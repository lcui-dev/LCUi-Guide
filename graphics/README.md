# 图形

本篇教程从一些基础开始，描述了如何使用 LCUI 提供的图形 API 来绘制 2D 图形。教程中提供的例子，会让你明白可以用图形 API 做什么，也会提供一些代码片段来帮助你开始构建自己的内容。

### 图形对象 <a id="hua-bu"></a>

图形对象是一个记录了位图的像素数据和宽高等信息的对象，在 LCUI 中它被定义为 `LCUI_Graph` 结构体类型，与之相关的函数都以 `Graph_` 前缀命名。为了书写方便，在下文中我们将用“图像”代替图形对象。

图像的主要属性有：

* `width` : 宽度，单位为像素。
* `height` : 高度，单位为像素。
* `opacity`: 透明度，用于在与其它图像混合时控制混合比例，取值范围 0.0 ~ 1.0，值为 0.0 时完全透明，值为 1.0 时不透明。
* `color_type`: 色彩类型，描述了图像中的像素数据的理解方式，目前该属性的值只支持 `LCUI_COLOR_TYPE_ARGB` 和 `LCUI_COLOR_TYPE_RGB`。
* `quote` : 其它图像的引用信息，调用 `Graph_Quote()` 函数可为指定图像中的区域创建一个引用，操作该图像时实质上是在操作它引用的图像，且读写操作都被限定在引用区域内。
* `bytes`: 像素数据缓存，允许以 1 字节为单位进行读写。
* `argb` : 像素数据缓存，允许以 4 字节为单位进行读写。你可以将它理解为 `LCUI_ARGB` 类型的数组。
* `bytes_per_pixel` : 每个像素占用的字节数。当色彩类型为 `LCUI_COLOR_TYPE_RGB` 时，该值为 3；当色彩类型为 `LCUI_COLOR_TYPE_ARGB` 时，该值为 4。
* `bytes_per_row`: 每行像素数据占用的字节数。在读写像素数据时，我们需要知道每行像素占用多少字节才能准确计算指定坐标的像素点在像素数据缓存中的下标，我们推荐你优先使用该属性，因为手动编码计算还得考虑色彩类型和字节对齐问题。

以下示例展示了如何创建一个 800x600 像素的图形对象然后释放它：

```c
#include <LCUI.h>
#include <LCUI/graph.h>

int main(void)
{
    LCUI_Graph graph;
    
    Graph_Init(&graph);
    Graph_Create(&graph, 800, 600);
    Graph_Free(&graph);
    return 0;
}
```

### 栅格 <a id="zha-ge"></a>

在我们开始画图之前，我们需要了解一下栅格（Grid）以及坐标空间。如下图所示，图像被栅格所覆盖，栅格中的一个单元对应图像中的一个像素，它的起点位于左上角坐标（0,0）处，所有元素的位置都相对于原点定位。所以图中蓝色方形左上角的坐标距离左边 x 像素，距离上边 y 像素。​

![](https://gblobscdn.gitbook.com/assets%2F-MJ04kFHYqrADYVyG9qI%2F-MYU1HhTEIZLRL62Ts4W%2F-MYU985OTEpPVV1IHNKT%2Fcanvas_default_grid.png?alt=media&token=3eb35b06-ad80-49be-9e72-6fa3b49237a6)

在概念上，图像被我们赋予了二维坐标系，使得图像的每个像素点都有 x 和 y 坐标，而在计算机的内存中，图像的像素数据都存储在一段连续的内存空间内，通过一维下标来访问某个像素点的数据。在日常的图像处理中，为了能够操作像素数据，我们会需要用到一种二维坐标到一维下标的变换方法：

```c
index = y * width + x
```

用 C 代码来表达它就是这样：

```c
LCUI_Graph graph;

// 以一个像素为单位来访问坐标 (x, y) 上的像素
graph->argb[graph->width * y + x];
```

当图像的色彩类型是 `LCUI_COLOR_TYPE_RGB` 时，我们只能以一个字节为单位访问像素数据，用于计算下标的表达式还得考虑到像素点占用的字节数。

```c
LCUI_Graph graph;

// 以一个像素为单位来访问坐标 (x, y) 上的像素
graph->bytes[graph->bytes_per_row * y + graph->bytes_per_pixel * x];
```

### 绘制上下文

绘制上下文描述了绘制时使用的画布和绘制区域，常用于背景图、阴影、边框等复杂的图形绘制操作，你可以将其理解为图形绘制函数的常用参数集合体。它被定义为 `LCUI_PaintContextRec` 结构体类型的对象，与之相关的函数都以 `LCUIPainter_` 前缀命名。

以下示例展示了如何创建和释放绘制上下文：

```c
LCUI_PaintContext paint;
LCUI_Graph canvas;
LCUI_Rect rect;

...

paint = LCUIPainter_Begin(&canvas, &rect);
LCUIPainter_End(paint);
```

### 待办事项

**重新设计图像处理 API**

以开发新的图形库为目的，设计一套图形 API，然后将现有的代码改用这套新 API 来实现。设计时需要考虑的因素有：

* 不依赖 LCUI 的数据类型和功能。
* 参考主流图形库的 API 设计，使得用过其它图形库的人能够快速上手。
* 能够切换多个渲染后端，例如：纯 CPU 渲染、DirectX、OpenGL、skia、cario。

**添加 gif 文件读取和渲染支持**

* 设计合适的数据结构来存储 gif 动画数据。
* 提供相应的函数以实现播放、暂停、渲染功能。
* 当组件的 `background-image` 属性指定了 gif 文件时，应创建一个定时器来渲染动画。

\*\*\*\*

