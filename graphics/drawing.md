---
description: 图形渲染相关 API 和概念的介绍。
---

# 绘制图形

### 绘制矩形 <a id="&#x7ED8;&#x5236;&#x77E9;&#x5F62;"></a>

与其它图形库不同，LCUI 提供的图形 API 只支持矩形这一种形式的图形绘制，不支持基于路径来绘制复杂图形。因此，对于其它复杂的图形，你需要手动编写代码填充像素来绘制。

LCUI 提供了一种方法绘制矩形：

```c
int Graph_FillRect(LCUI_Graph *graph, LCUI_Color color,
                   LCUI_Rect *rect, LCUI_BOOL with_alpha);
```

{% hint style="warning" %}
注意，该函数会将指定区域内的像素替换为指定颜色，而不是与原有颜色混合。
{% endhint %}

现在就来使用这个函数：

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>

int main(void)
{
        int i, j;
        LCUI_Graph canvas;
        LCUI_Color color;
        LCUI_Rect rect;

        Graph_Init(&canvas);
        Graph_Create(&canvas, 150, 150);
        for (i = 0; i < 6; ++i) {
                for (j = 0; j < 6; ++j) {
                        color.red = (unsigned char)(255 - 42.5 * i);
                        color.green = (unsigned char)(255 - 42.5 * j);
                        color.blue = 0;
                        rect.x = j * 25;
                        rect.y = i * 25;
                        rect.width = 25;
                        rect.height = 25;
                        Graph_FillRect(&canvas, color, &rect, FALSE);
                }
        }
        LCUI_WritePNGFile("test_fill_rect.png", &canvas);
        Graph_Free(&canvas);
        return 0;
}

```

在本示例里，我们用两层 `for` 循环来绘制方阵列，每个方格不同的颜色。结果如下图，但实现所用的代码却没那么绚丽。我们用了两个变量 `i` 和 `j` 来为每一个方格产生唯一的 RGB 色彩值，其中仅修改红色和绿色通道的值，而保持蓝色通道的值不变。你可以通过修改这些颜色通道的值来产生各种各样的色板。通过增加渐变的频率，你还可以绘制出类似 Photoshop 里面的那样的调色板。

![test\_fill\_rect.png](../.gitbook/assets/test_fill_rect.png)

### 透明度

除了可以绘制实色图形，我们还可以绘制半透明的图形。通过设置画布的 opacity 属性或使用一个半透明颜色作为填充颜色。

opacity 属性影响整个画布的透明度，有效的取值范围是 0.0（完全透明）到 1.0（完全不透明），默认是 1.0。

#### opacity 的使用示例

在这个例子里，我们使用两个画布，一个填充四色格作为背景，另一个用作前景，设置其 `opacity` 属性为 `0.2`，然后再上面画一系列尺寸递增的半透明矩形。最终结果是一个径向渐变效果。矩形叠加得越更多，原先所画的矩形的透明度会越低。通过增加循环次数，画更多的矩形，从中心到边缘部分，背景图会呈现逐渐消失的效果。

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>

int main(void)
{
        int i, size;
        LCUI_Graph canvas;
        LCUI_Graph fore_canvas;
        LCUI_Rect rect;

        Graph_Init(&canvas);
        Graph_Init(&fore_canvas);
        Graph_Create(&canvas, 150, 150);
        // 画背景
        rect.x = 0;
        rect.y = 0;
        rect.width = 75;
        rect.height = 75;
        Graph_FillRect(&canvas, RGB(255, 221, 0), &rect, FALSE);
        rect.x = 75;
        Graph_FillRect(&canvas, RGB(102, 204, 0), &rect, FALSE);
        rect.x = 0;
        rect.y = 75;
        Graph_FillRect(&canvas, RGB(0, 153, 255), &rect, FALSE);
        rect.x = 75;
        Graph_FillRect(&canvas, RGB(255, 51, 0), &rect, FALSE);
        // 设置前景的 opacity 值
        fore_canvas.opacity = 0.2f;
        // 仅当色彩模式为 ARGB 时 opacity 属性才会生效
        fore_canvas.color_type = LCUI_COLOR_TYPE_ARGB;
        for (i = 0; i < 7; ++i) {
                size = 2 * (10 + 10 * i);
                // 使用新尺寸重新创建前景图
                Graph_Create(&fore_canvas, size, size);
                // 重新填充颜色
                Graph_FillRect(&fore_canvas, RGB(255, 255, 255), NULL, TRUE);
                // 将前景图混合到背景图中
                Graph_Mix(&canvas, &fore_canvas, 75 - size / 2, 75 - size / 2, FALSE);
        }
        LCUI_WritePNGFile("test_mix_rect_with_opacity.png", &canvas);
        Graph_Free(&fore_canvas);
        Graph_Free(&canvas);
        return 0;
}

```

![test\_mix\_rect\_with\_opacity.png](../.gitbook/assets/test_mix_rect_with_opacity.png)

#### 半透明色的使用示例

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <stdio.h>

int main(void)
{
        int i, j;
        LCUI_Graph canvas;
        LCUI_Graph fore_canvas;
        LCUI_Color color;
        LCUI_Rect rect;

        Graph_Init(&canvas);
        Graph_Init(&fore_canvas);
        Graph_Create(&canvas, 160, 160);
        // 画背景
        rect.x = 0;
        rect.y = 0;
        rect.width = 160;
        rect.height = 40;
        Graph_FillRect(&canvas, RGB(255, 221, 0), &rect, FALSE);
        rect.y += 40;
        Graph_FillRect(&canvas, RGB(102, 204, 0), &rect, FALSE);
        rect.y += 40;
        Graph_FillRect(&canvas, RGB(0, 153, 255), &rect, FALSE);
        rect.y += 40;
        Graph_FillRect(&canvas, RGB(255, 51, 0), &rect, FALSE);
        color.red = 255;
        color.green = 255;
        color.blue = 255;
        fore_canvas.color_type = LCUI_COLOR_TYPE_ARGB;
        Graph_Create(&fore_canvas, 15, 30);
        // 画半透明矩形
        for (i = 0; i < 10; ++i) {
                color.alpha = (unsigned char)(255 * (i + 1) / 10.0);
                Graph_FillRect(&fore_canvas, color, NULL, TRUE);
                for (j = 0; j < 4; ++j) {
                        Graph_Mix(&canvas, &fore_canvas, 5 + i * 15, 5 + j * 40, TRUE);
                }
        }
        LCUI_WritePNGFile("test_fill_rect_with_rgba.png", &canvas);
        Graph_Free(&fore_canvas);
        Graph_Free(&canvas);
        return 0;
}

```

![test\_fill\_rect\_with\_rgba.png](../.gitbook/assets/test_fill_rect_with_rgba.png)

### 绘制背景

LCUI 将组件的背景绘制参数抽象成了 `LCUI_Background` 结构体类型的对象，并由 `Background_Paint()` 进行绘制。

现在就使用它在指定区域内填充绿色作为背景色：

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <LCUI/painter.h>

int main(void)
{
        LCUI_Graph canvas;
        LCUI_Color gray = RGB(240, 240, 240);
        LCUI_Color green = RGB(102, 204, 0);
        LCUI_Rect rect = { 200, 100, 400, 300 };
        LCUI_Background bg = { 0 };
        LCUI_PaintContext paint;

        Graph_Init(&canvas);
        Graph_Create(&canvas, 800, 600);
        Graph_FillRect(&canvas, gray, NULL, FALSE);
        // 设置背景色
        bg.color = green;
        // 创建绘制上下文
        paint = LCUIPainter_Begin(&canvas, &rect);
        // 绘制背景
        Background_Paint(&bg, &rect, paint);
        LCUI_WritePNGFile("test_paint_background_color.png", &canvas);
        LCUIPainter_End(paint);
        Graph_Free(&canvas);
        return 0;
}

```

示例中将画布中的区域 `(200, 100, 400, 30)` 作为背景区域，由于我们要让背景区域被完整绘制出来，所以又将该区域作为绘制区域。需要注意的是，背景区域和绘制都共用同一个原点，只有这两个区域相重叠部分才会被绘制出来。

运行结果：

![test\_paint\_background\_color.png](../.gitbook/assets/test_paint_background_color.png)

#### 使用背景图

只是填充颜色的话直接用 `Graph_FillRect()` 更简单些，接下来我们在背景区域内添加一张图片：

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <LCUI/painter.h>

int main(void)
{
        LCUI_Graph canvas;
        LCUI_Graph image;
        LCUI_Color gray = RGB(240, 240, 240);
        LCUI_Color green = RGB(102, 204, 0);
        LCUI_Rect rect = { 200, 100, 400, 300 };
        LCUI_Background bg = { 0 };
        LCUI_PaintContext paint;

        Graph_Init(&canvas);
        Graph_Init(&image);
        Graph_Create(&canvas, 800, 600);
        Graph_FillRect(&canvas, gray, NULL, FALSE);
        // 读取背景图片
        if (LCUI_ReadImageFile("test_image_reader.png", &image) != 0) {
                return -1;
        }
        // 设置背景色
        bg.color = green;
        // 设置背景图
        bg.image = &image;
        bg.size.width = image.width;
        bg.size.height = image.height;
        // 创建绘制上下文
        paint = LCUIPainter_Begin(&canvas, &rect);
        // 绘制背景
        Background_Paint(&bg, &rect, paint);
        LCUI_WritePNGFile("test_paint_background_image.png", &canvas);
        LCUIPainter_End(paint);
        Graph_Free(&image);
        Graph_Free(&canvas);
        return 0;
}
```

![test\_paint\_background\_image.png](../.gitbook/assets/test_paint_background_image.png)

#### 拉伸背景图

当背景图的尺寸与背景区域尺寸不同时，我们可以通过设置宽高属性来让背景图填满背景区域：

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <LCUI/painter.h>

int main(void)
{
        LCUI_Graph canvas;
        LCUI_Graph image;
        LCUI_Color gray = RGB(240, 240, 240);
        LCUI_Color green = RGB(102, 204, 0);
        LCUI_Rect rect = { 200, 100, 400, 300 };
        LCUI_Background bg = { 0 };
        LCUI_PaintContext paint;

        Graph_Init(&canvas);
        Graph_Init(&image);
        Graph_Create(&canvas, 800, 600);
        Graph_FillRect(&canvas, gray, NULL, FALSE);
        // 读取背景图片
        if (LCUI_ReadImageFile("test_image_reader.png", &image) != 0) {
                return -1;
        }
        // 设置背景色
        bg.color = green;
        // 设置背景图
        bg.image = &image;
        // 将背景图设置成与背景区域相同的尺寸
        bg.size.width = rect.width;
        bg.size.height = rect.height;
        // 创建绘制上下文
        paint = LCUIPainter_Begin(&canvas, &rect);
        // 绘制背景
        Background_Paint(&bg, &rect, paint);
        LCUI_WritePNGFile("test_paint_background_image_with_size.png", &canvas);
        LCUIPainter_End(paint);
        Graph_Free(&image);
        Graph_Free(&canvas);
        return 0;
}

```

![test\_paint\_background\_image\_with\_size.png](../.gitbook/assets/test_paint_background_image_with_size.png)

#### 设置背景图位置

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <LCUI/painter.h>

int main(void)
{
        LCUI_Graph canvas;
        LCUI_Graph image;
        LCUI_Color gray = RGB(240, 240, 240);
        LCUI_Color green = RGB(102, 204, 0);
        LCUI_Rect rect = { 200, 100, 400, 300 };
        LCUI_Background bg = { 0 };
        LCUI_PaintContext paint;

        Graph_Init(&canvas);
        Graph_Init(&image);
        Graph_Create(&canvas, 800, 600);
        Graph_FillRect(&canvas, gray, NULL, FALSE);
        // 读取背景图片
        if (LCUI_ReadImageFile("test_image_reader.png", &image) != 0) {
                return -1;
        }
        // 设置背景色
        bg.color = green;
        // 设置背景图
        bg.image = &image;
        bg.size.width = image.width;
        bg.size.height = image.height;
        // 让背景图居中
        bg.position.x = (rect.width - image.width) / 2;
        bg.position.y = (rect.height - image.height) / 2;
        // 创建绘制上下文
        paint = LCUIPainter_Begin(&canvas, &rect);
        // 绘制背景
        Background_Paint(&bg, &rect, paint);
        LCUI_WritePNGFile("test_paint_background_image_with_position.png",
                          &canvas);
        LCUIPainter_End(paint);
        Graph_Free(&image);
        Graph_Free(&canvas);
        return 0;
}

```

![test\_paint\_background\_image\_with\_position.png](../.gitbook/assets/test_paint_background_image_with_position.png)

### 绘制边框

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <LCUI/painter.h>

int paint_background(LCUI_PaintContext paint, LCUI_Rect *box)
{
        LCUI_Graph image;
        LCUI_Color green = RGB(102, 204, 0);
        LCUI_Background bg = { 0 };

        Graph_Init(&image);
        // 读取背景图片
        if (LCUI_ReadImageFile("test_image_reader.png", &image) != 0) {
                return -1;
        }
        // 设置背景色
        bg.color = green;
        // 设置背景图
        bg.image = &image;
        bg.size.width = image.width;
        bg.size.height = image.height;
        // 让背景图居中
        bg.position.x = (box->width - image.width) / 2;
        bg.position.y = (box->height - image.height) / 2;
        // 绘制背景
        Background_Paint(&bg, box, paint);
        Graph_Free(&image);
        return 0;
}

void paint_border(LCUI_PaintContext paint, LCUI_Rect *box)
{
        LCUI_Border border = { 0 };
        LCUI_Color black = RGB(0, 0, 0);

        border.top.color = black;
        border.top.style = SV_SOLID;
        border.top.width = 4;
        border.right.color = black;
        border.right.style = SV_SOLID;
        border.right.width = 4;
        border.bottom.color = black;
        border.bottom.style = SV_SOLID;
        border.bottom.width = 4;
        border.left.color = black;
        border.left.style = SV_SOLID;
        border.left.width = 4;
        border.top_left_radius = 32;
        border.top_right_radius = 32;
        border.bottom_left_radius = 32;
        border.bottom_right_radius = 32;
        Border_Paint(&border, box, paint);
}

int main(void)
{
        int border_size = 4;

        LCUI_Graph canvas;
        LCUI_Graph layer;
        LCUI_Color gray = RGB(240, 240, 240);
        LCUI_Rect border_box = { 0, 0, 400, 300 };
        LCUI_Rect bg_box = { border_box.x + border_size,
                             border_box.y + border_size,
                             border_box.width - border_size * 2,
                             border_box.height - border_size * 2 };
        LCUI_Rect layer_rect = { 0, 0, border_box.width, border_box.height };
        LCUI_PaintContext paint;

        Graph_Init(&canvas);
        Graph_Create(&canvas, 800, 600);
        Graph_FillRect(&canvas, gray, NULL, FALSE);

        Graph_Init(&layer);
        layer.color_type = LCUI_COLOR_TYPE_ARGB;
        Graph_Create(&layer, layer_rect.width, layer_rect.height);

        // 创建绘制上下文
        paint = LCUIPainter_Begin(&layer, &layer_rect);
        paint->with_alpha = TRUE;
        paint_background(paint, &bg_box);
        paint_border(paint, &border_box);
        Graph_Mix(&canvas, &layer, (canvas.width - layer_rect.width) / 2,
                  (canvas.height - layer_rect.height) / 2, FALSE);
        LCUI_WritePNGFile("test_paint_border.png", &canvas);
        Graph_Free(&canvas);
        return 0;
}

```

![test\_paint\_border.png](../.gitbook/assets/test_paint_border.png)

### 绘制阴影

```c
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/image.h>
#include <LCUI/painter.h>

int paint_background(LCUI_PaintContext paint, LCUI_Rect *box)
{
        LCUI_Graph image;
        LCUI_Color green = RGB(102, 204, 0);
        LCUI_Background bg = { 0 };

        Graph_Init(&image);
        // 读取背景图片
        if (LCUI_ReadImageFile("test_image_reader.png", &image) != 0) {
                return -1;
        }
        // 设置背景色
        bg.color = green;
        // 设置背景图
        bg.image = &image;
        bg.size.width = image.width;
        bg.size.height = image.height;
        // 让背景图居中
        bg.position.x = (box->width - image.width) / 2;
        bg.position.y = (box->height - image.height) / 2;
        // 绘制背景
        Background_Paint(&bg, box, paint);
        Graph_Free(&image);
        return 0;
}

void paint_border(LCUI_PaintContext paint, LCUI_Rect *box, int size, int radius)
{
        LCUI_Border border = { 0 };
        LCUI_Color black = RGB(0, 0, 0);

        border.top.color = black;
        border.top.style = SV_SOLID;
        border.top.width = size;
        border.right.color = black;
        border.right.style = SV_SOLID;
        border.right.width = size;
        border.bottom.color = black;
        border.bottom.style = SV_SOLID;
        border.bottom.width = size;
        border.left.color = black;
        border.left.style = SV_SOLID;
        border.left.width = size;
        border.top_left_radius = radius;
        border.top_right_radius = radius;
        border.bottom_left_radius = radius;
        border.bottom_right_radius = radius;
        Border_Paint(&border, box, paint);
}

int main(void)
{
        int border_size = 4;
        int border_radius = 32;
        int width = 800;
        int height = 600;

        LCUI_Graph canvas;
        LCUI_Graph layer;
        LCUI_Color gray = RGB(240, 240, 240);
        LCUI_BoxShadow shadow = { .x = 0,
                                  .y = 0,
                                  .blur = 40,
                                  .spread = 0,
                                  .color = ARGB(150, 0, 0, 0),
                                  .top_left_radius = border_radius,
                                  .top_right_radius = border_radius,
                                  .bottom_left_radius = border_radius,
                                  .bottom_right_radius = border_radius };
        LCUI_Rect shadow_box;
        LCUI_Rect border_box;
        LCUI_Rect bg_box;
        LCUI_Rect layer_rect;
        LCUI_PaintContext paint;

        Graph_Init(&canvas);
        Graph_Create(&canvas, width, height);
        Graph_FillRect(&canvas, gray, NULL, FALSE);

        // 设置居中的背景区域
        bg_box.width = 400;
        bg_box.height = 300;
        bg_box.x = (width - bg_box.width) / 2;
        bg_box.y = (height - bg_box.height) / 2;
        // 基于背景区域，计算边框区域
        border_box.x = bg_box.x - border_size;
        border_box.y = bg_box.y - border_size;
        border_box.width = bg_box.width + border_size * 2;
        border_box.height = bg_box.height + border_size * 2;
        // 基于边框区域，计算阴影区域
        BoxShadow_GetCanvasRect(&shadow, &border_box, &shadow_box);

        // 创建一个临时绘制层
        Graph_Init(&layer);
        layer_rect.x = 0;
        layer_rect.y = 0;
        layer_rect.width = shadow_box.width;
        layer_rect.height = shadow_box.height;
        layer.color_type = LCUI_COLOR_TYPE_ARGB;
        Graph_Create(&layer, layer_rect.width, layer_rect.height);

        // 基于临时绘制层创建绘制上下文
        paint = LCUIPainter_Begin(&layer, &layer_rect);
        paint->with_alpha = TRUE;
        // 将背景区域和边框区域的坐标转换成相对于阴影区域
        bg_box.x -= shadow_box.x;
        bg_box.y -= shadow_box.y;
        border_box.x -= shadow_box.x;
        border_box.y -= shadow_box.y;
        paint_background(paint, &bg_box);
        paint_border(paint, &border_box, border_size, border_radius);
        BoxShadow_Paint(&shadow, &layer_rect, border_box.width,
                        border_box.height, paint);

        // 将临时绘制层混合到画布中
        Graph_Mix(&canvas, &layer, shadow_box.x, shadow_box.y, FALSE);
        LCUI_WritePNGFile("test_paint_boxshadow.png", &canvas);
        Graph_Free(&canvas);
        return 0;
}

```

![test\_paint\_boxshadow.png](../.gitbook/assets/test_paint_boxshadow.png)

### 待办事项

**添加样式转绘制参数的函数**

从上面的绘制背景图的示例代码中我们可以看出像位置、尺寸这类参数都要我们编写代码去计算，要是能用 css 代码描述的话会方便很多，因此，我们需要一个函数能够读取样式表中的 `background-` 开头的属性然后输出成`LCUI_Background` 类型的对象。

