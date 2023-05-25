# Git 协作开发流程与规范

## 信息来源

知乎 [web 前端面试宝典](https://www.zhihu.com/people/webqian-duan-mian-shi-bao-dian) - [git 命名规范和协作开发流程](https://zhuanlan.zhihu.com/p/406903738)

稀土掘金 [稻草叔叔](https://juejin.cn/user/2031553216253559) - [您必须知道的 Git 分支开发规范](https://juejin.cn/post/6844903635533594632)

CSND [Git 常用命令及方法大全](https://www.cnblogs.com/miracle77hp/articles/11163532.html)

## 开始

Git 是目前最流行的源代码管理工具。为规范开发，保持代码提交记录以及 git 分支结构清晰，方便后续维护，现规范 git 的相关操作。

## 分支管理

### 常设分支

git 版本库的两条主要的分支： `master` 和 `develop` .

> 从 2020 年 10 月 1 日开始，GitHub 上的所有新库都将用中性词「main」命名，取代原来的「master」，因为后者是一个容易让人联想到奴隶制的术语。所以 `main` 和 `master` 都可以作为主分支名称

**主分支 [master | main]** 分支

- `master` 分支由版本库初始化后自动创建，主要用于部署生产环境的分支，要确保 `master` 分支的稳定性
- `master` 分支一般由 `develop` 以及 `hotfix` 分支合并，任何时间都不能直接修改代码
- `master`分支只能管理员可以进行 `push` 操作，他人若要合并分支到 `master` 需要提 `merge request` 由管理员进行 `code review` 之后再合并

**开发分支 [develop | dev]** 分支

- `develop` 为开发分支，始终保持最新开发完成以及 `bug` 修复后的代码
- 一般开发新的功能时，`feature` 分支都是基于 `develop` 分支创建的

### 临时性分支

**功能分支 feature**

- 开发新功能时，从 `develop` 分支上切出 `feature` 分支
- 分支命名规范：`feature/` 开头，后面跟有意义的新功能名或模块名，如：`feature/user_management`(用户管理需求)、`feature/power_manangement`(电源管理)
- 如果多人共用一个功能分支，那么本地代码 `push` 之前一定要经过自测，至少保证主流程走通，页面正常访问。

**预发布分支 release**

- 它是指发布正式版本之前（即合并到 Master 分支之前），我们可能需要有一个预发布的版本进行测试。
- 预发布分支是从 Develop 分支上面分出来的，预发布结束以后，必须合并进 Develop 和 Master 分支。它的命名，可以采用 release-\*的形式。

**修复分支 hotfix**

- 如果线上出现紧急问题，需及时处理时，则需要修复分支 `hotfix` 进行 `bug` 修复
- 分支命名规范：`hotfix/xxx`，命名规则和 `feature` 类似
- 修复分支需从 `master` 主分支上创建，修复完成后，需要合并到 `develop` 和 `master` 分支

## 开发流程

1.新增功能

当接到一个新需求，需要新建分支，开发需求，提交代码

```text
(master)$: git pull                           # 基于 master 分支 创建新分支之前 拉最新代码
(feature/xxx)$: git checkout -b feature/xxx   # 创建新的功能分支
# coding...
(feature/xxx)$: git add -A                    # 即将提交代码 将修改，删除和新增的文件存放在暂存区
(feature/xxx)$: git commit -m '提交内容描述'    # 将暂存区代码添加到本地仓库中
(feature/xxx)$: git push origin feature/xxx   # 将本地分支版本上传到远程仓库
```

**注意：** `git add<span> </span>.`和 `git add -A` 的区别 前者包括内容修改和新增但不包括删除 后者是所有

2.合并到开发分支

当需求开发完成后，需要合并到分支

```text
(feature/xxx)$: git checkout develop          # 将当前功能分支 切换到开发分支
(test)$: git pull origin develop              # 拉去 develop 分支最新代码
(test)$: git merge feature/xxx                # 将功能分支合并到测试分支
# 若有冲突 需要现在本地解决完所有冲突
(test|MERGING)$: git add -A                   # 将刚才修改的文件存放到暂存区
(test|MERGING)$: git commit -m '解决文件冲突..'# 将暂存区代码添加到本地仓库中
(test|MERGING)$: git push origin develop      # 将本地分支版本上传到远程仓库
# 若没有冲突 直接 push 到远程仓库
(test)$: git push origin develop              # 将本地分支版本上传到远程仓库
```

## 协作开发流程

以 GitHub 为例

如果你是项目的主导者，应该把 `main` 分支设置为受保护的分支，不能直接操作 `main` 分支

1. （远程 - 公共仓库）管理者 从 `develop` 分支新建 `feature/XX` 分支
2. （远程 - 公共仓库）GitHub 中先进入需要开发的公共仓库，然后 `Fork` 公共仓库到自己的个人空间中
3. （远程 - 个人空间）将个人空间中的仓库 `clone` 到本地（每次开发前最好先拉取一下公共仓库代码）
4. （本地-main 分支）切换到 `feature/XX` 分支
5. （本地-feature/XX 分支）在 `feature/XX` 分支进行编码
6. （本地-feature/XX 分支）将本地多次提交信息进行合并成一次提交
7. （本地-feature/XX 分支）将本地的 `feature/XX` 分支 push 推送到自己的远程个人空间仓库
8. （远程-feature/XX 分支）将远程 `feature/XX` 分支提交 `Pull Request` 到公共仓库的 `develop` 分支
9. （公共仓库）从 `develop` 分支分出 `release-*` 分支进行预发布
10. （公共仓库）预发布测试结束后，将 `release-*` 分支与 `develop` 分支、`main` 分支进行合并
