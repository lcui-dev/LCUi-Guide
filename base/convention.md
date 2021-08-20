---
description: 介绍 LCUI 的命名风格和推荐的 LCUI 应用项目的代码组织方式。
---

# 约定

### 命名约定

在正式开始使用 LCUI 前，我们先了解一下 LCUI 的命名约定，这将有助于记忆和查找你需要的 API。

#### 数据类型

大部分公共的数据类型都采用驼峰式命名法（Camel-Case），并带有 `LCUI_` 前缀，像链表（LinkedList）、红黑树（RBTree） 和字典（Dict）这类基础数据类型，由于名称长度和可替代性，未加上 `LCUI_` 前缀。

对于常以指针形态引用的数据类型，它的定义会是这样：

```c
typedef struct LCUI_FooRec_*  LCUI_Foo;

typedef struct  LCUI_FooRec_
{
  /* fields for the 'foo' class */
  ...

} LCUI_FooRec;
```

这种写法参考自 FreeType，如需了解更多，可参考它的设计文档：《[FreeType Design](https://www.freetype.org/freetype2/docs/design/design-3.html#section-1)》

#### 函数

由于 LCUI 在开发初期并未确定最佳的命名风格，受到维护人员的不稳定的编程习惯以及其它开源项目的影响，每当引入新功能的代码时候都有可能采用其它命名风格，因此，你会发现 LCUI 的源码中存在多种命名风格：

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

* **驼峰风格：** 函数以操作对象的类型名开头，并以下划线分隔对象类型名与方法名。如果函数操作的对象是采用单例模式的全局共享对象，通常还会加上 LCUI 前缀。

  ```c
  Widget_Append();
  Dict_FetchValue();
  LinkedList_Append();
  LCUIFont_RenderBitmap();
  LCUIFont_LoadFile();
  ```

* **全小写风格：**与标准库函数的命名风格类似。

  ```c
  strhash();
  strsplit();
  ```

### 推荐的目录结构

* **app/:** 应用的工作目录，用于存放运行时所需的资源文件。
* **build/:** 构建工具的工作目录，包含一些配置文件和构建产物。
* **config/:** 配置文件目录。
* **include/:** 头文件目录。
* **src/:** 源码目录。
  * **lib/:** 基础库目录，包含应用的核心功能源码。
  * **ui/:** 用户界面目录。
    * **components/:** 组件目录，包含能够在用户界面中复用的基础组件。
    * **effects/:** 效果目录，包含一些动效、手势、拖拽等界面交互效果的实现源码。
    * **views/:** 视图目录，包含能够完成特定功能的完整界面。
    * **app.c:** 应用的入口。
    * **ui.c:** 用户界面的入口。
* **vendor/:** 第三方库目录，包含第三方库的头文件和库文件。

### 待办事项

**重新设计编码规范**

参考主流的 C 开源项目的编码风格，为 LCUI 设计最佳编码规范。



