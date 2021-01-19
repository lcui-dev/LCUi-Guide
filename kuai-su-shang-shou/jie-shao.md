---
description: 一些基本用法、核心概念的介绍。
---

# 介绍

### LCUI 是什么

LCUI 是一个用于构建用户界面的 C 库。其定位是探索和实践新的用户界面开发方式，以体积小、易于使用、提供便捷的开发工具为特点，帮助 C 开发者快速开发带有图形用户界面的桌面端应用程序。不过由于人力有限，目前 LCUI 在用户界面开发方式的探索和实践还只停留在简单实现了类似于网页的描述风格和渲染效果的程度。

### 起步

尝试 LCUI 最简单的方法是使用 [lcui-quick-start](https://github.com/lc-ui/lcui-quick-start) 例子。你可以下载它然后按照 README.md 中描述的步骤编译并运行它，跟着例子学习一些基础用法。

[安装教程](an-zhuang.md)给出了更多安装 LCUI 的方式。请注意我们**不推荐**新手直接使用 `lcui-cli`，尤其是在你还不熟悉基于 Node.js 的构建工具时。

### 基本框架

先从最简单的代码开始：

```c
#include <LCUI.h>

int main(void)
{
    LCUI_Init();
    return LCUI_Main();
}
```

这段代码完成了一个 LCUI 应用程序的最基础且必要的工作：初始化 LCUI 的各个功能然后将主线程控制权交给 LCUI 的主循环。你可以通过编译这段代码来检查 LCUI 是否正确安装，如果编译失败，请检查编译器的头文件和库文件搜索路径是否配置正确、LCUI 的头文件和库文件的存放位置是否正确。

### 渲染文本

上一段代码只是让程序保持运行，不会在屏幕上输出任何内容，接下来我们再补充一些代码：

```c
#include <LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>

int main(void)
{
    LCUI_Widget root;
    LCUI_Widget text;

    LCUI_Init();
    root = LCUIWidget_GetRoot();
    text = LCUIWidget_New("textview");
    TextView_SetTextW(text, L"Hello, World!");
    Widget_Append(root, text);
    return LCUI_Main();
}
```

TextView 是预置的组件，它提供了文本渲染功能，我们可以使用它将 `Hello, World!` 文本渲染到屏幕上。在使用 TextView 组件之前，我们需要调用 `LCUIWidget_New()` 函数来创建一个组件实例，其中 `"textview"` 参数是 TextView 在组件原型库中注册的名字， `LCUIWidget_New()` 函数根据这个名字找到对应的组件原型后，会调用原型中的 `init` 函数对组件实例进行初始化，这个过程类似于 C++ 中的 `new Class()`。

在 LCUI 底层实现中，所有类型的组件都共用同一个数据结构，这意味着我们只需要用 `LCUI_Widget` 这一种类型的指针来引用组件，从 `LCUIWdget_New()` 函数拿到组件实例后，我们调用了一些函数设置它的文本内容并将它追加到根组件内，其中 `Widget_` 前缀的函数是所有组件通用的函数，可以用于操作组件的基本属性、样式、布局等，而 `TextView_` 前缀的函数则是 TextView 组件专用的函数。

### 处理用户输入

（略）添加按钮：

```c
#include <LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>
#include <LCUI/gui/widget/button.h>

void OnButtonClick(LCUI_Widget self, LCUI_WidgetEvent e, void *arg)
{
    LCUI_Widget text = e->data;
    
    TextView_SetText(text, "Hello, LCUI!");
}

int main(void)
{
    LCUI_Widget root;
    LCUI_Widget text;
    LCUI_Widget button;

    LCUI_Init();
    root = LCUIWidget_GetRoot();
    text = LCUIWidget_New("textview");
    button = LCUIWidget_New("button");
    TextView_SetTextW(text, L"Hello, World!");
    Button_SetTextW(button, L"Change");
    Widget_BindEvent(button, "click", OnButtonClick, text, NULL);
    Widget_Append(root, text);
    Widget_Append(root, button);
    return LCUI_Main();
}
```

（略）添加输入框：

```c
#include <LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>
#include <LCUI/gui/widget/button.h>

void OnButtonClick(LCUI_Widget self, LCUI_WidgetEvent e, void *arg)
{
    wchar_t str[256] = { 0 };
    LCUI_Widget text = ((LCUI_Widget*)e->data)[0];
    LCUI_Widget input = ((LCUI_Widget*)e->data)[1];
    
    TextEdit_GetTextW(input, 0, 255, str);
    TextView_SetText(text, str);
}

int main(void)
{
    LCUI_Widget root;
    LCUI_Widget text;
    LCUI_Widget button;
    LCUI_Widget input;
    static LCUI_Widget button_data[2];

    LCUI_Init();
    root = LCUIWidget_GetRoot();
    text = LCUIWidget_New("textview");
    input = LCUIWidget_New("textedit");
    button = LCUIWidget_New("button");
    button_data[0] = text;
    button_data[1] = input;
    TextView_SetTextW(text, L"Hello, World!");
    TextEdit_SetPlaceholderW(input, L"Please input...");
    Button_SetTextW(button, "Change");
    Widget_BindEvent(button, "click", OnButtonClick, button_data, NULL);
    Widget_Append(root, text);
    Widget_Append(root, input);
    Widget_Append(root, button);
    return LCUI_Main();
}
```

### 设置文本样式

（略）

```c
#include <LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>
#include <LCUI/gui/widget/button.h>

void OnButtonClick(LCUI_Widget self, LCUI_WidgetEvent e, void *arg)
{
    wchar_t str[256] = { 0 };
    LCUI_Widget text = ((LCUI_Widget*)e->data)[0];
    LCUI_Widget input = ((LCUI_Widget*)e->data)[1];
    
    TextEdit_GetTextW(input, 0, 255, str);
    TextView_SetText(text, str);
}

int main(void)
{
    LCUI_Widget root;
    LCUI_Widget text;
    LCUI_Widget button;
    LCUI_Widget input;
    LCUI_Color color = RGB(56, 132, 255);
    static LCUI_Widget button_data[2];

    LCUI_Init();
    root = LCUIWidget_GetRoot();
    text = LCUIWidget_New("textview");
    input = LCUIWidget_New("textedit");
    button = LCUIWidget_New("button");
    button_data[0] = text;
    button_data[1] = input;
    Widget_SetStyle(text, key_color, color, color);
    Widget_SetStyle(text, key_font_size, 24, px);
    Widget_SetPadding(text, 16, 16, 16, 16);
    WIdget_SetBorder(text, 1, SV_SOLID, color);
    TextView_SetTextW(text, L"Hello, World!");
    TextEdit_SetPlaceholderW(input, L"Please input...");
    Button_SetTextW(button, "Change");
    Widget_BindEvent(button, "click", OnButtonClick, button_data, NULL);
    Widget_Append(root, text);
    Widget_Append(root, input);
    Widget_Append(root, button);
    return LCUI_Main();
}
```

### 用 XML 和 CSS 描述用户界面

（略）

`main.css`:

```c
textview {
    color: #3884ff;
    font-size: 24px;
    padding: 16px;
    border: 2px solid #3884ff;
}
```

`main.xml`:

```c
<?xml version="1.0" encoding="UTF-8" ?>
<lcui-app>
  <resource type="text/css" src="main.css"/>
  <ui>
    <textview id="text" type="textview">
      Hello, World!
    </textview>
    <textedit id="input" placeholder="Please input..."></textedit>
    <button id="button">Change</button>
  </ui>
</lcui-app>
```

`main.c`:

```c
#include <LCUI.h>
#include <LCUI/gui/builder.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>
#include <LCUI/gui/widget/button.h>

void OnButtonClick(LCUI_Widget self, LCUI_WidgetEvent e, void *arg)
{
    wchar_t str[256] = { 0 };
    text = LCUIWidget_GetById("text");
    input = LCUIWidget_GetById("input");
    
    TextEdit_GetTextW(input, 0, 255, str);
    TextView_SetText(text, str);
}

int main(void)
{
    LCUI_Widget root;
    LCUI_Widget pack;
    LCUI_Widget button;

    LCUI_Init();
    pack = LCUIBuilder_LoadFile("main.xml");
    if (!pack) {
        return -1;
    }
    Widget_Append(root, pack);
    Widget_Unwrap(pack);
    root = LCUIWidget_GetRoot();
    button = LCUIWidget_GetById("button");
    Widget_BindEvent(button, "click", OnButtonClick, NULL, NULL);
    return LCUI_Main();
}
```

### 以声明式编写用户界面

（略\)

`main.jsx` 

```jsx
import {
  App,
  Widget,
  Button,
  TextView,
  TextEdit
} from 'lcui';
import './main.css';

class MyApp extends App {
  constructor() {
    super();

    this.state = {
      text: 'Hello, World!',
      input: ''
    };
  }

  template() {
    return (
      <Widget>
        <TextView>{this.state.text}!</TextView>
        <Widget class="form-control">
          <TextEdit
            ref="input"
            value={this.state.input}
            placeholder="Please input..."
          />
          <Button onClick={this.onBtnChangeClick}>Change</Button>
        </Widget>
      </Widget>
    );
  }

  onBtnChangeClick() {
    this.state.text = this.refs.input.value.toString();
  }
}

export function main() {
  const app = new MyApp();

  return app.run();
}
```

### 待办事项

#### 取一个新名字

现在的名字是当初作者一时想不出合适的名字而临时取的，四个字母读起来没单词顺口，而且全大写的四个字母在代码中看起来也不太美观。新的名字除了解决这两个问题外，还应该具备这几个特点：无歧义、长度短、可缩写为两个字母。

#### 为 LCUI CLI 添加 XML 编译器

LCUI 的 xml 文件解析功能是由 libxml 库提供支持的，为了缩减 LCUI 的库文件体积、提升用户界面的加载速度，我们可以在编译阶段将 xml 文件预先转换为 C 代码来使用。

#### 为 LCUI CLI 添加 CSS 编译器

集成 SASS 预处理器，支持对多个 scss 或 css 文件进行处理、合并和编译，编译只是简单的将处理后的结果转换成 C 中的字符串代码。



