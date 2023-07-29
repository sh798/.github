# 固件代码分支管理规范

Git提供了有效且开放的代码管理能力。正是由于其开放性，我们需要自建规则以便更好地利用其tag、branch特性，为我们提高代码开发、管理效率。

Git的使用必须遵守分支、经常合并和始终保持同步的原则。Git的分支是轻量级的操作，我们需要频繁建立分支，经常合并分支，并始终与中心库保持同步。

保持同步特别重要，其内涵是保持及时的推送和拉取。不及时同步将引起大量的冲突，而解决冲突浪费的时间会让习惯不好的开发人员进入恶性循环。

本规范规定了git代码库中分支和标签的命名、操作规范。

## 术语定义

* origin: GIT中心仓库。虽然Git是去中心化的，但是我们仍然定义一个中心仓库。如公司项目代码的中心仓库位于https://git.chingo.net上；而使用Github时，Github上的项目库就是中心仓库。具体例子，水位读数盒项目的origin就是https://github.com/sh798/QC-SWD-DIFD。

## 主分支

主分支特指以下2个具有特定名称的分支。这2个分支是每个代码库必须具有的且长期存在的（不删除）。

1. master
2. develop

### master

**master** 分支代表项目稳定、可发布的代码状态。**origin/master**的HEAD始终代表本代码库的最新release。

当需要发布一个新的稳定版本时，把相关更改合并到master分支中，并打上相应的版本号。

### develop

**develop** 分支用于开发，保存了为下一个发布版本要推出的更改。当分支也可以称为“集成分支”，如“每晚构建”等集成工作都是基于本分支执行。

当 **develop**分支的代码达到稳定可发布状态时，应将所有更改合并回主分支，并标记版本号。

因此，每次将修改合并回master时，必定会产生一个新的release版本。这是必须遵守的原则。基于该原则，我们可以使用一个 Git 钩子脚本，在每次主版本有提交时自动构建软件并将其发布到生产服务器上。

## 支持分支

支持分支是为了支持**并行开发**而规划的。并行开发是指多个开发人员的，或者同时需要开发不同的feature、bugfix等等而乱序执行的开发任务。

支持分支都是有有限的生命周期的，最终需要被删除。

支持分支类型有：

* 特性分支（Feature branches）
* 版本分支（Release branches）
* 补丁分支（Hotfix branches）

每种分支都有其特定的目的，并受严格的规则约束，如哪些分支可以作为其原始分支，哪些分支必须作为其合并目标。

### 特性分支（Feature branches）

>>> 需要从**develop**创建，并必须合并回**develop**分支。
>>> 
>>> 命名规范：除`master`、`develop`、`release-*`或`hotfix-*`之外的任意名称。

特性分支（有时也称作主题分支）用于为即将发布或未来版本开发新特性。在开始开发某个功能时，可能还不知道该功能将被整合到哪个目标版本中。只要特性还在开发中，特性分支就一直存在，但最终会被合并回**develop**中（将新特性添加到即将发布的版本中）或被丢弃（在实际结果令人失望的情况下）。

#### 创建特性分支

假设我们的水位读数盒中要添加有关电源关闭实现超低功耗待机的特性，我们就可以将分支命名为“ultra-low-power”。通过执行以下命令创建分支：

```
$ git checkout ultra-low-power
Switched to a new branch "ultra-low-power"
```

#### 完结特性分支

经过在上述新特性分支数次提交后，和代码测试，我们认为该特性功能已经开发完毕，接下来我们需要将分支合并回**develop**，以便在下个发布版本中发布。

```
$ git checkout develop
Switched to branch `develop`
$ git merge --no-ff ultra-low-power
Updating ea1b82a..05e9545
(Summary of changes)
$ git branch -d ultra-low-power
Deleted branch ultra-low-power (was 05e9545).
$ git push origin develop
```

**--no-ff**参数用来关闭git的"fast forward"，从而强制创建一个新commit，以便在提交历史中能记住有"ultra-low-power"特性分支存在过。

### 版本分支（Release branches）

>>> 需要从**develop**分支创建，必须合并回**develop**和**master**分支。
>>>
>>> 命名规范：release-*

发布分支用于新版本发布的准备工作。

在develop分支的代码状态达到了新版本的理想状态时，即该版本需要发布的feature、修复的错误都已经开发完毕，且已经合并到了develop分支，那此时可以创建一个发布分支。

在且仅在创建发布分支时，本次发布的版本号才需要被明确。此时应该根据项目的版本编号规范分配新的版本号，并把分支命名为“release-*”的格式。

#### 创建版本分支

```
$ git checkout -b release-1.2 develop
Switched to a new branch "release-1.2"

$ 执行一系列发布准备工作

$ git commit -a -m "Bumped version number to 1.2"
```
版本分支一旦创建完毕，develop分支就又可以接收为下一个版本准备的功能了。

版本分支创建后，可能需要做一些与发布有关的其他改动工作，如手册等各类文档中的软件版本号等等。

版本分支中允许再对一些小错误进行修改，但原则上不能再做大的feature的添加。


#### 完成版本分支

当新版本发布工作准备完毕后，就可以进行下一步来完成版本发布。

需要做的工作有：

1. 将发布分支合并到主干分支（注意：主干分支上的每次提交都是一个新版本）。
2. 对主分支上的提交进行标记，以方便将来引用这个历史版本。
3. 将发布分支上的修改合并回develop分支，以便未来的版本也包含发布分支中的错误修复等。

```
$ git checkout master
Switched to branch 'master'
$ git merge --no-ff release-1.2
Merge made by recursive.
(Summary of changes)
$ git tag -a 1.2
```
现在版本发布已经完成了，并且打上了版本标签，将来要继续引用这个版本将会很方便。

因为版本分支中可能会有一些错误修订等修改，我们需要把这些修改合并回develop分支中。

```
$ git checkout develop
Switched to branch 'develop'
$ git merge --no-ff release-1.2
Merge made by recursive
(Summary of changes)
```
上述合并可能会引起冲突，比如修改代码中有关版本号的地方。但这些冲突不难解决，把它们解决掉并再提交一次即可。

至此，有关版本发布的所有工作皆已完成，是时候删除版本分支了。

```
$ git branch -d release-1.2
Deleted branch release-1.2 (was ff452fe).
```

### 补丁分支（Hotfix branches）

>>> 可以从**master**分支创建，必须合并回**develop**和**master**。
>>>
>>> 命名规范：hotfix-*

补丁分支与版本分支非常相似，它们也是为新的生产版本做准备，尽管是计划外的。补丁分支产生的原因是，必须对发布版本中的问题采取行动。可以从发布版本的主分支上的相应tag中分出一个补丁分支。我们在补丁分支上只解决原有功能上的问题，而不添加新功能。

这样做的好处是，团队成员（在develop分支上）的工作可以继续进行，而另一个人则可以专注修复。

#### 创建补丁分支

假设发布版本1.2需要打补丁，那我们根据项目版本号规范，确定一个补丁版本号，假设为“1.2.1”。

```
$ git checkout -b hotfix-1.2.1 release-1.2
Switched to a new branch "hotfix-1.2.1"
$ 执行一系列发布准备工作

$ git commit -a -m "Bumped version number to 1.2"

```
接下来，在分支上修改问题，并做一些commit。

#### 完成补丁分支

补丁代码需要合并回**master**，也需要合并回**develop**，以便在后续版本中也包括这些修订。

完成补丁分支的操作与完成版本分支是类似的。先合并回master，再合并回develop，这里不再展示。

有一个特例需要强调，当版本分支还没删除时就已经创建了补丁分支，这时候，补丁分支需要合并回版本分支，再跟随版本分支最终被合并回**develop**主分支。

## 固件版本所支持硬件版本的管理

每个代码仓库中应有固件版本所兼容硬件版本的对照表。

当较旧版本硬件需要更新固件时，只能更新到兼容的版本，不能粗暴更新到最新版本。

当较旧版本硬件的固件需要打补丁时，若相关功能在新版本中仍然存在，则代码应合并回master和develop中，反之则不再合并，且补丁分支不再删除，当后续需要再次打补丁时，从该补丁分支创建新补丁分支。新补丁分支创建后，旧补丁分支可以删除。