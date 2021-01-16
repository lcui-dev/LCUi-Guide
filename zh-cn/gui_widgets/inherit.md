# 继承

有时候会发现某个部件的功能不够用，想扩展一些新功能，但不想直接修改它的代码，也 不想重写一个新部件，这个时候可以使用原型的“继承”功能来创建一个该部件的扩展版本， 即能保留原部件的功能，又能使用新加的功能。

以 textview 部件为例，现在有这样的功能需求：能够支持设置网址链接，在被点击时会调用浏览器打 开这个链接，网址链接能靠 `href` 属性来设置，以下是示例代码：

```c
#include <string.h>
#include <stdlib.h>
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/gui/widget.h>

typedef struct LinkRec_ {
        char *href;
} LinkedRec, *Link;

LCUI_WidgetPrototype prototype;

static void Link_OnClick( LCUI_Widget w, LCUI_WidgetEvent e, void *arg )
{
        Link link = Widget_GetData( w, prototype );
        if( link->href ) {
                // 调用浏览器打开链接
                // ...
        }
}

static void Link_OnInit( LCUI_Widget w )
{
        Link link;
        const size_t size = sizeof( LinkRec );
        link = Widget_AddData( w, prototype, size );

        link->href = NULL;
        Widget_BindEvent( w, "click", Link_OnClick, NULL, NULL );
        // 调用父级原型的 init() 方法，继承父级现有的功能
        prototype->proto->init( w );
}

static void Link_OnDestroy( LCUI_Widget w )
{
        Link link = Widget_GetData( w, prototype );
        free( link->href );
        prototype->proto->destroy( w );
}

void Link_SetHref( LCUI_Widget w, const char *href )
{
        Link link = Widget_GetData( w, prototype );
        if( link->href ) {
                free( link->href );
                link->href = NULL;
        }
        if( href ) {
                size_t len = strlen( href ) + 1;
                link->href = malloc( len * sizeof( char ) );
                strcpy( link->href, href );
        }
}

static void Link_OnSetAttr( LCUI_Widget w, const char *name, const char *value )
{
        // 当 XML 解析器解析到的元素属性是 href 时
        if( strcmp( name, "href" ) == 0 ) {
                Link_SetHref( w, value );
        }
}

void LCUIWidget_AddLink( void )
{
        // 创建一个名为 link 的部件原型，继承自 textview
        prototype = LCUIWidget_NewPrototype( "link", "textview" );
        prototype->init = Link_OnInit;
        prototype->destroy = Link_OnDestroy;
        prototype->setattr = Link_OnSetAttr;
}
```

以上代码创建了一个名为 link 的部件原型，接下来将展示如何使用它：

```c
// ...
LCUI_Widget link;
LCUIWidget_AddLink();
// ...
link = LCUIWidget_New( "link" );
Link_SetHref( link, "https://www.example.com" );
// ...
```

```markup
<?xml version="1.0" encoding="UTF-8" ?>
<lcui-app>
  <ui>
    <widget type="link" href="https://www.example.com">点击这里</widget>
  </ui>
</lcui-app>
```

