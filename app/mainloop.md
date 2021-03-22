---
description: 主循环的概念和相关功能介绍。
---

# 主循环

应用程序在运行的时候，为了能够不断的对用户的操作进行响应和反馈，通常的做法是将事件处理、状态更新和界面重绘等任务往复执行，而这一循环执行的过程即为主循环。

LCUI 的主循环所执行的任务包括处理定时器、处理事件队列、更新组件、渲染组件等，这些任务的调度代码都在 [src/main.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/main.c#L214-L224) 文件中的 `LCUI_RunFrame()` 函数中：

```c
void LCUI_RunFrame(void)
{
	LCUI_ProcessTimers();
	LCUI_ProcessEvents();
	LCUICursor_Update();
	LCUIWidget_Update();
	LCUIDisplay_Update();
	LCUIDisplay_Render();
	LCUIDisplay_Present();
}
```

### 帧率控制

主循环的每次循环即为一帧，为了避免不必要的 CPU 资源占用，主循环的执行频率会受到帧率控制，预设的帧率限制是 120 帧每秒，也就是主循环每秒最多执行 120 遍，每帧至少占用约 8.33 毫秒的时间，如果一帧的耗时低于 8.33 毫秒则会利用剩下的时间进入休眠状态。

如果你需要自定义帧率限制，可以调用 `LCUI_ApplySettings()` 修改全局设置中的 `frame_rate_cap` 设置项：

```c
#include <LCUI.h>
#include <LCUI/settings.h>

int main(void)
{
    LCUI_SettingsRec settings;
    
    Settings_Init(&settings);
    settings.frame_rate_cap = 60;
    LCUI_ApplySettings(&settings);
}
```

### 多个主循环

试着考虑这种场景：在用户点击按钮后弹出一个确认框，等待用户点击确认后再执行操作。这种场景比较常见，我们会希望有个 `ShowConfirmDialog()` 函数能够完成这件事情：

```c
LCUI_BOOL ShowConfirmDialog(const char *title, const char *content)
{
    ...
    if (isOkButtonClicked) {
        return TRUE;
    }
    return FALSE;
}

void OnButtonClick()
{
    if (ShowConfirmDialog("Confirm", "Are you sure you want to do it?")) {
        DoSomeThing();
    }
}
```

按钮的点击事件处理器都是在主循环中执行的，如果 `ShowConfirmDialog()` 函数要等到用户点击弹框里的按钮后才退出的话，它在这段等待时间内会一直阻塞主循环的执行，导致整个界面无法响应用户操作，由于界面无法响应操作， `ShowConfirmDialog()` 函数也无法得知用户是否点击了确认按钮或取消按钮，这就成了一个死循环，那么如何解决此问题？有一种方法是在 `ShowConfirmDialog()` 函数中再创建一个主循环以响应后续的用户操作和界面更新，示例如下：

```c
typedef struct DialogContextRec_ {
    LCUI_BOOL result;
    LCUI_MainLoop loop;
} DialogContextRec, *DialogContext;

static void OnBtnOkClick(LCUI_Widget w, LCUI_WidgetEvent e, void *arg)
{
    DialogContext ctx = e->data;
    ctx->result = TRUE;
    LCUIMainLoop_Quit(ctx->loop);
}

static void OnBtnCancelClick(LCUI_Widget w, LCUI_WidgetEvent e, void *arg)
{
    DialogContext ctx = e->data;
    ctx->result = FALSE;
    LCUIMainLoop_Quit(ctx->loop);
}

LCUI_BOOL ShowConfirmDialog(const wchar_t* title, const wchar_t *content)
{
    DialogContextRec ctx = { 0 };
    LCUI_Widget btn_cancel = LCUIWidget_New("button");
    LCUI_Widget btn_ok = LCUIWidget_New("button");

    ...

    Widget_BindEvent(btn_ok, "click", OnBtnOkClick, &ctx, NULL);
    Widget_BindEvent(btn_cancel, "click", OnBtnCancelClick, &ctx, NULL);
    ctx.loop = LCUIMainLoop_New();
    LCUIMainLoop_Run(ctx.loop);
    Widget_Destroy(dialog);
    return ctx.result;
}
```

{% hint style="info" %}
这段代码省略了弹框组件的构造代码，如需了解完整的实现代码可以查看 LC Finder 项目中的 [src/ui/components/dialog\_confirm.c](https://github.com/lc-soft/LC-Finder/blob/573f200698e2604450665716ebc6608837b4b73a/src/ui/components/dialog_confirm.c) 文件。
{% endhint %}

在这段代码中，先定义了`DialogContextRec` 类型的 ctx 变量用于记录按钮点击状态和主循环的指针，然后为确认按和取消按钮绑定点击事件处理器，之后调用 `LCUIMainLop_New()` 新建了一个主循环，再调用 `LCUIMainLoop_Run()` 执行这个新的主循环。在按钮被点击后，事件处理器会修改 ctx 中的按钮点击状态，然后调用 `LCUIMainLoop_Quit()` 退出指定的主循环。在`LCUIMainLoop_Run()` 函数退出后，销毁弹框并将用户的操作结果返回。

另一种方法是改用回调函数的响应操作结果：

```c
LCUI_BOOL ShowConfirmDialog(
    const char *title,
    const char *content,
    void (*onResult)(LCUI_BOOL, void*)
);

void OnConfirm(LCUI_BOOL isConfirmed)
{
    if (isConfirmed) {
        DoSomeThing();
    }
}

void OnButtonClick()
{
    ShowConfirmDialog(
        "Confirm",
        "Are you sure you want to do it?",
        OnConfirm
    );
}
```

我们不建议采用这种方法，因为它存在以下几个问题：

* 需要再定义一个函数接收操作结果，使得操作逻辑被分散。
* 如果这个函数需要额外的参数话，还要给 `ShowConfirmDialog()` 再增加一个参数，增加了函数复杂度和代码量。
* 由于 C 语言没有像 JavaScript 那样的闭包特性和对异步编程的 async/await 关键字支持，使得这种方法的实现代码和调用代码并不优雅。

### 不使用主循环

如果你的应用程序有自己的主循环，不希望为适应 LCUI 的主循环而做改动，那么可以在主循环中调用 `LCUI_RunFrame()` 函数：

```c
while (your_app.active) {
    your_app_main_loop_task1();
    your_app_main_loop_task2();
    your_app_main_loop_task3();
    ...
    LCUI_RunFrame();
}
```



