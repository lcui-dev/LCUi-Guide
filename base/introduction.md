---
description: 一些基本用法、核心概念的介绍。
---

# 介绍

### LCUI 是什么

LCUI 是一个用于构建用户界面的 C 库。其定位是探索和实践新的用户界面开发方式，以体积小、易于使用、提供便捷的开发工具为特点，帮助 C 开发者快速开发带有图形用户界面的桌面端应用程序。不过由于人力有限，目前 LCUI 在用户界面开发方式的探索和实践还只停留在简单实现了类似于网页的描述风格和渲染效果的程度。

### 起步

尝试 LCUI 最简单的方法是使用 [lcui-quick-start](https://github.com/lc-ui/lcui-quick-start) 例子。你可以下载它然后按照 README.md 中描述的步骤编译并运行它，跟着例子学习一些基础用法。

### 创建基本应用程序

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

上一段代码只是让程序保持运行，并不会在屏幕上输出任何内容，接下来我们再补充一些代码：

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

TextView 是一个提供文本渲染功能的预置组件，我们可以使用它将 `Hello, World!` 文本渲染到屏幕上。在使用 TextView 组件之前，我们需要调用 `LCUIWidget_New()` 函数来创建一个组件实例，其中 `"textview"` 参数是 TextView 在组件原型库中注册的名字， `LCUIWidget_New()` 函数根据这个名字找到对应的组件原型后，会调用原型中的 `init` 函数对组件实例进行初始化，这个过程类似于 C++ 中的 `new Class()`。

在组件系统的底层实现中，所有类型的组件都共用同一个数据结构，这意味着我们只需要用 `LCUI_Widget` 这一种类型的指针来引用组件实例。从 `LCUIWdget_New()` 函数拿到组件实例后，我们调用了一些函数设置它的文本内容并将它追加到根组件内，其中 `Widget_` 前缀的函数是所有组件通用的函数，可以用于操作组件的基本属性、样式、布局等，而 `TextView_` 前缀的函数则是 TextView 组件专用的函数。

### 处理用户输入

为了让用户和你的应用进行交互，我们可以用 `Widget_BindEvent()`函数绑定一个事件处理器，在用户点击时调用自定义函数：

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

这里我们用到了预置的 Button 组件，它提供了简单的交互反馈效果，用于告知用户点击它可以触发相应的操作。

在 `Widget_BindEvent()` 函数调用代码中，我们指定了事件名称 `"click"` 、事件处理器 `OnButtonClick` 以及传给它的 `text` 组件指针。当用户点击按钮时会触发 click 事件，然后调用与之绑定的 `OnButtonClick()` 函数，该函数从事件数据结构体中的 data 成员拿到绑定时指定的 text 组件，然后将文本内容更改为`"Hello, LCUI!"` 。

LCUI 还提供了 TextEdit 组件，它能响应并存储用户输入的文本内容：

```c
#include <LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>
#include <LCUI/gui/widget/textedit.h>
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
    LCUI_Widget button_data[2];

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

事件处理器 `OnButtonClick()` 的工作是从 TextEdit 组件中读取用户输入的内容然后写入到 TextView 组件中，为了让它能够获得 TextView 和 TextEdit 组件实例，我们在 `main()` 函数中定义了 `button_data` 数组来保存它们的引用，然后通过 `Widget_BindEvent()` 的第四个参数将该数组绑定到 `e->data` 成员以供事件处理器访问。

在使用局部变量向事件处理器传递数据前，我们需要注意变量的生命周期，以避免出现内存访问越界的问题。由于上面的示例中的 `main()` 受到 `LCUI_Main()` 函数的阻塞，它的局部变量的生命周期要等到 `LCUI_Main()` 函数返回后才会结束，也就是说 `main()` 函数的局部变量在 LCUI 的整个生命周期中都是有效的。

### 设置文本样式

到现在为止，程序输出的文本的大小和颜色都是默认的，未免有些过于简陋，我们可以调用 `Widget_SetStyle()` 函数式宏将自定义样式添加到组件的 `custom_style` 样式表中来自定义组件的渲染效果：

```c
#include <LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>
#include <LCUI/gui/widget/textedit.h>
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
    LCUI_Color blue = RGB(56, 132, 255);
    static LCUI_Widget button_data[2];

    LCUI_Init();
    root = LCUIWidget_GetRoot();
    text = LCUIWidget_New("textview");
    input = LCUIWidget_New("textedit");
    button = LCUIWidget_New("button");
    button_data[0] = text;
    button_data[1] = input;
    Widget_SetStyle(text, key_color, blue, color);
    Widget_SetStyle(text, key_font_size, 24, px);
    Widget_SetPadding(text, 16, 16, 16, 16);
    Widget_SetBorder(text, 1, SV_SOLID, color);
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

这段代码将文本颜色和字体大小分别设置成了蓝色和 24px，并增加了边框和内间距，其中的 `Widget_SetPadding()` 和 `Widget_SetBorder()` 都是用于简化样式修改操作的辅助函数，而 `key_` 前缀的标识符引用的是 `LCUI_StyleKeyName` 类型的枚举值，命名与 CSS 属性相同，你可以通过查看 [css\_library.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/gui/css\_library.h#L44) 文件来了解 LCUI 支持哪些 CSS 属性。

### 用 XML 和 CSS 描述 UI

当应用程序的界面变得复杂后，混合着界面布局和样式以及交互逻辑的代码也会变得难以理解和维护，面对满屏的 `Widget_` 函数调用和赋值语句，我们该如何快速找到需要修改的样式，又该如何快速调整界面的布局结构？

为了解决这个问题，我们可以使用 XML 和 CSS 代替 C 代码来描述界面的结构和样式，对上个程序的代码进行改写后可得到这三个文件：

`main.css`:

```css
textview {
    color: #3884ff;
    font-size: 24px;
    padding: 16px;
    border: 2px solid #3884ff;
}
```

`main.xml`:

```markup
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

经过改写后的`main.c` 只保留了处理界面交互逻辑的代码，通过对比改写前的 C 代码，我们可以看出 CSS 和 XML 能以更少量的代码表达更多的信息。

`main.c` 文件中调用 `LCUIBuilder_LoadFile()` 函数加载 `main.xml` 文件内容并构建成组件树，由于这颗组件树的根节点是个只用于包装子组件的容器组件，我们需要先调用 `Widget_Append()` 函数将它追加到根组件里然后调用 `Widget_Unwrap()` 函数移除包装组件。

### 以声明式编写 UI

&#x20;使用 XML 和 CSS 来描述 UI 以达到结构、表现和行为相分离的目的，这种做法和使用 HTML + CSS + JavaScript 开发网页一样，是数十年前就有的开发方式，算不上有多先进，LCUI 的存在目的如果只是为了模仿浏览器的话那也没什么意义，目前 LCUI 在新的 UI 开发方式的探索和实践成果中，能值得一提的是实验性的编程语言 ——Trad，我们在用 C 语言开发 UI 时或许已经意识到一些问题：

* 实现异步操作时，要写一些复杂的代码解决传参和同步问题
* 项目的源码文件多了后，需要花时间维护 Makefile 和构建脚本
* 用 C 语言以面向对象方式编程的体验较差

Trad 语言诞生的目的就是为了解决这些问题，首先我们看看上面的示例应用是如何以 Trad 语言表达的：&#x20;

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

在 Trad 语言中，LCUI 应用的 UI 编写方式参考了 [React](https://reactjs.org)，不再是采用将标记与逻辑分离到不同文件这种人为地分离方式，而是通过将二者共同存放在一个松散耦合单元之中，来实现[关注点分离](https://en.wikipedia.org/wiki/Separation\_of\_concerns)。

为了编译它，我们需要下载安装 Trad 语言的编译器：

```bash
npm install tradlang
```

然后，使用 tradc 命令将 trad 语言代码编译为 C 代码：

```
tradc main.jsx
```

之后使用 C 的编译器将它编译为可执行文件：

```
gcc -o main main.jsx.c -lLCUI
```

在 Trad 的代码库的 [example](https://github.com/lc-soft/trad/tree/master/example) 目录中还有另一个示例可供体验。

### 待办事项

#### 取一个新名字

现在的名字是当初作者一时想不出合适的名字而临时取的，四个字母读起来没单词顺口，而且全大写的四个字母在代码中看起来也不太美观。新的名字除了解决这两个问题外，还应该具备这几个特点：无歧义、长度短、可缩写为两个字母。

#### 为 LCUI CLI 添加 XML 编译器

LCUI 的 xml 文件解析功能是由 libxml 库提供支持的，为了缩减 LCUI 的库文件体积、提升用户界面的加载速度，我们可以在编译阶段将 xml 文件预先转换为 C 代码来使用。

#### 为 LCUI CLI 添加 CSS 编译器

集成 SASS 预处理器，支持对多个 scss 或 css 文件进行处理、合并和编译，其中的编译只是简单的将处理后的结果转换成 C 中的字符串代码，无需生成 `LCUI_StyleSheet` 对象的构造代码。

**支持编译为  WebAssembly 并在浏览器端运行**

详见 [#207](https://github.com/lc-soft/LCUI/issues/207)。

**重写 Trad 的编译器**

&#x20;Trad 的编译器的语法树和生成器的实现代码是混在一起的，代码中大量使用了 class 继承特性，导致功能模块间的耦合度较高，添加新语法解析支持的难度较大。为了解决这些问题，可以参考 [babel](https://github.com/babel/babel/tree/master/packages) 编译器的做法，将代码划分为语法树（AST）、解析器（Parser）、生成器（Generator）。 对于语法树的代码划分，可以参考 [babel-types](https://github.com/babel/babel/tree/master/packages/babel-types) 和 [babel-traverse](https://github.com/babel/babel/tree/master/packages/babel-traverse)，前者用于语法树结点的工具库，后者用于维护整棵树的状态，包括替换、移除和添加结点。

&#x20;**制定 Trad 的语言规范文档**

文档的内容组织方式可参考 [TypeScript 的语言规范文档](https://github.com/microsoft/TypeScript/blob/master/doc/spec-ARCHIVED.md)。



