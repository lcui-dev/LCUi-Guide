## 原型

在《[快速上手](getting_started/README.md)》章节中有讲到用 `LCUIWidget_New()` 函数创建部件，给定类型名称就能
够创建该类型的部件，实际上，这个函数会先找到与该名称对应的部件原型，然后基于这
个原型来创建部件。

图形界面会需要各种各样的部件以丰富用户交互体验，例如：文本框、滚动条、
进度条、单选框等类型的部件，这些部件都会有各自的数据和方法（函数），但在
LCUI 中所有类型的部件都有共同的方法，而这些方法都定义在一个结构体中，代码如下：

``` c
typedef struct LCUI_WidgetPrototypeRec_ *LCUI_WidgetPrototype;
typedef const struct LCUI_WidgetPrototypeRec_ *LCUI_WidgetPrototypeC;

typedef void( *LCUI_WidgetFunction )(LCUI_Widget);
typedef void( *LCUI_WidgetResizer )(LCUI_Widget, int*, int*);
typedef void( *LCUI_WidgetAttrSetter )(LCUI_Widget, const char*, const char*);
typedef void( *LCUI_WidgetTextSetter )(LCUI_Widget, const char*);
typedef void( *LCUI_WidgetPainter )(LCUI_Widget, LCUI_PaintContext);

/** 部件原型数据结构 */
typedef struct LCUI_WidgetPrototypeRec_ {
        char *name;                             /**< 名称 */
        LCUI_WidgetFunction init;               /**< 构造函数  */
        LCUI_WidgetFunction destroy;            /**< 析构函数 */
        LCUI_WidgetFunction update;             /**< 样式处理函数 */
        LCUI_WidgetFunction runtask;            /**< 自定义任务处理函数 */
        LCUI_WidgetAttrSetter setattr;          /**< 属性设置函数 */
        LCUI_WidgetTextSetter settext;          /**< 文本内容设置函数 */
        LCUI_WidgetResizer autosize;            /**< 内容尺寸计算函数 */
        LCUI_WidgetPainter paint;               /**< 绘制函数 */
        LCUI_WidgetPrototype proto;             /**< 父级原型 */
} LCUI_WidgetPrototypeRec;
```

以上代码中列出的方法是 LCUI 在处理部件时都会用到的，以下是简单的说明。

** init **

`LCUIWidget_New()` 函数在找到原型后，会调用 `init()` 函数按照该类型的部件预设
的方法初始化部件，不同类型的部件都会有自己的数据以及其它相关的设置，这些数据
可以在 `init()` 函数中初始化。

** destroy **

在 LCUI 销毁部件时会调用 `destroy()` 函数，通常这个函数主要负责销毁部件的私有
数据、解除相关设置等任务。

** update **

对于某些部件而言，预置的 CSS 样式属性无法满足需求，会需要用到扩展样式，而这些
扩展样式的处理方法是 LCUI 无法知道的，因此，LCUI 在处理完预置的样式属性后，会
将样式处理任务交给 `update()` 函数。

** runtask **

在 LCUI 处理完部件预设的一些任务后，会调用 `runtask()` 去处理部件自己设定的一
些任务。

** setattr **

`setattr()` 主要用于 XML 文档解析功能，当解析到 XML 文档元素的属性时，会调用该
函数让部件应用这些属性。

** settext **

`settext()` 也同样是用于 XML 文档解析功能，当解析到 XML 文档元素内的文本结点
时，会调用该函数让部件处理文本内容。

** autosize **

在 LCUI 计算部件宽高时，如果部件的宽高被设置为 auto，则会调用 `autosize()` 获
取该部件的尺寸，如果未设置 `autosize`，LCUI 会按照默认的方式计算部件的宽高。
通常像文本显示（TextView）这类有自己内容的部件会需要这个函数来调整自身宽高以
适应文本内容。

** paint **

在 LCUI 按设定的样式绘制好部件后，会调用 `paint()` 绘制部件自己的内容。

** proto **

父级原型，用于访问父级原型的方法，这个属性不需要手动设置。

### 基本用法

创建部件原型需要用到 `LCUIWidget_NewPrototype()` 函数，函数原型如下：

``` c
LCUI_WidgetPrototype LCUIWidget_NewPrototype( const char *name,
                                              const char *parent_name );
```

需要的参数有两个：原型的名称、继承的父级原型的名称，创建完后会返回部件原型，如果
已经存在同名的部件原型或者原型添加失败，则会返回 NULL。以下代码展示了部件原型的常规创建方法，如需详尽的参考代码请查阅 LCUI 预置部件的代码（例如：/src/gui/widget/textview.c）。

``` c
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/gui/widget.h>

static struct MyWidgetModule {
        LCUI_WidgetPrototype prototype;
        // 其它用得到的数据
        // xxxx
        // ...
} self;

static void MyWidget_OnInit( LCUI_Widget w )
{
        // 初始化一些数据
}

static void MyWidget_OnDestroy( LCUI_Widget w )
{
        // 释放相关数据
}

static void MyWidget_UpdateStyle( LCUI_Widget w )
{
        // 处理扩展的样式属性
}

static void MyWidget_AutoSize( LCUI_Widget w, int *width, int *height )
{
        // 根据自身的内容，计算合适的尺寸
}

static void MyWidget_OnTask( LCUI_Widget w )
{
        // 处理积累的任务
}

static void MyWidget_OnPaint( LCUI_Widget w, LCUI_PaintContext paint )
{
        // 利用 paint 上下文绘制自己的内容
}

static void MyWidget_OnParseText( LCUI_Widget w, const char *text )
{
        // 处理 XML 解析器传来的文本内容
}

void LCUIWidget_AddMyWidget( void )
{
        int i;
        self.prototype = LCUIWidget_NewPrototype( "mywidget", NULL );
        self.prototype->init = MyWidget_OnInit;
        self.prototype->paint = MyWidget_OnPaint;
        self.prototype->destroy = MyWidget_OnDestroy;
        self.prototype->autosize = MyWidget_AutoSize;
        self.prototype->update = MyWidget_UpdateStyle;
        self.prototype->settext = MyWidget_OnParseText;
        self.prototype->runtask = MyWidget_OnTask;
        // 如果需要用到全局的数据的话
        // self.xxxx = ???
        // ...
}
```
