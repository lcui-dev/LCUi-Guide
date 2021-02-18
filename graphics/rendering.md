---
description: 图形渲染相关 API 和概念的介绍。
---

# 图形渲染

图像渲染是 UI 库的核心能力，它直接影响到 UI 的视觉效果和渲染性能。

### 创建画布

在开始渲染我们的图像前，我们需要创建一个用于存储图像数据的画布（canvas）：

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>

int main(void)
{
    LCUI_Graph canvas;
    
    Graph_Init(&canvas);
    Graph_Create(&canvas, 800, 600);
    LCUI_WritePNGFile("canvas.png", &canvas);
    Graph_Free(&canvas);
    return 0;
}
```

这段代码做了这几件事：

* 初始化一个画布，默认的色彩类型是 RGB
* 为这块画布创建了能存储 800x600 像素的内存空间
* 将画布内的数据写入到 png 文件
* 释放画布占用的资源

### 填充颜色

打开 canvas.png 文件后我们可以发现图片内容是黑色的，因为给画布分配的内存空间初始填充的都是 0，RGB\(0,0,0\) 就是黑色，为了方便看到我们接下来绘制的内容，我们先将画布填充为白色：

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>

int main(void)
{
    LCUI_Graph canvas;
    LCUI_Color white = { .value = 0xffffffff };
    
    Graph_Init(&canvas);
    Graph_Create(&canvas, 800, 600);
    Graph_FillRect(&canvas, white, NULL, FALSE);
    LCUI_WritePNGFile("canvas.png", &canvas);
    Graph_Free(&canvas);
    return 0;
}
```

`Graph_FillRect()` 的第三个参数可以指定填充区域，我们可以试试用它将 \(0, 0 , 100, 200\) 区域填充为红色：

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>

int main(void)
{
    LCUI_Graph canvas;
    LCUI_Color white = { .value = 0xffffffff };
    LCUI_Color red = { .value = 0xffff0000 };
    LCUI_Rect red_area = { 0, 0, 100, 200 };
    
    Graph_Init(&canvas);
    Graph_Create(&canvas, 800, 600);
    Graph_FillRect(&canvas, white, NULL, FALSE);
    Graph_FillRect(&canvas, red, &red_area, FALSE);
    LCUI_WritePNGFile("canvas.png", &canvas);
    Graph_Free(&canvas);
    return 0;
}
```

### 绘制文本

填充色块只是最基本的功能，接下来我们再试试在画布上绘制一段文本：

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <LCUI/font.h>

int main(void)
{
    LCUI_Graph canvas;
    LCUI_Color white = { .value = 0xffffffff };
    LCUI_Color red = { .value = 0xffff0000 };
    LCUI_Color blue = { .value = 0xff0000ff };
    LCUI_Rect red_area = { 0, 0, 100, 200 };
    LCUI_Pos text_pos = { 0, 240 };
    LCUI_TextLayer text_layer = TextLayer_New();
    LCUI_TextStyleRec txxt_style;

    Graph_Init(&canvas);
    Graph_Create(&canvas, 800, 600);
    Graph_FillRect(&canvas, white, NULL, FALSE);
    Graph_FillRect(&canvas, red, &red_area, FALSE);

    LCUI_InitFontLibrary();
    TextStyle_Init(&text_style);
    TextStyle_SetSize(&text_style, 24);
    TextStyle_SetForeColor(&text_style, white);
    TextStyle_SetBackColor(&text_style, blue);

    TextLayer_SetTextStyle(text_layer, &text_style);
    TextLayer_SetTextW(text_layer, L"White text and blue background", NULL);
    TextLayer_Update(text_layer, NULL);
    TextLayer_Render(text_layer, NULL, 0, 240, &canvas);

    LCUI_WritePNGFile("canvas.png", &canvas);

    TextLayer_Destroy(text_layer);
    TextStyle_Destroy(text_style);
    Graph_Free(&canvas);
    LCUI_FreeFontLibrary();
    return 0;
}
```

这段示例代码用到了字体库、文本样式（TextStyle）和文本层（TextLayer），其中文本样式影响文本层的渲染效果，使用它可以设置文本的字族、风格、大小、颜色等样式，而文本层则是让 LCUI 具备文本渲染能力的关键，它的文字排版和渲染能力依赖于字体库提供的字形数据，在使用它之前需要先调用 `LCUI_InitFontLibrary()` 初始化字体库。

在准备好画布和文本样式后，调用文本层的操作函数设置固定大小、文本样式和文本内容，然后调用 `TextLayer_Update()` 应用这些改动，之后调用 `TextLayer_Render()` 将文本层绘制到画布上。

运行这个示例后打开 canvas.png，你会看到蓝底白字的 "White text and blue background" 文本。

### 绘制背景图

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <LCUI/painter.h>
 
int main(void)
{
    LCUI_Graph canvas;
    LCUI_Color white = { .value = 0xffffffff };
    LCUI_Rect bg_area = { 200, 100, 400, 300 };
    LCUI_Rect paint_area = { 0, 0, 400, 300 };
    LCUI_Background bg;
    LCUI_PaintContext paint;
    
    Graph_Init(&canvas);
    Graph_Create(&canvas, 800, 600);
    Graph_FillRect(&canvas, white, NULL, FALSE);
    
    Background_Init(&bg);
    paint = LCUIPainter_Begin(&canvas, &bg_area);
    Background_Paint(&bg, &paint_area, paint);

    LCUI_WritePNGFile("canvas.png", &canvas);

    LCUIPainter_End(paint);
    Graph_Free(&canvas);
    return 0;
}
```

### 绘制边框

### 绘制圆形

### 绘制阴影



