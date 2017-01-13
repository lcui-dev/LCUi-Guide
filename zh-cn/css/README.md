# CSS

CSS 的英文全称是 Cascading Style Sheets，即：层叠样式表，在 LCUI 中，图形界面元素的样式可以靠 CSS 代码来描述，“样式”指的是元素的视觉效果，包括元素的位置、尺寸、间距、背景色、边框、阴影等，这些样式都存储在样式表中，每个元素都有一张样式表。使用 CSS 能够解决内容与表现分离的问题，如需改变界面效果，只需要简单的修改 CSS 代码，无需修改应用程序的实现代码。

LCUI 的 CSS 解析器与浏览器中的有一定的差异，受限于 LCUI 现有的功能，很多样式属性都不支持，而作者并不打算让 LCUI 成为一个浏览器，毕竟这样做意义并不大，所以，请不要尝试将网上那些能够实现酷炫效果的 CSS 代码放在 LCUI 应用程序上使用。如果想了解目前支持哪些样式属性，可以查阅 `src/gui/css_parser.c`。

以下是目前支持的功能：

- 通用选择器：`* { display: block; }`
- type 选择器：`textview { font-size: 14px; }`
- class 选择器：`.button { padding: 5px 10px; }`
- id 选择器：`#name { background-color: #fff; }`
- 交集选择器：`textview#test.link { color: #f00; }`
- 多元素选择器：`textview, textedit { line-height: 1.42; }`
- 后代元素选择器：`.container button .text { border: 1px solid #000; }`
- 伪类选择器：`button:hover { background-color: #eee; }`
- 选择器权重值计算。样式表的选择器具体性越明确，权重值越高。
- 样式表层叠。当多张表都有定义相同样式时，能够根据它们的权重决定优先使用哪个样式。

不支持的有：

- 继承。这个功能在作者的实际开发中并不是特别需要，常见的继承属性是 font-family、font-size、line-height、color 等，这类属性与文字样式相关，即使不支持继承也能靠其它方法控制文字样式，通常的做法是为 textview 部件设置全局样式，作为正文默认的样式，然后为标题、副标题等内容设置额外的样式类，如：`.title`、`.subtitle` 等。

（待续...）
