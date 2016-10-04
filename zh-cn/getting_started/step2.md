
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
        LCUI_Color bdcolor = RGB( 0, 0, 0 );
        LCUI_Color bgcolor = RGB( 245, 245, 245 );

        LCUI_Init();
        root = LCUIWidget_GetRoot();
        txt = LCUIWidget_New( "textview" );
        Widget_SetMargin( txt, 25, 0, 0, 25 );
        Widget_SetPadding( txt, 25, 0, 0, 0 );
        Widget_SetBorder( txt, 1, SV_SOLID, bdcolor );
        Widget_SetStyle( txt, key_background_color, bgcolor, color );
        TextView_SetText( txt, "[size=18px]Hello, World![/size]" );
        TextView_SetTextAlign( txt, SV_CENTER );
        Widget_Resize( txt, 200, 100 );
        Widget_Append( root, txt );
        return LCUI_Main();
}
```

在原有的代码上补充了几行代码，运行效果如下：

![运行效果](../../images//getting_started_step_2.png)

为了缩短后面代码的宽度，预先定义了 bdcolor 和 bgcolor 两个变量，分别保存边框色和背景色。

在创建好部件后，调用 `Widget_SetMargin()`、`Widget_SetPadding()` 和 `Widget_SetBorder()` 这三个函数分别设置 txt 部件的外间距、内间距和边框，内间距和外间距有上、右、下、左这四个，`Widget_SetMargin()` 和 `Widget_SetPadding()` 函数的后面四个参数也是对应这个顺序的。`Widget_SetBorder()` 函数的后三个参数分别表示边框的大小、风格、颜色，目前边框风格只支持实线（solid），与之对应的值为 SV_SOLID。

之后调用 `Widget_SetStyle()` 函数设置 txt 部件的背景色。`Widget_SetStyle()` 是一个宏，主要用于简化部件样式表的修改操作，它的第二个参数是样式属性的编号，这些编号也就是样式属性在样式表中的位置，为方便记忆，已经将它们定义成以 key_ 的开头的枚举，你可以在 gui/css_library.h 中找到它们的定义，为方便识别，并没有采用和普通枚举那样的全大写字母的命名方式。第三个和第四个参数分别是属性值和属性值的类型，这里就不做过多的说明了。

在为 txt 部件设置的文本中包含了 `[size=18px]`，它表示的是将字体的大小设置为 18 像素。TextView 部件默认支持样式标签，但当前支持的样式标签只有 `[size]` 和 `[color]`，你可以直接在文本内使用它们来设置简单的样式。

`TextView_SetTextAlign()` 函数可以为 TextView 部件设置文本对齐方式，支持的对齐方式有三种：靠左（SV_LEFT）、居中（SV_CENTER）、靠右（SV_RIGHT），SV 是样式值（Style Value）的缩写。

`Widget_Resize()` 函数用于调整部件的尺寸，这里将部件的宽高分别为 200 像素和 100 像素。

看了上面的代码你可能会想：为什么修改部件的坐标需要设置外间距？不是直接设置 xy 坐标就可以了吗？

LCUI 有提供 `Widget_Move()` 函数，用于修改部件的坐标，但对于默认定位方式的部件来说，这个函数并没有任何效果。LCUI 的布局方式和浏览器中的网页布局类似，部件在默认的定位方式下，能够影响坐标的只有外间距，除非更改该部件的定位方式，具体内容在后面的章节中会介绍。
