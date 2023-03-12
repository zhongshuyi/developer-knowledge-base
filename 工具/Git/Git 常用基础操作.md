---
order: 1

author: 钟舒艺
---
# Git 常用基础操作

写了一些常用的 Git 命令与初次使用的配置

## 信息来源

博客园 [Git 常用命令及方法大全](https://www.cnblogs.com/miracle77hp/articles/11163532.html)

[Git 官方文档](https://git-scm.com/book/zh/v2)

[博客园 - ssh 公钥和私钥生成](https://www.cnblogs.com/Jamesblog/p/16216967.html)

## 准备

### 安装

[安装 Git](https://git-scm.com/book/zh/v2/%E8%B5%B7%E6%AD%A5-%E5%AE%89%E8%A3%85-Git)

### 配置信息

如果你需要检查你的当前配置（加上 --global 为全局配置）

```bash
git config --list [--global]
```

直接在文本编辑器中修改配置

```bash
git config -e [--global]
```

### 提交信息配置

安装完 Git 之后，要做的第一件事就是设置你的用户名和邮件地址。这一点很重要，因为每一个 Git 提交都会使用这些信息，它们会写入到你的每一次提交中，不可更改

```bash
# 设置用户名
git config --global user.name "John Doe"
# 设置用户邮箱
git config --global user.email johndoe@example.com
```

### 行尾序列配置

因为 widow 的换行是 CRLF 而 linux 的换行是 LF，所以本地去 git 上拉取代码的时候，如果是 window 的话会把你转成 CRLF，你提交时候会转成 LF 提交

但我希望这些都有一个规范自己控制

行尾部序列检查：

```bash
git config --global core.safecrlf [true | false | warn]
```

`true` ：拒绝提交包含混合换行符的文件

`false`：允许提交包含混合换行符的文件

`warn` ：提交包含混合换行符的文件时给出警告

行尾部序列转换：

```bash
git config --global core.autocrlf [true | false | input]
```

`true` ：提交时转换为 LF，检出时转换为 CRLF

`false`：提交检出均不转换提交时转换为 LF

`input` ：提交时转换为 LF，检出时不转换

这里推荐使用的配置，希望无论在什么时候都使用 LF 行尾序列

```bash
git config --global core.safecrlf true
git config --global core.autocrlf input
```

在编辑器中也设置默认的 换行符（新建文件时）

VS Code 设置：搜索 `files.eol`

IDEA 设置默认换行符： `File` -> `Settings` -> `Editor` -> `Code Style` 的 `Line separator (for new files)`: 设置为 `Unix and OS X`

如果项目中换行符不统一

可以通过 IDEA 转换，在 IDEA 中 选中当前项目然后 `File` -> `Line Separators`->`LF`

### ssh 公钥生成

ssh-keygen 用于为 ssh 生成、管理和转换认证密钥，包括 RSA 和 DSA 两种密钥。密钥类型可以用 -t 选项指定。如果没有指定则默认生成用于 SSH-2 的 RSA 密钥。

认证密钥非常重要，提交到远程 Git 仓库都需要使用密钥来完成身份认证

命令

```bash
ssh-keygen -t rsa
```

参数说明

```bash
-t： 指定密钥类型
-P： 指定密码（空字符串表示ssh连接时不需要输入密码）
-C： 注释，一般为用户的邮箱信息。缺省时为“当前系统用户@主机名”
```

执行完后一直按回车就可以了

生成的文件默认在： `C:\Users\用户名\.ssh`

## 代码库操作

### 初始化

1. 在当前目录新建一个 Git 代码库

    ```bash
    git init 
    ```

2. 新建一个目录，将其初始化为 Git 代码库

    ```bash
    git init [project-name]
    ```

3. 下载一个项目和它的整个代码历史

    ```bash
    git clone [url]
    ```

### 增加/删除文件

1. 添加指定文件到暂存区

    ```bash
    git add [file1] [file2] ...
    ```

2. 添加指定目录到暂存区，包括子目录

    ```bash
    git add [dir]
    ```

3. 添加当前目录的所有文件到暂存区

    ```bash
    git add .
    ```

4. 添加每个变化前，都会要求确认。对于同一个文件的多处变化，可以实现分次提交

    ```bash
    git add -p
    ```

5. 删除工作区文件，并且将这次删除放入暂存区

    ```bash
    git rm [file1] [file2] ...
    ```

6. 停止追踪指定文件，但该文件会保留在工作区

    ```bash
    git rm --cached [file]
    ```

7. 改名文件，并且将这个改名放入暂存区

    ```bash
    git mv [file-original] [file-renamed]
    ```

### 代码提交

1. 提交暂存区到仓库区

    ```bash
    git commit -m [message]
    ```

2. 提交暂存区的指定文件到仓库区

    ```bash
    git commit [file1] [file2] ... -m [message]
    ```

3. 提交工作区自上次 commit 之后的变化，直接到仓库区

    ```bash
    git commit -a
    ```

4. 提交时显示所有 diff 信息

    ```bash
    git commit -v
    ```

5. 使用一次新的 commit，替代上一次提交。如果代码没有任何新变化，则用来改写上一次 commit 的提交信息

    ```bash
    git commit --amend -m [message]
    ```

6. 重做上一次 commit，并包括指定文件的新变化

    ```bash
    git commit --amend [file1] [file2] ...
    ```

### 分支

1. 列出所有本地分支

    ```bash
    git branch
    ```

2. 列出所有远程分支

    ```bash
    git branch -r
    ```

3. 列出所有本地分支和远程分支

    ```bash
    git branch -a
    ```

4. 新建一个分支，但依然停留在当前分支

    ```bash
    git branch [branch-name]
    ```

5. 新建一个分支，并切换到该分支

    ```bash
    git checkout -b [branch]
    ```

6. 新建一个分支，指向指定 commit

    ```bash
    git branch [branch] [commit]
    ```

7. 新建一个分支，与指定的远程分支建立追踪关系

    ```bash
    git branch --track [branch] [remote-branch]
    ```

8. 切换到指定分支，并更新工作区

    ```bash
    git checkout [branch-name]
    ```

9. 切换到上一个分支

    ```bash
    git checkout -
    ```

10. 建立追踪关系，在现有分支与指定的远程分支之间

    ```bash
    git branch --set-upstream [branch] [remote-branch]
    ```

11. 合并指定分支到当前分支

    ```bash
    git merge [branch]
    ```

12. 选择一个 commit，合并进当前分支

    ```bash
    git cherry-pick [commit]
    ```

13. 删除分支

    ```bash
    git branch -d [branch-name]
    ```

14. 删除远程分支

    ```bash
    git push origin --delete [branch-name]
    git branch -dr [remote/branch]
    ```

### 标签

1. 列出所有 tag

    ```bash
    git tag
    ```

2. 新建一个 tag 在当前 commit

    ```bash
    git tag [tag]
    ```

3. 新建一个 tag 在指定 commit

    ```bash
    git tag [tag] [commit]
    ```

4. 删除本地 tag

    ```bash
    git tag -d [tag]
    ```

5. 删除远程 tag

    ```bash
    git push origin :refs/tags/[tagName]
    ```

6. 查看 tag 信息

    ```bash
    git show [tag]
    ```

7. 提交指定 tag

    ```bash
    git push [remote] [tag]
    ```

8. 提交所有 tag

    ```bash
    git push [remote] --tags
    ```

9. 新建一个分支，指向某个 tag

    ```bash
    git checkout -b [branch] [tag]
    ```

### 查看信息

1. 显示有变更的文件

    ```bash
    git status
    ```

2. 显示当前分支的版本历史

    ```bash
    git log
    ```

3. 显示 commit 历史，以及每次 commit 发生变更的文件

    ```bash
    git log --stat
    ```

4. 搜索提交历史，根据关键词

    ```bash
    git log -S [keyword]
    ```

5. 显示某个 commit 之后的所有变动，每个 commit 占据一行

    ```bash
    git log [tag] HEAD --pretty=format:%s
    ```

6. 显示某个 commit 之后的所有变动，其"提交说明"必须符合搜索条件

    ```bash
    git log [tag] HEAD --grep feature
    ```

7. 显示某个文件的版本历史，包括文件改名

    ```bash
    git log --follow [file]
    git whatchanged [file]
    ```

8. 显示指定文件相关的每一次 diff

    ```bash
    git log -p [file]
    ```

9. 显示过去 5 次提交

    ```bash
    git log -5 --pretty --oneline
    ```

10. 显示所有提交过的用户，按提交次数排序

    ```bash
    git shortlog -sn
    ```

11. 显示指定文件是什么人在什么时间修改过

    ```bash
    git blame [file]
    ```

12. 显示暂存区和工作区的差异

    ```bash
    git diff
    ```

13. 显示暂存区和上一个 commit 的差异

    ```bash
    git diff --cached [file]
    ```

14. 显示工作区与当前分支最新 commit 之间的差异

    ```bash
    git diff HEAD
    ```

15. 显示两次提交之间的差异

    ```bash
    git diff [first-branch]...[second-branch]
    ```

16. 显示今天你写了多少行代码

    ```bash
    git diff --shortstat "@{0 day ago}"
    ```

17. 显示某次提交的元数据和内容变化

    ```bash
    git show [commit]
    ```

18. 显示某次提交发生变化的文件

    ```bash
    git show --name-only [commit]
    ```

19. 显示某次提交时，某个文件的内容

    ```bash
    git show [commit]:[filename]
    ```

20. 显示当前分支的最近几次提交

    ```bash
    git reflog
    ```

### 远程同步

1. 下载远程仓库的所有变动

    ```bash
    git fetch [remote]
    ```

2. 显示所有远程仓库

    ```bash
    git remote -v
    ```

3. 显示某个远程仓库的信息

    ```bash
    git remote show [remote]
    ```

4. 增加一个新的远程仓库，并命名

    ```bash
    git remote add [shortname] [url]
    ```

5. 取回远程仓库的变化，并与本地分支合并

    ```bash
    git pull [remote] [branch]
    ```

6. 上传本地指定分支到远程仓库

    ```bash
    git push [remote] [branch]
    ```

7. 强行推送当前分支到远程仓库，即使有冲突

    ```bash
    git push [remote] --force
    ```

8. 推送所有分支到远程仓库

    ```bash
    git push [remote] --all
    ```

### 撤销

1. 恢复暂存区的指定文件到工作区

    ```bash
    git checkout [file]
    ```

2. 恢复某个 commit 的指定文件到暂存区和工作区

    ```bash
    git checkout [commit] [file]
    ```

3. 恢复暂存区的所有文件到工作区

    ```bash
    git checkout .
    ```

4. 重置暂存区的指定文件，与上一次 commit 保持一致，但工作区不变

    ```bash
    git reset [file]
    ```

5. 重置暂存区与工作区，与上一次 commit 保持一致

    ```bash
    git reset --hard
    ```

6. 重置当前分支的指针为指定 commit，同时重置暂存区，但工作区不变

    ```bash
    git reset [commit]
    ```

7. 重置当前分支的 HEAD 为指定 commit，同时重置暂存区和工作区，与指定 commit 一致

    ```bash
    git reset --hard [commit]
    ```

8. 重置当前 HEAD 为指定 commit，但保持暂存区和工作区不变

    ```bash
    git reset --keep [commit]
    ```

9. 新建一个 commit，用来撤销指定 commit 后者的所有变化都将被前者抵消，并且应用到当前分支

    ```bash
    git revert [commit]
    ```

10. 暂时将未提交的变化移除，稍后再移入

    ```bash
    git stash
    git stash pop
    ```

### 其他

生成一个可供发布的压缩包

```bash
git archive
```
