---
description: 图形渲染相关 API 和概念的介绍。
---

# 绘制简单的图形

### 绘制矩形 <a id="&#x7ED8;&#x5236;&#x77E9;&#x5F62;"></a>

与其它图形库不同，LCUI 提供的图形 API 只支持矩形这一种形式的图形绘制，不支持基于路径来绘制复杂图形。因此，对于其它复杂的图形，你需要手动编写代码填充像素来绘制。

LCUI 提供了一种绘制矩形的方法：

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

opacity 属性影响整个图像的透明度，有效的取值范围是 0.0（完全透明）到 1.0（完全不透明），默认是 1.0。

#### opacity 的使用示例

在这个例子里，我们使用两个图像，一个填充四色格作为背景，另一个用作前景，设置其 `opacity` 属性为 `0.2`，然后在前景图中填充一系列尺寸递增的半透明矩形并用 `Graph_Mix()` 函数将它们混合到背景图上。最终结果是一个径向渐变效果。矩形叠加得越更多，原先所画的矩形的透明度会越低。通过增加循环次数，画更多的矩形，从中心到边缘部分，背景图会呈现逐渐消失的效果。

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

上个例子虽然用填充半透明色代替 opacity 属性也能实现同样的效果，但两个示例都共用同一示例代码的话未免有些无聊，所以我们换一种绘制方式，画一个正方形，将它分成四个填充不同颜色的长方形，每个长方形中都画一系列透明度从左递增的白色矩形：

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



