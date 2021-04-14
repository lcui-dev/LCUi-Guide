# 字体渲染引擎

字体渲染引擎的工作主要是字体文件操作和文字渲染，LCUI 将其抽象成了 `LCUI_FontEngine` 接口，使得  LCUI 的字体渲染引擎可被切换和扩展。

目前基于该接口实现的引擎有内置引擎和 FreeType 引擎，接下来我们再深入了解它们。

### 内置引擎

内置引擎是 LCUI 初始化的第一个引擎，它主要用于在无其它可用引擎的情况下加载预置的字体位图数据，以确保界面中的文字能够被渲染出来。

内置引擎只能加载 `in-core.inconsolata` 字体，该字体已经转换为点阵字库嵌入在源码中，并随着 LCUI 应用程序的运行而一同被加载到内存中，对于内置引擎而言，渲染过程只是简单的将该字库中的文字位图取出来然后混合到画布上。

如果你开发的应用程序是在性能、内存和存储条件较为苛刻的环境中运行的，没有多余的资源供 FreeType 引擎使用，那么只使用内置引擎是开销最小一种办法，但内置的点阵字库包含 12px ~ 18px 的字体位图，你可能需要删除一些位图以缩减字库的大小。

### FreeType 引擎

[FreeType](https://www.freetype.org/) 是一个用于渲染字体的软件库，它采用 C 语言编写，被设计为小型、高效、高可定制、可移植性的同时还能够产生大多数矢量和位图字体格式的高质量输出。

### 添加新的引擎

`LCUI_FontEngine` 在[ include/LCUI/font/fontlibrary.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/font/fontlibrary.h#L82-L88) 中的定义如下：

```c
struct LCUI_FontEngine {
    char name[64];
    int(*open)(const char*, LCUI_Font**);
    int(*render)(LCUI_FontBitmap*, wchar_t, int, LCUI_Font);
    void(*close)(void*);
};
```

从中我们可以看出，我们需要给新的引擎定义 `open()`、`render()` 和 `close()` 方法，然后设置这四个字段，示例如下：

```c
static int MyFontEngine_Open(const char *filepath, LCUI_Font **outfonts)
{
    ...
}

static void MyFontEngine_Close(void *data)
{
    ...
}

static int MyFontEngine_Render(LCUI_FontBitmap *bmp, wchar_t ch,
                               int pixel_size, LCUI_Font font)
{
    ...
}

int LCUIFont_InitMyFontEngine(LCUI_FontEngine *engine)
{
    ....

    strcpy(engine->name, "MyFontEngine");
    engine->render = MyFontEngine_Render;
    engine->open = MyFontEngine_Open;
    engine->close = MyFontEngine_Close;
    return 0;
}

int LCUIFont_ExitMyFontEngine(void)
{
    ...
    return 0;
}
```

如需了解更多，可参考现有引擎的源码：

* [src/font/freetype.c](https://github.com/lc-soft/LCUI/blob/master/src/font/freetype.c)
* [src/font/in\_core\_font.c](https://github.com/lc-soft/LCUI/blob/master/src/font/in_core_font.c)

