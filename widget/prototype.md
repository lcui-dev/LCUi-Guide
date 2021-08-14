---
description: 介绍组件的原型相关概念和用法。
---

# 原型

在开发 UI 的时候，我们会用到各种各样的组件来丰富 UI 的交互体验，例如：文本输入框、滚动条、 进度条、单选框等，这些组件虽然有着不同的数据、方法、视觉效果和交互方式，但都能在 LCUI 中以相同的规则来工作，而这个规则就是原型。

### 原型方法

原型中记录了 LCUI 在更新和渲染组件时需要用到的方法，通过将这些方法与自定义函数绑定，可实现对组件的扩展。关于原型的定义，你可以在 [include/LCUI/gui/widget\_base.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/gui/widget_base.h#L140-L164) 中找到。

接下来让我们深入了解原型中的各个方法的用途。

 **init** 

`LCUIWidget_New()` 函数在找到原型后，会调用 `init()` 函数按照该类型的组件预设的方法初始化组件实例，不同类型的组件都会有自己的数据以及其它相关的设置，这些数据可以在 `init()` 函数中初始化。

 **destroy** 

在组件被销毁时会调用 `destroy()` 函数，通常这个函数主要负责销毁组件的私有数据、解除相关设置等。

 **update** 

对于某些组件而言，预置的 CSS 属性无法满足需求，会需要用到扩展 CSS 属性，而这些扩展 CSS 属性的处理方法是 LCUI 无法知道的，因此，LCUI 在处理完预置的 CSS 属性后，会将剩余的 CSS 属性交给 `update()` 函数去处理。

 **runtask** 

在 LCUI 处理完组件预设的一些任务后，会调用 `runtask()` 去处理组件自己设定的一 些任务。

 **setattr** 

在 `Widget_SetAttribute()` 设置完属性后会调用它。

 **settext** 

当解析到 XML 文档元素内的文本结点时，会调用该函数让组件处理文本内容。

 **autosize** 

在计算组件宽高时，如果组件的宽高被设置为 auto，则会调用 `autosize()` 获取该组件的尺寸，如果未设置 `autosize`，LCUI 会按照默认的方式计算组件的宽高。通常像文本显示（TextView）这类有自己内容的组件会需要这个函数来调整自身宽高以 适应文本内容。

**resize**

在组件宽高更新时调用。

 **paint** 

在 LCUI 按设定的样式绘制好组件后，会调用 `paint()` 绘制组件自己的内容。

 **proto** 

父级原型，用于访问父级原型的方法，这个属性不需要手动设置。

### 创建原型

创建原型需要用到 `LCUIWidget_NewPrototype()` 函数：

```c
LCUI_WidgetPrototype LCUIWidget_NewPrototype(const char *name, const char *parent_name);
```

它的参数有两个：原型的名称、继承的父级原型的名称，创建完后会返回原型，如果已经存在同名的原型或者原型添加失败，则会返回 NULL。

以下代码展示了原型的创建方法，如需详尽的参考代码请查阅 LCUI 预置组件的代码（例如：[src/gui/widget/textview.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/gui/widget/textview.c)）。

```c
#include <LCUI.h>
#include <LCUI/gui/widget.h>

static struct MyWidgetModule {
        LCUI_WidgetPrototype prototype;
        // 其它用得到的数据
        // xxxx
        // ...
} my_widget;

static void MyWidget_OnInit(LCUI_Widget w)
{
        // 初始化一些数据
}

static void MyWidget_OnDestroy(LCUI_Widget w)
{
        // 释放相关数据
}

static void MyWidget_UpdateStyle(LCUI_Widget w)
{
        // 处理扩展的样式属性
}

static void MyWidget_AutoSize(LCUI_Widget w, float *width, float *height)
{
        // 根据自身的内容，计算合适的尺寸
}

static void MyWidget_OnTask(LCUI_Widget w)
{
        // 处理积累的任务
}

static void MyWidget_OnPaint(LCUI_Widget w, LCUI_PaintContext paint)
{
        // 利用 paint 上下文绘制自己的内容
}

static void MyWidget_OnParseText(LCUI_Widget w, const char *text)
{
        // 处理 XML 解析器传来的文本内容
}

void LCUIWidget_AddMyWidget(void)
{
        int i;
        my_widget.prototype = LCUIWidget_NewPrototype("my-widget", NULL);
        my_widget.prototype->init = MyWidget_OnInit;
        my_widget.prototype->paint = MyWidget_OnPaint;
        my_widget.prototype->destroy = MyWidget_OnDestroy;
        my_widget.prototype->autosize = MyWidget_AutoSize;
        my_widget.prototype->update = MyWidget_UpdateStyle;
        my_widget.prototype->settext = MyWidget_OnParseText;
        my_widget.prototype->runtask = MyWidget_OnTask;
        // 如果需要用到全局的数据的话
        // my_widget.xxxx = ???
        // ...
}
```

### 使用私有数据

当组件的功能变多变复杂的时候，如果仅靠函数内的局部变量难以实现多个功能之间的数据共享的话，那么就会需要一个能在整个组件生命周期内有效的空间来存放数据，例如：文本编辑框，它会保存当前编辑的文本内容，调用相关函数可以对这个文本内容进行读写操作，在绘制时也会需要用到这些文本内容以在屏幕上绘制出相应的文字。

组件私有数据的操作函数有以下两个：

```c
void *Widget_AddData(LCUI_Widget widget, LCUI_WidgetPrototype proto, size_t data_size);
void *Widget_GetData(LCUI_Widget widget, LCUI_WidgetPrototype proto);
```

从以上代码中可以看出组件私有数据有添加和获取这两种方法，私有数据是与组件原型绑定的，添加时需要指定具体的内存占用大小。添加后，可以调用 `Widget_GetData()` 函数获取私有数据，这个函数也同样需要指定原型。

以下是这两个函数的基本用法示例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <LCUI.h>
#include <LCUI/gui/widget.h>

/** 组件私有数据的结构 */
typedef struct MyWidgetRec_ {
    int a;
    char b;
    double c;
    char *str;
} MyWidgetRec, *MyWidget;

static struct MyWidgetModule {
        LCUI_WidgetPrototype prototype;
        // 其它用得到的数据
        // xxxx
        // ...
} self;

static void MyWidget_OnInit(LCUI_Widget w)
{
        MyWidget data;
        const size_t size = sizeof(MyWidgetRec);
        data = Widget_AddData(w, self.prototype, size);
        // 初始化私有数据
        data->a = 123;
        data->b = 'b';
        data->c = 3.1415926;
        data->str = malloc(256 * sizeof(char));
        strcpy(data->str, "this is my widget.");
        printf("my widget is inited.\n");
}

static void MyWidget_OnDestroy(LCUI_Widget w)
{
        MyWidget data = Widget_GetData(w, self.prototype);
        // 释放私有数据占用的内存资源
        free(data->str);
        printf("my widget is destroied.\n");
}

void LCUIWidget_AddMyWidget(void)
{
        int i;
        self.prototype = LCUIWidget_NewPrototype("mywidget", NULL);
        self.prototype->init = MyWidget_OnInit;
        self.prototype->destroy = MyWidget_OnDestroy;
        // 如果全局用得到的数据的话
        // self.xxxx = ???
}
```

###  使用 CLI 添加组件

LCUI CLI 提供了组件生成器，你可以基于它生成的组件模板代码快速开发组件。

```text
lcui generate widget 你的组件名
```

### 继承

在某一个组件的功能不够用的时候，我们会想基于它扩展一些新功能，如果直接改代码的话会让它容易变得更复杂，而重新写一个成本太高，所以，这个时候我们可以使用原型的“继承”功能来创建一个组件的扩展版本， 即能保留原组件的功能，又能使用新加的功能。

以 TextView 组件为例，假设有这么个需求：能够支持设置网页链接，在组件被点击时调用浏览器打开这个链接，网页链接由 `href` 属性提供，以下是示例代码：

```c
#include <string.h>
#include <stdlib.h>
#include <LCUI.h>
#include <LCUI/gui/widget.h>

typedef struct LinkRec_ {
        char *href;
} LinkedRec, *Link;

LCUI_WidgetPrototype prototype;

static void Link_OnClick(LCUI_Widget w, LCUI_WidgetEvent e, void *arg)
{
        Link link = Widget_GetData(w, prototype);
        if (link->href) {
                // 调用浏览器打开链接
                // ...
        }
}

static void Link_OnInit(LCUI_Widget w)
{
        const size_t size = sizeof(LinkRec);
        Link link = Widget_AddData(w, prototype, size);

        link->href = NULL;
        Widget_BindEvent(w, "click", Link_OnClick, NULL, NULL);
        // 调用父级原型的 init() 方法，继承父级现有的功能
        // 在其它语言中（例如 JavaScript），这段代码类似于在构造函数中调用 super()
        prototype->proto->init(w);
}

static void Link_OnDestroy(LCUI_Widget w)
{
        Link link = Widget_GetData(w, prototype);
        free(link->href);
        prototype->proto->destroy(w);
}

void Link_SetHref(LCUI_Widget w, const char *href)
{
        Link link = Widget_GetData(w, prototype);
        if (link->href) {
                free(link->href);
                link->href = NULL;
        }
        if (href) {
                size_t len = strlen(href) + 1;
                link->href = malloc(len * sizeof(char));
                strcpy(link->href, href);
        }
}

static void Link_OnSetAttr(LCUI_Widget w, const char *name, const char *value)
{
        // 响应 href 属性值变化
        if (strcmp(name, "href") == 0) {
                Link_SetHref( w, value );
        }
}

void LCUIWidget_AddLink( void )
{
        // 创建一个名为 link 的原型，继承自 textview
        prototype = LCUIWidget_NewPrototype("link", "textview");
        prototype->init = Link_OnInit;
        prototype->destroy = Link_OnDestroy;
        prototype->setattr = Link_OnSetAttr;
}
```

以上代码创建了一个名为 link 的原型，接下来将展示如何使用它：

```c
...

LCUIWidget_AddLink();

...

LCUI_Widget link = LCUIWidget_New("link");
Link_SetHref(link, "https://www.example.com");
```

```markup
<?xml version="1.0" encoding="UTF-8" ?>
<lcui-app>
  <ui>
    <link href="https://www.example.com">点击这里</link>
  </ui>
</lcui-app>
```

LCUI 的预置组件 Anchor 是这个示例组件的完整实现，如需了解更多，可查看文件：[src/gui/widget/anchor.c](https://github.com/lc-soft/LCUI/blob/master/src/gui/widget/anchor.c#L206)

