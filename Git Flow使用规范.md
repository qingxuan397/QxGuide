# 1. Git Flow使用规范

## 1.1. 简介

　　Gitflow工作流程围绕项目发布定义了严格的分支模型。尽管它比Feature Branch Workflow更复杂一些，但它也为管理更大规模的项目提供了坚实的框架。
　　Gitflow流程特色在于，它为不同的分支分配了非常明确的角色，并且定义了使用场景和用法。除了用于功能开发的分支，它还使用独立的分支进行发布前的准备、记录以及后期维护。

## 1.2. 工作原理

　　流程仍然使用一个中央代码仓库，它是所有开发者的信息交流中心。跟其他的工作流程一样，开发者在本地完成开发，然后再将分支代码推送到中央仓库。唯一不同的是项目中分支的结构。

### 1.2.1. 用于功能开发的分支

　　每一个新功能的开发都应该各自使用独立的分支。为了备份或便于团队之间的合作，这种分支也可以被推送到中央仓库。但是，在创建新的功能开发分支时，父分支应该选择develop（而不是master）。当功能开发完成时，改动的代码应该被合并（merge）到develop分支。功能开发永远不应该直接牵扯到master。

![img](http://doc.gt.huayu.nd/images/Git%20Flow%E4%BD%BF%E7%94%A8%E8%A7%84%E8%8C%83/gitflow-dev.png) 　

### 1.2.2. 用于发布的分支

　　一旦develop分支积聚了足够多的新功能（或者预定的发布日期临近了），你可以基于develop分支建立一个用于产品发布的分支。这个分支的创建意味着一个发布周期的开始，也意味着本次发布不会再增加新的功能——在这个分支上只能修复bug，做一些文档工作或者跟发布相关的任务。在一切准备就绪的时候，这个分支会被合并入master，并且用版本号打上标签。另外，发布分支上的改动还应该合并入develop分支——在发布周期内，develop分支仍然在被使用（一些开发者会把其他功能集成到develop分支）。使用专门的一个分支来为发布做准备的好处是，在一个团队忙于当前的发布的同时，另一个团队可以继续为接下来的一次发布开发新功能。

![img](http://doc.gt.huayu.nd/images/Git%20Flow%E4%BD%BF%E7%94%A8%E8%A7%84%E8%8C%83/gitflow-release.png)

### 1.2.3. 用于维护的分支

　　发布后的维护工作或者紧急问题的快速修复也需要使用一个独立的分支。这是唯一一种可以直接基于master创建的分支。一旦问题被修复了，所做的改动应该被合并入master和develop分支（或者用于当前发布的分支）。在这之后，master上还要使用更新的版本号打好标签。

![img](http://doc.gt.huayu.nd/images/Git%20Flow%E4%BD%BF%E7%94%A8%E8%A7%84%E8%8C%83/gitflow-bug.png)

## 1.3. 实际应用

![img](http://doc.gt.huayu.nd/images/Git%20Flow%E4%BD%BF%E7%94%A8%E8%A7%84%E8%8C%83/gitflow-edu.png)

## 1.4. 资料参考

- [Git使用](http://wiki.sdp.nd/index.php?title=Git使用)
- [git-flow 备忘清单](http://danielkummer.github.io/git-flow-cheatsheet/index.zh_CN.html)