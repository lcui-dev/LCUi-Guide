# 键盘

鼠标驱动的工作是触发按键按下和释放事件，通常我们只需要绑定系统的按键事件然后转换成 LCUI 的按键事件对象即可。

如需了解更多，可参考现有的鼠标驱动代码：

* [src/platform/linux/linux\_keyboard.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/linux/linux_keyboard.c)
* [src/platform/linux/linux\_keyboard.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/linux/linux_x11keyboard.c)
* [src/platform/windows/windows\_keyboard.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/windows/windows_keyboard.c)
* [src/platform/windows/uwp\_input.cpp](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/platform/windows/uwp_input.cpp#L252-L273)

