---
description: 视频驱动的概念和开发方式介绍。
---

# 视频

视频驱动负责实现 LCUI 应用程序与操作系统中的视窗系统的交互，这些交互包括向 LCUI 应用程序提供屏幕尺寸、将组件的信息和渲染结果同步到对应的窗口中，是 LCUI 的几个驱动模块中最为复杂的一个。

### 表面

表面（Surface）是窗口的抽象，也是 LCUI 的组件与操作系统的窗口进行交互的中间层，它屏蔽了各个操作系统中的视窗操作接口的差异和实现细节，使得 LCUI 应用程序只需要专注于将图形内容渲染到表面上，剩下的工作则交给视频驱动，它会将表面的尺寸、位置、标题等信息以及图形内容同步到对应的窗口。

LCUI 将表面的数据结构交由视频驱动在内部定义，应用层代码仅靠 `LCUI_Surface` 类型的指针来引用表面，对表面的操作都是靠调用表面的函数来实现的。

### 显示模式

有三种显示模式：

* `LCUI_DMODE_WINDOWED`：窗口化模式，将根组件绑定到表面上，组件的宽高与表面宽高相同。
* `LCUI_DMODE_FULLSCREEN`：全屏模式，将根组件绑定到表面上，表面宽高与屏幕宽高相同。
* `LCUI_DMODE_SEAMLESS`：无缝模式，为根组件的每个直系子组件绑定一个表面。

初始显示模式是窗口化模式，你可以使用 `LCUIDisplay_SetMode()` 函数更改显示模式。

### 驱动接口

视频驱动接口在 [include/LCUI/display.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/display.h#L79-L102) 中的定义如下：

```c
typedef struct LCUI_DisplayDriverRec_ {
        char name[256];
        int (*getWidth)(void);
        int (*getHeight)(void);
        LCUI_Surface (*create)(void);
        void (*destroy)(LCUI_Surface);
        void (*close)(LCUI_Surface);
        void (*resize)(LCUI_Surface, int, int);
        void (*move)(LCUI_Surface, int, int);
        void (*show)(LCUI_Surface);
        void (*hide)(LCUI_Surface);
        void (*update)(LCUI_Surface);
        void (*present)(LCUI_Surface);
        LCUI_BOOL (*isReady)(LCUI_Surface);
        LCUI_PaintContext (*beginPaint)(LCUI_Surface, LCUI_Rect *);
        void (*endPaint)(LCUI_Surface, LCUI_PaintContext);
        void (*setCaptionW)(LCUI_Surface, const wchar_t *);
        void (*setRenderMode)(LCUI_Surface, int);
        void *(*getHandle)(LCUI_Surface);
        int (*getSurfaceWidth)(LCUI_Surface);
        int (*getSurfaceHeight)(LCUI_Surface);
        void (*setOpacity)(LCUI_Surface, float);
        int (*bindEvent)(int, LCUI_EventFunc, void *, void (*)(void *));
} LCUI_DisplayDriverRec, *LCUI_DisplayDriver;
```

接下来让我们深入了解这个结构体中的成员。

#### name

视频驱动的名称，用于标识当前使用的是哪个视频驱动。

#### getWidth

获取屏幕的宽度。

#### getHeight

获取屏幕的高度。

#### create

创建表面。你可以在这里初始化帧缓存、初始化表面信息、调用系统提供的接口创建窗口。

#### destroy

销毁表面。你可以在这里释放帧缓存、释放表面信息、调用系统提供的接口关闭窗口。

#### close

关闭表面。你可以在这里做销毁窗口的准备工作。这个函数是参考 Windows 的窗口关闭流程而设计的，关闭窗口时会收到 `WM_CLOSE` 消息，`DefWindowProc()` 函数对这个消息的处理就是调用 `DestroyWindow()` 销毁窗口。

#### resize

调整表面尺寸。你可以在这里重新分配帧缓存、调用系统提供的接口调整窗口尺寸。

#### move

移动表面位置。你可以在这里调用系统提供的接口调整窗口位置。

#### show

显示表面。你可以在这里调用系统提供的接口显示窗口。

#### hide

隐藏表面。你可以在这里调用系统提供的接口隐藏窗口。

#### update

更新表面。现有的视频驱动由于考虑到在其它线程上操作表面的情况，所以被设计成需要调用 update 函数才会应用所有的表面操作。不过现在还没有这种情况，你可以忽略这个函数。

#### present

呈现表面的最新内容。你可以在这里将帧缓存中的内容同步到主窗口中。

#### isReady

表面是否已经准备就绪。如果你的表面在 `create()` 函数中因某些原因无法立刻完成初始化，那么可以用这个函数返回 `FALSE` 告知 LCUI 这个表面暂时不能用，需要等待一会，直到返回 `TRUE` 为止。

#### beginPaint

开始绘制。你可以在这里完成绘制前的准备工作，例如创建绘制缓存区来存储接下来绘制的内容，然后返回绘制上下文。

#### endPaint

结束绘制。你可以在这里将已绘制的内容更新到帧缓存中。

#### setCaptionW

设置表面的说明文字。你可以在这里将表面的说明文字更新到窗口标题上。

#### setRenderMode

设置表面的渲染模式。现在的渲染模式有拉伸和直接填充这两种，不过自定义渲染模式的场景很少，你可以忽略这个函数。

#### getHandle

获取表面的窗口句柄。在 Windows 的视频驱动中，这个函数被用于在事件循环驱动中处理 `WM_CLOSE` 消息时判断应该关闭哪个表面。在其它系统的视频驱动中并没有这种处理，`getHandle()` 的返回值为 NULL，它们对窗口关闭事件的响应是直接退出 LCUI。

#### getSurfaceWidth

获取表面宽度。

#### getSurfaceHeight

获取表面高度。

#### setOpacity

设置表面透明度。其它视频驱动没有实现该功能，你可以忽略这个函数。

#### bindEvent

绑定事件。LCUI 会在主窗口主动触发尺寸变化、重绘、更新尺寸限制时做一些操作，例如：在用户主动拖拽调整窗口大小时，LCUI 会将表面的尺寸同步到与之绑定的组件上。

### 开发方式

综上所述，假设你想添加的是适用于 Mac OS 的驱动，那么需要如下步骤：

1. 按照 `LCUI_DisplayDriver` 中的函数指针的原型来定义函数。
2. 定义 `LCUI_CreateDarwinDisplayDriver` 函数，在这个函数创建一个 `LCUI_DisplayDriver` 类型的对象，然后给它的 `name` 和函数指针设置正确的值。
3. 在[ include/LCUI/platform.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform.h) 中添加针对该操作系统的预处理指令：

   ```c
   #elif __APPLE__
           #define LCUI_CreateDisplayDriver LCUI_CreateDarwinDisplayDriver
   ...
   ```

### 参考资料

如需了解更多，可参考现有的 Windows 和 Linux 系统的驱动：

* [include/LCUI/platform/linux/linux\_display.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform/linux/linux_display.h)
* [include/LCUI/platform/linux/linux\_fbdisplay.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform/linux/linux_fbdisplay.h)
* [include/LCUI/platform/linux/linux\_x11display.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform/linux/linux_x11display.h)
* [include/LCUI/platform/windows/windows\_display.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform/windows/windows_display.h)
* [src/platform/linux/linux\_display.c](https://github.com/lc-soft/LCUI/blob/master/src/platform/linux/linux_display.c)
* [src/platform/linux/linux\_x11display.c](https://github.com/lc-soft/LCUI/blob/master/src/platform/linux/linux_x11display.c)
* [src/platform/linux/linux\_fbdisplay.c](https://github.com/lc-soft/LCUI/blob/master/src/platform/linux/linux_fbdisplay.c)
* [src/platform/windows/windows\_display.c](https://github.com/lc-soft/LCUI/blob/master/src/platform/windows/windows_display.c)
* [src/platform/windows/uwp\_renderer.cpp](https://github.com/lc-soft/LCUI/blob/master/src/platform/windows/uwp_renderer.cpp)

### 待办事项

**调整视频驱动接口成员的命名**

`getWidth` 获取的是屏幕的宽度， `getSurfaceWidth` 获取的是表面的宽度，按照这种命名风格，容易让人以为与表面相关的函数指针的命名应该都带有 Surface，然而实际上并没有。我们应该考虑将 `getWidth` 改成 `getScreenWidth`，将 `getSurfaceWidth` 改成 `getWidth`，但这样改的话，`LCUI_DisplayDriver` 是不是应该重命名为 `LCUI_SurfaceDriver` ？毕竟这个接口的操作集大都是针对表面的，而不是屏幕。

**检验全屏模式和无缝模式是否工作正常**

这两个显示模式已经很久没有测试过了，需要添加测试用例来检验是否能够正常工作。

