---
description: CSS 解析器的流程和解析器的添加方法介绍。
---

# CSS 解析器

### 简单的例子

这个例子展示了如何从字符串中加载 CSS 样式：

```c
#include <LCUI.h>
#include <LCUI/gui/css_library.h>
#include <LCUI/gui/css_parser.h>

int main(void)
{
    LCUI_Selector selector = Selector(".button");
    LCUI_StyleSheet stylesheet = StyleSheet();

    LCUI_InitCSSLibrary();
    LCUI_InitCSSParser();
    LCUI_LoadCSSString(".toolbar .button { color: #000 }");
    LCUI_GetStyleSheet(selector, stylesheet);
    LCUI_PrintStyleSheet(stylesheet);
    StyleSheet_Delete(stylesheet);
    Selector_Delete(selector);
    LCUI_FreeCSSParser();
    LCUI_FreeCSSLibrary();
    return 0;
}
```

`LCUI_LoadCSSString()` 函数用于从字符串中加载 CSS 样式到数据库中，它内部会调用一系列的解析器对字符串进行解析，最终转换成便于操作的数据结构。

### 工作流程

CSS 解析器在解析时会将解析目标分为：无、注释、规则名称、规则数据、选择器、CSS 属性名 和 CSS 属性值，例如以下 CSS 代码：

```css
@import "common.css";

/* button style */
.btn {
  font-size: 16px;
}
```

CSS 解析器会将它识别为以下目标进行解析：

```css
@[rule-name] [rule-data];

[comment]
[selector] {
    [css-property-name]: [css-property-value];
}
```

每个解析目标都有对应的解析器，它们的解析行为大致是这样的：

* **无：**寻找解析目标，遍历字符串，根据当前读取到的字符来决定切换到哪种解析目标，例如读取到的字符是 `@` ，则切换解析目标为规则名称。
* **规则名称：**存储每个字符直到读取的字符是空白符为止，解析完后切换解析目标为规则数据。
* **规则数据：**如果有与规则名称对应的规则解析器，则调用它解析，否则忽略。
* **选择器：**存储每个字符直到解析到左括号 `{` 为止，转换字符串为选择器，然后切换解析目标为 CSS 属性名。
* **CSS 属性名：**存储每个字符直到冒号 `:`为止，将存储的字符串作为属性名，并记录对应的 CSS 属性解析器，然后切换解析目标为 CSS 属性值。
* **CSS 属性值：**存储每个字符直到右括号 `}` 或分号 `;` 为止，如果存在对应的 CSS 属性解析器，则调用它更新当前解析上下文中的样式表。
* **注释：**以上解析器在解析到斜杆 `/` 时都会切换解析目标为注释，当解析完注释后再恢复解析目标为上个解析目标。

### 添加 at 规则解析器

目前预设的 at 规则有：`@font-face`、`@import` 和 `@media`，其中 `@media` 规则解析暂未实现，如果你想添加新的 at 规则解析器，则必须修改 CSS 解析器源码才能做到。

首先编辑[ include/LCUI/gui/css\_parser.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/gui/css_parser.h#L62) 头文件，在 `LCUI_CSSRule` 枚举的定义里追加你的新规则的枚举值（例如：`CSS_RULE_MY_RULE`）。

然后定义你的规则解析上下文和解析器函数，例如：

```c
#define GetParserContext(CTX) (CTX)->rule.parsers[CSS_RULE_MY_RULE].data

typedef struct MyRuleParserContextRec_ {
    // Define your data
    // ...
} MyRuleParserContextRec, *MyRuleParserContext;

static int MyRuleParser_Begin(LCUI_CSSParserContext ctx)
{
    // Initialize your parser context
    // ...
    // ctx->
    ctx->rule.state = /* your parser initial state */;
    return 0;
}

static void MyRuleParser_End(LCUI_CSSParserContext ctx)
{
    MyRuleParserContext data = GetParserContext(ctx);

    // Destroy your data
    // ...
    // free(data->xxx);
}

int CSSParser_InitMyRuleParser(LCUI_CSSParserContext ctx)
{
    LCUI_CSSRuleParser parser;
    FontFaceParserContext data;

    parser = &ctx->rule.parsers[CSS_RULE_MY_RULE];
    data = NEW(MyRuleParserContextRec, 1);
    if (!data) {
        return -ENOMEM;
    }
    // Initialize your data
    // ...
    // data->xxxx = xxxx;
    parser->data = data;
    parser->parse = MyRuleParser_Parse;
    parser->begin = MyRuleParser_Begin;
    strcpy(parser->name, "my-rule");
    return 0;
}


void CSSParser_FreeMyRuleParser(LCUI_CSSParserContext ctx)
{
    FontFaceParserContext data = GetParserContext(ctx);

    MyRuleParser_End(ctx);
    free(data);
    ctx->rule.parsers[CSS_RULE_MY_RULE].data = NULL;
}

```

定义好后，在 [src/gui/css\_parser.c](https://github.com/lc-soft/LCUI/blob/master/src/gui/css_parser.c) 的中追加调用你的解析器的初始化和销毁函数：

```c
LCUI_CSSParserContext CSSParser_Begin(size_t buffer_size, const char *space)
{
    ...
    
    CSSParser_InitMyRuleParser(ctx);
    return ctx;
}

void CSSParser_End(LCUI_CSSParserContext ctx)
{
    ...
    CSSParser_FreeMyRuleParser(ctx);
    ...
    free(ctx->buffer);
    free(ctx);
}
```

如需了解更详细的规则解析器实现方式，可参考 `@font-face` 规则解析器的源码：[src/gui/css\_rule\_font\_face.c](https://github.com/lc-soft/LCUI/blob/master/src/gui/css_rule_font_face.c)

### 添加 CSS 属性解析器

在添加新的 CSS 属性解析器前，我们需要先用 `LCUI_AddCssPropertyName()` 注册自定义属性名称，拿到该属性的标识号，也就是该属性在样式表中的下标，然后调用`LCUI_AddCSSPropertyParser()` 函数注册我们的解析器的自定义标识号、解析函数和名称。

接下来我们从下面这个简单的示例来了解如何添加 CSS 属性解析器：

```c
#define GetMYPropertyKey(CTX) keys[(CTX)->parser->key]
#define SetMyProperty(CTX, S) \
	CSSStyleParser_SetCSSProperty(CTX, GetMYPropertyKey(CTX), S);

enum CSSMyPropertyKey {
    key_my_css_property,
    // define other css property keys
    // key_xxx
    // ...
    MY_PROPERTY_KEY_TOTAL
};

static int keys[MY_PROPERTY_KEY_TOTAL];

static int OnParseMyCSSProperty(LCUI_CSSParserStyleContext ctx, const char *str)
{
    LCUI_StyleRec style;

    // parse data from str
    // ...
    
    CSSStyleParser_SetCSSProperty(ctx, keys[ctx->parser-key], style);
    return 0;
}

void InitMyCSSProperties(void)
{
    LCUI_CSSPropertyParserRec my_parser = {
        key_my_css_property,
        "my-css-property",
        OnParseMyCSSProperty
    };
    
    keys[my_parser->key] = LCUI_AddCSSPropertyName(my_parser->name);
    LCUI_AddCSSPropertyParser(my_parser);
}
```

如需了解更详细的 CSS 属性解析器实现方式，可参考字体样式解析器的源码：[src/gui/css\_fontstyle.c](https://github.com/lc-soft/LCUI/blob/master/src/gui/css_fontstyle.c)

### 待办事项

**完善 CSS 解析器的错误处理**

CSS 解析器是以 CSS 代码完全正确为前提而工作的，一旦加载的 CSS 代码存在语法错误或不支持的语法时，可能会出现怪异的解析行为和解析结果，为解决这一问题，我们应该补充错误判断并输出相关错误信息，以便开发者快速定位问题。



