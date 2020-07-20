---
title: ionic在移动端react项目中的使用
date: 2020-07-20 10:10:04
categories:
  - web
tags:
  - coding
  - 实践
---

最近接触了一个移动端的 2C 项目， 上一次做 2C 的项目已经可以追溯到遥远的毕业第一年了。能够把这些年间歇看到的 2C 的一些知识点进行应用还是挺有趣的。 在项目中，我们使用了 ionic 来作为移动端 h5 的组件库，经过一个多月的使用，也总结了一些经验。

# why？

我们都知道做 h5 或者 SPA 较之原生 app 在性能上的巨大差距。于是市面上有很多 hybird 的库都是封装原生的实现，提供 js 的编码接口，其代表就是 React Naive。 但使用 RN 本身需要一定的原生开发经验，对开发人员有一定的要求，一方面我担心未来的坑不好踩，另一方面也担心新人培养成本和未来团队在维护的时候产生较大的技术困难。还有一点也是今年 airbnb 等使用 RN 的代表公司放弃 RN 给我留下的心理阴影- -。

之所以选用 ionic 作为前端的组件库是因为我在做调研的时候被它的介绍所吸引了。跨平台、高性能保证、使用 web-component 作组件封装、对目前的前端三大框架都做了支持、简洁响应式并支持主题切换的 UI 设计、同样封装了大量原生方法并提供 api。官网的介绍确实是比较诱人了，另一方面，ionic 本身也是一个维护了很多年的项目，且 4.0+新版本的发布也才过去不久，github 上的更新频率也足够快，整体值得信赖。  
而我们项目本身又是一个 MVP 试错的新项目以及我作为程序员喜欢尝鲜的特点，我最终决定试一试。

# 使用体验

先说使用一个多月后的一个总体感觉。  
首先，坑确实不少。

1. 组件样式修改和自定义有难度。这一点有点成也 web-compnent，败也 web-component。ionic 使用 web-component 作组件封装，组件的 dom 结构和 css 都封装在 shadow dom 中，修改样式的方式就是使用 css 变量并向外暴露。这样的方式除了修改 css 变量本身稍显麻烦意外本来没有什么问题，但 ionic 给我的感觉是做的不够彻底。经常遇到需要修改的 css 样式没通过 css 变量做，而是需要直接写样式覆盖，或者写死于全局样式让人担心覆盖后产生的影响，或者写死于 shadow-dom 内无法修改的情况。一方面找这种 css 存在层级比较麻烦，另一方面也增加了开发者和 UI 的沟通成本。

2. 提供的组件库不是特别全。就组件库的完整度来说，ionic 还有一段路要走。在实际使用的时候，因为已经引入了 ionic，所以我们暂时没有考虑引入其它的组件库，那么就会遇到不少组件需要自己开发的情况，对于小项目组来说也是一项负担。项目做了一个月，自开发的比较耗时的公共组件数量就达到 4 个，未来肯定会更多。而且自开发的组件和 DOM 结构要接入 ionic 生态，比如支持 theme，使用 css 变量等，也算是负担。

3. Documentation 的完整度不足且只有英文，虽然有文档也有 example，但 example 有些敷衍，文档整体结构和描述也不够清晰。且目前没有中文文档，对于团队成员的培养也是一个问题。

4. 对 react 支持不足，不少组件目前还没有 react 版本，或者有了 react 版本文档却没有更新，依然写着“not support react”。

优点当然也是存在的：

1. 主题切换确实比较方便，只要定义好 css 结构并且做好自定义组件和各种 html css 的接入(按照 ionic 的方式使用 root 和 host 下的 css 变量)。那么主题就真正可以做到一键切换。对于我们这种用户独立部署的产品来说非常实用。
2. 性能还不错。不同组件在 ios 和 android 都使用了原生来实现，且通过 mode 来进行样式的切换。不管从样式和手势体验还是性能上给到用户的使用感受确实很有原生 app 的感觉。
3. 因为使用了 web-component，所以 dom 结构非常清晰。对 css 调试和单元测试都提供了一些方便。

# 实践经验总结

ionic 官网的指南和 doc 在[这里](https://ionicframework.com/docs)。

下面总结一些使用过程中记录下来的点。关于安装这些就跳过不讲了，ionic 提供了自己的 cli，但功能并不多，我们也没有使用。

## 关于样式

ionic 提供的组件间的样式是自隔离的，但 ionic 本身也有一些全局的样式。如果你想要使用所有 ionic 的特性，你需要引入它们。主要有以下几个文件：

- **core.css**：这是唯一一个必须引入的全局样式文件。它包含了 app 的特定样式，允许 color 属性跨组件工作。如果不引入这个文件，一些组件元素的颜色可能会出现问题。
- **structure.css**: 推荐。应用于 html 标签的样式，以及类似 box-sizing=border-box 等默认样式，保证了滚动等行为更像原生体验。
- **typography.css**:推荐。字体相关
- **normalize.css**: 推荐。使浏览器渲染元素更加稳定。
- **padding.css**:定义了一些修改 padding 或者 margin 的样式。[这里查看详情](https://ionicframework.com/docs/layout/css-utilities#content-space)
- **float-elements.css**: 提供了一些 float 相关的样式，给予 breakpoint 和 side，[这里查看详情](https://ionicframework.com/docs/layout/css-utilities#element-placement)
- **text-alignment.css**:提供了一些关于字体排列的样式。[这里查看详情](https://ionicframework.com/docs/layout/css-utilities#text-alignment)
- **text-transformation.css**:提供一些字母大小写，首字母大写等样式。[这里查看详情](https://ionicframework.com/docs/layout/css-utilities#text-transformation)
- **flex-utils.css**:flex 相关的一些样式，[这里查看详情](https://ionicframework.com/docs/layout/css-utilities#flex-properties)
- **display.css**: 基于 breakpoint 用于控制显示隐藏的样式。[这里查看详情](https://ionicframework.com/docs/layout/css-utilities#element-display)

在项目中不一定要全部引入，根据实际情况按需进行引入即可。[这里有上述样式列表的各种样式使用例子](https://ionicframework.com/docs/layout/css-utilities)。

## 关于主题 theming

### colors

ionic 有九种默认的颜色可供切换并应用到众多组件上。每种颜色有多个属性，包括本来的颜色和 shade 以及 tint。

所以要修改一个颜色，你得修改这个颜色的所有相关属性。每个颜色由 `base,contrase,shade,tint`几种属性组成。前面两者还需要一个 rgb 属性的额外设置。听起来比较麻烦，但 ionic 提供了颜色生成器，你只需要提供一个主题 color，其它有生成器帮你生成。[生成器在这里](https://ionicframework.com/docs/theming/colors)

比如要向 ionic 中添加一个新的 color，第一步，先在全局中加入变量：

```
:root {
  --ion-color-favorite: #69bb7b;
  --ion-color-favorite-rgb: 105,187,123;
  --ion-color-favorite-contrast: #ffffff;
  --ion-color-favorite-contrast-rgb: 255,255,255;
  --ion-color-favorite-shade: #5ca56c;
  --ion-color-favorite-tint: #78c288;
}
```

接着，创建一个使用这些变量的 class。class 名字需要是 .ion-color-{COLOR-NAME}格式。如下：

```
.ion-color-favorite {
  --ion-color-base: var(--ion-color-favorite);
  --ion-color-base-rgb: var(--ion-color-favorite-rgb);
  --ion-color-contrast: var(--ion-color-favorite-contrast);
  --ion-color-contrast-rgb: var(--ion-color-favorite-contrast-rgb);
  --ion-color-shade: var(--ion-color-favorite-shade);
  --ion-color-tint: var(--ion-color-favorite-tint);
}
```

接着你就可以在任何的组件中使用它了：

```
<ion-button color="favorite">Favorite</ion-button>
```

当然，你在 style 中定义的变量，你还可以在其它任何地方使用。

### CSS 变量

所有的主题实现基于 css 变量。所以修改主题样式非常方便，通过修改 ionic 的 css 变量即可实现。通过在一处定义变量，其它地方使用的方式。ionic 可以非常方便的对样式进行动态修改。这以前需要像 sass 这样的预处理器，但 css 变量让这一切变得简单。  
可以将变量声明在:root 下创建全局的 css 变量，也可以声明在上面说到的 mode 样式下来将变量作用域缩小到一种 mode。当然还可以进一步声明在一个组件上或者某个具体的 class 下。

比如希望把某 container 下的--background 变量进行修改：

```
.xxx_container {
  --ion-color-secondary: #006600;
  --ion-color-secondary-rgb: 0,102,0;
  --ion-color-secondary-contrast: #ffffff;
  --ion-color-secondary-contrast-rgb: 255,255,255;
  --ion-color-secondary-shade: #005a00;
  --ion-color-secondary-tint: #1a751a;
}
```

这样，xxx_container 下面的 ionic 元素使用这些 css 变量的地方就会改变。

### ionic modes

ionic 使用 modes 来定制化组件在不同平台的样式，每个平台的默认 mode 不同，但可以通过全局配置被重写。

默认配置如下：
|Platform | Mode | Description|
|---|---|---|
|ios | ios | ios Viewing on an iPhone, iPad, or iPod will use the iOS styles.|
|android | md | Viewing on any Android device will use the Material Design styles.|
|core | md | Any platform that doesn't fit any of the above platforms will use the Material Design styles.|

举个例子，安卓平台的 ionic app 将会使用 md 模式。即`<html class="md">`

**覆盖 mode 样式**：  
如上我们可以看到 ionic 根据不同平台在 html 标签上加了一个 class，那么我们可以巧妙地使用它来对某一种模式的样式进行覆盖：

```
.ios ion-badge {
  text-transform: uppercase;
}
```

同样，有很多全局的 css 变量可以被覆盖，如 [ionic 的颜色变量](https://ionicframework.com/docs/theming/colors)，[主题变量](https://ionicframework.com/docs/theming/themes)和[全局组件变量](https://ionicframework.com/docs/theming/advanced)。

### themes 主题

ionic 提供了一些全局变量，它们在整个组件库中使用，从而通过修改它们可以修改整个 app 的主题。  
主要分为 Application Colors 和 Stepped Colors 两类，前者可以修改大多数 ionic 组件的样式，后者则用于为某些组件提供更多的变化。

**Application Colors**：

这个类型的 colors 会在 ionic 的大多数地方用到。  
注意其中的 background 和 text color 也都需要设置一个 rgb 变量。[原因](https://ionicframework.com/docs/theming/advanced#the-alpha-problem)

| Name                                     | Description                               |
| ---------------------------------------- | ----------------------------------------- |
| --ion-background-color                   | 整个 app 的背景色                         |
| --ion-background-color-rgb               | 整个 app 的背景色, rgb format             |
| --ion-text-color                         | 整个 app 的文字颜色                       |
| --ion-text-color-rgb                     | 整个 app 的文字颜色, rgb format           |
| --ion-backdrop-color                     | 背景组件的颜色                            |
| --ion-backdrop-opacity                   | 背景组件透明度                            |
| --ion-overlay-background-color           | 叠加层(overlays)的背景色                  |
| --ion-border-color                       | 边框颜色                                  |
| --ion-box-shadow-color                   | Box shadow 颜色                           |
| --ion-tab-bar-background                 | Tab Bar 背景                              |
| --ion-tab-bar-background-focused         | 焦点 Tab Bar 的背景                       |
| --ion-tab-bar-border-color               | Tab Bar 的边框色                          |
| --ion-tab-bar-color                      | Tab Bar 的颜色                            |
| --ion-tab-bar-color-selected             | 选中的 Tab Button 颜色                    |
| --ion-toolbar-background                 | Toolbar 背景色                            |
| --ion-toolbar-border-color               | Toolbar 边框颜色                          |
| --ion-toolbar-color                      | Toolbar 内组件的颜色                      |
| --ion-toolbar-segment-color              | Toolbar 中细分按钮的颜色                  |
| --ion-toolbar-segment-color-checked      | Toolbar 中选中的细分按钮的颜色            |
| --ion-toolbar-segment-background         | Toolbar 中细分按钮的背景色                |
| --ion-toolbar-segment-background-checked | Toolbar 中选中的细分按钮的背景色          |
| --ion-toolbar-segment-indicator-color    | Toolbar 中细分按钮指示器(indicator)的颜色 |
| --ion-item-background                    | Item 的背景色                             |
| --ion-item-border-color                  | Item 的边框颜色                           |
| --ion-item-color                         | Item 内组件的颜色                         |
| --ion-placeholder-color                  | Inputs 中 placeholder 的颜色              |

**stepped colors**：

当然，整个 ionic 的颜色没有那么简单，在设计时还使用来不同的背景和文字颜色的阴影。为适应这种模式，ionic 创建来阶梯色(stepped color)。

简单的梗概 `ion-background-color`和`--ion-text-color`将会改变大部分组件的表现，但某些 ionic 组件可能会看起来有一些 broken。特别是在应用一些暗色主题的时候。

这是因为在一些组件中为 background-color 和 text-color 加了一些 lighten 或者 darken 的阴影。所以，如果你要修改这二者，那么也请同步更改对应的 steped colors。

[官方文档的最下方](https://ionicframework.com/docs/theming/themes)提供了一个 stepped color 的生成器，我们只需要选择好再复制即可。不用劳神。

## 在 react 中的使用

先说一说 ionic 的一些组件

### IonReactRouter

```
export declare class IonReactRouter extends React.Component<BrowserRouterProps> {
    render(): JSX.Element;
}
```

上述是它的声明，所以这就是一个 React-router 中的 BrowserRouter 的 wrap。它与后者大体表现相同，直接使用它替代后者即可。 一些微小的区别可以在[这里](https://ionicframework.com/docs/react/navigation)查看。
例如：

```
<IonApp>
  <IonReactRouter>
    <Switch>
      <Redirect path="/" exact to="/home" />
      <Route path="/home" component={HomePage} />
      <Route path="/login" exact component={Login} />
    </Switch>
  </IonReactRouter>
</IonApp>
```

### IonPage,IonHeader,IonContent

- **IonPage**: 是每个页面的基本组件(拥有 route/url 的组件)。包含一些主要的全屏组件模块，比如 header，title 和 content 组件。将 IonPage 用作根组件很重要，因为它有助于确保过渡正常工作，并提供 Ionic 组件依赖的基本 CSS。

- **IonHeader**: 组件顾名思义，存在于页面顶部并且不会随着内容变化滚动。除了处理一些 flex 布局意外，主要是用于装载其它组件，比如 IonToolbar(返回等按钮) 或者 IonSearchbar(搜索框)。

- **IonContent**: 则是页面的内容区域。用于提供和装载用户交互的可滚动内容。

基本上一个正常的 page 的 dom 最外层就是

```
<IonPage>
  <IonHeader>{/* content of header */}</IonHeader>
  <IonContent>{/* content of content */}</IonContent>
</IonPage>
```

### 关于 slot 属性

用于表示你所插入的组件的放置位置，此属性来源于 web-component，如下是 webcomponent 标准中对它的解释:  
**web component 中的一个占位符，你可以填充自己的标记，这样你就可以创建单独的 DOM 树并将它们呈现在一起。关联的 DOM 接口是 HTMLSlotElement。**

比如我想把一个退出按钮放在 header 的末尾：

```
<IonHeader>
  <IonToolbar>
    <IonButtons slot="end">
      <IonButton onClick={exit}>退出</IonButton>
    </IonButtons>
  </IonToolbar>
</IonHeader>
```

[这里还有一个官方的例子](https://github.com/mdn/web-components-examples/tree/master/element-details)。

### 组件封装和使用 styled-component 时的样式覆盖

贴个最简单的例子：

```
const DEMOWarning: FC<{ text?: string; content?: string }> = ({
  text = 'Default Title',
  content,
}) => {
  return (
    <WarningContainer>
      <IonItem>
        <IonIcon icon={informationCircleOutline} />
        <IonLabel>{text}</IonLabel>
      </IonItem>
      <p>{content}</p>
    </WarningContainer>
  );
};

const WarningContainer = styled.div`
  padding: 0 1rem;
  background-color: props.theme.bg_color;
  ion-item {
    --background-color: props.theme.bg_color;
    --inner-border-width: 0;
    --padding-start: 0;
    --padding-top: 0;
    --inner-padding-start: 0;
  }
  p {
    margin: 0;
  }
`;
```

可以看到可以直接使用 web-component 的名字选中 dom。  
一些样式直接写，一些样式用 css 变量覆盖。这也是我上面说的不统一的地方之一。

# 未完待续

todo...

# Reference

- [Ionic 官网组件文档](https://ionicframework.com/docs/components)
- [Ionic 官方指南，使用文档](https://ionicframework.com/docs/)
- [Ionic github repo](https://github.com/ionic-team/ionic-framework)
