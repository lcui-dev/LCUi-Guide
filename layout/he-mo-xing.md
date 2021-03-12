# 盒模型

盒模型用于定义组件的布局和渲染参数，它由以下几个区域组成：

* **Content box**: 这个区域是用来显示内容，大小可通过 `width` 和 `height` 属性设置。
* **Padding box**: 包围在内容区域外部的空白区域，大小通过 `padding` 相关属性设置。
* **Border box**: 边框盒包裹内容和内边距，大小通过 `border` 相关属性设置。
* **Canvas box:** 画布区域包裹了边框盒，与外边距区域重叠，它定义了组件在渲染时所使用的画布的大小，组件的阴影参数会影响它的大小，如果组件没有阴影，则它的大小与边框盒相同。
* **Margin box**: 这是最外面的区域，是盒子和其他元素之间的空白区域。大小通过 `margin` 相关属性设置。

你可以通过组件对象中的 box 属性来访问这些区域，例如：

```c
printf(
    "content_box: (%f, %f, %f, %f)\n",
    w->box->content.x, w->box->content.y,
    w->box->content.width, w->box->content.height,
);
printf(
    "border_box: (%f, %f, %f, %f)\n",
    w->box->border.x, w->box->border.y,
    w->box->border.width, w->box->border.height,
);

```



