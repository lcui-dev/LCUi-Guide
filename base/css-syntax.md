---
description: CSS 的介绍以及 LCUI 对 CSS 特性的支持情况。
---

# CSS

 **层叠样式表** \(Cascading Style Sheets，缩写为 **CSS**），是一种用来为结构化文档（如 HTML 文档或 XML 应用）添加样式（字体、间距和颜色等）的语言。CSS 使我们可以将 UI 中的显示信息隔离出来，存放在用 CSS 语言写的文件中，从而简化 UI 的代码，使其只包含结构和内容的描述代码。

出于开发成本的考虑，LCUI 内置的 CSS 解析器仅支持一些常用的 CSS 特性，你可以从以下列表来了解 CSS 特性支持情况，已勾选的是表示已支持（至少支持基本功能），未列出的属性则默认不支持。

* at rules
  * [x] `@font-face`
  * [ ] `@keyframes`
  * [ ] `@media`
* keywords
  * [ ] `!important`
* selectors
  * [x] `*`
  * [x] `type`
  * [x] `#id`
  * [x] `.class`
  * [x] `:hover`
  * [x] `:focus`
  * [x] `:active`
  * [x] `:first-child`
  * [x] `:last-child`
  * [ ] `[attr="value"]`
  * [ ] `:not()`
  * [ ] `:nth-child()`
  * [ ] `parent > child`
  * [ ] `a ~ b`
  * [ ] `::after`
  * [ ] `::before`
  * [ ] ...
* units
  * [x] px
  * [x] dp
  * [x] sp
  * [x] pt
  * [x] %
  * [ ] rem
  * [ ] vh
  * [ ] vw
* properties
  * [x] top, right, bottom, left
  * [x] width, height
  * [x] visibility
  * [x] display
    * [x] none
    * [x] inline-block
    * [x] block
    * [x] flex
    * [ ] inline-flex
    * [ ] inline
    * [ ] grid
    * [ ] table
    * [ ] table-cell
    * [ ] table-row
    * [ ] table-column
    * [ ] ...
  * [x] position
    * [x] static
    * [x] relative
    * [x] absolute
    * [ ] fixed
  * [x] box-sizing
    * [x] border-box
    * [x] content-box
  * [x] border
  * [x] border-radius
  * [x] background-color
  * [x] background-image
  * [x] background-position
  * [x] background-cover
  * [ ] background
  * [x] pointer-events
  * [x] font-face
  * [x] font-family
  * [x] font-size
  * [x] font-style
  * [x] flex
  * [x] flex-shrink
  * [x] flex-grow
  * [x] flex-basis
  * [x] flex-wrap
  * [x] flex-direction
  * [x] justify-content
    * [x] flex-start
    * [x] center
    * [x] flex-end
  * [x] align-items
    * [x] flex-start
    * [x] center
    * [x] flex-end
    * [x] stretch
  * [ ] float
  * [ ] transition
  * [ ] transform
  * [ ] ...



