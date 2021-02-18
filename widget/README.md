---
description: 组件允许你将 UI 拆分为独立可复用的代码片段，并对每个片段进行独立构思。
---

# 组件

组件（Component）是构成 LCUI 应用程序的 UI 的基本元素，其它 UI 开发库可能会将这类元素称之为控件（Control）或部件（Widget），你可以按照自己的习惯称呼它们，不过由于作者受到 Web 前端开发技术的影响，本文档将统一采用“组件”这一名字称呼它，即使它的数据类型是 `LCUI_Widget`。

### 待办事项

**添加 img 组件**

功能与 HTML 中的 img 元素相同，组件宽高自适应图片尺寸。

**改进组件增量更新机制**

为了应对数十万量级的组件更新，保证界面的流畅度，现有的做法是统计组件的更新耗时然后为每帧更新的组件总量设置一个合适的限制，这种做法并不是最优的，可参考 React 的 [Fiber reconciler](https://zh-hans.reactjs.org/docs/codebase-overview.html#fiber-reconciler) 的架构，重新设计 LCUI 的组件增量更新机制。

\*\*\*\*

