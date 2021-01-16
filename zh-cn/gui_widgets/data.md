# 私有数据

一个部件的各种功能的实现都会用到数据，例如：文本编辑框，它会保存当前编辑的文本 内容，调用相关函数可以对这个文本内容进行读写操作，在绘制时也会需要用到这些文本 内容以在屏幕上绘制出相应的文字。

部件私有数据的操作函数有以下两个：

```c
void *Widget_AddData( LCUI_Widget widget, LCUI_WidgetPrototype proto, size_t data_size );
void *Widget_GetData( LCUI_Widget widget, LCUI_WidgetPrototype proto );
```

从以上代码中可以看出部件私有数据有添加和获取这两种方法，私有数据是与部件原型 绑定的，添加时需要指定具体的内存占用大小。添加后，可以调用 `Widget_GetData()` 函数获取私有数据，这个函数也同样需要指定原型。

以下是这两个函数的基本用法示例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/gui/widget.h>

/** 部件私有数据的结构 */
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

static void MyWidget_OnInit( LCUI_Widget w )
{
        MyWidget data;
        const size_t size = sizeof( MyWidgetRec );
        data = Widget_AddData( w, self.prototype, size );
        // 初始化私有数据
        data->a = 123;
        data->b = 'b';
        data->c = 3.1415926;
        data->str = malloc( 256 * sizeof(char) );
        strcpy( data->str, "this is my widget." );
        printf( "my widget is inited.\n" );
}

static void MyWidget_OnDestroy( LCUI_Widget w )
{
        MyWidget data = Widget_GetData( w, self.prototype );
        // 释放私有数据占用的内存资源
        free( data->str );
        printf( "my widget is destroied.\n" );
}

void LCUIWidget_AddMyWidget( void )
{
        int i;
        self.prototype = LCUIWidget_NewPrototype( "mywidget", NULL );
        self.prototype->init = MyWidget_OnInit;
        self.prototype->destroy = MyWidget_OnDestroy;
        // 如果全局用得到的数据的话
        // self.xxxx = ???
}
```

