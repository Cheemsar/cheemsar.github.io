---
layout: wiki
title: Android Studio 常用快捷键
cate1: Android
cate2: 提高生产力
description: 提升Android开发效率，IDE使用心得
keywords: Android
type:
link:
mermaid: false
sequence: false
flow: false
mathjax: false
mindmap: false
mindmap2: false
---

# 代码折叠

在阅读一些开源项目时候，你会发现一个代码文件动辄上千行，如果你想要找到一个方法，受罪的就是你的滚轮键了。 

如果只是 `.java` 或者 `.kt` 代码文件，我们可以在 `Structed Dialog` 也能看到文档结构，那如果我们需要查看 上千行的 `layout` 布局结构呢？

通过代码折叠可以让你快速的折叠代码并更快的找到要定位的方法。

## 展开选中项
1. 展开单个  `Ctrl + Number +`
2. 递归展开所有后代 `Ctrl + Alt + Number +`

3. 折叠单个  `Ctrl + Number -`
4. 递归折叠所有后代 `Ctrl + Alt + Number -`

## 展开文档折叠

1. 展开文档所有折叠 `Ctrl + Shift + Number +`
2. 折叠文档所有折叠 `Ctrl + Shift + Number -`

## 展开选中项至某级

1. 展开到 1 级 `Ctrl + Number *` then `Number 1`
2. 展开到 2 级 `Ctrl + Number *` then `Number 2`
3. 展开到 3 级 `Ctrl + Number *` then `Number 3`
4. 展开到 4 级 `Ctrl + Number *` then `Number 3`

## 展开文档折叠至某级

1. 展开到 1 级 `Ctrl + Shift + Number *` then `Number 1`
2. 展开到 2 级 `Ctrl + Shift + Number *` then `Number 2`
3. 展开到 3 级 `Ctrl + Shift + Number *` then `Number 3`
4. 展开到 4 级 `Ctrl + Shift + Number *` then `Number 3`

PS: 上面的 `Number *` 需要小键盘支持，没有小键盘的同学需要手动修改下快捷键，我个人配置了 `Ctrl + Alt + 1` 。

## 展开所有注释
- 展开所有文档注释  `Code->Folding->Expand Doc Comments`
- 折叠所有文档注释  `Code->Folding->Collapse Doc Comments`

PS: 文档折叠操作默认没有快捷键，有需要的小伙伴可手动在 `Setting` 中进行配置。
