# 布局

布局模式，有时简称为布局，是一种基于盒子与其兄弟和祖辈盒子的交互方式来确定盒子的位置和大小的算法。在 LCUI 中参与布局计算的数据包括 display 属性、定位属性、几何属性、盒模型、尺寸规则、布局规则等，这些数据的用途大致如下：

* 在布局开始前，组件的 display 属性、定位属性和几何属性会被用于计算盒模型和尺寸规则。
* 在布局开始时，组件的 display 属性、定位属性和尺寸规则用于选择合适的布局规则。
* 在布局时，布局算法会根据组件的盒模型计算其兄弟和祖辈组件位置和尺寸。

出于学习和开发成本上的考虑，LCUI 的布局引擎以网页浏览器为参考对象，实现了 CSS 布局中常见的几种布局模式，在大多数情况下，同一种布局模式在 LCUI 应用程序和网页浏览器中的效果是一样的，因此，你也可以通过学习 CSS 布局相关文档来加深对布局的理解。

{% hint style="info" %}
本文假定你已经熟悉 CSS 布局技术，如果你对 CSS 布局技术还不熟悉，我们建议你阅读 MDN 上的文档：《[CSS 布局 - 学习 Web 开发 \| MDN](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS_layout)》
{% endhint %}

在介绍布局模式之前，我们先深入了解一下这些数据的概念和实现细节。

### 盒模型

盒模型用于定义组件的布局和渲染参数，它由以下几个部分组成：

* **Content box**: 这个区域是用来显示内容，大小可通过 `width` 和 `height` 属性设置。
* **Padding box**: 包围在内容区域外部的空白区域，大小通过 `padding` 相关属性设置。
* **Border box**: 边框盒包裹内容和内边距，大小通过 `border` 相关属性设置。
* **Canvas box:** 画布区域包裹了边框盒，与外边距区域重叠，它定义了组件在渲染时所使用的画布的大小，组件的阴影参数会影响它的大小，如果组件没有阴影，则它的大小与边框盒相同。
* **Margin box**: 这是最外面的区域，是盒子和其他元素之间的空白区域。大小通过 `margin` 相关属性设置。

如下图所示：

![&#x76D2;&#x6A21;&#x578B;&#x7684;&#x7EC4;&#x6210;](../.gitbook/assets/box-model.png)

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

### Display 属性

 实现页面布局的主要方法是设定`display`属性的值，它允许我们更改默认的显示方式。LCUI 目前支持 `display` 属性的三种值： `block`、`inline-block` 、`flex`，该属性的默认值是 `block`。

### 定位

定位 \(positioning\) 能够让我们把一个组件从它原本在布局中应该在的位置移动到另一个位置。定位 \(positioning\) 并不是一种用来给你做主要界面布局的方式，它更像是让你去管理和微调界面中的一个特殊项的位置。

LCUI 仅支持以下三种定位：

* **静态定位（Static positioning）：**默认定位，表示使用布局引擎计算好的位置。
* **相对定位（Relative positioning）：**相对定位，它允许我们相对于布局引擎计算好的位置来移动组件，这对于微调和精准设计 \(design pinpointing\) 非常有用。
* **绝对定位（Absolute positioning）：**将组件从布局中移出，不占据空间，通过指定组件相对于父组件的偏移来确定位置。该定位适用于精确控制组件位置，例如：让组件停靠在右上角、或是让组件随着鼠标移动。

{% hint style="warning" %}
注意，绝对定位在浏览器中是相对于元素的最近被定位祖先元素 \(nearest positioned ancestor element\)，而在 LCUI 中是直属父组件。
{% endhint %}

### 尺寸规则

尺寸规则影响到组件的布局规则和布局后的实际尺寸，也会影响到子组件的尺寸规则，它在组件的样式计算和布局阶段会被重新计算。以下是这些规则的作用说明：

* **FIXED：**固定。表示尺寸属性可在布局前直接计算出确切的值，除了设置 100px 这类值外，父组件的尺寸规则是固定时也会采用该规则，因为在父组件尺寸已知的情况下，即便子组件的尺寸是百分比值或 auto 也能直接计算出来。
* **FILL：**填充。组件的尺寸将填满父组件内容区域。大多数情况下该规则都会转换为固定，因为根组件的尺寸规则必定是固定。
* **PERCENT：**百分比。表示在计算尺寸属性的值之前必须先计算出父组件尺寸的实际值，然后再按照百分比计算实际值。在父组件尺寸规则不为固定且尺寸属性为 50% 这类值时，会采用百分比规则。
* **FIT\_CONTENT：**适应内容。表示在布局完后使用内容区域的尺寸作为实际值。该规则常用于组件的高度，因为在大多数情况下宽度是固定的，由内容撑开高度。当组件采用绝对定位或者显示方式为内联块（inline-block）时，它的宽高都会采用该规则。

### 布局规则

布局规则在布局前由组件的定位方式、尺寸规则和初始布局规则计算而来，它告诉布局引擎在布局前如何确定最大内容宽度、在布局后如何计算组件实际尺寸。以下是这些规则的作用说明：

* **AUTO**：自动。这是初始规则，由布局引擎根据组件的显示方式和尺寸规则来选择合适的布局规则。
* **MAX\_CONTENT**：最大内容。在布局时尽可能扩大尺寸以呈现更多的内容。当尺寸规则都不为固定时会采用此布局规则。
* **FIXED\_WIDTH：**固定宽度。在布局时将组件内容区域宽度作为最大宽度，排列的每行子组件都不会超出该宽度。当宽度的尺寸规则是固定时会采用该规则。
* **FIXED\_HEIGHT**：固定高度。与固定宽度相似，但它在正常流布局下不会影响子组件的排列位置。
* **FIXED**：固定。当组件的宽高属性的尺寸规则都是固定时采用该规则。

### 最大内容尺寸

最大内容尺寸是组件在空间无限的情况下可以容纳全部内容并避免内容溢出的最小尺寸。它在组件主动布局时重新计算，主要用于在弹性盒子布局中为组件提供初始尺寸。

### 主动与被动布局

因组件自身样式变化而触发的布局是主动布局。在主动布局过程中，如果子组件的尺寸因此发生变化而触发的布局就是被动布局。这两种布局的区别是，主动布局有选择布局规则和更新最大内容尺寸的权力，而被动布局没有，只能基于父组件的布局规则进行布局。

区分布局的主动和被动是为了解决父子组件宽高都不确定时的尺寸计算问题。以下拉菜单为例，下拉菜单的宽高是自适应的，里面的每个菜单项的宽度都是相同的，且正好能容纳全部内容而不换行或溢出，那么为了计算它们的尺寸，我们需要先计算每个菜单项的尺寸，根据它们的尺寸进行简单的布局来得出菜单的内容尺寸，然后再对每个菜单项布局一次，让它们根据菜单的内容尺寸计算出自己的实际尺寸。大致布局流程如下：

* 触发每个菜单项的被动布局，初始布局规则为 `MAX_CONTENT`。
* 利用菜单项的尺寸进行布局，得到菜单的内容尺寸。
* 再次触发每个菜单项的被动布局，初始布局规则为 `FIXED`。
* 布局完成，此时每个菜单项的宽度都等于最大菜单项的宽度，菜单的内容宽度等于菜单项的宽度，菜单的内容高度等于全部菜单项的高度之和。

### 布局流程

在 LCUI 当前的布局引擎设计中，正常流布局和弹性盒子布局的流程是相似的，都需要经历以下几个阶段：

* **开始：**创建布局上下文，初始化布局所需的一些数据。
* **载入：**遍历子组件，根据子组件的盒模型对其进行分组，每组子组件即为一行。载入完后根据每行的宽高来估算内容区域的宽高。
* **应用尺寸：**根据布局规则为组件设置合适的尺寸。
* **布局：**遍历子组件，为子组件的宽高属性计算实际值。这时如果子组件的宽高因此发生变化的话会对子组件进行布局。
* **布局自由组件：**计算绝对定位的子组件的位置和宽高。对于绝对定位的子组件，它们的位置和宽高常常依赖父组件的宽高，需要等到这个阶段再计算实际值。
* **结束：**销毁布局上下文。

### 待办事项

**重新设计布局引擎**

现有的布局引擎是与 LCUI 的组件系统绑定在一起的，而且布局规则的处理流程也比较复杂，为了降低维护成本，简化布局流程，我们应该重新设计布局引擎，使它能够作为一个独立的项目被用于其他项目。可供参考的同类案例有：[https://github.com/facebook/yoga](https://github.com/facebook/yoga)



