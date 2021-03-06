---
description: CSS 数据库的增删改查操作和相关概念介绍。
---

# CSS 数据库

### 简单的例子

```c
#include <LCUI.h>
#include <LCUI/gui/css_library.h>

void PutStyleSheet(void)
{
    LCUI_Color black = RGB(0, 0, 0);
    LCUI_Selector selector = Selector(".toolbar .button");
    LCUI_StyleSheet stylesheet = StyleSheet();

    SetStyle(stylesheet, key_color, black, color);
    LCUI_PutStyleSheet(selector, stylesheet, NULL);
    StyleSheet_Delete(stylesheet);
    Selector_Delete(selector);
}

void GetStyleSheet(void)
{
    LCUI_Selector selector = Selector(".button");
    LCUI_StyleSheet stylesheet = StyleSheet();
    LCUI_GetStyleSheet(selector, stylesheet);
    LCUI_PrintStyleSheet(stylesheet);
    StyleSheet_Delete(stylesheet);
    Selector_Delete(selector);
}

int main(void)
{
    LCUI_InitCSSLibrary();
    PutStyleSheet();
    GetStyleSheet();
    LCUI_FreeCSSLibrary();
    return 0;
}
```



