### Flutter 滑动冲突解决方案

#### flutter 中的布局

- Flutter 中有两种布局模型：
  - 基于 RenderBox 的盒模型布局
  - 基于 Sliver ( RenderSliver ) 按需加载列表布局

- Flutter 中的可滚动主要由三个角色组成：Scrollable、Viewport 和 Sliver：
  - Scrollable ：用于处理滑动手势，确定滑动偏移，滑动偏移变化时构建 Viewport 。
  - Viewport：显示的视窗，即列表的可视区域；
  - Sliver：视窗里显示的元素

#### 为什么会滑动冲突

#### 带有滑动功能的组件有哪些

#### 如何解决滑动冲突

#### 总结