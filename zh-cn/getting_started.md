# 快速上手

本文会介绍 LCUI 的基本用法，以便快速上手。这里假定你已经安装好了 LCUI ，并且能够配置应用程序的相关编译参数，否则请先阅读《[安装](install)》。

## 一个最小的应用

``` c
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>

int main( int argc, char **argv )
{
        LCUI_Widget root, txt;

        LCUI_Init();
        root = LCUIWidget_GetRoot();
        txt = LCUIWidget_New( "textview" );
        TextView_SetText( txt, "Hello, World!" );
        Widget_Append( root, txt );
        return LCUI_Main();
}
```

把它保存为 `helloworld.c` 或其它类似名称，然后用你的编译器编译它，编译成功后运行输出的可执行文件，你会看到如下的效果：

![运行效果](../images/getting_started_step_1.png)

那么，这些代码是什么意思呢？

- 开头两行引入了 `LCUI_Build.h` 和 `LCUI/LCUI.h`，这两个头文件中主要包含了 LCUI 应用程序必要的相关参数、数据结构定义以及常用函数声明。
- 接着又引入了 `LCUI/gui/widget.h` 和 `LCUI/gui/widget/textview.h`，这两个头文件是属于图形界面相关的，在 LCUI 中，组成图形界面的基本元素是部件（widget），你也可以将其理解为控件（Control）， `widget.h` 包含了部件相关的通用操作函数，而 `textview.h` 包含了预置的文本显示（textview）部件的相关操作函数。
- 在 main() 函数中，定义了两个类型为 `LCUI_Widget` 的变量，用于保存后面将要用到的部件。
- 调用 LCUI_Init() 函数初始化 LCUI，LCUI 大部分功能都必须在初始化后才能使用。
- 调用 LCUIWidget_GetRoot() 函数获取根级部件，在 LCUI 中，只有根级部件内的部件才能显示在屏幕上。
- 调用 LCUIWidget_New() 函数创建一个类型为 `textview` 的部件，并用 `txt` 变量保存。
- 调用 TextView_SetText() 函数指定 textview 部件所显示的文本，注意，该函数的第二个参数（字符串）的编码方式必须是 UTF-8，否则非英文字符可能会显示成乱码。
- 在部件创建完后，需要调用 Widget_Append() 函数将它追加到根级部件上，以让 LCUI 能够处理并将它显示在屏幕上。
- 最后，调用 LCUI_Main() 函数进入 LCUI 的主循环，执行后续产生的各种任务，顺便让程序保持运行。

## 改进视觉效果

以上程序主要实现了简单的文本显示，接下来将介绍如何调整文字的位置、大小、颜色。

``` c
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/widget/textview.h>

int main( int argc, char **argv )
{
        LCUI_Widget root, txt;
        LCUI_StyleSheet style;

        LCUI_Init();
        root = LCUIWidget_GetRoot();
        txt = LCUIWidget_New( "textview" );
        style = txt->custom_style;
        SetStyle( style, key_margin_top, 25, px );
        SetStyle( style, key_margin_left, 25, px );
        SetStyle( style, key_padding_top, 25, px );
        SetStyle( style, key_border_width, 1, px );
        SetStyle( style, key_border_color, RGB( 0, 0, 0 ), color );
        SetStyle( style, key_background_color, RGB( 245, 245, 245 ), color );
        TextView_SetText( txt, "[size=18px]Hello, World![/size]" );
        TextView_SetTextAlign( txt, SV_CENTER );
        Widget_Resize( txt, 200, 100 );
        Widget_Append( root, txt );
        return LCUI_Main();
}
```

在原有的代码上补充了几行代码，运行效果如下：

![运行效果](../images/getting_started_step_2.png)

继续讲解代码：
- 为节省后面代码的宽度，定义了一个类型为 `LCUI_StyleSheet` 的 `style` 变量来引用 `txt->custom_style`。每个部件都有四张样式表：style（当前应用的样式）、inherited_style（继承的样式）、custom_style（自定义样式）、cached_style（缓存的样式），style 表是由 inherited_style 和 custom_style 这两张样式表合并而成的，通常为部件添加额外的样式只需要修改 custom_style 表，而其它三张表是由 LCUI 负责生成的，即使手动修改它的内容也会很快还原。
- 调用 SetStyle() 为部件设置外边距、内边距、边框和背景色，上外边距和左外边距都为 25 像素，上内边距为 25 像素，边框线的宽度为 1 像素，颜色为纯黑，背景色为灰色。
- SetStyle() 是一个宏，主要用于简化部件样式表的修改操作，感兴趣的话可以查看它的定义。
- `key_` 的开头标识符都是枚举，用于表示各个样式属性在样式表中的位置，为方便识别，并没有和普通枚举那样采用全大写字母的命名方式。
- `[size=18px]` 表示的是将字体的大小设置为 18 像素。TextView 部件默认支持样式标签，但当前支持的样式标签只有 `[size]` 和 `[color]`，你可以直接在文本内使用它们来设置简单的样式。
- TextView_SetTextAlign() 函数为 TextView 部件设置文本对齐方式，支持的对齐方式有三种：靠左（SV_LEFT）、居中（SV_CENTER）、靠右（SV_RIGHT），SV 是样式值（Style Value）的缩写。
- 调用 Widget_Resize() 调整部件的尺寸为 200 x 100。

看了上面的代码你可能会想：为什么修改部件的坐标需要设置外边距？不是直接设置 xy 坐标就可以了吗？

LCUI 有提供 `Widget_Move()` 函数，用于修改部件的坐标，但对于默认定位方式的部件来说，这个函数并没有任何效果。LCUI 的布局方式和浏览器中的网页布局类似，部件在默认的定位方式下，能够影响坐标的只有外边距，除非更改该部件的定位方式，具体内容在后面的章节中会介绍。

## 使用 XML 和 CSS 简化界面描述代码

LCUI 支持使用 XML 和 CSS 代码描述界面的布局和样式，以下是代码，请将其保存为 `helloworld.css` 文件。

``` css
textview.text-hello {
  color: #8cc63f;
  font-size: 18px;
  font-family: "Comic Sans MS";
  text-align: center;
  padding: 25px;
  margin: 25px 0 0 25px;
  border: 1px solid #000;
  background-color: #aaa;
}
```

这里的 CSS 代码对于有写过网页的人来说应该很容易理解，如果你对 CSS 代码的语法规则并不了解，可以参考网上的相关教程。LCUI 虽然支持 CSS 代码解析，但与网页浏览器不同，只支持处理简单的 CSS 样式。

`textview.text-hello` 是选择器，`textview` 指的是部件类型，而 `.text-hello` 指的是样式类，也就是说 {} 花括号里的 CSS 样式只对拥有 `text-hello` 类的 `textview` 部件有效。

color、font-size、font-family、text-align 这四个属性是 textview 部件扩展的属性，仅对 textview 类型的部件有效，分别用于设置文字的颜色、字体大小、字族名称、对齐方式。

新建 `helloworld.xml` 文件，保存以下代码：

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<lcui-app>
  <resource type="text/css" src="helloworld.css"/>
  <resource type="application/font-ttf" src="C:/Windows/Fonts/comic.ttf"/>
  <ui>
    <widget type="textview" class="text-hello">Hello, World!</widget>
  </ui>
</lcui-app>
```

以下是这段 XML 代码的说明：
- 第一行指定了 XML 的版本及文档编码方式。
- `<lcui-app>` 标签是表示里面的代码是针对 LCUI 应用程序的。
- `<resource>` 标签用于指示 LCUI 需要加载的资源，`type` 属性表示资源的类型，`src` 属性表示资源文件的位置。
- 加载 `helloworld.css` 资源文件，将其内容作为 CSS 文本来处理。
- 加载 `C:/Windows/Fonts/comic.ttf` 资源文件，将其作为 ttf 字体文件来处理。
- `<ui>` 标签用于容纳所有与界面相关的内容，一个 XML 文档中只能有一个 `<ui>` 标签，相当于 HTML 文档中的 `<body>` 标签。
- `<widget>` 标签指示 LCUI 创建一个部件，`type` 属性表示部件类型，`class` 属性表示该部件拥有的样式类。`<widget>` 标签内可以嵌套文本，但文本是否有用取决于该类型的部件是否支持。
- 创建一个 `textivew` 类型的部件，拥有 `text-hello` 样式类，并设置其文本内容为 `Hello, World!`。

``` c
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/builder.h>

int main( int argc, char **argv )
{
        LCUI_Widget root, pack;

        LCUI_Init();
        root = LCUIWidget_GetRoot();
        pack = LCUIBuilder_LoadFile( "helloworld.xml" );
        if( !pack ) {
                return -1;
        }
        Widget_Append( root, pack ); 
        Widget_Unwrap( pack );
        return LCUI_Main();
}
```

当有了 XML 和 CSS 文件后，需要让程序载入它们，XML 文件的载入与解析功能由 LCUIBuilder_LoadFile() 函数提供，该函数在 `LCUI/gui/builder.h` 头文件有声明。如果 XML 文件载入失败，LCUIBuilder_LoadFile() 函数会返回 NULL，如果载入成功则会返回一个部件，这个部件主要用于容纳 `<ui>` 标签中出现的各个部件，相当于一个容器，对于这个容器，可以先将它追加到根级部件中，然后调用 Widget_Unwrap() 函数展开该部件的内容，在展开后该部件会被销毁。

以下是程序的运行效果：

![运行效果](../images/getting_started_step_3.png)

## 为界面添加用户交互

当前应用程序的界面还只是单纯的向用户展示信息，接下来将介绍如何让应用程序通过界面与用户进行交互。

首先，修改 XML 文件，添加按钮部件，为了能够在程序中操作它们，还需要并为它们加上 ID，ID 内容可以由你自己定义，但 ID 必须是唯一的。

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<lcui-app>
  <resource type="text/css" src="helloworld.css"/>
  <resource type="application/font-ttf" src="C:/Windows/Fonts/comic.ttf"/>
  <ui>
    <widget id="text-hello" type="textview" class="text-hello">Hello, World!</widget>
    <widget id="btn-ok" type="button">确定</widget>
  </ui>
</lcui-app>
```

接下来补充 CSS 代码，设置按钮的外边距，调整按钮的位置。

``` css
textview.text-hello {
  color: #8cc63f;
  font-size: 18px;
  font-family: "Comic Sans MS";
  text-align: center;
  padding: 25px;
  margin: 25px 0 0 25px;
  border: 1px solid #000;
  background-color: #fafafa;
}
#btn-ok {
  margin: 25px 0 0 25px;
}
```

最后，补充事件绑定与事件响应代码：

``` c
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/builder.h>
#include <LCUI/gui/widget/textview.h>

static void OnBtnClick( LCUI_Widget self, LCUI_WidgetEvent e, void *arg )
{
        LCUI_Widget txt = e->data;
        TextView_SetTextW( txt, L"第一个 LCUI 应用程序" );
}

int main( int argc, char **argv )
{
        LCUI_Widget root, pack, btn, txt;

        LCUI_Init();
        root = LCUIWidget_GetRoot();
        pack = LCUIBuilder_LoadFile( "helloworld.xml" );
        if( !pack ) {
                return -1;
        }
        Widget_Append( root, pack ); 
        Widget_Unwrap( pack );
        txt = LCUIWidget_GetById( "text-hello" );
        btn = LCUIWidget_GetById( "btn-ok" );
        Widget_BindEvent( btn, "click", OnBtnClick, txt, NULL );
        return LCUI_Main();
}
```

以上代码的功能是让按钮在点击后将 "hello, world!" 更改为 "第一个 LCUI 应用程序"，代码具体说明如下：

- 使用 LCUIWidget_GetById() 函数根据 ID 来获取需要操作的部件。
- 为按钮绑定 `click` 事件，事件处理函数为 OnBtnClick()，附加的数据是 txt，该数据不需要析构函数，所以为 NULL。
- 在 OnBtnClick() 函数中，第一个参数是绑定该事件的部件，第二个参数是事件相关的数据，第三个是触发该事件时传递的额外参数，这个参数通常用不到。
- 绑定事件时保存的附加数据存在于 data 成员变量中，即：`e->data`。
- TextView_SetTextW() 函数是 TextView_SetText() 函数的宽字符版本，它的第二个参数是类型为 `wchar_t*` 的指针，这里设置的文本内容包含中文，由于 TextView_SetText() 函数是默认将第二个参数作为 UTF-8 编码的字符串进行处理的，而 windows 系统的编译器会将字符串以 ANSI 编码方式存储，为避免乱码，所以改用宽版本的 TextView_SetTextW() 函数。

以下是该应用程序的运行效果：

![运行效果](../images/getting_started_step_4.gif)

以上供用户操作的只有按钮，接下来将添加文本编辑框，让用户输入自己的内容。

修改 `helloworld.xml` 文件，添加文本编辑框，并设置初始文本为 `Hello, World!`。

``` xml
<?xml version="1.0" encoding="UTF-8" ?>
<lcui-app>
  <resource type="text/css" src="helloworld.css"/>
  <resource type="application/font-ttf" src="C:/Windows/Fonts/comic.ttf"/>
  <ui>
    <widget id="text-hello" type="textview" class="text-hello">Hello, World!</widget>
    <widget id="edit" type="textedit">Hello, World!</widget>
    <widget id="btn-ok" type="button">确定</widget>
  </ui>
</lcui-app>
```

修改 `helloworld.css` 文件，调整文本编辑框的外边距。

``` css
textview.text-hello {
  color: #8cc63f;
  font-size: 18px;
  font-family: "Comic Sans MS";
  text-align: center;
  padding: 25px;
  margin: 25px 0 0 25px;
  border: 1px solid #000;
  background-color: #fafafa;
}
#btn-ok, #edit {
  margin: 25px 0 0 25px;
}
```

然后修改 `helloworld.c` 文件，让程序能够在按钮点击后取出文本编辑框内的文本，并将这些文本显示出来。

``` c
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/gui/widget.h>
#include <LCUI/gui/builder.h>
#include <LCUI/gui/widget/textview.h>
#include <LCUI/gui/widget/textedit.h>

static void OnBtnClick( LCUI_Widget self, LCUI_WidgetEvent e, void *arg )
{
        wchar_t str[256];
        LCUI_Widget edit = LCUIWidget_GetById( "edit" );
        LCUI_Widget txt = LCUIWidget_GetById( "text-hello" );
        TextEdit_GetTextW( edit, 0, 255, str );
        TextView_SetTextW( txt, str );
}

int main( int argc, char **argv )
{
        LCUI_Widget root, pack, btn;

        LCUI_Init();
        root = LCUIWidget_GetRoot();
        pack = LCUIBuilder_LoadFile( "helloworld.xml" );
        if( !pack ) {
                return -1;
        }
        Widget_Append( root, pack ); 
        Widget_Unwrap( pack );
        btn = LCUIWidget_GetById( "btn-ok" );
        Widget_BindEvent( btn, "click", OnBtnClick, NULL, NULL );
        return LCUI_Main();
}
```

TextEdit_GetTextW() 函数用于取出文本编辑框内的文本，它的第二个参数是起始读取位置，第三个参数是文本最大长度，返回值为实际读取的文本长度。

以下是该应用程序的运行效果：

![运行效果](../images/getting_started_step_5.gif)