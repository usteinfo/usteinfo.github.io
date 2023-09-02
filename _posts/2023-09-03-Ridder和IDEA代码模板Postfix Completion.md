---
layout: post
title: Ridder和IDEA代码模板(Postfix)
categories: [Ridder, IDEA]
description: Ridder和IDEA代码模板
keywords: Ridder, IDEA
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---
Postfix Completion (下称Postfix) 是一种通过 . + 模板Key 来对当前已经输出的表达式，添加和应用预设代码模板的编码增强能力。

其核心要解决的问题是，将编码过程中一些通用的代码结构范式进行抽象和沉淀，并能在同类型的场景下，通过 . + 模板Key 的方式进行唤醒和复用。

1. psvm : 可生成 main 方法
1. sout : System.out.println() 快捷输出

    ```
    soutp => System.out.println("方法形参名 = " + 形参名);
    soutv => System.out.println("变量名 = " + 变量);
    soutm => System.out.println("当前类名.当前方法");
    “abc”.sout => System.out.println("abc");
    ```

1. fori : 可生成 for 循环

    ```
    iter：可生成增强 for 循环
    itar：可生成普通 for 循环
    ```

1. list.for : 可生成集合 list 的 for 循环

    List<String> list = new ArrayList<String>();
    
    输入: list.for 即可输出 for(String s:list){ }
    又如：list.fori 或 list.forr



1. ifn：可生成 if(xxx = null)
    类似的： inn：可生成 if(xxx != null) 或 xxx.nn 或 xxx.null


1. prsf：可生成 private static final

    类似的：
    psf：可生成 public static final
    psfi：可生成 public static final int
    psfs：可生成 public static final String
    
    模式|说明
    -|-
    var|快速定义一个局部变量，自带IDE的类型推断
    notnull|快速进行NPE的判空保护：
    nn|同notnull，是它的简写，推荐用这个，更加便捷：
    try catch|快速对当前语句添加try catch异常捕获，同时IDE还会对catch中的Exception自动做类型推断：
    cast|快速实现类型强转，不需要反复使用()包裹和光标切换；配合instanceof使用时还能自动实现cast类型的推断：
    if|快速实现if判断的代码范式：
    throw|快速实现抛异常：
    for|快速实现集合或数组的迭代：
    fori|快速实现集合或数组的带索引值迭代；同时对整型数字也支持：
    sout/soutv|快速实现（不带参数/带参数）的打印功能：
    return|快速实现方法中的值返回逻辑
    format|快速实现字符串格式化：
    if/else/|
    null|
    switch|
    while|
    fori/forr|
    new/field|
    return|
