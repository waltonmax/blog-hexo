---
title: Git 工作流与规范
comments: false
toc: true
date: 2020-08-12 20:31:41
categories:
  - 开发规范
tags:
  - git
---

## 分支约定

　　Git Flow有主分支和辅助分支两类分支。其中主分支用于组织与软件开发、部署相关的活动；辅助分支组织为了解决特定的问题而进行的各种开发活动。

### 主分支

　　主分支是所有开发活动的核心分支。所有的开发活动产生的输出物最终都会反映到主分支的代码中。主分支分为**master**分支和**develop**分支。

#### master 分支

- master分支存放的是随时可供在生产环境中部署的稳定版本代码
- master分支保存官方发布版本历史，release tag标识不同的发布版本
- 一个项目只能有一个master分支
- **仅在发布新的可供部署的代码时才更新master分支上的代码**
- 每次更新master，都需对master添加指定格式的tag，用于发布或回滚
- master分支是保护分支，不可直接push到远程仓master分支
- master分支代码只能被release分支或hotfix分支合并

#### develop 分支

- develop分支是保存当前最新开发成果的分支
- 一个项目只能有一个develop分支
- develop分支衍生出各个feature分支
- develop分支是保护分支，不可直接push到远程仓库develop分支
- develop分支不能与master分支直接交互

### 辅助分支

　　辅助分支是用于组织解决特定问题的各种软件开发活动的分支。辅助分支主要用于组织软件新功能的并行开发、简化新功能开发代码的跟踪、辅助完成版本发布工作以及对生产代码的缺陷进行紧急修复工作。这些分支与主分支不同，通常只会在有限的时间范围内存在。

辅助分支包括：

- 用于开发新功能时所使用的feature分支
- 用于辅助版本发布的release分支
- 用于修正生产代码中的缺陷的hotfix分支

　　以上这些分支都有固定的使用目的和分支操作限制。从单纯技术的角度说，这些分支与Git其他分支并没有什么区别，但通过命名，我们定义了使用这些分支的方法。

#### feature 分支

使用规范：

- 命名规则：`feature/*`
- develop分支的功能分支
- feature分支使用develop分支作为它们的父类分支
- 以功能为单位从develop拉一个feature分支
- 每个feature分支颗粒要尽量小，以利于快速迭代和避免冲突
- 当其中一个feature分支完成后，它会合并回develop分支
- 当一个功能因为各种原因不开发了或者放弃了，这个分支直接废弃，不影响develop分支
- feature分支代码可以保存在开发者自己的代码库中而不强制提交到主代码库里
- feature分支只与develop分支交互，不能与master分支直接交互

　　如有几个同事同时开发，需要分割成几个小功能，每个人都需要从develop中拉出一个feature分支，但是每个feature颗粒要尽量小，因为它需要我们能尽早merge回develop分支，否则冲突解决起来就没完没了。同时，当一个功能因为各种原因不开发了或者放弃了，这个分支直接废弃，不影响develop分支。

#### release 分支

使用规范：

- 命名规则：`release/*`，“*”以本次发布的版本号为标识
- release分支主要用来为发布新版的测试、修复做准备
- 当需要为发布新版做准备时，从develop衍生出一个release分支
- release分支可以从develop分支上指定commit派生出
- release分支测试通过后，合并到master分支并且给master标记一个版本号
- release分支一旦建立就将独立，不可再从其他分支pull代码
- 必须合并回develop分支和master分支

　　release分支是为发布新的产品版本而设计的。在这个分支上的代码允许做小的缺陷修正、准备发布版本所需的各项说明信息（版本号、发布时间、编译时间等）。通过在release分支上进行这些工作可以让develop分支空闲出来以接受新的feature分支上的代码提交，进入新的软件开发迭代周期。

　　当develop分支上的代码已经包含了所有即将发布的版本中所计划包含的软件功能，并且已通过所有测试时，我们就可以考虑准备创建release分支了。而所有在当前即将发布的版本之外的业务需求一定要确保不能混到release分支之内（避免由此引入一些不可控的系统缺陷）。

　　成功的派生了release分支，并被赋予版本号之后，develop分支就可以为“下一个版本”服务了。所谓的“下一个版本”是在当前即将发布的版本之后发布的版本。版本号的命名可以依据项目定义的版本号命名规则进行。

#### hotfix 分支

使用规范：

- 命名规则：`hotfix/*`
- hotfix分支用来快速给已发布产品修复bug或微调功能
- 只能从master分支指定tag版本衍生出来
- 一旦完成修复bug，必须合并回master分支和develop分支
- master被合并后，应该被标记一个新的版本号
- hotfix分支一旦建立就将独立，不可再从其他分支pull代码

　　除了是计划外创建的以外，hotfix分支与release分支十分相似：都可以产生一个新的可供在生产环境部署的软件版本。

　　当生产环境中的软件遇到了异常情况或者发现了严重到必须立即修复的软件缺陷的时候，就需要从master分支上指定的TAG版本派生hotfix分支来组织代码的紧急修复工作。

　　这样做的显而易见的好处是不会打断正在进行的develop分支的开发工作，能够让团队中负责新功能开发的人与负责代码紧急修复的人并行的开展工作。

## 使用规范

> 所有使用了本规范的项目，必须严格规范操作，否则不予以合并代码、提测、打包上线等后续操作。

### 基本要求

- 所有commit必须有注释，内容必须简洁明了的描述本次commit涵盖了哪些内容。**严禁注释内容过于简单或不能明确表达提交内容的！**
- 合理控制提交内容的颗粒度，一次commit含一个独立功能点。严禁一次提交涵盖多个功能项。
- 正确为每个项目设置Git提交用到的user.name和user.email信息，以公司邮箱为准，不可随意设置以影响无法正确识别。 查看当前项目配置信息的命令：`git config -l`

### 版本号(tag)

- 版本号(tag)命名规则：`主版本号.次版本号.修订号`，如`2.1.13`。(遵循GitHub[语义化版本]命名规范)
- 版本号仅标记于master分支，用于标识某个可发布/回滚的版本代码
- 对master标记tag意味着该tag能发布到生产环境
- 对master分支代码的每一次更新(合并)必须标记版本号
- 仅项目管理员有权限对master进行合并和标记版本号

### 项目权限

- Git权限分管理员、开发者、浏览者三种类型
- 浏览者只能浏览代码，无push、pull request等所有写权限
- 开发者拥有浏览、push非主分支、提交pull request工单权限
- 管理员拥有建立和管理Git项目、合并分支和代码、给master打tag版本号等权限

### 分支使用

- 每个Git项目固定含有上述所有分支类型。主分支master和develop是保护分支，只能进行合并请求，均不可直接提交代码。
- 功能需求或常规Bug修复，请从develop拉取feature分支；线上紧急问题修复，请从master拉去hotfix分支。

### 代码提交

- 一个提交就代表解决一个问题
- 大问题适当地分解为多个小问题，以便每次小型提交都更易于理解

### 注释格式

- 第1行，标题行，对提交的简要总结，长度不超过50个字符，用语采用命令式而非过去式
- 第2行，空行
- 第3行开始，正文内容，对改动的详细介绍），可以是多行内容，建议每行长度不超过72个字符
- 正文用于解释`是什么`和`为什么`，而不是如何做
- 如果有对应的issue，则将issue的id或链接放在注释正文中
- 并非强制所有提交都要求3行+格式的注释，但除非标题行内容就能清晰的描述提交内容，否则必须采用3行+的注释格式

> **为什么要约定注释格式？** 1. 加快 Reviewing Code 的过程 2. 帮助我们写好 release note 3. 5年后帮你快速想起来某个分支，tag 或者 commit 增加了什么功能，改变了哪些代码 4. 让其他的开发者在运行 `git blame` 的时候想跪谢 5. 其他小伙伴不会出现想抽你的冲动 6. 总之，一个好的提交信息，会帮助你提高项目的整体质量
>
> 看看 Linus Torvalds 在Linux项目上写的提交注释：https://github.com/torvalds/linux/commits/master 以及 Linus Torvalds 关于提交注释的讨论：https://github.com/torvalds/linux/pull/17#issuecomment-5659933

如果怕自己忘记或尚未养成习惯，可以设置模板：

```
50-character subject line

72-character wrapped longer description. This should answer:

* Why was this change necessary?
* How does it address the problem?
* Are there any side effects?

Include a link to the ticket, if any.
```

下面是一则推荐的提交注释内容：

```
Redirect user to the requested page after login

https://trello.com/path/to/relevant/card

Users were being redirected to the home page after login, which is less
useful than redirecting to the page they had originally requested before
being redirected to the login form.

* Store requested path in a session variable
* Redirect to the stored location after successfully logging in the user
```



## 推荐工具

- [SourceTree](https://www.sourcetreeapp.com/) SourceTree集成了Git Flow功能，能简单方便的操作和实现常规的工作流程。支持OSX和Windows平台。
- [Git Flow 扩展](https://github.com/nvie/gitflow) Git Flow模型提出者nvie写的Git命令集扩展，提供了极出色的命令帮助以及输出提示。支持OSX，Linux和Windows平台