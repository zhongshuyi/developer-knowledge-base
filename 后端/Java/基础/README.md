---
order: 0

author: 钟舒艺
---
# Java 基础

## 基本语法

编写 Java 程序前应注意以下几点：

- **大小写敏感**：Java 是大小写敏感的，这就意味着标识符`Hello`与`hello`是不同的。
- **类名**：对于所有的类来说，类名的首字母应该大写。如果类名由若干单词组成，那么每个单词的首字母应该大写，例如`**MyFirstJavaClass**`。
- **方法名**：所有的方法名都应该以小写字母开头。如果方法名含有若干单词，则后面的每个单词首字母大写，例如`**myFirstJavaMethod**`。
- **源文件名**：源文件名必须和文件内名字唯一且公开 (由 public 关键词修饰) 的类名相同。当保存文件的时候，你应该使用该类名作为文件名保存（切记 Java 是大小写敏感的），文件名的后缀为`.java`。（如果文件名和类名不相同则会导致编译错误）。
- **主方法入口**：所有的 Java 程序由`public static void main(String[] args)`方法开始执行。

## Java 关键字

- 访问控制
  - private
  - protected
  - public
- 类、方法和变量修饰符
  - abstract
  - class
  - extends
  - final
  - implements
  - interface
  - native
  - new
  - static
  - strictfp
  - synchronized
  - transient
  - volatile
- 程序控制语句
  - break
  - case
  - continue
  - default
  - do
  - else
  - for
  - if
  - instanceof
  - return
  - switch
  - while
- 错误处理
  - assert
  - catch
  - finally
  - throw
  - throws
  - try
- 包相关
  - import
  - package
- 基本类型
  - boolean
  - byte
  - char
  - double
  - float
  - int
  - long
  - short
  - null
- 变量引用
  - super
  - this
  - void
- 保留关键字
  - goto
  - const
