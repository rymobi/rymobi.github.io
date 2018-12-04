---
layout: post
title: Android控件使用 之 BottomNavigationView
description: "主要记录Android Material Design控件BottomNavigationView的使用过程和遇到的一些问题及解决方案。"
tags: [Android, Material Design, 控件]

image:
  path: /images/android.jpg
  feature: android.jpg
---

BottomNavigationView就是安卓APP中常用到的底部导航栏，它由官方的support包内提供，按照[官方的设计规范](https://material.io/design/components/bottom-navigation.html)，这个控件的item要控制在3 ~ 5之间。当少于3个item时不会报错，但是超过5个item，则会报错抛出异常，通常每个item就是由一个icon和title组成的。
使用BottomNavigationView来实现底部导航栏，还是比较方便的。


# BottomNavigationView的使用方法



## 导包

BottomNavigationView是放置在design包中的，所以，使用前需要先引入`com.android.support:design:当前版本号`包，引入方式有3种：

* 直接从当前**module**的**gradle**文件中编辑
* 从**Project Structure**界面的**dependences**选项卡中导入
* 在任意一个布局文件中切换到**Design**窗口，在**Palette**窗口找到**Containers**里的BottomNavigationView，点击下载按钮/右击Add



## 布局

在需要使用的布局文件`xxx.xml`中，添加BottomNavigationView

```xml
<android.support.design.widget.BottomNavigationView
	android:id="@+id/bottom_nav_view"
	android:layout_width="match_parent"
	android:layout_height="48dp"
	android:layout_alignParentBottom="true"
	android:background="@color/whiteFull"
	app:menu="@menu/bottom_nav_view_menu"
	app:itemIconTint="@color/color_state_menu_bottom_nav"
	app:itemTextColor="@color/color_state_menu_bottom_nav"
	app:itemIconSize="20dp" />
```



## 添加Item

在`res`目录下创建一个`menu`文件夹，然后添加一个`bottom_nav_view_menu.xml`文件，内容如下：

```xml
<menu xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <item
        android:id="@+id/bottom_nav_item_1"
        android:icon="@drawable/ic_bottom_nav_find"
        android:title="@string/bottom_nav_item1_name" />

    <item
        android:id="@+id/bottom_nav_item_2"
        android:icon="@drawable/ic_bottom_nav_hierarchy"
        android:title="@string/bottom_nav_item2_name" />

    <item
        android:id="@+id/bottom_nav_item_3"
        android:icon="@drawable/ic_bottom_nav_todo"
        android:title="@string/bottom_nav_item3_name" />

</menu>
```



## 添加点击Item的监听

在`xxx.java`中加入如下代码：

```java
public class MainActivity extends AppCompatActivity {
    private BottomNavigationView mNavigationView;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mNavigationView = (BottomNavigationView) findViewById(R.id.navigation);
        mNavigationView.setOnNavigationItemSelectedListener(new BottomNavigationView.OnNavigationItemSelectedListener() {
            @Override
            public boolean onNavigationItemSelected(@NonNull MenuItem item) {
                //执行的逻辑
                return true;
            }
        });
    }
}
```



## 属性说明

* **app:itemIconTint：**Item的图标颜色设定
* **app:itemTextColor:**Item的文字颜色设定
* **app:itemIconSize：**Item的图标尺寸设定

#### 想要设定选中状态的颜色和非选中状态的颜色，可以这样做：
在`res`目录下创建一个`color`文件夹，然后添加一个`color_state_menu_bottom_nav.xml`资源文件，内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="@color/colorAccent" android:state_checked="true"/>
    <item android:color="@color/grayFull" android:state_checked="false"/>
</selector>
```



## 需求

### 默认选中非第一个Item

在`menu`目录下的`bottom_nav_view_menu.xml`中使用
```xml
android:checked="true"
```

或者在java代码中加入：
```java
mNavigationView.getMenu().getItem(你想选中的位置).setChecked(true);
```


### 修改点击动画效果字体变化大小

在修改之前，我们先要明白BottomNavigationView Item的一个特点，从它的源码中能看出，**它有两个TextView**,正常显示时一个TextView，文字放大以后是另一个TextView，因此改变字体变化大小，就是去改变这两个TextView的尺寸
因为BottomNavigationView的一些属性已经写死，所以我们要重写去覆盖它，在`values`目录下添加一个`values.xml`资源文件，覆盖BottomNavigationView的一些属性，重新设定内容如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <dimen name="design_bottom_navigation_active_text_size">12sp</dimen>
    <dimen name="design_bottom_navigation_text_size">10sp</dimen>
    <dimen name="design_bottom_navigation_margin">6dp</dimen>
    <dimen name="design_bottom_navigation_height">48dp</dimen>
</resources>
```

#### 注意
* **design_bottom_navigation_margin**属性是设置图片距离自身外部的边距
* **design_bottom_navigation_height**属性是设置BottomNavigationView的高度，源码中写死了是56dp，如果要显示效果好看，只改变**android:layout_height="xxdp"**是不够的，边距控制不了，会很难看，需要配合上面那个高度属性一起使用。
* 如果不要点击字体放大效果，将两个字体的尺寸设置成一样的数字。



### 点击涟漪效果

如果设置了 **app:itemBackground=""** 这个属性，则水波纹效果失效。
那么既需要保留水波纹效果，又要改变BottomNavigationView的背景颜色，我们必须使用 **android:background=""** 这个属性去设置背景色。



### 超过3个Item，动画效果会有很多改变，比如只有点中的Item显示文字等等

很多人说这个动画太过于浮夸，所以用方法取消了这个动画，但我个人很喜欢这样的动画效果，所以没有试着去取消，以后如果有这样的需求了，我再去补充这个解决方案。




# End...


