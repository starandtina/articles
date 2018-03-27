# My Personal Git Styleguide

## Git 通用规则

* 只能在功能分支上开发
* 禁止直接推送代码到 **maser** 分支（设置为受保护分支），推荐使用合并请求（PR/MR）
* 在发起合并请求前，请更新你本地的 **master** 分支并且使用交互式 [**rebase**](https://dev.to/maxwell_dev/the-git-rebase-introduction-i-wish-id-had) 操作(eg. `git rebase -i --autosquash master`)，从而保证你本地的提交应用于所有历史提交的最顶端，而不会去创建 Merge Commit
* 在发起合并请求前，必须确保你的功能分支可以成功构建，并且通过了所有的代码规则检查和测试
* 合并完成后，删除本地和对应的远程分支（Gitlab 或 Gitee 有选项可以自动完成）

## Git 分支命名规则

* 功能分支使用 **feature/名字-项目名-简短描述**
* Bug 分支使用 **fix/名字-项目名-简短描述**
* 发布分支使用 **release-发布日期**，如果有多个发布使用 \_1 形式的尾数来标识 (eg. **release-20180318**)
* Hotfix 分支 **hotfix/日期-Bug 标题**

针对更加复杂的项目（如一个项目里面牵涉多个子项目），为了不影响发布，Git 分支原则如下：

* 迭代开始时，针对每个子项目新建分支，如 feature/wms，feature/tms, feature/pms 等等
* 迭代过程中，功能分支一律合并入对应的子项目分支，不污染主分支
* 联调和冒烟结束后，提交到对应测试分支，如 test-wms, test-tms 等等
* 迭代结束时，如果子项目确定需要发布，则在发布前将对应子项目分支合并入 master 分支发布完成后（基于 master），基于 master 分支拉取发布分支

## Git 工作流

基于上面提到的 Git 通用规则和一些常用的分支命名规则，我们会采用功能分支策略 + 交互式 Rebase:

### 针对一个新项目, 在您的开发目录下初始化您的项目

```Bash
git clone ssh://git@git.guoxiaomei.cn:2224/fee/supply_chain.git
cd <项目目录>
```

如果是（已有项目）随后的功能开发/代码变动，这一步请忽略。

### 检出（Checkout） 一个新的功能或故障修复（feature/fix）分支

```Bash
git checkout -b <分支名称>
```

### 新增一下代码变更

```Bash
git add .
git commit -m 'your commit message'
```

Git 提交消息格式也需要满足规范要求，见下文。

### 保持与远程的同步，以便拿到最新变更

```Bash
git checkout master
git pull --rebase
```

> 时刻保持与远程仓库的同步是一个好习惯，可以提前解决有可能的代码冲突。

### 切换至功能分支后，通过交互式 rebase 从您的 master 分支中获取最新的代码提交，以更新您的功能分支

```Bash
git checkout <your-feature-branch>
git rebase -i --autosquash master
```

没有人会愿意看到单个功能分支中的功能开发就占据如此多的提交历史。某些第三方 Git 代码管理平台（如 企业级的 Gitlab）提供该功能，在代码合并会时会自动帮你完成。

### 如果没有冲突请跳过此步骤，如果您有冲突, 就需要解决它们并且继续变基操作

```Bash
git add <file1> <file2> ...
git rebase --continue
```

### 推送您的（功能）分支。

```Bash
git push -f
```

当您进行变基操作时，您会改变功能分支的提交历史，那么您必须使用 **-f** 或 **--force** 才能数强制推送到远程功能分支。

### 提交一个合并请求（Merge Request）

### 合并请求会被负责代码审查的同事接受，合并和关闭

### 如果您完成了开发，请记得删除您的本地分支

```Bash
git branch -D <your-feature-branch>
```

某些第三方 Git 代码管理平台（如 Github/Gitlab）在合并 PR/MR 时提供自动删除分支功能。

## Git 提交消息格式

请移玉步至 [How to Write a Git Commit Message](https://chris.beams.io/posts/git-commit/)

TLDR:

* 用一个空行来分隔开主题和正文
* 主题限制在 50 个字符内
* 主题行首字母开始
* 主题行不要以句号结尾
* 主题行使用祈使语气
* 正文的每行限制在 72 个字符内，超过部分折行显示
* 使用正文部分去解释 是什么 和 为什么 而不是 怎么做

推荐二个值得参考的 Git 提交消息最佳实践：

* [Udacity Git Commit Message Style Guide](https://udacity.github.io/git-styleguide/)
* [AngluarJS Commit Message Format](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#commit-message-format)

二者大同小异，完全满足上面这七大原则，都可以帮助团队统一 Git 提交消息格式，从而与他人协作更加方便。

> 推荐使用 commitizen 来帮助团队规范化整个流程，与 npm scripts 配合自动化整个流程。

## 自动检测 Git 提交消息格式

随着团队人员增多，我们需要一种工具能够自动化帮助我们检测 Git 提交消息格式是否满足规范要求。推荐使用 [marionebl/commitlint](https://github.com/marionebl/commitlint) 来自动化 Lint Git 提交的消息格式，并且配合 **prepush hook** 一起使用。

## [自动生成 CHANGELOG](https://github.com/conventional-changelog/conventional-changelog)

如果是库或是框架的开发，我们需要提供 **CHANGELOG**，以让团队内部和外部人员知道我们最近所做的更新，如修改的 Bugs，新增加的功能或是其它性能上的提升等等。
