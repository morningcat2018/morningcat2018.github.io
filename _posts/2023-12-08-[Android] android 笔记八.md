---
layout:     post
title:      android 笔记八
subtitle:   广播
date:       2023-12-08
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Android
---

## 详解广播机制

广播消息机制

## 6.1 广播机制简介

Android中的广播主要可以分为两种类型:标准广播和有序广播。

标准广播(normal broadcasts)是一种完全异步执行的广播，BroadcastReceiver 之间没有任何先后顺序。这种广播的效率会比较高，但同时也意味着它是无法被截断的

有序广播(ordered broadcasts)则是一种同步执行的广播，在广播发出之后，同一时刻 只会有一个BroadcastReceiver能够收到这条广播消息，当这个BroadcastReceiver中的 逻辑执行完毕后，广播才会继续传递。所以此时的BroadcastReceiver是有先后顺序的， 优先级高的BroadcastReceiver就可以先收到广播消息，并且前面的BroadcastReceiver 还可以`截断`正在传递的广播，这样后面的BroadcastReceiver就无法收到广播消息了。

## 6.2 接收系统广播

#### 6.2.1 动态注册监听时间变化

注册 BroadcastReceiver的方式一般有两种:

- 在代码中注册: 动态注册
- 在 AndroidManifest.xml 中注册: 静态注册

新建一个类，让它继承自 BroadcastReceiver，并重写父类的onReceive()方法; 当有广播到来时， onReceive()方法就会得到执行

```kotlin
import android.content.BroadcastReceiver
import android.content.Context
import android.content.Intent
import android.content.IntentFilter
import android.os.Bundle
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity

class MainActivity4 : AppCompatActivity() {
    lateinit var timeChangeReceiver: TimeChangeReceiver
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val intentFilter = IntentFilter()
        intentFilter.addAction(Intent.ACTION_TIME_TICK) // android.intent.action.TIME_TICK

        timeChangeReceiver = TimeChangeReceiver()
        registerReceiver(timeChangeReceiver, intentFilter)
    }

    override fun onDestroy() {
        super.onDestroy()
        unregisterReceiver(timeChangeReceiver)
    }

    inner class TimeChangeReceiver : BroadcastReceiver() {
        override fun onReceive(context: Context, intent: Intent) {
            Toast.makeText(context, "Time has changed", Toast.LENGTH_SHORT).show()
        }
    }
}
```

动态注册的BroadcastReceiver一定要取消注册才行，这里我们是在 onDestroy()方法中通过调用unregisterReceiver()方法来实现的。

系统每隔一分钟就会发出一条 `android.intent.action.TIME_TICK` 的广播，因此我们最多只需要等待一分钟就可以收到这条广播了

如果你想查看完整的系统广播列表，可以到如下的路径中去查看 : `${Android SDK路径}/platforms/${任意android api版本}/data/broadcast_actions.txt` ,本机为( ~/Library/Android/sdk/platforms/android-33/data/broadcast_actions.txt )

#### 6.2.2 静态注册实现开机启动

动态注册的BroadcastReceiver可以自由地控制注册与注销，在灵活性方面有很大的优势。但是它存在着一个缺点，即`必须在程序启动之后才能接收广播`，因为注册的逻辑是写在 onCreate()方法中的。

从理论上来说，动态注册能监听到的系统广播，静态注册也应该能监听到，在过去的 Android系统中确实是这样的。但是由于大量恶意的应用程序利用这个机制在程序未启动的情况 下监听系统广播，从而使任何应用都可以频繁地从后台被唤醒，严重影响了用户手机的电量和 性能，因此Android系统几乎每个版本都在削减静态注册BroadcastReceiver的功能。

在Android 8.0系统之后，所有隐式广播都不允许使用静态注册的方式来接收了。隐式广播指的是那些没有具体指定发送给哪个应用程序的广播，大多数系统广播属于隐式广播，但是少数特殊的系统广播目前仍然允许使用静态注册的方式来接收。这些特殊的系统广播列表详见[隐式广播例外情况](https://developer.android.google.cn/guide/components/broadcast-exceptions?hl=zh-cn)


例外情况其中之一就是 android.intent.action.BOOT_COMPLETED ，这是一条开机广播

在开机的时候，我们的应用程序肯定是没有启动的， 因此这个功能显然不能使用动态注册的方式来实现，而应该使用静态注册的方式来接收开机广播，然后在onReceive()方法里执行相应的逻辑，这样就可以实现开机启动的功能了

继承BroadcastReceiver的类
```kotlin
class BootCompleteReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "Boot Complete", Toast.LENGTH_LONG).show()
    }
}
```

注册到 AndroidManifest.xml 上

```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
    
    <!-- 开机广播 -->
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

    <receiver
        android:name=".BootCompleteReceiver"
        android:enabled="true"
        android:exported="true">
        <intent-filter>
            <action android:name="android.intent.action.BOOT_COMPLETED" />
        </intent-filter>
    </receiver>
</application>
```

- 需要 RECEIVE_BOOT_COMPLETED 权限, 接收系统的开机广播就是需要进行权限声明的
- Exported 属性表示是否允许这个 BroadcastReceiver 接收本程序以外的广播
- Enabled 属性表示是否启用这个 BroadcastReceiver
- 需要配置 intent-filter , 并在里面声明相应的action。

需要注意的是，不要在onReceive()方法中添加过多的逻辑或者进行任何的耗时操作，因为 BroadcastReceiver 中是不允许开启线程的，当onReceive()方法运行了较长时间而没有结束时，程序就会出现错误。

## 6.3 发送自定义广播

#### 6.3.1 发送标准广播

```kotlin
class MyActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val button = findViewById<Button>(R.id.button_count)
        button.setOnClickListener {
            // 发送标准广播
            val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
            intent.setPackage(packageName)
            sendBroadcast(intent)
        }
    }
}
```

在Android 8.0系统之后，静态注册的BroadcastReceiver是无法接收隐式广播的，而默认情况下我们发出 的自定义广播恰恰都是隐式广播。因此这里一定`要调用setPackage()方法，指定这条广播是发送给哪个应用程序的`，从而让它变成一条显式广播，否则静态注册的BroadcastReceiver将无法接收到这条广播。

---

接收方

```xml
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter>
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>
</receiver>
```

```kotlin
class MyBroadcastReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "received in MyBroadcastReceiver", Toast.LENGTH_SHORT).show()
    }
}
```

#### 6.3.2 发送有序广播

1. 发送方

```kotlin
class MyActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val button = findViewById<Button>(R.id.button_count)
        button.setOnClickListener {
            // 发送标准广播
            val intent = Intent("com.example.broadcasttest.MY_BROADCAST")
            intent.setPackage(packageName)
            sendOrderedBroadcast(intent, null)
        }
    }
}
```

发送有序广播只需要改动一行代码，即将sendBroadcast()方法改成 sendOrderedBroadcast()方法

2. 接收方

设置权重

```xml
<receiver
    android:name=".MyBroadcastReceiver"
    android:enabled="true"
    android:exported="true">
    <intent-filter android:priority="100">
        <action android:name="com.example.broadcasttest.MY_BROADCAST"/>
    </intent-filter>
</receiver>
```

通过android:priority属性给BroadcastReceiver设置了优先级，优先级比较高的BroadcastReceiver就可以先收到广播

```kotlin
class BootCompleteReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        Toast.makeText(context, "Boot Complete", Toast.LENGTH_LONG).show()
        // 截断
        abortBroadcast()
    }
}
```

如果在onReceive()方法中调用了abortBroadcast()方法，就表示将这条广播截断，后面 的BroadcastReceiver将无法再接收到这条广播。











