---
layout: post
title: Ridder和IDEA常用快捷键
categories: [IDE,ridder, IDEA]
description: Ridder和IDEA常用快捷键，提供快速查看
keywords: ridder, IDEA,快捷键
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

在使用IDE编码的过程，如果能记住常用快捷键，工作效率会有很大的提升。同时要想记住这些快捷键也比较困难，可以通过分步的方式来实现肌肉记忆，如：每个月记住几个，需要的时候直接使用，忘记的情况查询后，再使用。

### 常用快捷键

快捷键|说明
-|-
Ctrl+Shift+Enter|代码补全后，自动在代码末尾添加分号结束符(常用)
Ctrl + Alt + T|自动生成具有环绕性质的代码，比如：if..else,try..catch, for, synchronized 等等，使用前要先选择好需要环绕的代码块。（常用）
Ctrl + Alt + O|去除没有实际用到的包，这在 java 类中特别有用。（常用）CTRL+ALT+V自动生成变量
Ctrl + NumPad(+/-)|展开或收缩代码段。 （常用）
Ctrl + Shift + NumPad(+)|展开所有代码段。（常用）
Ctrl + Shift + NumPad(-)|收缩所有代码段。（常用）
Ctrl + o|查看本类的继承或者实现的方法
Ctrl + shift + F12|编辑器全屏）
Ctrl + ]|跳转到{（常用）
Ctrl + [|跳转到}（常用）

### 1、编辑【Editing】

快捷键|说明
-|-
Ctrl + Shift + Space|在列出的可选项中只显示出你所输入的关键字最相关的信息。（常用）
Ctrl + Shift + Enter|代码补全后，自动在代码末尾添加分号结束符(常用)
Ctrl + mouse|跳进到某个类或者方法源代码中进行查看。（常用）
Alt + Inser|自动生成某个类的 Getters, Setters, Constructors, hashCode/equals, toString 等代码。（常用）
Ctrl + Alt + T|自动生成具有环绕性质的代码，比如：if..else,try..catch, for, synchronized 等等，使用前要先选择好需要环绕的代码块。（常用）
Ctrl + /|对单行代码，添加或删除注释。分为两种情况：如果只是光标停留在某行，那么连续使用该快捷键，会不断注释掉下一行的代码；如果选定了某行代码（选定了某行代码一部分也算这种情况），那么连续使用该快捷键，会在添加或删除该行注释之间来回切换。（常用）
Ctrl + Shift + /|对代码块，添加或删除注释。它与Ctrl + / 的区别是，它只会在代码块的开头与结尾添加注释符号！（常用）
Ctrl + Alt + L|格式化代码 （常用）
Ctrl + Alt + O|去除没有实际用到的包，这在 java 类中特别有用。（常用）
Tab / Shift + Tab|缩进或者不缩进一次所选择的代码段。（常用）
Ctrl + X 或 Shift Delete|剪切当前代码。 （常用）
Ctrl + C 或 Ctrl + Insert|拷贝当前代码。 （常用）
Ctrl + V 或 Shift + Insert|粘贴之前剪切或拷贝的代码。（常用）
Ctrl + Shift + V|从之前的剪切或拷贝的代码历史记录中，选择现在需要粘贴的内容。（常用）
Ctrl + D|复制当前选中的代码。（常用）
Ctrl + Y|删除当前光标所在的代码行。（常用）
Shift + Enter|当前代码行与下一行代码之间插入一个空行，原来光标现在处于新加的空行上。（常用）
Ctrl + Alt+ Enter|当前代码行与上一行代码之间插入一个空行，原来光标现在处于新加的空行上。（常用）
Ctrl + Shift + U|所选择的内容进行大小写转换。。（常用）
Ctrl + NumPad(+/-)|展开或收缩代码段。 （常用）
Ctrl + Shift + NumPad(+)|展开所有代码段。（常用）
Ctrl + Shift + NumPad(-)|收缩所有代码段。（常用）
Ctrl + F4|关闭当前标签页。（常用）
Shift + F6|修改名字。（常用）
Ctrl + Q|显示文本提示（光标放在变量上）
Ctrl + o|查看本类的继承或者实现的方法


### 2. 查找或替换【Search/Replace】

快捷键|说明
-|-
Ctrl + F|在当前标签页中进行查找，还支持正则表达式哦。（常用）
Ctrl + R|在当前标签页中进行替换操作。（常用）
Ctrl + Shift + F|通过路径查找。（常用）
Ctrl + Shift + R|通过路径替换。（常用）
Ctrl + Alt + F7|打开使用情况列表。 （常用）

### 3 导航【Navigation】

快捷键|说明
-|-
Double Shifft|查询（常用）
Ctrl Shfit X(show in explorer)|打开代码所在硬盘路径(需要自己设置)
Ctrl + G|跳转至某一行代码。（常用）
Ctrl + B 或 Ctrl + 鼠标左键|如果是类，那么会跳转到当前光标所在的类定义或者接口；如果是变量，会打开一个变量被引用的列表。（常用）
Ctrl + F12|打开类的结构列表。（常用）
Ctrl + H|打开类的继承关系列表。（常用）
Ctrl + Alt + H|打开所有类的方法列表，这些方法都调用了当前光标所处的某个类方法。（常用）

### 4、调试

快捷键|说明
-|-
双击Ctrl|运行所有
Ctrl + Alt + F5|附加到进程
Ctrl + F2|停止
Ctrl + Shift + F2|停止后台进程
	  Ctrl + F8|切换行断点
			F8|跨过调用
	  Shift + F8|跳出调用
Alt + Shift + F8|强制跨过调用
	   Alt + F8|评估表达式
Ctrl + Alt + F8|快速评估表达式
Ctrl + Shift + F8|查看/编辑断点
Ctrl + Alt + Shift + F8|切换临时行断点
			F7|进入调用
	  Shift + F7|智能进入调用
Alt + Shift + F7|强制进入调用
		     F9|运行至下一断点
	   Alt + F9|运行至光标处
	  Shift + F9|启动当前配置项目调试
Alt + Shift + F9|打开调试窗口
Ctrl + Alt + F9|强制运行至光标处
Ctrl + F9|构建当前项目
Ctrl + shift + F9|编译当前类
Alt + F10|显示执行点	  
Shift + F10|启动当前配置项目
Alt + Shift + F10|打开运行窗口
Alt + 4|显示运行窗口
Alt + 5|显示调试窗口
Alt + 8|显示服务窗口

