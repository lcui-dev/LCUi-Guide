# 鼠标

鼠标驱动的工作是触发包括移动、点击、滚轮在内的鼠标事件，通常我们只需要绑定系统的鼠标事件然后转换成 LCUI 的鼠标事件对象即可。

如需了解更多，可参考现有的鼠标驱动代码：

* [src/platform/linux/linux\_mouse.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/linux/linux_mouse.c)
* [src/platform/linux/linux\_x11mouse.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/linux/linux_x11mouse.c)
* [src/platform/windows/windows\_mouse.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/windows/windows_mouse.c)
* [src/platform/windows/uwp\_input.cpp](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/windows/uwp_input.cpp#L105-L251)





