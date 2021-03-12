# 布局

LCUI 的布局与网页中的 CSS 布局是相似的，我们假定你已经熟练掌握 CSS 布局技术，本章节将简单介绍 LCUI 布局与网页差异，并通过示例代码的形式来展现常见布局在 LCUI 中的实现方式，相信你在看了这些内容后能够快速上手。

{% hint style="info" %}
如果你对 CSS 布局技术还不熟悉，我们建议你阅读 MDN 上的文档：《[CSS 布局 - 学习 Web 开发 \| MDN](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/CSS_layout)》
{% endhint %}

### 待办事项

**重新设计布局引擎**

现有的布局引擎是与 LCUI 的组件系统绑定在一起的，而且布局规则的处理流程也比较复杂，为了降低维护成本，简化布局流程，我们应该重新设计布局引擎，使它能够作为一个独立的项目被用于其他项目。可供参考的同类案例有：[https://github.com/facebook/yoga](https://github.com/facebook/yoga)



