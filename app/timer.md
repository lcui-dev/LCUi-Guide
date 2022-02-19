---
description: 定时器的概念及用法介绍。
---

# 定时器

定时器用于将一些操作推迟到指定时间之后执行。LCUI 的定时器都是在主线程中处理的，这意味着定时器的时间粒度受到帧率的限制，不能小于每帧的停留时间。举个例子：当前帧率为 120 帧每秒，那么时间粒度就是 8.33 毫秒，如果你设置定时器的等待时间是 20 毫秒，那么实际的等待时间就会大于等于 25 毫秒，也就是在设置定时器后的第三帧时处理这个定时器。这种精确度的定时器能够应付大多数场景，如果你需要更加精确的定时器，我们建议你选择其它定时器库，或自行编码实现。

### 简单的例子

这个例子展示了如何设置和释放定时器：

```c
#include <stdio.h>
#include <LCUI.h>

void OnTimeout(void *arg)
{
    int *timer_id = arg;

    LCUITimer_Free(*timer_id);
    LCUI_Quit();
    printf("timeout\n");
}

void OnInterval(void *arg)
{
    printf("interval\n");
}

int main(void)
{
    int timer_id;

    LCUI_Init();
    timer_id = LCUI_SetInterval(1000, OnInterval, 0);
    LCUI_SetTimeout(5000, OnTimeout, &timer_id);
    return LCUI_Main();
}
```

首先调用 `LCUI_SetInterval()` 设置定时器每隔 1 秒调用一次 `OnInterval()` 函数，并将它返回的定时器标识号赋值给 `timer_id` ，然后调用 `LCUI_SetTimeout()` 设置一个定时器在 5 秒后调用 `OnTimeout()` 函数，并将 `timer_id` 的引用作为参数传给 `OnTimeout()` 函数。
