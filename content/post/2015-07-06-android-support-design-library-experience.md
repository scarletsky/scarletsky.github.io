---
title: Android Support Design Library 注意事项
date: 2015-07-05 23:45:02
categories: [android]
tags: [android, material design]
---

# CoordinatorLayout

这个新 Layout 是 FrameLayout 的加强版，用来协调各个子 view 的行为。最主要是用来实现 Toolbar 的折叠效果，也可以用来实现 FAB 自动消失的效果。

常见的用法如下：

```
<android.support.design.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v4.widget.NestedScrollView
        xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_gravity="fill_vertical"
        android:layout_marginBottom="?attr/actionBarSize"
        android:background="@android:color/white"
        app:layout_behavior="@string/appbar_scrolling_view_behavior"
        >

        <FrameLayout
            android:id="@+id/frame_image_toolbar_content"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            >
        </FrameLayout>

    </android.support.v4.widget.NestedScrollView>

    <android.support.design.widget.AppBarLayout
        android:id="@+id/appbar"
        android:layout_width="match_parent"
        android:layout_height="256dp"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar">

        <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/collapsing_toolbar_wrapper"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            app:contentScrim="?attr/colorPrimary"
            app:expandedTitleMarginEnd="64dp"
            app:expandedTitleMarginStart="48dp"
            app:layout_scrollFlags="scroll|exitUntilCollapsed"
            >

            <ImageView
                android:id="@+id/collapsing_toolbar_image"
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:scaleType="centerCrop"
                app:layout_collapseMode="parallax"
                />

            <android.support.v7.widget.Toolbar
                android:id="@+id/collapsing_toolbar"
                android:layout_width="match_parent"
                android:layout_height="?attr/actionBarSize"
                app:layout_collapseMode="pin"
                app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
                app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
                />

        </android.support.design.widget.CollapsingToolbarLayout>

    </android.support.design.widget.AppBarLayout>

</android.support.design.widget.CoordinatorLayout>
```

- CollapsingToolbarLayout.setTitle 代替 Toolbar.setTitle
- NestedScrollView 中必须添加 `app:layout_behavior="@string/appbar_scrolling_view_behavior"` 属性之后，里面的内容才会位于 AppBarLayout 之下
- NestedScrollView 中 `android:layout_gravity="fill_vertical"` 和 `android:layout_marginBottom="?attr/actionBarSize"` 属性不是必须的，但如果内容不够高，则内容会位于屏幕底部，而不是 AppBarLayout 下面。`layout_gravity="fill_vertical"` 可以修复这个问题。



# 参考资料

- [Android Support Design 中 CoordinatorLayout 与 Behaviors 初探](http://segmentfault.com/a/1190000002888109)
- http://stackoverflow.com/questions/30612310/android-nestedscrollview-has-wrong-size-after-applayout-behavior
