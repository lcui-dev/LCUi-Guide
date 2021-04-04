# 触控

触控驱动的工作是收集触点的状态和坐标然后触发触控事件，通常我们只需要绑定系统的触控事件然后转换成 LCUI 的触控事件对象即可。

如需了解更多，可参考现有的触控驱动：

* [src/platform/windows/uwp\_input.cpp](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/windows/uwp_input.cpp#L213-L224)
* [src/platform/windows/windows\_mouse.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/windows/windows_mouse.c#L95-L143)

