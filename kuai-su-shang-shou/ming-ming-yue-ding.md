---
description: 介绍 LCUI 的命名风格和推荐的 LCUI 应用项目的代码组织方式。
---

# 约定

### 命名约定

#### 数据类型

大部分公共的数据类型都采用驼峰式命名法（Camel-Case），并带有 `LCUI_` 前缀，像链表（LinkedList）、红黑树（RBTree） 和字典（Dict）这类基础数据类型，由于名称长度和可替代性，未加上 `LCUI_` 前缀。

#### 函数

由于 LCUI 在开发初期并未确定最佳的命名风格，而在开发的过程中我们也有尝试过其它命名风格，因此，你会发现 LCUI 源码中有多种命名风格：

* **面向对象 + 驼峰风格：** 使用与类型名同名的函数作为构造函数，该对象的所有操作函数都以类型名开头，并以下划线分隔，对象的析构函数名称都是 Delete。

  ```c
  LCUI_StyleSheet sheet1 = StyleSheet();
  LCUI_StyleSheet sheet2 = StyleSheet();
  StyleSheet_Merge(sheet1, sheet2);
  StyleSheet_Delete(sheet2);
  ```

* `LCUI_` **前缀 + 驼峰风格：** 依赖 LCUI 核心功能的公共函数大都采用此风格，有的函数为避免命名冲突也会加上 `LCUI_` 前缀。

  ```c
  LCUI_Init():
  LCUI_SetTimeout();
  LCUI_Quit();
  LCUI_Main();
  LCUI_EncodeString();
  ```

* **全小写风格：**与标准库函数的命名风格类似。

  ```c
  strhash();
  strsplit();

  ```

### 推荐的目录结构



