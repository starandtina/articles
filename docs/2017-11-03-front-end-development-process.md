# 前端开发流程

1.  分配任务

> 团队成员应该积极主动去领取任务，并且将相应的 Story 分配给自己，做好前期技术需求调查，或者是由 Team Lead 分配相应的任务给团队成员

2.  基于 [develop](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow) 分支拉取最新的代码

> 做相应 Story 的人必须与其他同事（后端，PM, QA, etc.）一起过一下业务需求，并且尽早提前调查，越早越好。必须保证什么是我们将要做的。有任何问题，如果有必要随时去找 PM/QA 并且需要添加你的子任务到 Story 中。

3.  基于 feature branch 开发

> 建议以 **姓名-ticket-简短描述**命名你的 feature brnach，比如，_shantinggui-supplyChain-ad_。

> 这样做的话，我们就可以隔离我们的开发代码在一个特定分支上，而不是在我们的主分支上，从而保证主分支是随时可以发布的。

4.  一旦 Story 分配至相应开发人员，Story 的开发人员必须清楚知道自己是 Owner，如果用了 Jira，必须将 **Assignee** 指定给自己。

5.  如果某个 Story 过于复杂，或是它需要修改我们的底层框架核心（如 Redux Middlewares）或是共享组件，Story Owner 应该创建更加详细的设计文档

    * 在写代码之前多想想，你会发现其实你并不需要花更多的时间在实现上，比如我们应该先设计我们的 State（数据模型），才开始写代码可能效果更好
    * 一旦设计文档写完了，Story Owner 应该计划一次设计评审会议，与其他的开发人员一起评审
    * 一旦设计文档没有问题了，Story Owner 可以开始根据已有设计文档写代码
    * 关于设计如果有任何改变，尽早通知团队成员，从而做出相应调整

6.  根据自己的时间和所需要交付的东西规划开发时间，同时也要考虑留更多的时间在写测试上面

7.  在你本地的开发机器上先测试，保证基本需求跑通

8.  如果使用 Jira，那么需要记录你的开发时间，并且当代码合并到主干分支后，关闭相应的子任务

9.  基于你的 feature branch 提交代码

> 确保 Git 提交消息格式满足[Git Commit Guidelines](https://github.com/angular/angular.js/blob/master/CONTRIBUTING.md#-git-commit-guidelines)。

> 同时更新你本地的 develop 分支，在提交代码前，执行一次[交互式的 rebase](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)，这样可以保证提交历史记录干净并且是线性的。

10. 创建一个 **Merge Request**，用于 Code Review

    * 为 MR 添加合适的标题，描述和相应的 Jira 链接等等
    * 添加 Labels，如 **Code Review**, **Feature**, **Enhancement**, **Bug** or **Test**
    * 指定相应的 **Assignee** 来做 CR，举个例子，沙师弟可能会添加小尉迟作为 **Assignee**，那可能是因为他们共同一起开发一个项目
    * 如果需要其他团队成员参与 CR，可以在 Comment 中 @ 相应人员
    * 每次代码评审必须少于 200-400 行，尽量避免一次性提交数十个文件或是数千行代码 :(
    * 每个 MR 会触发 CI pipeline，每次它都会跑代码 Lint 和测试并且生成代码覆盖率报告。如果最小测试覆盖率没有达到，那么测试框架会返回失败，从而触发这次构建失败，那么你的 MR 也不能自动合并。

> CR 的目的是尽量减少代码的 Bugs，而并不会管是谁造成的这个 Bug。同时，我们必须严格遵循 **pre-commit** CR。

11. 一旦 CR 完成了，接受 MR，并且删除 **Code Review** 标签。

大多数时候，你可能会看见代码合并冲突，请提前解决合并冲突。为了一个更加干净线性的提交历史记录，推荐使用 `git rebase`。

> 无论何时创建 MR 时，请确保**Remove source branch when merge request is accepted** 和 **Squash commits when merge request is accepted** 是勾选上的。因为如果勾选上的话，那么接受 MR 后，你的 feature branch 就会被自动删除掉，同时它也会整理该 feature branch 上的提交历史记录。它只会生成一个提交，之后使用项目设置的合并方法来合并这个提交。

12. 设置相应的 Jira 任务为完成，之后将 Story 指派给 QA 测试

指派给 QA 时，尽量添加更多的在用的信息为测试，如项目发布的版本，测试的范围，以及未完成的任务等等。

## 重要提示

* 请确保更新合并后，主分支的构建是成功的
* 请确保代码合并后，已经在集成开发环境中进行了充分的测试
* 提交 MR 时，请确保满足 [SRP](https://en.wikipedia.org/wiki/Single_responsibility_principle) 原则，这是软件开发过程中的一个基本最佳实践，只把相关的放在一起，并且隔离其它不相关的。这个同样也适用于 MR:
  * 如果 MR 的 scope 是简明直接的，那么会更加容易去做代码评审
  * 更容易去 revert 更新（因为相关的更新都在同一个 MR 中）
