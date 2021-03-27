# 字体数据库

### 简单的示例

以下示例代码展示了如何加载一个字体文件：

```c
#include <stdio.h>
#include <LCUI.h>
#include <LCUI/font.h>

int main(int argc, char *argv[])
{
    int ret;
    if (argc < 2) {
        printf("Please specify the font file path");
        return -1;
    }
    LCUI_InitFontLibrary();
    ret = LCUIFont_LoadFile(argv[1]);
    LCUI_FreeFontLibrary();
    return ret;
}
```

### 选择字型

字体渲染引擎的接口都依赖于字型 id，为了得到字型 id，我们需要用到这两个函数：

```c
int LCUIFont_GetId(const char *family_name, LCUI_FontStyle style,
									 LCUI_FontWeight weight);

size_t LCUIFont_GetIdByNames(int **font_ids, LCUI_FontStyle style,
														 LCUI_FontWeight weight, const char *names);
```

`LCUIFont_GetId()` 用于根据传入的字族名称、样式和字重获取匹配的字型 id，示例：

```c
int font_id = LCUIFont_GetId("Verdana", FONT_STYLE_NORMAL, FONT_WEIGHT_NORMAL);
```

`LCUIFont_GetIdByNames()` 是它的批量版本，可获取多个字型 id，示例：

```c
int *font_ids = NULL;

LCUIFont_GetIdByNames(
    &font_ids,
    FONT_STYLE_NORMAL,
    FONT_WEIGHT_NORMAL,
    "Verdana, Arial, Helvetica"
);
```

{% hint style="info" %}
当这两个函数未找到与指定的风格和字重相匹配的字型时，会按照回退规则选择相近的字型，其中风格的回退规则是按照 oblique -&gt; italic -&gt; normal 顺序回退直到 normal 为止，而字重的回退规则则是采用了与浏览器相同的做法，详见 MDN 上的[字重的回退规则](https://developer.mozilla.org/zh-CN/docs/Web/CSS/font-weight#%E5%9B%9E%E9%80%80%E6%9C%BA%E5%88%B6)。
{% endhint %}

### 渲染文字

渲染文字前，需要先准备一块画布，然后用 `LCUIFont_RenderBitmap()` 函数将指定字符的字型渲染为位图，之后再用 `FontBitmap_Mix()` 将位图混合到画布上。

以下例子展示了如何渲染文字并输出到 PNG 图片中：

```c
#include <LCUI.h>
#include <LCUI/image.h>
#include <LCUI/font.h>

int main(void)
{
    int ret, fid;
    LCUI_Graph img;
    LCUI_FontBitmap bmp;
    LCUI_Pos pos = { 25, 25 };
    LCUI_Color bg = RGB(240, 240, 240);
    LCUI_Color color = RGB(255, 0, 0);

    /* 初始化字体处理功能 */
    LCUI_InitFontLibrary();

    /* 创建一个画布，并填充背景为灰色 */
    Graph_Init(&img);
    Graph_Create(&img, 100, 100);
    Graph_FillRect(&img, bg, NULL, FALSE);

    /* 载入字体文件 */
    ret = LCUIFont_LoadFile("C:/Windows/fonts/simsun.ttc");
    while (ret == 0) {
        /* 获取字体ID */
        fid = LCUIFont_GetId("SimSun", 0, 0);
        if (fid < 0) {
            break;
        }
        /* 渲染对应的文字位图，大小为 48 像素 */
        ret = LCUIFont_RenderBitmap(&bmp, L'字', fid, 48);
        if (ret != 0) {
            break;
        }
        /* 绘制红色文字到图像上 */
        FontBitmap_Mix(&img, pos, &bmp, color);
        LCUI_WritePNGFile("test_char_render.png", &img);
        /* 释放内存资源 */
        FontBitmap_Free(&bmp);
        Graph_Free(&img);
        break;
    }

    /* 释放字体处理功能相关资源 */
    LCUI_FreeFontLibrary();
    return ret;
}
```

运行该程序后打开 `test_char_render.png` 文件，你会看到如下内容：

![&#x6587;&#x5B57;&#x6E32;&#x67D3;&#x6548;&#x679C;](../.gitbook/assets/test_char_render.png)

### 字体数据库

字体数据库由如下数据结构组成：

* **字族表（Dict）：**记录了所有字族结点的哈希表，以字族名称作为索引键值。
* **字族结点（FontFamilyNode）：**记录该字族下的所有字体样式结点列表。
* **字体样式结点（FontStyleNode）：**记录了该字体样式下的字体列表，字体在数组重的下标与字重对应。
* **字型（Font）：**记录了字型的 id、所属字族、样式、字重，以及字体渲染引擎所需的一些数据。

### 字型缓存

字体文件中的字型数据会在加载时被添加到字型缓存中，该缓存是一个数组，每个元素都包含固定长度数组，结构与二维数组类似。缓存的目的就是为了快速访问字型数据，字型的 id 就是它在缓存中的位置，经过如下计算即可访问：

```c
font_cache[id / FONT_CACHE_SIZE]->fonts[id % FONT_CACHE_SIZE]
```

### 位图缓存

渲染文本就是将每个字符的字形栅格化成位图然后绘制到目标面上，其中栅格化的耗时相比读取位图的耗时要高一点，而又由于一段文本通常都会包含一些重复的字符，尤其是由 26 个字母和一些符号组成的英文文本，为减少因栅格化这些重复的字符而增加的耗时，于是就有了位图缓存。

位图缓存的数据结构由嵌套三层的红黑树（RBTree）组成，这三层红黑树分别以字符码、字型 id 和 字体大小为索引键，实现了对每个字符位图的分类功能。

### 待办事项

**优化位图缓存的存储结构和性能**

优化目标如下：

* 提高缓存读写性能：考虑改用数组代替红黑树。
* 减少内存占用：测试字形的栅格化耗时，如果耗时可以忽略，则可改为缓存字形数据。

**重新设计字体数据库的接口**

`LCUIFont_LoadFile()` 的返回值只表示加载是否成功，应用程序无法得知已加载的字体文件的信息，是否需要改造接口使其返回字族名称？或者参考其它字体库，重新设字体数据库的全部接口？





