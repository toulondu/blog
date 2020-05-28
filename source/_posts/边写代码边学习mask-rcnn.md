---
title: 边写代码边学习Mask-rcnn
date: 2020-05-19 00:42:15
categories:
- 深度学习
tags:
- 目标检测
- Mask R-CNN
- coding
- 实践
---
作为一个深度学习的学习者，你是不是苦于自己看了非常多的理论却难以下手实践？是否总觉得自己还没到写代码的时候？  
如果你这么想，那么你看再多的理论，也无法真正踏进深度学习的大门。
就我个人而言，很多时候即使读完了论文仍然有点懵，很多地方感觉都一知半解，还有些地方可能直接就是不太明白。此时，如果能有源码看一看，才能真正将这篇文章搞明白。  
so, "talk is cheap, show me the code" 实乃金玉良言。

这篇文章将使用pytorch modelzoo提供的现成Mask R-CNN预训练模型来进行fine-turing，实现一个目标检测和语义分割应用。并且在这个过程中来重新复习一下Mask R-CNN这个经典网络的一些原理。  

## Mask R-CNN简介
Mask R-CNN来自何恺明大神2017年的论文，是一个通用的目标检测和实例分割的模型。它基于作者团队在2015年提出的faster rcnn模型，最主要的改动就是增加了一个分支来用于分割任务。
Mask R-CNN是anchor-based的模型，依然采用Faster RCNN的2-stage结构，首先用RPN找出候选region，然后在此基础上计算ROI并完成分类、检测和分割任务。  
并没有添加各种trick，Mask RCNN就超过了当时所有的sota模型。

## 定义DataSet并处理
我们将使用Penn-Fudan数据库中的行人图片数据来对模型进行微调。它包含170个图像和345个行人实例。[数据在此](https://www.cis.upenn.edu/~jshi/ped_html/PennFudanPed.zip)。  
数据文件结构大致如下：
```
PennFudanPed/
  PedMasks/
    FudanPed00001_mask.png
    FudanPed00002_mask.png
    FudanPed00003_mask.png
    FudanPed00004_mask.png
    ...
  PNGImages/
    FudanPed00001.png
    FudanPed00002.png
    FudanPed00003.png
    FudanPed00004.png
```
PedMasks中数据为PNGImages文件夹下对应图片的实例分割掩膜，如下：
{% asset_img mask_sample.png 原图片%}
{% asset_img mask_sample1.png mask图片%}
即掩膜中不同的数值对应不同的实例的分割。

### 定义Dataset类
上一篇文章说过，Dataset类是帮助我们处理原始数据并产出模型需要的输入数据的类。  
而在我们这次的Mask R-CNN模型中，我们希望Dataset通过__getitem__能返回我们图像数据(H,W)以及图像的以下信息：
- boxes: 这张图片里所有的目标区域,格式为[x0,x1,y0,y1]，x∈[0,W], y∈[0,H]
- labels: 每个边框的标签
- masks: 每个图像的掩膜
- image_id: 图片id
- area：每个bbox的面积，用于计算IoU
- iscrowd: 每个区域是否是人群
代码如下：
```
class PennFudanDataset(Dataset):
    def __init__(self, root, transforms):
        self.root = root
        self.transforms = transforms
        # 下载所有图像文件，为其排序。确保它们对齐,而且这样就把图片名字列出来了，方便了加载图片
        self.imgs = list(sorted(os.listdir(os.path.join(root, "PNGImages"))))
        self.masks = list(sorted(os.listdir(os.path.join(root, "PedMasks"))))

    def __getitem__(self, idx):
        # load images ad masks
        img_path = os.path.join(self.root, "PNGImages", self.imgs[idx])
        mask_path = os.path.join(self.root, "PedMasks", self.masks[idx])
        img = Image.open(img_path).convert("RGB")
        # 请注意我们还没有将mask转换为RGB,
        # 因为每种颜色对应一个不同的实例。0是背景
        mask = Image.open(mask_path)
        # 将PIL图像转换为numpy数组
        mask = np.array(mask)
        # 实例被编码为不同的颜色
        obj_ids = np.unique(mask)
        # 第一个id是背景(即0)，所以删除它
        obj_ids = obj_ids[1:]

        # 将相同颜色编码的mask分成一组
        # mask为2维，用None扩充obj_ids维度，masks为3维，因为一张图片可能有多个实例分割
        # 二进制格式
        masks = mask == obj_ids[:, None, None]

        # 获取每个mask的边界框坐标
        num_objs = len(obj_ids)
        boxes = []
        for i in range(num_objs):
            # masks[i]为2维，所以np.where返回2个tuple，分别为此颜色编码的元素在各个维度的下标
            # 这里的数据中不同颜色的mask是语义分割的像素点，选出最大最小的x坐标和y坐标就得到了目标区域(x0,y0),(x1,y1)
            pos = np.where(masks[i])
            xmin = np.min(pos[1])
            xmax = np.max(pos[1])
            ymin = np.min(pos[0])
            ymax = np.max(pos[0])
            boxes.append([xmin, ymin, xmax, ymax])

        # 将所有转换为torch.Tensor
        boxes = torch.as_tensor(boxes, dtype=torch.float32)
        # 我们只检测行人这一个类(行人，所以直接全部置为1)
        labels = torch.ones((num_objs,), dtype=torch.int64)
        masks = torch.as_tensor(masks, dtype=torch.uint8)

        image_id = torch.tensor([idx])
        area = (boxes[:, 3] - boxes[:, 1]) * (boxes[:, 2] - boxes[:, 0])
        # 假设所有实例都不是人群
        iscrowd = torch.zeros((num_objs,), dtype=torch.int64)

        target = {}
        target["boxes"] = boxes  # 这张图片里所有的目标区域
        target["labels"] = labels   # 每个目标区域的类型
        target["masks"] = masks    # 图像掩膜 mask
        target["image_id"] = image_id  # 图片id
        target["area"] = area          # 每个区域的面积
        target["iscrowd"] = iscrowd    # 每个区域是否是人群(这里假设的都不是)

        if self.transforms is not None:
            img, target = self.transforms(img, target)

        return img, target
        
    def __len__(self):
        return len(self.imgs)
```

## 定义模型
Mask-RCNN结构如下：
{% asset_img mask_rcnn_model.jpg 模型结构%}
torchvision.models.detection.fasterrcnn_resnet50_fpn(pretrained=True) 为我们提供了一个预训练的Mask-RCNN模型。  
修改这种预训练模型一般有2种思路，第一就是当我们数据较少时，我们就只将最后一层替换成我们的输入目标，然后进行微调。
另一种思路则是可以在原模型基础上进行修改，比如替换backbone，修改RPN的anchor数量，调整ROI维度等。  

我们先来看看第二种方式：
**修改backbone**: Mask-RCNN 的backbone使用的Resnet101，整体还是比较大的，假如你想使用一些轻量的backbone，比如mobileNet，那么你可以进行替换  
**修改rpn**:Mask-RCNN 的anchor是如何生成的呢，注意看上面的结构图。输入数据在经过backbone之后，得到的feature-map其实是在原输入的基础上进行了32倍下采样。基于这个feature-map的每个元素，我们再进行一个3×3的卷积来增加感受野，然后对每个元素生成9个anchor来生成候选区域。这9个初始anchor包含3种长宽比(1:1,1:2,2:1),每种长宽比包含3种不同的面积。结构如下：
{% asset_img rpn_structure.jpg rpn结构%}
注意图上的各种数字，256表示的是骨干网络输出的通道数，k表示生成的anchor的数量。因为每个anchor有positive和negative，所以有2k个打分。而每个anchor会经过后续的回归找到针对正确区域的4个偏移量(x,y,w,h)，所以是4k个coordinates。
对于这些参数，我们也可以修改。  
**修改RoI pooling**：RoI pooling层复杂收集proposal，然后选取特征并送入后续的分类和检测FC网络。 要知道为了保留图片中事物的特性，我们很少对图片采用resize或者裁剪操作，而Mask-RCNN接受不同大小的图片输入，那么经过骨干和RPN网络后，各图片到此时的数据维度是不一样的。这种情况下我们没有办法通过FC等网络进行特征组合。RoI pooling层就是来解决这个问题的。它将收集到的proposal分为固定个数的区域(比如7*7)，然后对每这些区域使用max_pool处理，这样就得到了固定维度的输出。
其次，Faster RCNN在处理RoI pooling的过程中有2次取整操作：
- region proposal的xywh通常是小数，但是为了方便操作会把它整数化。
- 将整数化后的边界区域平均分割成 k x k 个单元，对每一个单元的边界进行整数化。
这将会导致RoI pooling后的输出与原图像对应的区域产生一些偏离，导致不能完全对应。第一个问题很好解决，不再取整即可。而解决第二个问题，则是使用双线性插值的方式来更加精确的找到每个边界的特征。我们在下面代码中看到的sampling_ratio=2就是这个方法的体现。

代码如下：
```
from torchvision.models.detection import FasterRCNN
from torchvision.models.detection.rpn import AnchorGenerator

# 加载预先训练的模型进行分类和返回
# 只有功能 
# 主干采用mobileNet V2
backbone = torchvision.models.mobilenet_v2(pretrained=True).features
# FasterRCNN需要知道骨干网中的输出通道数量。对于mobilenet_v2，它是1280，所以我们需要在这里添加它
backbone.out_channels = 1280

# 我们让RPN在每个空间位置生成5 x 3个锚点 PS：这里原文是3*3，即3种大小3种宽高比
# 改成5种不同的大小和3种不同的宽高比。 
# 因为每个特征映射可能具有不同的大小和宽高比，size为anchor box大小，aspect_ratios则是宽高比
anchor_generator = AnchorGenerator(sizes=((32, 64, 128, 256, 512),),
                                   aspect_ratios=((0.5, 1.0, 2.0),))

# 定义一下我们将用于执行感兴趣区域裁剪的特征映射，以及重新缩放后裁剪的大小。 
# 如果您的主干返回Tensor，则featmap_names应为[0]。 
# 更一般地，主干应该返回OrderedDict [Tensor]
# 并且在featmap_names中，您可以选择要使用的功能映射。
# 这里为RoIPooling层，将feature_map对应的原图中部分处理成7*7(output_size=7)的大小然后再进行后续的分类和回归操作
# 而sampling_ratio=2则是原文中进行插值所选取的采样点，简单的说：采样点为2就是说7*7的每个区域内，都要再分成2*2个grid，然后对每个grid中心点进行采样，将这4个点的值求平均就是这个区域最终的值。
roi_pooler = torchvision.ops.MultiScaleRoIAlign(featmap_names=[0],
                                                output_size=7,
                                                sampling_ratio=2)

# 将这些pieces放在FasterRCNN模型中
model = FasterRCNN(backbone,
                   num_classes=2,
                   rpn_anchor_generator=anchor_generator,
                   box_roi_pool=roi_pooler)
```


虽然第二种方式明显比较酷，但鉴于本示例中样本数据比较少，所以我们使用第一种方式:
```
def get_model_instance_segmentation(num_classes):
    # 加载在COCO上预训练的预训练的实例分割模型
    model = torchvision.models.detection.maskrcnn_resnet50_fpn(pretrained=True)

    # 获取分类器的输入特征数
    in_features = model.roi_heads.box_predictor.cls_score.in_features
    # 用新的头部替换预先训练好的头部
    model.roi_heads.box_predictor = FastRCNNPredictor(in_features, num_classes)

    # 现在获取掩膜分类器的输入特征数
    in_features_mask = model.roi_heads.mask_predictor.conv5_mask.in_channels
    hidden_layer = 256
    # 并用新的掩膜预测器替换掩膜预测器
    model.roi_heads.mask_predictor = MaskRCNNPredictor(in_features_mask,
                                                       hidden_layer,
                                                       num_classes)
```


## 实例化模型
在torchvision的官方库中，references/detection/里有很多辅助函数来简化训练和评估检测模型。
这里我们需要用到references/detection/engine.py，references/detection/utils.py和references/detection/transforms.py。
去[这里](https://github.com/pytorch/vision) download代码并将这几个文件拷贝到你的目录中即可。

其次，提前安装[cocoapi](https://github.com/cocodataset/cocoapi/tree/master/PythonAPI),如果你在windows上，可能需要安装visial studio。
windows上也可以通过安装pycocotools来解决。whl见：https://pypi.org/project/pycocotools-windows/#files

这一步没什么好说的，代码里也有足够的注释：
```
# 训练阶段按0.5几率水平翻转图像
def get_transform(train):
    transforms = []
    transforms.append(T.ToTensor())
    if train:
        transforms.append(T.RandomHorizontalFlip(0.5))
    return T.Compose(transforms)


# 在GPU上训练，若无GPU，可选择在CPU上训练
device = torch.device('cuda') if torch.cuda.is_available() else torch.device('cpu')

# 我们的数据集只有两个类 - 背景和人
num_classes = 2
# 使用我们的数据集和定义的转换
dataset = PennFudanDataset('data/PennFudanPed', get_transform(train=True))
dataset_test = PennFudanDataset('data/PennFudanPed', get_transform(train=False))

# 在训练和测试集中拆分数据集
indices = torch.randperm(len(dataset)).tolist()
dataset = torch.utils.data.Subset(dataset, indices[:-50])
dataset_test = torch.utils.data.Subset(dataset_test, indices[-50:])

# 定义训练和验证数据加载器
data_loader = torch.utils.data.DataLoader(
    dataset, batch_size=2, shuffle=True, num_workers=4,
    collate_fn=utils.collate_fn)

data_loader_test = torch.utils.data.DataLoader(
    dataset_test, batch_size=1, shuffle=False, num_workers=4,
    collate_fn=utils.collate_fn)

# 使用我们的辅助函数获取模型
model = get_model_instance_segmentation(num_classes)

# 将我们的模型迁移到合适的设备
model.to(device)
```

## 训练阶段
我们使用SGD进行优化，训练10个epoch。并且通过比较在测试集上的mAP，保存效果最好的参数到best_state_dict中。
```
def train():
    # 构造一个优化器
    params = [p for p in model.parameters() if p.requires_grad]
    optimizer = torch.optim.SGD(params, lr=0.005,
                                momentum=0.9, weight_decay=0.0005)
    # 和学习率调度程序
    lr_scheduler = torch.optim.lr_scheduler.StepLR(optimizer,
                                                   step_size=3,
                                                   gamma=0.1)

    # 训练10个epochs
    num_epochs = 10

    best_mAp = 0
    for epoch in range(num_epochs):
        # 训练一个epoch，每10次迭代打印一次
        train_one_epoch(model, optimizer, data_loader, device, epoch, print_freq=10)
        # 更新学习速率
        lr_scheduler.step()
        # 在测试集上评价
        eval_res = evaluate(model, data_loader_test, device=device)

        # 将结果最好的参数保存下来
        mAp_epoch = float(eval_res.coco_eval['bbox'].stats[0])
        if mAp_epoch > best_mAp:
            torch.save(model.state_dict(),'./best_state_dict')
            best_mAp = mAp_epoch

    print("Finish training the model.")
```
训练过程中你可以看到各项指标，我忘了截图，最后的COCO-style mAP大概是81左右，mask mAP为在78左右。

## 使用效果最好的参数进行预测
完成了训练，接下来肯定就是我们的show time了。
```
model.load_state_dict(torch.load('./best_state_dict'))
    # # 切换为评估模式
    model.eval()

    # 让我们瞅一瞅效果
    img, _ = dataset_test[0]

    with torch.no_grad():
        prediction = model([img.to(device)])

    img_ori = Image.fromarray(img.mul(255).permute(1, 2, 0).byte().numpy())
    draw = ImageDraw.Draw(img_ori)

    masks = prediction[0]['masks']
    masks_all = Image.fromarray(np.sum(np.sum(masks.mul(255).byte().cpu().numpy(),axis=0),axis=0))
    
    
    for [x1,y1,x2,y2] in prediction[0]['boxes']:
        draw.rectangle([(x1,y1),(x2,y2)],outline=(255,0,0))

    imgs = [img_ori,masks_all]

    for i,im in enumerate(imgs):
        ax = plt.subplot(1, 2, i + 1)
        plt.tight_layout()
        ax.axis('off')
        plt.imshow(im)

    plt.show()
```
效果如下：
{% asset_img result01.jpg 效果图1%}
{% asset_img result02.jpg 效果图2%}
{% asset_img result03.jpg 效果图3%}

## 源码
本文源码在[这里](https://github.com/toulondu/mask-rcnn-brief)

## reference
[pytorch官网教程:TORCHVISION OBJECT DETECTION FINETUNING TUTORIAL](https://pytorch.org/tutorials/intermediate/torchvision_tutorial.html)
[Faster R-CNN 论文](https://arxiv.org/abs/1506.01497)
[Mask R-CNN 论文](https://arxiv.org/abs/1703.06870)、
[令人拍案称奇的Mask RCNN](https://zhuanlan.zhihu.com/p/37998710)