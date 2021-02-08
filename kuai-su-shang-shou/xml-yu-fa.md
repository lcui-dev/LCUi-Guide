---
description: XML 的写法和常用标签的介绍。
---

# XML

XML 是一种可扩展的标记语言，LCUI 之所以采用 XML 而不是 HTML，主要有以下理由：

* XML 比 HTML 更简单。
* LCUI 不是浏览器，不打算实现 HTML 中的所有标签的功能，因为这是既浪费时间又没有意义的事情。
* 使用 HTML 的话会让用户以开发网页的思维方式去编写 LCUI 应用，然后误以为 `<div>`、`<img>`、`<table>`、`<video>`、`<audio>`、`<table>` 等标签在 LCUI 中会有效果。

在开始前，我们假定你已经熟悉 HTML 或 XML 这类标记语言的语法，本文将跳过基本语法和相关术语的介绍，直接讲解 LCUI 的 XML 文档写法和常用元素的用法，如需了解请查阅相关文档。

LCUI 的 XML 文档内容都是这样的结构：

```markup
<?xml version="1.0" encoding="UTF-8" ?>
<lcui-app>
  <ui>
    ...
  </ui>
</lcui-app>
```

第一行声明文档的类型，第二行的`<lcui-app>` 声明了它包裹的内容适用于 LCUI 应用，第三行的 `<ui>` 包裹了整个用户界面的结构及其所有组件的信息。

### &lt;resource&gt; 

声明资源信息，可用于加载资源文件。

* **type:** 资源类型，可选值有：
  * `application/font-ttf` ：作为 ttf 字体文件加载。
  * `text/css` ：作为 css 文本文件加载。
  *  `text/xml` ：作为 xml 文档加载，效果相当于将目标 xml 文档的内容插入到 `<resource>` 所在位置。
* **src:** 资源文件的来源路径。

### &lt;widget&gt; 

描述组件，仅限在 `<ui>` 内使用。

* **type:** 组件的类型名称，需要是组件原型库中已注册的名称。
* **id:** 唯一标识符。
* **class:** 类名称。

在 `<ui>` 中，所有元素都会视为 `<widget>` ，因此，你可以使用组件类型名作为标签名，例如以下两行元素是等效的：

```markup
<widget type="textview" class="text">hello</widget>
<textview class="text">hello</textview>
```





