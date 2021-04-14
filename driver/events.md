---
description: 事件循环的概念和驱动开发方式的介绍。
---

# 事件循环

事件是应用程序与自身各个功能模块以及与操作系统进行通讯的手段，也是实现事件驱动编程模型的基础，应用程序如果要响应这些事件，通常是创建一个事件队列来集中存放它们，从事件队列取出事件并调用对应处理器就是一次事件响应，而往复执行这个操作的过程就是事件循环。

### 驱动接口

LCUI 对事件循环的操作有处理事件、绑定事件和解绑事件，驱动模块的职责就是基于操作系统接口向 LCUI 提供实现了这些操作的接口。首先我们看看 [include/LCUI/main.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/main.h#L133-L142) 中的 `LCUI_AppDirver` 定义：

```c
typedef enum LCUI_AppDriverId_ {
        LCUI_APP_UNKNOWN,
        LCUI_APP_LINUX,
        LCUI_APP_LINUX_X11,
        LCUI_APP_WINDOWS,
        LCUI_APP_UWP
} LCUI_AppDriverId;

typedef struct LCUI_AppDriverRec_ {
        LCUI_AppDriverId id;
        void (*ProcessEvents)(void);
        int (*BindSysEvent)(int, LCUI_EventFunc, void *, void (*)(void *));
        int (*UnbindSysEvent)(int, LCUI_EventFunc);
        int (*UnbindSysEvent2)(int);
        void *(*GetData)(void);
} LCUI_AppDriverRec, *LCUI_AppDriver;
```

`id` 是该驱动的标识，在 Linux 系统中，LCUI 针对字符界面和 X11 环境提供了两套驱动，而这个 `id` 的作用就是用来决定该使用哪套驱动，当然 `id` 不只有这一种用途，例如：在非字符界面模式下隐藏鼠标光标。 

`ProcessEvents` 函数用于处理事件队列里的所有事件，如果事件队列中没有事件则会立刻退出，不需要阻塞等待事件。

`UnbindSysEvent` 和 `UbindSysEvent2` 函数都是用于解绑事件，前者的解绑依据 `eventId + eventHandler` ，后者是 `handlerId` 。

剩下的 `GetData` 函数主要用于向其它驱动模块提供数据，例如在 Windows 系统中，虽然各个驱动的代码因出于模块化的考虑而被分割到多个源文件中，但它们都是基于同一个主窗口的消息循环且都依赖主窗口句柄，为了让这些驱动能拿到主窗口句柄，那么就可以靠 `GetData` 函数来获取。

### 开发方式

综上所述，假设你添加的是适用于 Mac OS 的驱动，那么需要如下步骤：

1. 在 `LCUI_AppDriverId` 中的末尾添加 `LCUI_APP_DARWIN` 。
2. 按照函数指针的原型来定义一些事件处理函数。
3. 定义 `LCUI_CreateDarwinAppDriver` 函数，在这个函数创建一个 `LCUI_AppDriver` 类型的对象，然后给它的 `id` 和函数指针设置正确的值。
4. 在[ include/LCUI/platform.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform.h) 中添加针对该操作系统的预处理指令：

   ```c
   #elif __APPLE__
           #define LCUI_CreateAppDriver LCUI_CreateDarwinAppDriver
   ...
   ```

### 参考资料

如需了解更多，可参考现有的 Windows 和 Linux 系统的驱动：

* [include/LCUI/platform/windows/windows\_events.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform/windows/windows_events.h)
* [include/LCUI/platform/linux/linux\_events.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform/linux/linux_events.h)
* [include/LCUI/platform/linux/linux\_x11events.h](https://github.com/lc-soft/LCUI/blob/master/include/LCUI/platform/linux/linux_x11events.h)
* [src/platform/windows/windows\_events.c](https://github.com/lc-soft/LCUI/blob/master/src/platform/windows/windows_events.c)
* [src/platform/linux/linux\_events.c](https://github.com/lc-soft/LCUI/blob/master/src/platform/linux/linux_events.c)
* [src/platform/linux/linux\_x11events.c](https://github.com/lc-soft/LCUI/blob/master/src/platform/linux/linux_x11events.c)

