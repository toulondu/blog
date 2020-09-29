---
title: 常用的git命令介绍和简单的git工作流
date: 2020-09-29 14:00:47
categories:
  - 基础
tags:
  - 效率
---

# 常用的 git 命令介绍

Git 是一款非常强大的工具，但如果你并不了解每个命令敲下去之后分支会如何交互，交互之后又会怎样影响历史记录，那 git 使用起来可能也没那么让你的同事舒服。

比如当你发现 push 有问题而在不知道问题原因时强行执行了 git push，或者习惯来使用 pull+merge 来合并代码，你的同事也许会在心里默默地画了一个圈圈。

所以在讲解我们的 git 工作流前，我们先来学习一下基本的 git 命令，明白这些命令背后的逻辑。

## 1.merge

我们都知道 git 代码管理是基于多分支，你可以通过分支很方便地将不同的代码修改隔离开，并且很好地和同事完成分工且不会互相影响。并且即使你提交了错误的代码，也能够保证不影响生产环境。

而`git merge`则是可以把一个分支的代码与另一个分支进行合并的命令，合并的方式有 2 种，`fast-forward` 和 `no-fast-forward`。

### fast-forward

`fast-forward` 很简单，当你的分支跟你想合并的分支想比没有额外的提交时，merge 就会使用 `fast-forward` 合并，这也是 git 首先尝试的合并方式。合并时不会创建任何额外的提交，而是将我们正在合并的分支上的 commit 直接合并到当前分支。

比如我从 dev 分支 checkout 了一个新分支 feat/nothing，然后修改了一些东西并提交。

此时我 dev 分支相对于之前没有任何变化，此时在 dev 分支上执行`git merge feat/nothing`，执行的就是 fast-forward 合并。简单修改了 dev 分支 HEAD 的指针指向就达到了合并的效果：

合并前:
{% asset_img before_git_merge.png fast-forward-merge%}

合并后：
{% asset_img after_git_merge.png fast-forward-merge%}

### No-fast-forward (—no-ff)

但实际情况肯定没有这么美好，很多时候我们在新分支上修改代码后，原分支也会有一些新的提交。此时我们要在原分支合并我们的新分支，就需要执行`no-fast-forward`合并。

`no-fast-forward`的合并，git 会创建一个新的 commit，这个新的 commit 的上一次提交会同时指向合并的 2 个分支。

举个例子，我在`dev`上 checkout 一个新分支`feat/no-fast-forward`，然后在`feat/no-fast-forward`上新建了 c.js 文件并提交，同时在 dev 上新建了 d.js 文件并提交。此时我们的分支情况如下：
{% asset_img before_no_fast_merge.png no-fast-forward-merge%}

当我们在 dev 分支上执行`git merge feat/no-fast-forward`之后，我们看看发生了什么：
{% asset_img after_no_fast_merge.png no-fast-forward-merge后%}

很清晰的可以看到，正如我们上面所说，git 创建了一个新的 commit，同时指向了 2 个合并的分支提交。

## 2.rebase

除了 `git merge` 以外，还有一种可以合并分支代码的方式，叫做变基，即 `git rebase`。 这也是我们推荐使用的命令，因为它比 merge 强大得多！

`git rebase` 会把当前分支的提交复制到指定的分支之后。  我将这句话进行了加粗，希望你多看两遍。这也是 rebase 和 merge 最大的区别，执行 rebase 时，git 不会纠结要保留或者要删除哪些修改，执行 rebase 命令的分支总是含有我们最新的修改。这样我们可以保留一个漂亮的、线性的 git 历史。

看一个例子：

我在 `dev` 上 checkout 了一个新分支 `feat/rebase-feat`，然后分别在 `dev` 和 `feat/rebase-feat` 上进行了一次提交。然后我们的分支变成了这样：
{% asset_img before_git_rebase.png rebase前%}

此时我们在 `feat/rebase-feat` 上执行 `git rebase dev` 命令，至于为什么我们要在 feat 分支上执行 rebase 而不是 dev 上执行，接下来会说到。

执行之后分支变成了这样：
{% asset_img after_git_rebase.png rebase后%}

再回头来看变基这个名字，即“重新设置基线”。 重新设置的是你的当前分支的开始点，而这个开始点则是基于你 rebase 的目标分支(称作跟踪分支)。

执行 git rebase 命令后，git 会把你当前分支的开始时间轴移动到跟踪分支的最新提交之后。这样，你的当前分支也就成为了最新的跟踪分支。

而这一系列的操作都是基于文件事务进行处理的，这个事务类似于数据库的事务，所以你不用担心事务失败等异常情况影响文件的一致性。你可以在 rebase 的过程中随时回退和取消 rebase 事务。

看懂了上面这段，你也就明白了我们为什么不在公共分支 dev 上执行 rebase 了。如果你不明白，我提示一下，新的 commit 会向后放，那么其它从公共分支拉分支的用户，都需要再 rebase，这显然不科学。而且在公共分支上 rebase 还会修改公共分支的提交历史，这也很不 ok。

综上，当你在 feat 分支上开发完成之后，想要把主分支 master 的修改合到你的分支上，这种情况使用 rebase 就要优于 merge，把你的提交放在主分支的提交之后，且不会创建新的提交。历史更加清晰。

除了合并代码之外，git rebase 还有很多功能，它可以让我们自由控制我们的提交。

- reword：修改提交信息；
- edit：修改此提交；
- squash：将提交融合到前一个提交中；
- fixup：将提交融合到前一个提交中，不保留该提交的日志消息；
- exec：在每个提交上运行我们想要 rebase 的命令；
- drop：移除该提交。

### squash

我们来尝试一下 squash，其它的举一反三即可。

我们很多人在自己的分支上开发时，因为各种原因，commit 也许会很频繁。比如：
{% asset_img many_commits.png many_commits%}

很显然，这很糟糕，如果让你的同事或者上级看到这些令人尴尬的提交记录，想必是你的一次社会性死亡。

这个时候 squash 就可以派上用场了,它可以帮助我们将多个提交合并成一个。

(1) 首先执行 `git log` 查看提交记录：
{% asset_img git_logs.png git_logs%}

此时我们的目的是将最近的这三个提交进行合并，那么我们就复制它们之前的那一次提交记录的 commit sha，即 69d79e992e480999a5e89957bcf8e01b3e9e6933。

(2) 执行 `git rebase -i 69d79e992e480999a5e89957bcf8e01b3e9e6933`

也可以执行 `git rebase -i HEAD~3`，跟上诉命令效果一样。执行后我们会看到如下的 vim 编辑界面，让我们对这几次提交作处理：
{% asset_img squash_step_1.png squash_step_1%}

将你要 squash 的提交标识为 squash(至少保留一个 pick)。
{% asset_img squash_step_2.png squash_step_2%}

接着保存并退出,我们会进入下一个 vim 编辑界面，这里让我们处理这三次提交的提交信息：
{% asset_img squash_step_3.png squash_step_3%}

这里你可以任意修改，比如只保留:
{% asset_img squash_step_4.png squash_step_4%}

接着保存退出，我们就完成了 rebase，重新查看历史：
{% asset_img squash_step_5.png squash_step_5%}

很好！只剩下一次提交了！

PS:至于 fixup,drop,edit 等功能，尝试在第一次 vim 时将对应提交的 pick 修改为这些关键字，建议你自己去试一试。

## 3.代码冲突

当你在做 merge 或者 rebase 时，都难免会出现代码冲突。

git 会向你展示冲突出现的位置，你可以手动移除你不想保留的修改，保存你想要的修改。(merge 时的冲突处理后需要重新提交，rebase 则不用。)

这里建议使用编译器或者 git 可视化工具提供的功能来解决冲突。比如 vscode、fork 等。

## 4.reset(重置)

当你进行了错误的提交(比如有 bug)时，或许你需要使用到 git reset。

它可以修改当前 HEAD 的指向，从而不再使用那次错误的提交。

重置分为软重置和硬重置。

### 软重置

软重置只会移动 HEAD 的指向，而不会删除该指向之后的修改。

我们在 2 次提交中分别提交 something.js 和 anything.js 后：
{% asset_img before_soft_reset.png before_soft_reset%}

此时执行 `git reset --soft HEAD~2`：
{% asset_img after_soft_reset.png after_soft_reset%}

提交回到了 2 次提交之前，但通过 git status 可以看到，something.js 和 anything.js 并未删除：
{% asset_img soft_reset_files.png soft_reset_files%}

### 硬重置

硬重置与软重置唯一的不同，就是你重新指向的那次提交之后的修改都会被丢弃。这包括工作目录中的和暂存区的修改。

对应上面的例子，则 anything.js 和 something.js 将会被删除。

## 5.revert

git revert 是另一种撤销修改的方式。

不同于 git reset，它不会修改分支的历史(git reset 会直接删除提交记录)。

执行 `git reset commit-id` 将会创建一次新的提交，并且会将 commit-id 对应的提交的修改移除。

## 6.fetch

在我们进行开发时，我们会拥有一个远程分支，指向我们在 gitlab 上的库的某个分支。当你想获取远程分支上的最新代码时，便可以使用 fetch。

fetch 不会以任何方式影响你的本地分支，它只是单纯的下载新数据。要将这些修改合并到你的代码中，你依然需要执行 merge 或者 rebase。

## 7.remote

remote 可以用来管理你的远程仓库地址。

一般来说，当我们通过 clone 来拉取了一个远端的代码后，git 会默认在 remote 中给我们添加一个 origin 源。

通过 `git remote -v` 可以查看当前的 remote 源：
{% asset_img git_remote_v.png git_remote_v%}

此时你应该明白，当你执行 `git push origin master` 的时候，你是在做什么了，是在向远端 origin 对应的仓库的 master 分支提交代码。

如果你是想把本地的一个目录提交到某个新建的远端 git 仓库上，怎么做呢？

首先，我在 gitlab 上新建了一个名为 my-project 的 project，地址为 git@gitlab.xxxx.com:dulb/my-project.git

接着我在我想要提交的本地目录通过 git init 命令将此目录初始化为一个 git 目录:
{% asset_img git_init.png git_init%}

接着使用 `git remote add` 来添加远端 git 仓库地址
{% asset_img git_remote_add.png git_remote_add%}

ps:这个 origin 可以是任何名字，你也可以通过 `git remote rename` 随时修改成任何你觉得更清晰的名字：
{% asset_img git_remote_rename.png git_remote_rename%}

接着你就可以使用 git push 将你的代码提交到远端了。

remote 可以有多个，如果你在开源社区 github 并希望对某个 project 进行代码贡献，一般的做法就是 fork 该库到你自己的空间。然后在本地时，你通过 remote 维护两个远端仓库，一个指向你 fork 的远端库，一个指向原 project 本身。这样，到你最终的 merge request 被通过合并前，你的代码不会以任何形式出现在该 project 的 github 空间中。

## 其它

还有一些命令，比如 cherry-pick，reflog 等，略。

# 最简单的 git 工作流

掌握了基本的 git 命令之后，在我们具体的项目中，我们使用什么方式来进行代码提交呢？
{% asset_img git_work_flow.png git_work_flow%}
如上是一个比较完整的 git 工作流，但大多数的实际项目不比开源项目，开发人员一般不会太多。

如果我们的开发团队只有 5 个人左右，我们并不需要这么完整的工作流。

所以在当前公司我们并没有使用上图这种相对复杂的 git 工作流，而是采用一种相对简单的方式，这里简单地介绍一下流程。

## 简单工作流

假设目前我们的远端代码仓库为git@gitlab.xxxx.com:dulb/my-project.git

而 master 分支为我们的保护分支(不允许直接 push 代码)。

当我们想要为其贡献代码的时候，步骤如下

### 1.克隆代码

`git clone git@gitlab.seeyon.com:dulb/my-project.git`

### 2.修改代码

当开发一个新的 feature 时，在 master 分支上通过`git checkout -b feat/feature-desc`切换一个新的 feature 分支，再在此分支上进行开发。

### 3.提交代码

分支开发完成后，首先通过`git add` 和 `git commit`将代码将提交到本地分支中。  
在将此代码 push 到远端仓库前，_需要在本地获取最新的 master 代码并合并到分支代码中_，这样能够保证管理员在合并代码时不会出现冲突。  
首先通过`git fetch origin master`下载远端 master 分支最新的代码。  
接着使用`git rebase origin master`合并远端 master 最新的代码并处理可能的冲突。
最后使用`git push origin feat/feature-desc`将分支代码推送到远端仓库。

### 4.提交 merge request

merge request 顾名思义，即一个合并请求，即请求将分支代码合并到另一个分支上，此处我们是请求将 feat/feature-desc 分支合并到 master 中。

上一步完成之后，登录到 gitlab，你会在顶部看到这么一条信息：
{% asset_img auto_mr_btn.png auto_mr_btn%}

这是 gitlab 提供的提交 merge request 的快捷方式。
如果你没看到这条消息也没关系，进入对应的仓库点击左边的 merge request 选项卡即可：
{% asset_img menu_tab.png menu_tab%}

点击创建一个 merge request，将来到如下页面:
{% asset_img mr_detail.png mr_detail%}

`title`,`description`我就不作介绍了。 不过这里注意 Title 输入框下面的这个`start the title with WIP`按钮，如果你的分支尚在开发中并不希望被合并，你就可以点击这个按钮，它会在 title 前面添加一个 WIP 前缀,即 work-in-progress，这样可以阻止这个分支被合并，当你完成开发之后再取消这个状态即可。

`assignee`是指定合并人。 `milestone`是 project 的里程碑，`label`则是给你的这个 mr 打上标签，这三个属性都选择合适的值即可。

`Source branch`和`Target branch`则分别是你要合并的源分支和目标分支，即将源分支的代码合并到目标分支中，我们这里是将 feat/feature-desc 合并到 master 中，则如上图所示。

下面还有 2 个 checkbox，第一个表示管理员点击合并后，将删除你提交到远端的这个`feature-desc`分支(不是本地)，故当你完成了这个 feature 开发的开发时，勾选此选项。  
第二个则是合并你分支上的 commit 记录，勾选此选项后，即使你的分支拥有 10 个 commit，在 master 上只会看到一个。

最下面的三个选项卡 Commit/pipelines/Changes 则分别是 commit 信息，流水线和代码修改记录。也是 code reviewer 或者管理员主要关注的区域。  
在这里可以看到本次 mr 的代码变化，pipeline 是否通过(目前我们通过 Jenkins 构建而不是 gitlabci，故这里的 pipeline 没用。)

提交之后，提醒管理员(可以考虑假如通过 hook 给管理员发送邮件)合并代码即可。

## 分支命名规范和代码提交规范

这里介绍一种基于提交消息的轻量级约定---约定式提交，它主要是用于规范 git commit 信息。使用这种规则，我们可以得到更加清晰可读的提交记录，还能方便我们之后编写基于这种规范的自动化工具。

主要的做法是在提交信息中描述新特性、bug 修复和破坏性变更。

这里我直接引用官方说明：

提交说明的结构如下所示：

```
<类型>[可选的作用域]: <描述>

[可选的正文]

[可选的脚注]
```

提交说明包含了下面的结构化元素，以向类库使用者表明其意图：

fix:  类型   为  fix  的提交表示在代码库中修复了一个 bug（这和语义化版本中的  PATCH  相对应）。
feat:  类型   为  feat  的提交表示在代码库中新增了一个功能（这和语义化版本中的  MINOR  相对应）。
BREAKING CHANGE:  在可选的正文或脚注的起始位置带有  BREAKING CHANGE:  的提交，表示引入了破坏性 API 变更（这和语义化版本中的  MAJOR  相对应）。 破坏性变更可以是任意   类型   提交的一部分。
其它情况:  除  fix:  和  feat:  之外的提交   类型   也是被允许的，例如  @commitlint/config-conventional（基于  Angular 约定）中推荐的  chore:、docs:、style:、refactor:、perf:、test:  及其他标签。 我们也推荐使用 improvement，用于对当前实现进行改进而没有添加新功能或修复错误的提交。 请注意，这些标签在约定式提交规范中并不是强制性的。并且在语义化版本中没有隐式的影响（除非他们包含 BREAKING CHANGE）。  可以为提交类型添加一个围在圆括号内的作用域，以为其提供额外的上下文信息。例如  feat(parser): adds ability to parse arrays.。

比如我要提交的代码是在开发一个新的特性，开发了一个订单详情页面。那么我在提交代码时，应该：

```
git commit -m "feat: 增加了一个订单详情页面. (...更多的描述)"
```

更多的例子可以查看[官方介绍](https://www.conventionalcommits.org/zh-hans/v1.0.0-beta.4/)。

进一步的，你可以使用 git hook 来对 commit 信息进行校验，禁止不符合规范的提交。

将之引申，我们在创建分支的名字时，也可以使用类似的约定，比如 feature 分支，分支名为 feat/xxxx， 比如修复 bug，分支名为 fix/xxxx。
