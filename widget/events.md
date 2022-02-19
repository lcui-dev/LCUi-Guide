---
description: 组件事件的相关概念和用法介绍。
---

# 事件

组件事件来源于 LCUI 的核心事件，当核心事件触发时，组件系统中的相关事件处理器会对其进行处理，然后转换成组件事件派发给对应的组件。

组件事件与核心事件的区别在于事件的种类和事件对象的内容，事件种类包括悬停、单击、双击、焦点等事件，而事件对象则包含了关联的组件和事件冒泡控制。

### 事件对象

首先，让我们看看组件事件对象在[ include/LCUI/gui/widget\_event.h](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/gui/widget\_event.h#L81-L95) 文件中的定义：

```c
typedef struct LCUI_WidgetEventRec_ {
        uint32_t type;
        void *data;
        LCUI_Widget target;
        LCUI_BOOL cancel_bubble;
        union {
                LCUI_WidgetMouseMotionEvent motion;
                LCUI_WidgetMouseButtonEvent button;
                LCUI_WidgetMouseWheelEvent wheel;
                LCUI_WidgetKeyboardEvent key;
                LCUI_WidgetTouchEvent touch;
                LCUI_WidgetTextInputEvent text;
        };
} LCUI_WidgetEventRec, *LCUI_WidgetEvent;
```

我们可以发现，组件事件对象与核心事件对象的结构相似，只是多了 `target` 和 `cacnel_buble` 成员：

* `target` 成员指向的是事件触发时的组件，当你想让多个组件在事件发生时执行某些操作而给它们设置相同的事件处理器时，`target` 非常有用，例如，有 16 个按钮，按钮被点击时会更改文本，那么在事件处理器中，`target` 指向的就是当前被点击的按钮，你只需要修改它的文本即可，无需用复杂的方式去获取它。
* &#x20;`cancel_bubble` 成员用于标识是否取消该事件的冒泡，将它赋值为 TRUE 时，该事件对象不会冒泡到父级组件。

### 事件冒泡

事件从触发它的组件向上级组件逐层传递的过程就是事件冒泡。

### 事件委托

利用事件冒泡机制，我们可以实现事件委托，也就是将子组件的事件委托给父组件处理，这种做法适用于需要为大量组件设置事件处理器的场景，能避免因设置大量事件处理器而导致的性能降低和内存占用增加的问题。

### 触控事件

触控事件目前仅在 Windows 系统中有效，事件对象的数据结构定义如下：

```c
typedef struct LCUI_TouchPointRec_ {
        int x;
        int y;
        int id;
        int state;
        LCUI_BOOL is_primary;
} LCUI_TouchPointRec, *LCUI_TouchPoint;

typedef struct LCUI_TouchEvent_ {
        int n_points;
        LCUI_TouchPoint points;
} LCUI_TouchEvent;

typedef LCUI_TouchEvent LCUI_WidgetTouchEvent;
```

触控事件的数据结构设计参考自 Windows API，只保留了主要的成员变量。LCUI 内部的触控事件和组件的触控事件是一样的数据结构。

从上述代码中我们可以很容易的理解到：触控事件包含多个触点的信息，`n_points` 表示当前共有多少个触点，每个触点都有自己的 x、y 坐标，并且有个 id 用于标识该触点，而 `state` 表示该触点的状态，它的值有三种：`LCUI_WEVENT_TOUCHDOWN`、`LCUI_WEVENT_TOUCHUP`、`LCUI_WEVENT_TOUCHMOVE`，这些值分别对应：触点按下、触点释放、触点移动这三个状态。

以下是测试触控事件的程序：

```c
/** test_touch.c -- test touch support */

#include <stdio.h>
#include <stdlib.h>
#include <LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/display.h>
#include <LCUI/gui/widget.h>

/** 触点绑定记录 */
typedef struct TouchPointBindingRec_ {
        int point_id;        /**< 触点 ID */
        LCUI_Widget widget;  /**< 组件 */
        LinkedListNode node; /**< 在链表中的结点 */
        LCUI_BOOL is_valid;  /**< 是否有效 */
} TouchPointBindingRec, *TouchPointBinding;

/** 触点绑定记录列表 */
static LinkedList touch_bindings;

static void OnTouchWidget(LCUI_Widget w, LCUI_WidgetEvent e, void *arg)
{
        LCUI_TouchPoint point;
        TouchPointBinding binding;

        if (e->touch.n_points == 0) {
                return;
        }
        binding = e->data;
        point = &e->touch.points[0];
        switch (point->state) {
        case LCUI_WEVENT_TOUCHMOVE:
                Widget_Move(w, point->x - 32.0f, point->y - 32.0f);
                break;
        case LCUI_WEVENT_TOUCHUP:
                if (!binding->is_valid) {
                        break;
                }
                /* 当触点释放后销毁组件及绑定记录 */
                Widget_ReleaseTouchCapture(w, -1);
                LinkedList_Unlink(&touch_bindings, &binding->node);
                binding->is_valid = FALSE;
                Widget_Destroy(w);
                free(binding);
                break;
        case LCUI_WEVENT_TOUCHDOWN:
        default:
                break;
        }
}

static void OnTouch(LCUI_SysEvent e, void *arg)
{
        int i;
        LCUI_Widget w;
        LinkedListNode *node;
        LCUI_TouchPoint point;
        LCUI_Color bgcolor = RGB(255, 0, 0);

        for (i = 0; i < e->touch.n_points; ++i) {
                TouchPointBinding binding;
                LCUI_BOOL is_existed = FALSE;
                point = &e->touch.points[i];
                _DEBUG_MSG("point: %d\n", point->id);
                /* 检查该触点是否已经被绑定 */
                for (LinkedList_Each(node, &touch_bindings)) {
                        binding = node->data;
                        if (binding->point_id == point->id) {
                                is_existed = TRUE;
                        }
                }
                if (is_existed) {
                        continue;
                }
                w = LCUIWidget_New(NULL);
                /* 新建绑定记录 */
                binding = NEW(TouchPointBindingRec, 1);
                binding->point_id = point->id;
                binding->node.data = binding;
                binding->is_valid = TRUE;
                binding->widget = w;
                Widget_Resize(w, 64, 64);
                Widget_Move(w, point->x - 32.0f, point->y - 32.0f);
                /* 设置让该组件捕获当前触点 */
                Widget_SetTouchCapture(w, binding->point_id);
                Widget_BindEvent(w, "touch", OnTouchWidget, binding, NULL);
                Widget_SetStyle(w, key_position, SV_ABSOLUTE, style);
                Widget_SetStyle(w, key_background_color, bgcolor, color);
                LinkedList_AppendNode(&touch_bindings, &binding->node);
                Widget_Top(w);
        }
}

int main(int argc, char **argv)
{
        LCUI_Init();
        LinkedList_Init(&touch_bindings);
        LCUI_BindEvent(LCUI_TOUCH, OnTouch, NULL, NULL);
        return LCUI_Main();
}

```

编译后运行，如果你的计算机自带触屏，则可以用手指在程序窗口内点击和移动，然后会看到类似于如下图所示的效果：

![运行效果](../.gitbook/assets/gui\_events\_touch.gif)

这个程序实现的功能是捕获各个触点并用对应的红色方块表示触点的位置，有个 `touch_bindings` 全局变量用于保存各个触点和组件的绑定记录，在响应触控事件时会遍历每个触点，根据触点的 id 在绑定记录中找对应的组件，找到后会根据触点的坐标来更新组件的坐标。为了让组件能够捕获触点在它外面产生的触控事件，并且不被其它组件捕获到，调用了 `Widget_SetTouchCapture()` 函数实现对该触点的独占，在触点释放后，调用 `Widget_ReleaseTouchCapture()` 函数解除对触点的独占。

### 待办事项

**完善焦点管理**

需求如下：

* 添加支持用 Tab 键和 Shift+Tab 组合键上下移动焦点。
* 已获得焦点的组件应该有显眼的外边框。

关于外边框的实现，我们可以先为 CSS 解析器添加  outline 属性解析支持，然后在组件渲染流程中增加一个外边框绘制操作。
