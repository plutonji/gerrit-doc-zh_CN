# 项目所有者指南

> 版本：V2.13

[TOC]

本指南面向项目所有者。本文解释了 Gerrit 提供的多种自定义项目工作流程的可能性。

## 什么是项目所有者？

作为项目所有者意味着你拥有 Gerrit 中的一个项目。从技术上来说就是在项目的 `refs/*` 引用上具有 Owner 访问权限。作为项目所有者，你有权编辑访问控制列表和项目的设置。同时这也意味着你应当熟悉这些设置，这样你才能使设置符合项目的需要。

作为项目所有者意味着需对项目的管理负责。这要求你比一般用户具备更深刻的 Gerrit 知识。一般来说，一个团队中应该有 2 到 3 名具备足够 Git/Gerrit 知识水平的项目所有者。团队中的所有人都是项目所有者的做法一般来说没什么意义。对于普通的团队成员而言，作为 Committer 或贡献者就已经足够了。

## 访问权限

作为项目所有者，你可以编辑项目的访问控制列表。这就允许你将项目的权限分配给不同的用户组。

Gerrit 提供了丰富的权限种类，可以对谁能在项目上做什么进行非常细粒度的控制。访问控制是 Gerrit 最强大的特性之一，但也是一个复杂的话题。本指南仅强调访问控制最重要的方面，[访问控制](./access-control.md)一章详细解释了所有细节。

### 编辑访问权限

要查看项目的访问权限：

- 打开 Gerrit 网页
- 点击 `Projects` -> `List` 菜单项
- 在项目列表中找到你的项目并单击
- 点击 `Access` 菜单项

通过点击 `Edit` 按钮即可编辑访问权限，然后你可以点击 `Save Changes` 按钮保存修改。你也可以选择性地提供一个 commit 信息（Commit Message）来解释改变权限的原因。

访问权限以名为 `refs/meta/config` 的特殊分支保存在项目的 Git 仓库中。在这个分支中有一个包含了访问权限的 `project.config` 文件 。更多关于这个存储格式的信息可以在[项目配置文件格式](./config-project-config.md)一章查看。重要的是要知道，通过查看 `refs/meta/config` 分支上 `project.config` 文件的历史记录，你可以随时查看访问权限如何变化以及由谁更改。如果提供了良好的 commit 消息，你还可以从历史记录中查看修改访问权限的原因。

如果为 Gerrit 服务器配置了 git 浏览器（如 gitweb ），那么你可以在 Web UI 中找到指向 `project.config` 文件历史记录的链接。否则，你可以在本地查看历史记录。如果已克隆仓库，则可以通过执行以下命令来执行此操作：

```console
  $ git fetch origin refs/meta/config:config
  $ git checkout config
  $ git log project.config
```

非项目所有者仍可以通过单击 `Save for Review` 按钮来编辑访问权限并向项目所有者提出修改建议。这将创建一个新的变更，其中包含可由项目所有者批准的访问权限修改。项目所有者将自动添加为此更改的审阅者，以便他们通过电子邮件获得相关信息。

### 继承

通常，当一个新项目在 Gerrit 中创建时，它就已经拥有一些从父项目继承的访问权限。Gerrit 中的项目按层次结构组织为一个树，以 “All-Projects” 项目为根，所有项目都从该项目继承。每个项目只能有一个父项目，不支持多继承。

当你在 Gerrit Web UI 中查看项目的访问权限时，你只能看到在该项目上定义的访问权限。 要查看继承的访问权限，你必须查看 `Rights Inherit From`下的父项目的链接。

继承的访问权限除非定义为 [BLOCK 规则](./access-control.md#block)，否则是可以被覆盖的。BLOCK 规则用于限制所继承项目的项目所有者覆盖权限的可能性。有了这个，就可以对所有项目实施全局策略。请注意，Gerrit 不会阻止你分配与继承的BLOCK 规则相矛盾的访问权限，但这些访问权限将无效。

如果你负责几个需要相同权限的项目，那么为它们建立一个共同父项目并维护该公共父项目的访问权限是有意义的。只有 Gerrit 管理员才可以更改项目的父项目。这意味着如果要重新选择父项目，你需要联系 Gerrit 服务器的管理员。 一种方法是在 Web UI 中更改父项目，保存修改以供审阅，并经 Gerrit 管理员批准和合并后获取变更。

### 引用

Gerrit 中的访问权限分配在引用（refs）上。Git中的引用存在于不同的名字空间中，例如：所有的分支（branch）通常存在于 `refs/heads/`下，而所有的标签（tag）则在 `refs/tags/` 下。另外还有一些特殊引用和魔法引用。

访问权限既可以分配在具体的引用上，如 `refs/heads/master` ，也可以分配在引用模式和正则表达式上。

以 `/*` 结尾的引用模式描述了一个完整的引用名字空间，比如分配在 `refs/heads/*` 上的访问权限应用到了所有分支上。

正则表达式必须以 `^` 开头，比如分配在 `^refs/heads/rel-.*` 上的访问权限应用到了所有开头是 `rel-` 的分支上。

### 组

访问权限是分配给组的。有必要知道的是，Gerrit 不但维护自己内部的组，还能支持不同的外部组数据源。Gerrit 内部组可以在 Gerrit Web UI 中通过点击 `Groups` > `List` 项看到。点击一个组，你可以编辑组成员（`Members` 标签卡）和组选项（`General` 标签卡）。

Gerrit 内部组的成员可以是用户，也可以包括其他的组，甚至是外部组。

每一个组都归所有者组所有。只有所有者组的成员才可以管理所拥有的组，包括分配成员、编辑组选项。一个组可以拥有自己。在这种情况下，组成员就可以添加其他成员到这个组。当你为项目创建新组来为 Committer 或其他角色分配访问权限时，请确保它们归项目所有者组所有。

组的一个重要设置是选项 `Make group visible to all registered users.`，它定义了组以外的成员是否可以查看该组的成员信息。

新的 Gerrit 内部组可以在 `Groups` > `Create New Group` 下创建。这个菜单只有在你有全局能力 [Create Group](./access-control.html#capability_createGroup) 时才可用。

Gerrit 还有一组你可能会觉得有用的[特殊系统组](access-control.html#system_groups)。

外部组在赋予权限时需要加上前缀。比如 [LDAP 组名](access-control.html#ldap_groups)前需加上 `ldap/` 前缀。

如果安装了 [singleusergroup](https://gerrit-review.googlesource.com/admin/repos/plugins%2Fsingleusergroup) 插件，你就可以通过在用户名前面加上前缀 `user/` 或在用户 ID 前面加上前缀 `userid/` 的方式直接将权限赋予用户。

### 常用的访问权限

项目中的不同角色，例如开发者（Committer）或贡献者，需要不同的访问权限。“访问控制”一文的[此章节](access-control.html#example_roles)中描述了为哪个角色分配哪些访问权限的经典示例。

### 代码评审

尽管 Gerrit 的主要功能是代码评审，然而使用代码评审只是可选项，你可以决定只把 Gerrit 当做一个具有访问控制能力的 Git 服务器使用。是否允许评审或直接推送取决于项目的访问权限。

要推送 commit 以进行审核，必须将其推送到魔法引用 [`refs/for/<branch-name>`](access-control.html#refs_for) 上。也就是说 Push 访问权限必须分配到 `refs/for/<branch-name>` 上。

要允许绕过代码评审而直接推送，分支引用 `refs/heads/<branch-name>` 就需要 Push 访问权限。

通过推送评审，您不仅可以启用审核工作流，还可以在合并变更之前从构建服务器上获取自动验证。 此外，您可以从Gerrit的合并策略中受益，如果需要，可以在服务器端自动合并/变基 commit。 您可以通过在项目上配置提交类型来控制合并策略。 如果您绕过代码检查，则在目标分支的提示已移动的情况下总是需要手动合并/变基。 请记住这一点，如果您因为认为避免审核工作流程的额外复杂性而选择不使用代码审核，但实际上可能并不会更简单。

你也可以启用[推送时自动合并](user-upload.html#auto_merge)，以在直接推送到仓库时自动合并/变基。

## 项目选项

作为项目所有者，你可以控制项目的几个选项。各选项的详细释义参见“项目配置”的[项目选项](https://gerrit-documentation.storage.googleapis.com/Documentation/2.13/project-configuration.html#project_options)一节。

要查看项目的选项：

- 打开 Gerrit Web UI
- 点击 `Projects` > `List` 菜单项
- 在列表中找到你的项目并点击
- 点击 `General` 按钮

### 提交类型

对于提交类型和内容合并设置（见 `Allow content merges` 选项）的选择是项目的一个重大决定。提交类型（submit type）是 Gerrit 提交变更到项目的使用方式，其定义了在变更评审期间目的分支如果已删除的情况下 Gerrit 应该做什么。内容合并（content merge）设置了当一个文件同时被修改时，Gerrit 是否需要尝试合并文件。

在选择提交类型和内容合并选项时，项目所有者需要在开发舒适性和不破坏远程分支的安全性之间权衡。

最具约束力的提交类型是 `Fast Forward Only`。使用此提交类型意味着在提交一个变更之后，同一目标分支上所有其他打开的变更必须手动变基。这是相当繁重的，而且在实践中只对很少更改的分支可行。另一方面，如果在提交之前验证了变更，例如通过使用这种提交类型的 CI 集成自动验证变更，则可以确保目标分支不会被破坏。

选择 `Merge If Necessary` 作为提交类型可以使开发人员的生活更加舒适，尤其是在启用了内容合并的情况下。 如果使用此提交策略，则开发人员只有在同时修改了相同的文件或者此类文件上的内容合并失败时再手动变基。 这种提交类型的缺点是存在破坏目标分支的风险，例如， 如果一个变更将一个类移动到另一个包中，另一个变更从旧位置导入该类。 经验表明，在实践中，启用了内容合并的 `Merge If Necessary` 设置效果很好，很少会破坏目标分支。 这就是为什么至少对开发分支推荐此设置的原因。 您可能希望从启用内容合并的 `Merge If Necessary` 开始，如果您遇到目标分支的构建和测试稳定性问题，则只需切换到更严格的策略即可。

也可以通过 Prolog 动态定义提交类型。这种方式让你可以对于不同的分支使用不同的提交类型。

## 标签（Label）

在形式上，代码评审过程包括评审者通过在不同的标签上投票表达他们对于一个变更的观点。Gerrit 默认带有 Code-Review 标签，许多 Gerrit 服务器全局配置了 Verified 标签。项目还可以自定义他们自己的标签来实现项目相关的工作流。举个例子，如果一个项目需要具有特定 IP 的团队的同意，那么可以定义一个 `IP-Review` 的标签，并将相应的权限赋给这个 IP 团队。

 一个标签的行为可以通过其 function 控制，比如它可以设置必须有最大同意票分才能提交，也可以设置投票为可选项。

通过设置自定义的提交规则，可以控制每个变更是否必须某个标签才能提交。

标签有个有用的特性，就是在三方变基或没有代码变化时（比如只编辑了提交信息）自动复制之前的分数到新的补丁集上来。

## 提交规则

提交规则是变更是否可以提交的逻辑。默认情况下，当变更在所有标签上获得至少一个最大同意分且没有最大反对分时就可以提交。

Gerrit 的提交规则通过 Prolog 语言实现。项目如果需要更灵活的规则，可以定义自己的提交规则来决定变更是否可以提交。Prolog cookbook 中有一个很好的例子，展示了一个变更如何只在拥有 `Code-Review+2` ,且投票提交者不是变更的作者的情况下才允许提交。这样，可以强制执行评审的四眼原则。

Prolog 提交规则可以访问所检测可提交性的变更的信息。在其他可访问的已修改文件列表中，允许创建特定文件时应用特殊的逻辑。例如，如果项目的依赖关系发生更改，通常的做法是要求对额外的标签（如`Library-Compliance`）进行投票。

也可以用 Prolog 控制提交类型。比如说对于稳定分支使用严格的提交类型（如 `Fast Forward Only`），而对于开发分支则使用宽松的提交类型（如 `Merge If Necessary`）。可以在 Prolog cookbook 中看到实现的[示例](prolog-cookbook.md#SubmitTypePerBranch)。

提交规则在项目分支 `refs/meta/config` 中的 rules.pl 文件中维护。[Prolog cookbook](prolog-cookbook.md#HowToWriteSubmitRules) 中解释了如何编写提交规则。开发提交规则时也对[测试](prolog-cookbook.md#TestingSubmitRules)做了很好的支持。

## 持续集成

## Commit 验证

## 分支管理

作为项目所有者，你可以在 Gerrit Web UI 的 `Projects` > `List` > `<your project>` > `Branches` 下管理你的项目分支。对于项目所有者来说，Web UI 中的[分支创建](project-configuration.md#branch-creation)和[分支删除](project-configuration.html#branch-deletion)都无需额外的访问权限。

通过设置项目的 `HEAD` 你可以定义[默认分支](project-configuration.md#default-branch)。为方便起见，当克隆仓库时，Git会为此默认分支创建一个本地分支并将其检出。

## 邮件提醒

## 仪表盘

## 问题跟踪集成

## 评价链接

## 评审者

## 下载命令

## 主题

## 与其他工具集成

Gerrit 提供了很多与其他工具集成的可能性：

- 流事件（Stream Events）：

  SSH 命令 [stream events](cmd-stream-events.md) 允许监听 Gerrit [事件](cmd-stream-events.md#events)。其他工具可以用来对 Gerrit 的动作做出响应。

  使用 `stream-events` 命令需要 [Stream Events](access-control.md#capability_streamEvents) 全局能力。

- REST API：

  Gerrit 提供了丰富的 [REST API](rest-api.md)，其他工具可以用来查询 Gerrit 的信息或触发 Gerrit 的动作。

- Gerrit 插件：

  Gerrit 的功能可以通过插件扩展，并且有很多扩展点，比如插件可以：

  - 添加新的菜单项
  - 扩展已有屏幕或添加新的屏幕
  - 验证，比如对于新的 commit 做验证
  - 添加新的 REST 接口和 SSH 命令

  [插件开发](dev-plugins.md) 一节描述了如何开发一个 Gerrit 插件。

## 项目生命周期

### 项目创建

新项目可以在 Gerrit Web UI 的 `Projects` > `Create Project` 下创建。你只有被赋予 `Create Project` 这个全局能力时才能看到 `Create Project` 菜单项。

新项目也可以通过 REST 接口或 SSH 命令创建。

一般建议以一个初始化的空白 commit 创建项目，因为某些工具克隆完全空白的仓库时有些问题。不过，如果你计划向新项目导入已有的历史记录，那么创建项目时最好不要初始化的空白 commit。

### 导入已有历史记录

你可以将历史记录导入新的 Gerrit 项目。

你本地要有一个包含历史记录的 Git 仓库。如果你的代码基于其他的 VCS 系统，那么首先你需要迁移到 Git 上来。如 [Subversion 迁移指南](http://git-scm.com/book/en/Git-and-Other-Systems-Migrating-to-Git#Subversion) 所述，Subversion 的项目可以使用 `git svn` 命令。Perforce 的导入工具可以在 Git 源码的 `contrib` 部分找到。在 [Perforce 迁移指南](http://git-scm.com/book/en/Git-and-Other-Systems-Migrating-to-Git#Perforce) 中描述了如何使用 `git p4` 命令导入 Perforce 项目。

你需要有绕过代码评审直接推送到 `refs/heads/<branch>` 的权限才能导入已有历史记录。如果 Gerrit 仓库的目的分支已存在一份历史（比如初始化的空白 commit），你可以通过强制推送来覆盖。在这种情况下，你必须有项目的强制推送访问权限。

某些 Gerrit 服务器可能通过 BLOCK Forge Committer 全局禁止伪造 Committer。这时你必须使用 `git filter-branch` 命令重写Committer 信息（记录谁写了代码的作者信息保持原样；已签名标签会丢失他们的签名）：

```console
$ git filter-branch --tag-name-filter cat --env-filter 'GIT_COMMITTER_NAME="John Doe"; GIT_COMMITTER_EMAIL="john.doe@example.com";' -- --all
```

如果服务器上配置了 [max object size limit](config-gerrit.md#receive.maxObjectSizeLimit) ，你可能需要在推送前从历史记录上移除大对象。要在历史记录中找到大对象，你可以使用 Gerrit 服务器上下载下来的 `reposize.sh` 脚本：

```console
  $ curl -Lo reposize.sh http://review.example.com:8080/tools/scripts/reposize.sh

或

  $ scp -p -P 29418 john.doe@review.example.com:scripts/reposize.sh .
```

然后你可以使用 `git filter-branch` 命令从所有分支的历史中删除大对象：

```console
  $ git filter-branch -f --index-filter 'git rm --cached --ignore-unmatch path/to/large-file.jar' -- --all
```

因为这个命令重写了仓库中的所有 commit，所以从这个重写后的仓库中创建全新的克隆是个好主意，这样可以保证在推送到 Gerrit 之前原始被重写的对象已被移除。

### 项目删除

Gerrit 内核不支持项目的删除。你可以设置项目状态为 `ReadOnly` 或 `Hidden`。

如果安装了 [delete-project](https://gerrit-review.googlesource.com/#/admin/projects/plugins/delete-project) 插件，你可以在 Gerrit Web UI 的 `Projects` > `List` > `<project>` > `General` 下通过点击 `Project Commands` 的  `Delete` 命令删除项目。你应当有 `Delete Projects` 全局能力,或者如果你拥有项目并且你有 `Delete Own Projects` 全局能力才能看到 `Delete` 命令 。如果两种能力都没有，那么你就需要联系 Gerrit 管理员请求删除项目了。

### 项目重命名

Gerrit 内核不支持项目的重命名。

解决方案是：

1. 使用新的名称新建项目
2. 从旧项目导入历史记录
3. 删除旧项目

这种解决方案的缺点是所有的评审记录（变更、评审评论）都会丢失。

作为替代，你可以使用 [importer](https://gerrit.googlesource.com/plugins/importer/) 插件复制包括评审记录的项目，然后删除旧项目。