---
layout: post
title: "【笔记】android nested scroll 分析"
categories: 编程
tags: android
author: adai
date: 2020-05-11 14:43 +0800
---
* content
{:toc}





### 1、接口简介



**NestedScrollingChild**


```java
// 设置支持嵌套滚动
void setNestedScrollingEnabled(boolean enabled);

// 是否支持嵌套滚动
boolean isNestedScrollingEnabled();

// 开始轴向嵌套滚动
boolean startNestedScroll(@ViewCompat.ScrollAxis int axes);

// 停止嵌套滚动
void stopNestedScroll();

// 是否有parent
boolean hasNestedScrollingParent();

// child 消耗嵌套滚动后，让parent继续处理滚动
boolean dispatchNestedScroll(int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow);

// child 准备嵌套滚动之前，优先传递给parent处理
boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed, @Nullable int[] offsetInWindow);

// child 处理fling滚动后，传递给parent处理
boolean dispatchNestedFling(float velocityX, float velocityY, boolean consumed);

// child 处理fling 消耗之前，传递给parent处理(默认递归到parent去处理，最终给了false，新版本fling都是让child控制的)
boolean dispatchNestedPreFling(float velocityX, float velocityY);


```



> **NestedScrollingChild2** 增加了 type 用于判断滚动是**TYPE_TOUCH** 还是 **TYPE_NON_TOUCH** 区分fling事件
>
> **NestedScrollingChild3** 对 **dispatchNestedScroll** 方法增加了 **consumed** 参数用于传递所有滚动信息



**NestedScrollingParent**

```java
boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes); // 开始制定轴向的嵌套滚动

void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes); // child开始嵌套滚动的时候，通知当前类嵌套滚动开始

void onStopNestedScroll(@NonNull View target); // 停止嵌套滚动

void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed, int dxUnconsumed, int dyUnconsumed); // child 消耗滚动后，处理滚动

void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed); // child 消耗滚动前，处理滚动

boolean onNestedFling(@NonNull View target, float velocityX, float velocityY, boolean consumed); // child 消耗滚动后，处理fling

boolean onNestedPreFling(@NonNull View target, float velocityX, float velocityY); // child 消耗fling滚动前，处理fling

int getNestedScrollAxes(); // 当前嵌套滚动轴向
```

> **NestedScrollingParent2** 增加了 type 用于判断滚动是**TYPE_TOUCH** 还是 **TYPE_NON_TOUCH** 区分fling事件
>
> **NestedScrollingParent3** 对 **onNestedScroll** 方法增加了 **consumed** 参数用于传递所有滚动信息





### 2、嵌套滚动流程解析



![scroll anim]({{site.url}}/assets/2020-05-01/scroll anim.gif)



如动画所示，android嵌套滚动的过程

1、view 接收触摸事件，开始嵌套滚动

2、parent优先处理滚动事件

3、child消耗剩余滚动事件

4、parent结束消耗



多级嵌套滚动和fling总体逻辑复用上述逻辑





<img src="{{site.url}}/assets/2020-05-01/scrollend.png" alt="scrollend" style="zoom:50%;" />







嵌套滚动简易流程图



![scroll simple]({{site.url}}/assets/2020-05-01/scroll simple.png)





详细嵌套滚动流程图



![Android NestedScrolling]({{site.url}}/assets/2020-05-01/Android NestedScrolling.png)









### 3、嵌套滚动应用 -悬停header处理









<img src="{{site.url}}/assets/2020-05-01/suspendLayout.gif" alt="suspendLayout" style="zoom:30%;" />



期望达到如图效果





#### 自定义view方式

1、继承LinearLayout，实现NestedScrollingParent3接口

2、将第一个View作为header，可滚动，滚出界面





核心代码

```java
// 使用第一个view 作为header
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
if (getChildCount() > 0) {
View headerView = getChildAt(0);

measureChildWithMargins(headerView, widthMeasureSpec, 0, MeasureSpec.UNSPECIFIED, 0);
headerViewHeight = headerView.getMeasuredHeight();
// 添加了悬停的高度，避免上拉的时候把底裤漏出来
super.onMeasure(widthMeasureSpec, MeasureSpec.makeMeasureSpec(MeasureSpec.getSize(heightMeasureSpec) + headerViewHeight, MeasureSpec.EXACTLY));
return;
}

super.onMeasure(widthMeasureSpec, heightMeasureSpec);
}

// 限制滚动范围为 0~悬停布局高度 (即第一个视图高度)
public void scrollTo(int x, int y) {
int validY = MathUtils.clamp(y, 0, headerViewHeight);
super.scrollTo(x, validY);
}

// 下拉处理
private void scrollDown(int dyUnconsumed, @NonNull int[] consumed) {
int oldScrollY = getScrollY();
scrollBy(0, dyUnconsumed);
if (consumed != null) {
consumed[1] += getScrollY() - oldScrollY;
}
}

// 上拉处理
private void scrollUp(int dy, @NonNull int[] consumed) {
int oldScrollY = getScrollY();
scrollBy(0, dy);
if (consumed != null) {
consumed[1] = getScrollY() - oldScrollY;
}
}

```



> 缺陷，由于布局未实现touch事件，滚动处理全是支持嵌套滚动的子View实现的，导致在不支持嵌套滚动的View上滑动，是无效的，比如上图的tabLayout. 自己实现touch事件不失为一种好方法，就是比较麻烦



#### 使用系统方案

AppBarLayout + CollapsingToolbarLayout

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:fitsSystemWindows="true"
android:orientation="vertical">


<com.google.android.material.appbar.AppBarLayout
android:layout_width="match_parent"
android:layout_height="wrap_content">

<com.google.android.material.appbar.CollapsingToolbarLayout
android:layout_width="match_parent"
android:minHeight="48dip"
app:layout_scrollFlags="scroll|exitUntilCollapsed"
android:layout_height="wrap_content">


<androidx.recyclerview.widget.RecyclerView
android:id="@+id/recycleView"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:layout_marginBottom="48dip"
/>


<com.google.android.material.tabs.TabLayout
android:id="@+id/tabLayout"
android:layout_gravity="bottom"
app:tabSelectedTextColor="#333333"
app:tabTextColor="#dddddd"
app:layout_collapseMode="pin"
android:text="header"
android:layout_width="match_parent"
android:layout_height="48dip" />

</com.google.android.material.appbar.CollapsingToolbarLayout>

</com.google.android.material.appbar.AppBarLayout>

<androidx.viewpager.widget.ViewPager
android:id="@+id/viewPager"
android:layout_width="match_parent"
android:layout_height="match_parent"
app:layout_behavior="@string/appbar_scrolling_view_behavior" />
</androidx.coordinatorlayout.widget.CoordinatorLayout>

```



如图

<img src="{{site.url}}/assets/2020-05-01/appbar.gif" alt="appbar" style="zoom:33%;" />



>  虽然能实现悬停和滑动效果，但是却显示自view在fling的时候按不住。因为touchdown的时候是通过子view的scroller控制滚动的，无法通知到子view停止滚动









### 参考

https://juejin.im/post/5c60562e51882562e17c092b

https://juejin.im/post/5c3c8d2ae51d4552475fcef7#heading-8




