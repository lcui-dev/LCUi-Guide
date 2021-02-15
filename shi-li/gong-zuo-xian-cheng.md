---
description: 工作线程的概念、用途以及相关函数的介绍。
---

# 工作线程

主循环执行频率影响界面的流畅度，在主循环的章节中我们已经了解到每一次循环都会按顺序执行处理定时器、处理事件队列、更新组件、渲染组件等任务，其中最容易影响到主循环的执行频率的任务是处理事件队列，因为大部分的事件处理器都是应用程序主动绑定的，对于缺乏性能优化意识的新手而言，可能会在事件处理器中直接进行一些耗时较高的操作，从而导致界面在操作结束前一直处于未响应状态。

解决这一问题的常见做法是将操作移动到另一个线程上执行，我们可以用 LCUI 提供的工作线程池来实现：

```c
void DoSomeThing(void *arg1, void *arg2)
{
    printf("key: %s\n", arg1);
    printf("value: %s\n", arg2);
}

void OnButtonClick()
{
    LCUI_TaskRec task = { 0 };
    
    LCUI_Init();
    task.arg[0] = strdup("color");
    task.arg[1] = strdup("red");
    task.destroy_arg[0] = free;
    task.destroy_arg[1] = free;
    task.func = DoSomeThing;
    
    LCUI_PostAsyncTask(&task);
}
```

`LCUITaskRec` 类型的 task 变量描述了任务的执行函数及其参数，`arg` 成员变量记录了参数列表，`destroy_arg` 则是这些参数的销毁函数，这里我们用了 `strdup()` 分配了新的内存存储字符串，并指定 `free()` 作为参数的销毁函数。准备完任务后，调用 `LCUI_PostAsyncTask()` 函数将任务添加到工作线程的任务队列中等待执行。

LCUI 的工作线程池中默认创建了 4 个工作线程，为了让这些工作线程都有任务执行，`LCUI_PostAsyncTask()` 函数会在每次投递完任务后将目标切换到下一个工作线程，如果这种简单的任务分配策略不符合你的需求，你也可以基于 [src/worker.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/worker.c) 提供的函数创建自己的工作线程池：

```c
// 创建一个带有任务队列的工作线程，然后运行它
LCUI_Worker worker = LCUIWorker_New();
LCUIWorker_RunAsync(worker);

...

// 给工作线程投递任务
LCUI_TaskRec task = { 0 };

task.arg[0] = strdup("color");
task.arg[1] = strdup("red");
task.destroy_arg[0] = free;
task.destroy_arg[1] = free;
task.func = DoSomeThing;
LCUIWorker_PostTask(worker, task);

...

// 销毁工作线程
LCUIWorker_Destroy(worker);
```

### 总结

由于主循环负责的都是 UI 相关的工作，而且它通常在主线程上执行，因此主线程也是 UI 线程，为了不影响 UI 线程上的主循环执行频率，我们应该尽可能将耗时较高的操作移动到 UI 线程之外的线程上执行，而这个线程就是工作线程。

