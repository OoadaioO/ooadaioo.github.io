---
layout: post
title: "【笔记】android tablayout 改变字体大小"
categories: 编程
tags: android 安卓
author: adai
date: 2020-04-23 14:00 +0800
---
* content
{:toc}




# android TabLayout调整tab字体大小





>  组件版本com.google.android.material:material:1.1.0 



最近项目中要用到tab，设计师希望能让tab选中的字体显示比较大，未选中的显示比较小，并且tab上面还要使用徽标。



如果使用主题的话主题只能设置选中和未选中颜色，没有字体大小选项



简单布局完了之后，想在**OnTabSelectedListener**中直接设置 tab.textSize,但是没有找到对应的设置属性

所以优先用了方案一



### 方案一、自定义customView



自定义tab布局 

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical">

    <TextView
        android:id="@android:id/text1"
        android:text="tab"
        style="@style/TextAppearance.Tab"
        android:layout_width="wrap_content"
        android:paddingStart="3dip"
        android:paddingEnd="3dip"
        android:layout_height="wrap_content"
        android:layout_gravity="center" />

    <androidx.appcompat.widget.AppCompatImageView
        android:id="@android:id/icon1"
        android:visibility="gone"
        android:layout_marginTop="3dip"
        tools:visibility="visible"
        android:layout_width="6dip"
        android:src="@drawable/tab_indicator"
        android:layout_gravity="right|top"
        android:layout_height="6dip" />

</FrameLayout>
```



页面初始化的时候更新一遍自定义view

```java
for (int i = 0; i < tabLayout.getTabCount(); i++) {
    TabLayout.Tab tab = tabLayout.getTabAt(i);
    if (tab == null) {
        return;
    }
    tab.setCustomView(R.layout.tab_layout);
}

```



**OnTabSelectedListener**回调中动态更新tab字体大小

```java
public void updateSelectTabTextSize(int position) {
    for (int i = 0; i < tabLayout.getTabCount(); i++) {
        TabLayout.Tab tab = tabLayout.getTabAt(i);

        if (tab.getCustomView() != null) {
            TextView text = tab.getCustomView().findViewById(android.R.id.text1);
             if (position == i) {
                 text.setTextSize(TypedValue.COMPLEX_UNIT_SP, 20);
              }else{
                 text.setTextSize(TypedValue.COMPLEX_UNIT_SP, 15);
              }
         }

    }
}
```



为什么，这里不直接用系统自带的BadgeDrawable，看了下源码，因为tab设置了customView之后，BadgeDrawable直接移除了。所以tab布局的文本和徽标都需要自定义。



想了一下，其实就差一个字体大小系统不能设置。看了下接口方法，tab.setText()参数是Charsequence，突然想到可以用SpannableString设置字体大小，就有了方案二



### 方案二、**OnTabSelectedListener**中直接给tab设置一个带字体大小的Charsequence即可



```java
   /**
     * 动态变更tabLayout的字体大小
     * @param tabLayout tabLayout布局
     * @param selectPosition 位置
     * @param selectTextSize 选中字体大小
     * @param normalTextSize 未选中字体大小
     */
public static void setTextSize(TabLayout tabLayout, int selectPosition, int selectTextSize, int normalTextSize) {
        for (int i = 0; i < tabLayout.getTabCount(); i++) {
            TabLayout.Tab tab = tabLayout.getTabAt(i);
            if(tab != null) {
                SpannableStringBuilder builder = new SpannableStringBuilder();
                String text = tab.getText() == null ? "" : tab.getText().toString();
                if (selectPosition == i) {
                    builder.append(text);
                    builder.setSpan(new AbsoluteSizeSpan(selectTextSize, true), 0, builder.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
                    tab.setText(builder);
                }
                else {
                    builder.append(text);
                    builder.setSpan(new AbsoluteSizeSpan(normalTextSize, true), 0, builder.length(), Spanned.SPAN_INCLUSIVE_EXCLUSIVE);
                    tab.setText(builder);
                }
            }
        }
    }
```



**OnTabSelectedListener** 回调中设置一下即可， 顺便设置下系统徽标颜色和显示



```java
    /**
     * 设置tabLayout徽标颜色
     * @param tab 目标tab
     * @param color 颜色
     */
    public  static void setBadgeColor(TabLayout.Tab tab, String color) {
        if(tab != null) {
            if (tab.getCustomView() != null) {
                ImageView image = tab.getCustomView().findViewById(android.R.id.icon1);
                if(image != null) {
                    ImageViewCompat.setImageTintList(image, ColorStateList.valueOf(Color.parseColor(color)));
                }
            }else {
                BadgeDrawable bd = tab.getOrCreateBadge();
                bd.setBackgroundColor(Color.parseColor(color));
            }
        }
    }

    /**
     * 设置tabLayout徽标可见性
     * @param tab 目标tab
     * @param visible 颜色
     */
    public static void setBadgeVisible(TabLayout.Tab tab, boolean visible) {
        if(tab != null){
            if (tab.getCustomView() != null) {
                ImageView image = tab.getCustomView().findViewById(android.R.id.icon1);
                if(image != null) {
                    image.setVisibility(visible ? View.VISIBLE : View.GONE);
                }
            }else{
                BadgeDrawable bd = tab.getOrCreateBadge();
                bd.setVisible(visible);
            }
        }
    }
```



玩美

当然谷歌能给个设置字体大小的接口就更好了。











