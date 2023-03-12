---
order: 2

author: 钟舒艺
---
# Git 提交信息规范

## 信息来源

CSDN [半夏\_2021](https://blog.csdn.net/fd2025) [Git 提交规范](https://blog.csdn.net/fd2025/article/details/124543690)

[Git Guide](https://zj-git-guide.readthedocs.io/zh_CN/latest/)

知乎 [leohxj](https://www.zhihu.com/people/leohxj) [优雅的提交你的 Git Commit Message](https://zhuanlan.zhihu.com/p/34223150)

## 开始

Git 是目前世界上最先进的分布式版本控制系统，在我们平时的项目开发中已经广泛使用。而当我们使用 Git 提交代码时，都需要写 Commit Message 提交说明才能够正常提交。

我们平时在编写提交说明时，通常会直接填写如"fix"或"bug"等不规范的说明，不规范的提交说明很难让人明白这次代码提交究竟是为了什么。而在工作中，一份清晰简介规范的 Commit Message 能让后续代码审查、信息查找、版本回退都更加高效可靠。因此我们需要一些工具来约束开发者编写符合规范的提交说明。

这里我们选用 Conventional 提交规范， [Conventional Commits](https://www.conventionalcommits.org/)(约定式提交) 脱胎于 `Angular` 提交信息准则，提供了更加通用、简洁和灵活的提交规范

提交格式如下

```plaintext
<类型>[可选的作用域]: <描述>
# 空一行
[可选的正文]
# 空一行
[可选的脚注]
```

## 页眉

以下为页眉

```text
<类型>[可选的作用域]: <描述>
```

页眉中必须有类型与描述，强制支持的类型名是 `fix` 和 `feat`，同样支持 `Angular` 准则中推荐的其他类型

### 类型

其中 Angular 准则中的类型有：

1. `build`：对构建系统或者外部依赖项进行了修改
2. `ci`：对 CI 配置文件或脚本进行了修改
3. `docs`：对文档进行了修改
4. `feat`：增加新的特征
5. `fix`：修复 `bug`
6. `pref`：提高性能的代码更改
7. `refactor`：既不是修复 `bug` 也不是添加特征的代码重构
8. `style`：不影响代码含义的修改，比如空格、格式化、缺失的分号等
9. `test`：增加确实的测试或者矫正已存在的测试

此外还推荐使用

1. `improvement` : 表示在不添加新功能或修复 `bug` 的情况下改进当前的实现
2. `init` : 项目初始化
3. `chore` : 改变构建流程、或者增加依赖库、工具等
4. `revert` : 回滚到上一个版本

### 作用域

作用域（scope）表示此次提交影响的范围。比如可以取值 api，表明只影响了接口

### 主题

主题（subject）描述是简短的一句话，简单说明此次提交的内容。

## 正文与页尾

正文（body）和页眉（footer）这两部分不是必须的

正文描述为什么修改，做了什么样的修改，以及开发的思路等等

也可以使用 Angular 的规范：

1. 使用命令式，现在时态：“改变”不是“已改变”也不是“改变了”
2. 不要大写首字母
3. 不在末尾添加句号

页脚注释放 Breaking Changes 或 Closed Issues

强制支持在页脚使用 `BREAKING CHANGES` 。也就是说，当有破坏性更新时那就必须在提交的正文或脚注加以展示。一个破坏性变更必须包含大写的文本 BREAKING CHANGE，紧跟冒号和空格。脚注必须只包含 BREAKING CHANG E、外 部 链 接、i s s u e 引 用 和 其 它 元 数 据 信 息 BREAKING CHANGE、外部链接、issue 引用和其它元数据信息 BREAKINGCHANGE、外部链接、issue 引用和其它元数据信息。例如修改了提交的流程，依赖了一些包，可以在正文写上：

下面给出了一个 Commit Message 例子，该例子中包含了 header 和 body 例子：

```plaintext
chore: 引入commitizen

BREANKING CHANGE：需要重新npm install，使用npm run cm代替git commit

```

当然，在平时的提交中，我们也可以只包含 header，比如我们修改了登录页面的某个功能，那么可以这样写 Commit Message

```text
feat(登录）：添加登录接口
```

## 规范

结合[RFC2019](https://www.ietf.org/rfc/rfc2119.txt)，在下面规范中使用关键字来指示需求级别

- `MUST`（必须）
- `MUST NOT`（禁止）
- `REQUIRED`（需要）
- `SHALL`（应当）
- `SHALL NOT`（不应当）
- `SHOULD`（应该）
- `SHOULD NOT`（不应该）
- `RECOMMENDED`（推荐）
- `MAY`（可以）
- `OPTIONAL`（可选）
- 每次提交**必须**添加类型名为前缀。类型名是一个名词，比如 `feat、fix`，后跟冒号和一个空格
- 类型 `feat` **必须**在提交新特征时使用
- 类型 `fix`**必须**用于修复 `bug`的提交
- 类型后面**可以**添加一个可选的作用域。作用域名是一个短语，表示代码库中的一个小节，比如 `fix(parser)`
- 在类型/作用域前缀后面**必须**跟上一个简短描述，关于本次提交的代码修改，比如 `fix: 字符串中包含多个空格时的数组分析问题`
- **可以**在描述后面添加一个长的正文，用于提供额外的上下文信息。正文内容**必须**在短描述后空一行开始
- **可以**在正文后空一行开始页脚，页脚**应该**包含这次代码修改的相关问题，比如 `Fixes #13`
- 不兼容修改 (`breaking change`)**必须**放置在提交的页脚或正文部分的最开始处，**必须**使用大写文本 `BREAKING CHANGE`作为前缀，后跟冒号和一个空格
- 在 `BREAKING CHANGE:`后面**必须**提供一个描述，关于 `API`的修改，比如 `BREAKING CHANGE: 环境变量目前优先于配置文件`
- 页脚**必须**包含不兼容修改、额外链接、问题引用和其他元信息
- 除了 `feat`和 `fix`以外的类型**可以**在提交信息中使用

## 示例

只有页眉和页尾

```plaintext
feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files
```

使用了作用域

```plaintext
feat(lang): added polish language
```

修复了 `bug`

```plaintext
fix: minor typos in code

see the issue for details on the typos fixed

fixes issue #12
```

## 工具

虽然有了规范，但是还是无法保证每个人都能够遵守相应的规范，因此就需要使用一些工具来保证大家都能够提交符合规范的 Commit Message

VS Code ： [git-commit-plugin](https://marketplace.visualstudio.com/items?itemName=redjue.git-commit-plugin) 插件可以实现可视化的提交

IDEA : Git Commit Template 插件
