---
layout: post
title:  "Dalvik 虚拟机 - Android逆向"
categories: Android
tags: Android 非虫 逆向 Dalvik
excerpt: 点进来在告诉你~
---

### Dalvik 虚拟机 - Android逆向


&emsp;&emsp;Android 使用 Java语言开发应用程序，但是 Andorid 程序不是运行在标准的 Java 的虚拟机上的。

---
#### 1、Dalvik 虚拟机的特点

Dalvik 虚拟机是 Android 平台的核心组件
- 体积小，占用内存空间少。
- 专有的 DEX (Dalvik Executable) 可执行文件格式，体积小，执行速度快。
- 常量池才用 32 位索引值，对类方法名、字段名、常量的寻址速度快。
- 基于寄存器架构，同时拥有一套完整的指令系统。
- 提供了对象生命周期管理、堆栈管理、线程管理、安全和异常管理及垃圾回收等重要功能
- **所有的 Android 程序都运行在 Android 系统进程中，每个进程都与一个 Dalvik 虚拟机实例对应。**

#### 2、Java 转 Dalvik

Java class 转换 Dalvik dex 是通过 <Android SDK 安装目录>\build-tools\xx.x.x 目录下 的 dx 工具实现的。dx 能够重新排列 Java 类文件，消除在类文件中出现的所有冗余信息。

#### 3、Dalvik 指令格式
一段 Dalvik 汇编代码由一系列 Dalvik 指令组成，指令语法由指令的位描述和指令格式标识决定。位描述约定如下。


- 每 16 位的字用空格分开
- 每个字母表示 4 位，每个字母按顺序从高字节到低字节排列，每 4 位之间可能使用竖线 "\|" 将不同的内容分开。
- 顺序才用英文大写字母 A ~ Z 表示 4 位的操作码。op 表示 8 位的操作码。
- ∅ 表示字段所有位的值为 0。

&emsp;&emsp;以指令格式 “A|G|op BBBB F|E|D|C” 为例进行分析。两个空格将这条指令分成了大小均为 16 位的三部分，因此，这条指令由三个 16 位的字组成。第 1 个 16 位部分是 “A|G|op”，其高 8 位由 “A” 和 “G” 组成，低字节则由操作码 “op” 组成。第 2 个 16 位部分由 “BBBB” 组成，标识一个 16 位的偏移量。第 3 个 16 位部分由 “F” “E” “D” “C” 四个 4 字节组成，它们分别表示寄存器的参数。
&emsp;&emsp;单独使用位标识是无法确定一条指令的，必须通过指令格式标识来指令的格式编码，约定如下。
- 指令格式标识大都由三个字符组成。其中，前两个是数字，最后一个是字母。
- 第 1 个数字表示指令是由多少个 16 位的字组成的。
- 第 2 个数字标识指令最多使用的寄存器的格式。特殊标记 r 用于标识所使用的寄存器的范围。
- 第 3 个字母为类型码，表示指令所使用的额外数据的类型。

| 助记符 | 位大小 | 说明 |
|:-:|----|----|
| b | 8 | 8位有符号立即数|
| c | 16, 32 | 常量池索引 |
| f | 16 | 接口常量（仅对静态链接格式有效）|
| h | 16 | 有符号立即数（ 32 位或 64 位）|
| i | 32 | 立即数，有符号整数或 32 位浮点数|
| l | 64 | 立即数，有符号整数或 64 位双精度浮点数|
| m | 16 | 方法常量（仅对静态链接格式有效）|
| n | 4 | 4 位立即数 |
| s | 16 | 短整型立即数 |
| t | 8, 16, 32 | 跳转，分支|
| x | 0 | 无额外数据 |

&emsp;&emsp;一种特殊的情况是指令的末尾多出一个字母。如果字母是 s ，表示指令才用静态链接；如果是字母 i ，表示指令应该被内联处理。
&emsp;&emsp;以指令格式标识 “22x” 为例，第 1 个数字 2 表示指令由两个 16 位字组成，第 2 个数字 2 表示指令使用两个寄存器，字母 x 表示没有使用额外的数据。
&emsp;&emsp;Dalvik 指令对语法进行了一些说明，约定如下。
- 每条指令都是从操作码开始的，后面紧跟参数。参数的个数不定，参数之间用逗号分隔。
- 每条指令的参数都是从指令的第一部分开始的。op 位低 8 位。高 8 位可以是一个 8 位的参数，也可以是两个 4 位的参数，还可以为空。如果指令超过 16 位，则将之后的部分依次作为参数。
- 如果参数采用 “vX” 的形式表示，说明它是一个寄存器，例如 v0、v1 等。这里用 “v” 而不用 “r” 的目的是避免与基于该虚拟机架构本身的寄存器产生命名冲突（例如，ARM 架构的寄存器名称以 “r” 开头）
- 如果参数采用 “#+X” 的形式表示，说明它是一个常量数字。
- 如果参数采用 “+X”  的形式表示，说明它是一个相对指令的地址偏移量。
- 如果参数采用 “kind@X” 的形式表示，说明它是一个常量池索引值。其中，“kind” 表示常量池的类型，可以是 string （字符串常量池索引）、 type （类型常量池索引）、 field （字段常量池索引）或者 meth （方法常量池索引）。

&emsp;&emsp;以指令 op vAA, string@BBBB 为例，该指令使用了一个寄存器参数 vAA ，附加了一个字符串常量池索引值 string@BBBB。其实，这种指令格式代表 const-string 指令。
&emsp;&emsp;Android 4.0 源码目录 Dalvik/docs 下的文档 instruction-formats.html 列举了 Dalvik 指令的所有格式，可以通过阅读该文档了解更加完整的 Dalvik 指令信息。

#### 4、寄存器命名法
v 命名法与 p 命名法，它们是 Dalvik 字节码的两种寄存器表示方法。

<br>

##### v 命名法：
假设一个函数使用了 M 个寄存器，且该函数有 N 个参数。根据 Dalvik 虚拟机对参数传递方式的规定，参数使用最后的 N 个寄存器，局部变量使用从 v0 开始的 M - N 个寄存器。在 《 Android 软件安全权威指南》3.1.2 节的实例中，foo() 函数使用了五个寄存器和两个显式的整数参数。该函数是 Hello 类的非静态方法，它在被调用时会传入一个隐式的 Hello 对象引用，因此，实际上传入了三个参数。根据传参规则，局部变量将使用前两个寄存器，参数将使用后三个寄存器。

采用以小写字母 v 开头的形式表示函数所使用的局部变量与参数，寄存器命名从 v0 开始递增。

##### p 命名法：
p 命名法对函数的局部变量寄存器命名没有影响，它的命名规则是：函数中引入的参数命名从 p0 开始递增。对于 foo() 函数，p 命名法会使用 v0、v1、p0、p1、p2 五个寄存器，v0 与 v1 同样用于表示函数的局部变量寄存器，p0 用于表示传入的 Hello 对象引用，p1 与 p2 分别用于表示两个传入的整形参数。

| v 命名法 | p 命名法 | 寄存器含义 |
|----|----|----|
| v0 | v0 | 第 1 个局部变量寄存器 |
| v1 | v1 | 第 2 个局部变量寄存器 |
| ······ | ······ | 中间的局部变量寄存器递增且名称相同 |
| v *M-N* | p0 | 第一个参数寄存器 |
| ······ | ······ | 中间的参数寄存器递增 |
| v *M*-1 | p *N*-1 | 第 N 个参数寄存器|

#### Dalvik 字节码

1.类型
Dalvik 字节码只有两种类型，分别是基本类型和引用类型。Dalvik 使用这两种类型来表示 Java 语言的全部类型。除了对象和数组属于引用对象，其它的 Java 类型都属于基本类型。baksmali 严格遵守 DEX 文件格式类型描述符的定义

##### Dalvik 字节码类型描述

| 语法 | 含义 |
|----|----|
| v | void，只用于返回值类型 |
| Z | boolean |
| B | byte |
| S | short |
| C | char |
| I | int |
| J | long |
| F | float |
| D | double |
| L | Java 类类型|
| [ | 数组类型 |