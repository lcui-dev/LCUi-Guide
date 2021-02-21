---
description: 事件的概念和相关函数的介绍。
---

# 事件

事件是系统内发生的动作或者发生的事情，系统响应事件后，如果需要，你可以某种方式对事件做出回应。例如：如果用户在界面上单击一个按钮，你可能想通过显示一个信息框来响应这个动作。在这篇文章中，我们将讨论一些关于事件的重要概念，并且观察它们在 LCUI 上如何运行。这篇文章不会面面俱到，仅聚焦于你现阶段需要掌握的知识。

在 LCUI 中，事件在应用程序窗口中被触发并且通常被绑定到窗口内部的特定部分 —— 可能是一个组件、一系列组件、应用程序内的代码或者是整个应用程序窗口。举几个可能发生的不同事件：

* 用户在某个组件上点击鼠标或悬停光标。
* 用户在键盘中按下某个按键。
* 用户调整应用程序窗口的大小或者关闭应用程序窗口。
* 全局设置被改变。

如果您想看看更多其他的事件 ，请查阅 [include/LCUI/main.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/main.h#L38-L58) 中定义的事件。

每个可用的事件都会有一个**事件处理器**，也就是事件触发时会运行的代码块。当我们定义了一个用来回应事件被激发的代码块的时候，我们说我们**注册了一个事件处理器**。注意事件处理器有时候被叫做**事件监听器**——从我们的用意来看这两个名字是相同的，尽管严格地来说这块代码既监听也处理事件。监听器留意事件是否发生，然后处理器就是对事件发生做出的回应。

### 简单的例子

```c
#include <LCUI.h>

void OnKeyDown(LCUI_SysEvent e, void *data)
{
    printf("key code: %d\n", e->key.code);
    LCUI_Quit();
}

int main(void)
{
    LCUI_Init();
    LCUI_BindEvent(LCUI_KEYDOWN, OnKeyDown, NULL, NULL);
    return LCUI_Main();
}
```

这个例子实现了按任意键退出的功能，我们定义了一个 `OnKeyDown()` 函数作为按键按下事件的处理器，然后使用 `LCUI_BindEvent()` 函数将它与事件的标识 `LCUI_KEYDOWN` 绑定，当按下任意键时就会调用我们的 `OnKeyDown()` 函数将事件对象中记录的按键码打印出来，然后退出应用程序。

### 事件对象

 有时候在事件处理函数内部，你可能会看到一个固定指定名称的参数，例如`event`，`evt`或简单的`e`。 这被称为**事件对象**，它被自动传递给事件处理函数，以提供额外的信息。

你可以在 [include/LCUI/main.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/main.h#L109-L121) 中找到事件对象的定义：

```c
typedef struct LCUI_SysEventRec_ {
	uint32_t type;
	void *data;
	union {
		LCUI_MouseMotionEvent motion;
		LCUI_MouseButtonEvent button;
		LCUI_MouseWheelEvent wheel;
		LCUI_TextInputEvent text;
		LCUI_KeyboardEvent key;
		LCUI_TouchEvent touch;
		LCUI_PaintEvent paint;
	};
} LCUI_SysEventRec, *LCUI_SysEvent;
```

从这个定义我们可以看出，所有类型的事件对象都共用同一个数据结构，并以 `type` 成员变量来标识该作为哪种事件对象来使用。在上个例子中，由于我们给 `OnKeyDown()` 函数绑定的是 `LCUI_KYEDOWN` 事件，那么传入该函数的事件对象必定是键盘事件对象，因此我们就能直接从 `key.code` 中获取按键码。

{% hint style="info" %}
你可以使用任何您喜欢的名称作为事件对象 - 您只需要选择一个名称，然后可以在事件处理函数中引用它。 开发人员最常使用 e / evt / event，因为它们很简单易记。 坚持标准总是很好。
{% endhint %}

### 核心事件

核心事件是指 LCUI 内部触发的事件，这些事件的标识号被定义为命名以 `LCUI_` 开头的枚举值，详见 [include/LCUI/main.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/main.h#L109-L121) 中的 `LCUI_SysEventType` 定义。

### 系统事件

系统事件是指操作系统中的 GUI 系统所提供的事件，它在 Windows 系统中是消息机制中的窗口消息，而在带有 X11 视窗系统的 Linux 系统中则是 X 窗口事件。它是核心事件的来源，LCUI 的驱动模块主要工作流程就是响应系统事件，从中提取必要的数据，然后转换成核心事件。

系统事件的相关函数如下：

```c
LCUI_API int LCUI_BindSysEvent(int event_id, LCUI_EventFunc func, void *data,
			       void (*destroy_data)(void *));

LCUI_API int LCUI_UnbindSysEvent(int event_id, LCUI_EventFunc func);
```

Windows 系统的事件绑定例子：

```c
LCUI_BindSysEvent(WM_KEYDOWN, OnKeyDown, NULL, NULL);
LCUI_UnbindSysEvent(WM_KEYDOWN, OnKeyDown);
```

Linux 系统的事件绑定例子：

```c
LCUI_BindSysEvent(KeyPress, OnKeyDown, NULL, NULL);
LCUI_UnbindSysEvent(KeyPress, OnKeyDown);
```

如需了解更多，可查看 [src/platform](https://github.com/lc-soft/LCUI/tree/master/src/platform) 目录中的驱动源码。

### 自定义事件

核心事件的标识号是有序递增的，它的取值范围是 `LCUI_NONE` 到 `LCUI_USER`，大于或等于 `LCUI_USER` 的都属于自定义事件。

创建自定义事件很简单，你只需要让事件标识号在  `LCUI_USER` 的基础上递增即可：

```c
enum MyCustomEvent {
    MY_CUSTOM_EVENT_A = LCUI_USER,
    MY_CUSTOM_EVENT_B,
    MY_CUSTOM_EVENT_C,
    MY_CUSTOM_EVENT_D
};
```

### 待办事项

**纠正事件的数据结构和函数的命名**

LCUI 的事件结构体命名用的是 `LCUI_SysEvent` ，绑定窗口事件的函数命名是 `LCUI_BindSysEvent` ，这两处的 Sys 所指的 System 并不是同一个，前者是指 LCUI 内部系统，后者是指操作系统，应该纠正它们的命名。

**重新设计事件接口**

事件解绑方式有 `eventId + eventHandler` 和 `eventHandlerId` 这两种，LCUI 提供了 `LCUI_UnbindEvent()` 函数作为第一种的实现，而第二种却没有，虽然早期版本提供了 `LCUI_UnbindEvent2()` 作为第二种方式的实现，但由于这是个糟糕的设计，在后废弃了。我们可以借此机会为事件模块重新设计一套接口。

