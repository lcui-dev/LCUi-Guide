# 输入法

输入法是一种将输入设备输入的数据翻译成字符的方法，这个方法可以表示编码方案和输入平台两种含义，本文将输入法作为输入平台来讲解 LCUI 对它的支持方案。

LCUI 的输入法引擎负责的工作很简单：记录支持的输入法列表，在有按键输入时调用当前激活的输入法驱动进行处理，然后转换成 TEXTINPUT 事件，让 TextEdit 等具有文本输入支持的组件接收输入法输入的文本。

### 添加输入法

LCUI 将输入法驱动抽象成了 `LCUI_IMEHandler` 接口，它在 [include/LCUI/ime.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/ime.h#L36-L42) 中的定义如下：

```c
typedef struct LCUI_IMEHandlerRec_ {
    LCUI_BOOL (*prockey)(int, int);
    void (*totext)(int);
    LCUI_BOOL (*open)(void);
    LCUI_BOOL (*close)(void);
    void (*setcaret)(int, int);
} LCUI_IMEHandlerRec, *LCUI_IMEHandler;
```

添加输入法过程就是补全这些方法供 LCUI 的输入法引擎调用，示例模板代码如下：

```c
static LCUI_BOOL X11IME_ProcessKey(int key, int key_state)
{
    // ...
    return FALSE;
}

static void X11IME_ToText(int ch)
{
    wchar_t text[256];
    size_t text_len;

    // ...
    LCUIIME_Commit(text, text_len);
}

static LCUI_BOOL X11IME_Open(void)
{
    // ...
    return TRUE;
}

static LCUI_BOOL X11IME_Close(void)
{
    // ...
    return TRUE;
}

int LCUI_RegisterLinuxIME(void)
{
    LCUI_IMEHandlerRec handler;
    handler.prockey = X11IME_ProcessKey;
    handler.totext = X11IME_ToText;
    handler.close = X11IME_Close;
    handler.open = X11IME_Open;
    handler.setcaret = NULL;
    return LCUIIME_Register("My Input Method", &handler);

}
```

接下来我们将以中文输入法为例，介绍这些方法：

`prockey()` 方法用于判断输入法是否处理按键，第一个参数是按键码，第二个参数是按键的状态，当返回值为 TRUE 时会调用 `totext()` 方法。

`totext()` 用于将按键码转换为文本。当为中文模式时，可以根据输入的按键码组成拼音，并更新候选词列表供用户选择；当为英文模式时，则可以将按键码转换为字符码，然后调用 `LCUIIME_Commit()` 函数提交它。

`open()` 方法用于在输入法被激活时调用。你可以在这里做一些与初始化相关的操作。

`close()` 方法用于在输入法被关闭时调用。你可以在这里终止输入法的工作并清理相关资源。

`setcaret()` 方法用于设置输入光标的位置，参数是光标的 x、y 坐标。TextEdit 组件中的 TextCaret 组件会在它更新时调用该方法来同步输入法光标的位置，以让输入法的界面定位在 TextEdit 组件附近。

如需了解更多，可参考现有的 Windows 和 Linux 输入法驱动：

* [src/platform/windows/windows\_ime.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/windows/windows_ime.c#L90)
* [src/platform/linux/linux\_ime.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/linux/linux_ime.c#L83)

### 待办事项

**重构输入法引擎**

LCUI 的输入法引擎在设计之初由于可参考的相关资料很少，对输入法的功能需求也不明确，因此只以“能够接收文本输入”为目的做了简单设计，其中一些 API 是参考了 WIndows 输入法相关的文档而设计的。如果你有输入法开发经验，可以帮助我们改进输入法支持模块。

**将 proc\(\) 的第二个参数改为布尔类型**

按键的状态有按下和释放，将其改为布尔类型的 `isPressed` 比判断 `state == LCUI_KSTATE_PRESSED` 更简单。

**改进 Linux 输入法引擎**

现有的 Linux 输入法引擎过于简单，支持输入英文字母和符号。

