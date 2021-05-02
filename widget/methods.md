---
description: 介绍组件的常用方法。
---

# 方法

### 组件树的操作方法

* `Widget_Append()`：给组件追加子组件。
* `Widget_Prepend()`：给组件前置插入子组件。
* `Widget_Unlink()`：将组件从父组件的子组件列表中移除。
* `Widget_Unwrap()`：将组件内的子组件提取出来，插入到该组件所处的位置，然后销毁该组件。
* `Widget_GetPrev()`：获取上个组件。
* `Widget_GetNext()`：获取下个组件。
* `Widget_GetChild()`：获取指定序号的子组件。
* `Widget_PrintTree()`：打印整颗组件树。
* `Widget_Each()：`遍历组件树内的每个组件，该遍历方法采用非递归的深度优先算法实现。
* `Widget_At()`：获取指定坐标命中的子级组件。



