---
description: 介绍组件属性的分类及用途。
---

# 属性

### 基础属性

基础属性是每个组件都有的属性，可通过组件指针直接访问到它们。你可以在 [include/LCUI/gui/widget\_base.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/gui/widget_base.h#L253-L336) 文件中找到它们的定义。

基础属性分为只读和可写两种：

* 只读的基础属性是经过计算后的结果，手动修改它们的值是没有意义的，因为它们的值会在下次计算后更新，例如：x、y、width、height 这些几何属性，它们是组件的样式经过布局引擎计算后的结果。
* 可写的基础属性能影响组件的功能和渲染效果，例如：修改 `disabled` 和 `event_blocked` 能控制组件的事件响应行为， 修改 `custom_style` 能覆盖组件原本的样式。这些属性大都有相关的函数来负责修改它们，我们只需要调用它们即可。

接下来让我们通过示例代码来了解一些常用的基础属性。

```c
LCUI_Widget w = LCUIWidget_New(NULL);

// 几何属性的读取
printf("coordinate: (%f, %f)\n", w->x, w->y);
printf("size: (%f, %f)\n", w->width, w->height);

// 类的增删查
Widget_AddClass(w, "button disabled");
if (Widget_HasClass(w, "disabled")) {
    Widget_RemoveClass(w, "disabled");
}

// id
Widget_SetId(w, "btn-submit");
printf("is same widget? %d\n", w == LCUIWidget_GetById("btn-submit"));

// 状态/伪类
if (Widget_HasStatus(w, "focus")) {
    printf("widget has focus");
}

// 打印最终样式
LCUI_PrintStyleSheet(w->style);
// 打印匹配的样式
LCUI_PrintStyleSheet(w->inherited_style);
// 设置自定义样式
Widget_SetStyle(w, key_margin_left, 10, px);
Widget_SetStyleString(w, "margin-left", "10px");
// 获取已计算的样式
if (w->computed_style.visible) {
    Widget_Hide(w);
} else {
    Widget_Show(w);
}
```

### 扩展属性

扩展属性是可以任意添加、修改和删除的属性，常用于保存自定义数据，或是让特定组件支持用 xml 语言来设置相关功能。它们主要来自于 xml 文档和手动调用 `Widget_SetAttribute()`  函数，且都以字符串的形式保存在一个以属性名为索引键的哈希表中。

扩展属性的操作例子：

```c
LCUI_Widget w = LCUIWidget_New(NULL);

Widget_SetAttribute(w, "attr", "value");
printf("%s\n", Widget_GetAttribute(w, "attr"));
```

如果你想让你的组件支持响应 xml 文档中设置属性，举个例子，假设你有个 Timeago 组件提供这些方法：

```c
Timeago_SetDate(w, "2021-01-01T08:00:00.000Z");
Timeago_SetAutoUpdate(w, 60);
Timeago_SetLocale(w, "zh-CN");
```

那么，首先你需要添加一个 `Timeago_SetAttribute()` 函数来集中处理属性：

```c
void MyWidget_SetAttribute(
    LCUI_Widget w,
    const char *name,
    const char *val
)
{
    if (strcmp(name, "date") === 0) {
        ...
    }
    ...
}
```

然后，将它与组件原型上 `setattr` 方法进行绑定：

```c
my_widget_proto->setattr = MyWidget_SetAttribute;
```

这样修改后，在 XML 文档中就能这样使用 Timeago 组件：

```markup
<timeago date="2021-01-01T08:00:00.000Z" auto-update="60" locale="zh-CN" />
```

而且还能支持用 `Widget_SetAttribute()` 函数设置属性：

```c
Widget_SetAttribute(w, "date", "2021-01-01T08:00:00.000Z");
Widget_SetAttribute(w, "auto-update", "60");
Widget_SetAttribute(w, "locale", "zh-CN");
```

### 待办事项

**给 `LCUI_WidgetRec_` 结构体中的只读成员加上只读注释**

修改  [include/LCUI/gui/widget\_base.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/gui/widget_base.h#L253-L336) 中的 `LCUI_WidgetRec_` 结构体定义，给只读属性加上 `(Readonly)` 之类的注释。

