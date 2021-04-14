# 文本排版与渲染

文字是程序通过界面向用户传递信息的一种最为简单高效的方式，而将这些文字以合理方式的排列并渲染到屏幕上也是界面的基本能力。本章节将介绍 LCUI 中的文字排版与渲染相关概念和用法，并通过一些示例来帮助你快速理解。

### 简单的例子

以下例子展示了如何使用文本层（TextLayer）将一段文本渲染到 320x240 尺寸的图片中。

```c
#include <LCUI.h>
#include <LCUI/image.h>
#include <LCUI/font.h>

int main(void)
{
    int ret;
    LCUI_Graph img;
    LCUI_Pos pos = { 0, 80 };
    LCUI_Rect area = { 0, 0, 320, 240 };
    LCUI_TextLayer txt = TextLayer_New();
    LCUI_TextStyleRec txtstyle;

    /* 初始化字体处理功能 */
    LCUI_InitFontLibrary();

    /* 创建一个图像，并使用灰色填充 */
    Graph_Init(&img);
    Graph_Create(&img, 320, 240);
    Graph_FillRect(&img, RGB(240, 240, 240), NULL, FALSE);

    /* 设置文本的字体大小 */
    TextStyle_Init(&txtstyle);
    txtstyle.pixel_size = 24;
    txtstyle.has_pixel_size = TRUE;

    /* 设置文本图层的固定尺寸、文本样式、文本内容、对齐方式 */
    TextLayer_SetFixedSize(txt, 320, 240);
    TextLayer_SetTextStyle(txt, &txtstyle);
    TextLayer_SetTextAlign(txt, SV_CENTER);
    TextLayer_SetTextW(txt, L"这是一段测试文本\nHello, World!", NULL);
    TextLayer_Update(txt, NULL);

    /* 将文本图层绘制到图像中，然后将图像写入至 png 文件中 */
    TextLayer_RenderTo(txt, area, pos, &img);
    ret = LCUI_WritePNGFile("test_string_render.png", &img);
    Graph_Free(&img);

    /* 释放字体处理功能相关资源 */
    LCUI_FreeFontLibrary();
    return ret;
}
```

### 最大尺寸和固定尺寸

文本的排版效果和尺寸受到最大尺寸和固定尺寸的影响，虽说尺寸包括宽度和高度，但能直接对它们产生影响的主要是宽度，细节如下：

* 如果两者都未设置，则排版时不对文本做换行处理。文本层宽度取所有行宽度中的最大值，高度是所有行高之和。
* 如果仅设置最大尺寸，则排版时仅在文本行宽度超出最大宽度时做换行处理。文本层宽度取所有行宽度中的最大值，不大于最大宽度；高度是所有行高度之和，不大于最大高度。
* 如果仅设置固定尺寸，则排版时仅在文本行宽度超出固定宽度时做换行处理。文本层的尺寸等于固定尺寸。
* 如果两者都被设置，则效果与仅设置固定尺寸时的效果相同。

之所以有这两种尺寸，是为了解决 TextView 组件的尺寸计算问题。当 TextView 组件的显示模式是内联块（inline-block）时，为了让它的尺寸自适应内容尺寸，它会在布局时将文本层的实际尺寸返回给布局引擎，等布局引擎计算出 TextView 组件的合适尺寸后，再将实际尺寸作为固定尺寸，对文本层重新排版。

### 排版

文字排版的过程是将每个字从左至右排列，每行文本的行高取每个字的高度的最大值。

影响排版效果的有两个属性：

* `enable_mulitiline`：多行模式。启用时，文本中的换行符后面的文本都会被移动到下一行中；禁用时，无视换行符，将所有文本放在一行中。
* `enable_autowrap`：自动换行。仅在多行模式启用的情况下有效，当文本超出最大宽度时会将剩余文本移动到下一行。

### 样式标签

样式标签用于对标记的文本设置样式，在启用该功能后，文本层会维护一个样式列表，记录每段文本样式，可用 `TextLayer_EnableStyleTag()` 函数设置是否启用该功能。

目前支持的标签有：

* `[color]`：颜色
* `[bgcolor]`：背景色
* `[size]`：大小
* `[b]`：加粗
* `[i]`：斜体

用法如下：

```text
[color="#f00"]red[/color]
[bgcolor="#f00"]red Background[/color]
[size="18px"]18px size[/size]
[b]bold[/b]
[i]italic[/i]
```

### 光标

光标（Caret）主要为 TextEdit 组件服务，它的位置即是文本插入位置，访问文本层对象的 `insert_x` __和  `insert_y` 成员可获取该位置。当调用 `TextLayer_InsertText()` 函数时，文本就会插入到光标所处的位置。

`insert_x` __和  `insert_y` 成员分别记录光标所在的列和行，对于像 TextEdit 组件这种需要光标的实际像素坐标来绘制光标的情况，则需要调用：

```c
int TextLayer_GetCaretPixelPos(LCUI_TextLayer layer, LCUI_Pos *pixel_pos);
```

光标的位置操作函数有两个：

```c
void TextLayer_SetCaretPos(LCUI_TextLayer layer, int row, int col);

int TextLayer_SetCaretPosByPixelPos(LCUI_TextLayer layer, int x, int y);
```

`TextLayer_SetCaretPos()` 函数适用于 TextEdit 组件响应按键控制移动光标的情况，而 `TextLayer_SetCaretPosByPixelPos()` 函数则适用于 TextEdit 组件响应鼠标点击时将点击处坐标转换为光标位置的情况。

### 坐标偏移量

坐标偏移量（Offset）影响文本的绘制位置，主要用于实现滚动功能，访问文本层对象的 `offset_x`  和 `offset_y` 成员可获得它的坐标偏移量。典型的例子就是 TextEdit 组件，当它的文本内容超出自身尺寸时就会显示滚动条，用户拖动滚动条本质上就是在更改坐标偏移量。

坐标偏移量的修改函数是：

```c
LCUI_BOOL TextLayer_SetOffset(LCUI_TextLayer layer, int offset_x, int offset_y);
```

该函数会将传入的偏移量赋值给 `new_offset_x` 和 `new_offset_y`，等 `TextLayer_Update()` 被调用时才会更新到 `offset_x` 和 `offset_y`。

### 渲染

渲染文本层的内容需要用到：

```c
int TextLayer_RenderTo(LCUI_TextLayer layer, LCUI_Rect area, LCUI_Pos layer_pos,
		       						 LCUI_Graph *canvas);
```

在它的参数中，`area` 指定了文本层中需要渲染的区域，`pos` 指定了这块区域在画布中的坐标，而`canvas` 就是用于存储渲染结果的画布。

如需了解更多用法，可参考预置组件的源码：

* [src/gui/widget/textview.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/gui/widget/textview.c#L281)
* [src/gui/widget/textedit.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/gui/widget/textedit.c#L933)

### 待办事项

**改进文本的渲染算法**

现在绘制文本时所用的基线位于行高的 4 / 5，这可能不是最佳做法。

**重写文本排版算法**

在重写前考虑 LCUI 新布局引擎设计方案，使两者能够共用同一布局引擎。

**添加文本选中高亮功能**

提供一个函数用于选中一段文本，并给这段文本填充背景色，效果类似于用鼠标点选拖动选择网页中的一段文本。

**添加支持其它书写方向**

现在只支持从左至右排列文本，应该添加支持从上至下、从右至左等书写方向。

**重写 TextLayer 模块**

TextLayer 的设计从开发之初到现在都没有多大改动，其源文件中的源码已经有一千多行，是时候重写它了，重写前需要考虑内存占用、接口设计和模块划分的问题，例如：采用内存占用更少的数据结构，应用主流设计思想重新设计接口，按照功能职责划分排版、编辑、渲染、光标定位等模块。

