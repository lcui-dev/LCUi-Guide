# CSS 解析器

### 简单的例子

```c
#include <LCUI.h>
#include <LCUI/gui/css_library.h>
#include <LCUI/gui/css_parser.h>

int main(void)
{
    LCUI_Selector selector = Selector(".button");
    LCUI_StyleSheet stylesheet = StyleSheet();

    LCUI_InitCSSLibrary();
    LCUI_LoadCSSString(".toolbar .button { color: #000 }");
    LCUI_GetStyleSheet(selector, stylesheet);
    LCUI_PrintStyleSheet(stylesheet);
    StyleSheet_Delete(stylesheet);
    Selector_Delete(selector);
    LCUI_FreeCSSLibrary();
    return 0;
}
```

### 添加自定义解析器



