---
description: 介绍组件的样式表操作方法和相关该概念。
---

# 样式

每个组件都有自定义样式、已匹配样式、最终样式和已计算样式共四张样式表，其中已计算样式由最终样式计算而来，而最终样式则是已匹配样式与自定义样式合并而来的。

当组件的自定义样式、状态、结构以及样式库有改动的时候会触发样式计算，计算过程是先从样式库中查询与组件匹配的样式表，然后合并为一张样式表作为组件的已匹配样式，之后将已匹配样式与自定义样式合并成一张样式表作为最终样式，最后将最终样式转换为已计算样式，以供布局引擎和渲染引擎使用。

{% hint style="warning" %}
除自定义样式外，其它样式表都是只读的，你不应该修改它们。
{% endhint %}

### 已匹配样式

已匹配样式是样式库中所有与组件匹配的样式表的合集，可通过组件的 `inherited_style` 成员访问它，它的生成过程是先在样式库中找到与组件匹配的选择器，然后再合并该选择器关联的样式表，如果组件有样式哈希值，则已匹配样式还会被缓存起来，以供其它同类组件使用，减少重复的样式匹配过程。

这张样式表仅在样式计算阶段用到，大多数情况下你用不到它。

### 自定义样式

自定义样式记录了该组件专属的样式，具有比已匹配样式更高的优先级，可通过组件的 `custom_style`  成员访问它，常见的使用场景是在实现动画、拖拽操作或是自定义布局时动态修改组件的坐标、宽高、透明度等样式。为了节省内存占用，它采用了链表结构来存储样式数据。

自定义样式的操作示例：

```c
Widget_SetStyle(w, key_width, 100, px);
Widget_SetStyleString(w, "width", "100px");

if (Widget_CheckStyleValid(w, key_width)) {
    Widget_UnsetStyle(w, key_width);
}
```

有些常用的样式操作可以改用更精简更具语义的函数代替，示例：

```c
Widget_Move(w, 10, 10);
Widget_Resize(w, 100, 200);
Widget_SetPadding(w, 10, 20, 10, 20);
Widget_SetMargin(w, 5, 5, 5, 5);
Widget_SetBorder(w, 1, SV_SOLID, RGB(200, 200, 200));
Widget_SetBoxShadow(w, 0, 0, 4, ARGB(30, 0, 0, 0));
Widget_Show(w);
Widget_Hide(w);
```

### 最终样式

最终样式是已匹配样式与自定义样式合并后的产物，它记录了组件用到的全部 CSS 属性，可通过组件的 `style` 成员访问它。出于数据的完整性和读写性能的考虑，它采用了内存占用较大的数组来存储样式数据。

最终样式是 `LCUI_StyleSheet` 类型的，你可以用样式表的辅助方法操作它：

```c
LCUI_Widget w = LCUIWidget_New(NULL);
LCUI_Style style = StyleSheet_GetStyle(w->style, key_width);

if (style.is_valid && style.type == LCUI_STYPE_PX) {
    printf("widget width: %fpx\n", style.val_px);
}
```

### 已计算样式

已计算样式是最终样式经计算后的产物，它记录了组件用到的全部 CSS 属性的实际值，可通过组件的 `computed_style` 成员访问它。相比最终样式，在访问它的属性时无需判断属性的有效性和属性值的类型，而且可供布局引擎和渲染引擎直接使用。

常用操作示例：

```c
// 给组件加上淡出动画
void FadeOutAnimation_OnFrame(LCUI_Widget w)
{
    float opacity = w->computed_style.opacity;
    
    if (w->computed_style.visible) {
        if (opacity > 0) {
            opacity -= 0.01;
        }
        if (opacity > 0) {
            Widget_SetOpacity(w, opacity);
        } else {
            Widget_Hide(w);
        }
    }
}

...

LCUI_SetInterval(5, FadeOutAnimation_OnFrame, w);

...
```

### 待办事项

**重新设计组件样式表的存储方式**

由于每个组件都要存储三张样式表，内存占用很大，尤其是组件数量达到上万数量级的时候，我们需要为样式表设计一个内存占用更低的存储方式。

**移除组件结构体中的已计算样式**

组件结构体中的 `computed_style` 成员通常只在更新和渲染组件时使用，大小为 328 字节，可以尝试移除它，让它只在更新和渲染组件前临时创建以节省内存占用。

****
