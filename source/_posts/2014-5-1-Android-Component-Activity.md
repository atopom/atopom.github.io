---
title: Android-Component-Activity
date: 2014-05-01 14:56:06
tags:
- Android
- Activity
categories:
- Work
- Technology
- Note
---

# Activity介绍

Android应用组件Activity是Android程序的呈现层，显示可视化的用户界面，并接收与用户交互所产生的界面事件。对于一个Android应用程序来说，可以包含一个或多个Activity，一般在程序启动后会呈现一个Activity，用于提示用户程序已经正常启动。当它不积极运行时，Activity可以被操作系统终止以节省内存。

# Activity四种状态

1. 活动状态，Activity在用户界面中处于最上层，完全能被用户看到，且能和用户进行交互。（前台进程）
2. 暂停状态，Activity在界面上被部分遮挡，该Activity不再处于用户界面的最上层，且不能够与用户进行交互。（可见但非前台进程）
3. 停止状态，Activity在界面上完全不能被用户看到，也就是说该Activity被其他Activity全部遮挡。（后台进程）
4. 非活动状态，不在以上3种状态的Activity则处于非活动状态。

# Activity生命周期

1. onCreate：     Activity被创建的时候调用（一般做初始化操作）
2. onStart：     Activity变成用户可见的时候调用
3. onResume：     界面获取焦点的时候调用
4. onPause：     界面失去焦点的时候调用
5. onStop：     Activity变成用户不可见的时候调用
6. onDestroy：     Activity被系统消毁的时候调用

![Activity生命周期](https://developer.android.com/images/activity_lifecycle.png)

> 如：同一个APP中一个Activity（A）跳转到另一个Activity（B）时：
A   onPause （A先进入onPause状态，B才能创建）
B   onCreate、onStart、onResume
A   onStop
所以，为了防止界面间切换的卡顿，不要在onPause和onResume方法中些许耗时的操作。
注意：当界面A调用界面B时，如果界面B是以对话框形式来打开的话（也就是界面A还能看得见），界面A只会执行onPause方法，同时返回到界面A时，也只会执行onResume。

## 内存不足被销毁的生命周期

1. onSaveInstanceState：Android系统因资源不足终止Activity前，调用此方法。用于保存Activity的状态信息，供onRestoreInstanceState()或onCreate()恢复时用。
2. onRestoreInstanceState()：恢复onSaveInstanceState()保存的Activity状态信息，在onStart()之后调用。

![Activity异常状态的工作过程](/images/Android/Activity_Exception_RecycleLife.png)

## configChanges配置

Activity在横竖屏切换的时候，会重走生命周期方法。
横竖屏切换activity不被消毁的设置：
> android:configChanges="orientation|keyboardHidden|screenSize" // 4.0以下只需前两个参数

注意：如果你的app一直是竖着显示的话，必须在所有Activity中设置：
> android:screenOrientation="portrait"

## 横竖屏切换时候activity的生命周期

1. 不设置Activity的android:configChanges时，屏幕方向发生改变，会重新调用各个生命周期。
2. 设置Activity的android:configChanges=“orientation|screenSize"时，屏幕方向发生改变，生命周期不会重新调用，只会执行onConfigurationChanged方法。（screenSize，API13新添加）

> 获取任务栈【代码】
ActivityService.getRunningTasks()

> killProcess【代码】
android.os.Process.killProcess(android.os.Process.myPid()); // Process类在lang中也存在，所以加上包名调用
System.exit(0);

# 清单文件默认配置
```
<intent-filter>
     <action android:name="android.intent.action.MAIN" /> // 启动程序优先启动
     <category android:name="android.intent.category.LAUNCHER" /> // 桌面图标
</intent-filter>
```
两个配置同时存在才会在桌面创建图标且点击图标优先启动该Activity

# Activity启动模式
1. standard：标准模式

2. singleTop：栈顶复用模式
> 开启Activity时，会检查当前与之和匹配的Task的栈顶是否存在该Activity，如果存在，则复用，并切会回掉Activity.onNewIntent方法。

3. singleTask：栈内复用模式
> 开启Activity时，会检查当前与之匹配的Task的任务栈中是否已有该Activity，如果有，则将任务栈中的该Activity上面的所有Activity全部Remove，然后显示出该Activity，并且回掉onNewIntent方法。这种启动模式主要是为了让某些比较消耗资源且只需一个的Activity在任务栈中只存在一个，防止创建太多而造成内存不足等。

4. singleInstance：单实例模式
> 让同一个Activity在任务栈中只存在一个（换句话说就是整个手机内存中该Activity只会存在一个，如我的程序开了该Activity，别的程序再开启，也是同一个）底层实现是新建一个任务栈专门存放该Activity，在用户调用该Activity时，该任务栈就会被放到原任务栈的前面。

## taskAffinity配置
> TaskAffinity任务相关性。
主要和SingleTask启动模式或AllowTaskReparenting配置配对使用。

## allowTaskReparenting配置
> 是否允许Activity从一个Task迁移到另一个Task。

## Activity的Flags
1. FLAT_ACTIVITY_SINGLE_TOP，和XML中singleTop启动模式相同。
2. FLAG_ACTIVITY_NEW_TASK，和XML中singleTask启动模式相同。
3. FLAT_ACTIVITY_CLEAR_TOP，
在同一个任务栈中，所有位于它上面的Activity都会出栈。如果Activity使用standard启动模式，那么它连同它之上的Activity都要出栈，系统会创建新的Activity实例放入栈顶。
4. FLAT_ACTIVITY_EXCLUDE_FROM_RECENTS，
具有这个标记的Activity不会出现在历史Activity的列表中，当某些情况下我们不希望用户通过历史列表回到我们到Activity的时候，这个标记比较有用。它等同于XML中指定Activity的属性android:excludeFromRecents=“true”。

# Activity常用方法
> 用于别的线程更新UI的操作：runOnUiThread(Runnable action)，将更新ui的操作写到这个Runnable的run()中。该方法将会判断当前线程是否是创建UI的线程。如果是，则会直接运行。如果不是，将会跳到UI线程中执行。

# Activity切换动画
> overridePendingTransition(R.anim.tran_in, R.anim.tran_out);
其中overridePendingTransition是Activity的方法，传递两个动画的xml配置文件。如：
```
tran_in.xml
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500"
    android:fromXDelta="100%p"
    android:fromYDelta="0"
    android:toXDelta="0"
    android:toYDelta="0" >
</translate>
tran_out.xml
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="500"
    android:fromXDelta="0"
    android:fromYDelta="0"
    android:toXDelta="-100%p"
    android:toYDelta="0" >
</translate>
```

# 伪桌面（透明背景、无标题）
> 在清单文件中加入 android:theme="@android:style/Theme.Translucent.NoTitleBar"

# 有关setContentView()方法的底层原理
> 首先每个控件都是不能自己设定宽高的，需要外面有一个ViewGroup包起来，设定宽高才有效。
而如果是这样的话，那样每个xml的最外层的ViewGroup的宽高都是无效的吗？其实不是，其实每个Activity默认就存在着一个FrameLayout。
