# 不能使用 ScrollView 包裹 ListView/GridView/ExpandableListVIew;因为这样会把 ListView 的所有 Item 都加载到内存中，要消耗巨大的内存和 cpu 去绘制图面。

不能使用 ScrollView 包裹 ListView/GridView/ExpandableListVIew;因为这
样会把 ListView 的所有 Item 都加载到内存中，要消耗巨大的内存和 cpu 去绘制图
面。
说明：
ScrollView 中嵌套 List 或 RecyclerView 的做法官方明确禁止。除了开发过程中遇到
的各种视觉和交互问题，这种做法对性能也有较大损耗。ListView 等 UI 组件自身有
垂直滚动功能，也没有必要在嵌套一层 ScrollView。目前为了较好的 UI 体验，更贴
近 Material Design 的设计，推荐使用 NestedScrollView。