---
description: XML 的写法和常用元素的介绍。
---

# XML

XML 是一种可扩展的标记语言，LCUI 之所以采用 XML 而不是 HTML，主要有以下理由：

* XML 比 HTML 更简单。
* LCUI 不是浏览器，不打算实现 HTML 中的所有标签的功能，因为这是既浪费时间又没有意义的事情。
* 使用 HTML 会让用户以开发网页的思维方式去编写 LCUI 应用，然后误以为 `<div>`、`<img>`、`<table>`、`<video>`、`<audio>`、`<table>` 等标签在 LCUI 中会有效果。

在开始前，我们假定你已经熟悉 HTML 或 XML 这类标记语言的语法，本文将跳过基本语法和相关术语的介绍，直接讲解 LCUI 的 XML 文档写法和常用元素的用法，如需了解更多请查阅相关文档。

在前面的章节中我们已经了解到 LCUI 的 XML 文档内容格式和预定义元素的用法：

```markup
<?xml version="1.0" encoding="UTF-8" ?>
<lcui-app>
  <resource type="text/css" src="main.css"/>
  <ui>
    <textview id="text" type="textview">
      Hello, World!
    </textview>
  </ui>
</lcui-app>
```

第一行声明文档的类型，第二行的`<lcui-app>` 声明了它包裹的内容适用于 LCUI 应用，第三行的 `<ui>` 包裹了整个用户界面的结构及其所有组件的信息。

### 常用元素

#### \<resource>&#x20;

声明资源信息，可用于加载资源文件。

| 属性   | 说明                                                                                                                                                                                                                                                                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| type | <p>资源类型，可选值有：</p><p></p><ul><li><code>application/font-ttf</code> ：作为 ttf 字体文件加载</li><li><code>text/css</code> ：作为 css 文本文件加载</li><li> <code>text/xml</code> ：作为 xml 文档加载</li></ul><p>当值为 <code>text/xml</code> 时，效果相当于将目标 xml 文档的内容插入到 <code>&#x3C;resource></code> 所在位置。</p> |
| src  | 资源文件的来源路径                                                                                                                                                                                                                                                                      |

#### \<widget>&#x20;

声明组件，仅限在 `<ui>` 内使用。

| 属性    | 说明                      |
| ----- | ----------------------- |
| type  | 组件的类型名称，需要是组件原型库中已注册的名称 |
| id    | 唯一标识符                   |
| class | 类名称                     |

在 `<ui>` 中，如果元素的标签名不是预定义的，则会视为 `<widget>` 元素，因此，你可以使用组件类型名作为标签名，例如以下两行元素是等效的：

```markup
<widget type="textview" class="text">hello</widget>
<textview class="text">hello</textview>
```

### API

LCUI 提供的 XML 文档相关的函数有两个：

```c
LCUI_Widget LCUIBuilder_LoadString(const char *str, int size);

LCUI_Widget LCUIBuilder_LoadFile(const char *filepath);
```

从函数原型可以知道，这两个函数分别用于从字符串和文件中加载 XML 文档内容，它们的返回值都是一个根组件，这个根组件只是充当包含了所有组件的容器，真正有用的是它里面组件，因此我们需要使用 `Widget_Unwrap()` 函数展开该容器组件，将它里面的组件暴露到外面。
