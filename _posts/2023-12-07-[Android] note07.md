---
layout:     post
title:      android 笔记七
subtitle:   Services
date:       2023-12-07
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Android
---

[Android developer document - 7.4: Services](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-3-working-in-the-background/lesson-7-background-tasks/7-4-c-services/7-4-c-services.html)

## What is a service?

A service is an app component that performs long-running operations, usually in the background. Unlike an Activity, a service doesn't provide a user interface (UI). Services are defined by the Service class or one of its subclasses.

服务是一个应用程序组件，通常`在后台`执行长时间运行的操作。 与活动不同，服务不提供用户界面 (UI)。 服务由 Service 类或其子类之一定义。

A service can be started, bound, or both: 服务可以启动、绑定或两者兼而有之：

- A started service is a service that an app component starts by calling startService(). 启动的服务是应用程序组件通过调用startService()启动的服务。
    - Use started services for tasks that run in the background to perform long-running operations. Also use started services for tasks that perform work for remote processes. 对在后台运行的任务使用已启动的服务来执行长时间运行的操作。 还可以将已启动的服务用于为远程进程执行工作的任务。
- A bound service is a service that an app component binds to itself by calling bindService(). 绑定服务是应用程序组件通过调用bindService() 与其自身绑定的服务。
    - Use bound services for tasks that another app component interacts with to perform interprocess communication (IPC). For example, a bound service might handle network transactions, perform file I/O, play music, or interact with a database. 将绑定服务用于与另一个应用程序组件交互的任务，以执行进程间通信 (IPC)。 例如，绑定服务可能会处理网络事务、执行文件 I/O、播放音乐或与数据库交互。

A service runs in the main thread of its hosting process—the service doesn't create its own thread and doesn't run in a separate process unless you specify that it should.

服务`在其托管进程的主线程中运行` - 服务不会创建自己的线程，也不会在单独的进程中运行，除非您指定它应该这样做。

If your service is going to do any CPU-intensive work or blocking operations (such as MP3 playback or networking), create a new thread within the service to do that work. By using a separate thread, you reduce the risk of the user seeing "application not responding" (ANR) errors, and the app's main thread can remain dedicated to user interaction with your activities.

如果您的服务要执行任何 `CPU 密集型工作`或阻塞操作（例如 MP3 播放或网络），请在服务中`创建一个新线程`来完成该工作。 通过使用单独的线程，您可以降低用户看到“应用程序未响应”(ANR) 错误的风险，并且应用程序的主线程可以继续专用于用户与您的活动的交互。

在 Android 8.0（Oreo、API 26）或更高版本中，当应用程序本身不在前台时，系统会对运行后台服务施加一些新的限制。 

To implement any kind of service in your app, do the following steps: 要在您的应用程序中实现任何类型的服务，请执行以下步骤：

- Declare the service in the manifest. 在`清单 AndroidManifest.xml`中声明该服务。
- Extend a Service class such as IntentService and create implementation code, as described in Started services and Bound services, below. 扩展服务类（例如 IntentService）并创建实现代码，如下面的已启动服务和绑定服务中所述。
- Manage the service lifecycle. 管理`服务生命周期`。

## Declaring services in the manifest

As with activities and other components, you must declare all services in your `Android manifest`. To declare a service, add a `<service>` element as a child of the `<application>` element. For example:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.AndroidAppcc"
        tools:targetApi="31">

        <!-- 声明服务 -->
        <service
            android:name=".service.ExampleService"
            android:exported="false" />

        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name"
            android:theme="@style/Theme.AndroidAppcc">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

To block access to a service from other apps, declare the service as private. To do this, set the android:exported attribute to false. This stops other apps from starting your service, even when they use an explicit intent.

要`阻止其他应用程序访问某个服务`，请将该服务声明为私有。 为此，请将 `android:exported` 属性设置为 false。 这会阻止其他应用程序启动您的服务，即使它们使用显式意图也是如此。

## Started services

How a service starts:

1. An app component such as an Activity calls startService() and passes in an Intent. The Intent specifies the service and includes any data for the service to use.

应用程序组件（例如 Activity）调用 startService() 并传入 Intent。 Intent 指定 service 并包括服务要使用的任何数据。

2. The system calls the service's onCreate() method and any other appropriate callbacks on the main thread. It's up to the service to implement these callbacks with the appropriate behavior, such as creating a secondary thread in which to work.

系统在主线程上调用服务的 onCreate() 方法和任何其他适当的回调。 由服务通过适当的行为来实现这些回调，例如创建一个工作的辅助线程。

3. The system calls the service's onStartCommand() method, passing in the Intent supplied by the client in step 1. (The client in this context is the app component that calls the service.)

系统调用服务的 onStartCommand() 方法，传入客户端在步骤 1 中提供的 Intent。（此上下文中的客户端是调用服务的应用程序组件。）

Once started, a service can run in the background indefinitely, even if the component that started it is destroyed. Usually, a started service performs a single operation and does not return a result to the caller. For example, it might download or upload a file over the network. When the operation is done, the service should stop itself by calling stopSelf(), or another component can stop it by calling stopService().

一旦启动，服务就可以`无限期地在后台运`行，即使启动它的组件被销毁。 `通常，启动的服务执行单个操作并且不会将结果返回给调用者`。 例如，它可能通过网络下载或上传文件。 当操作完成后，服务应该通过调用 stopSelf() 自行停止，或者另一个组件可以通过调用 stopService() 来停止它。

For instance, suppose an Activity needs to save data to an online database. The Activity starts a companion service by passing an Intent to startService(). The service receives the intent in onStartCommand(), connects to the internet, and performs the database transaction. When the transaction is done, the service uses stopSelf() to stop itself and is destroyed. (This is an example of a service you want to run in a worker thread instead of the main thread.)

例如，假设一个活动需要将数据保存到在线数据库中。 Activity 通过将 Intent 传递给 startService() 来启动配套服务。 该服务接收 onStartCommand() 中的意图，连接到互联网并执行数据库事务。 当事务完成时，服务使用 stopSelf() 来停止自身并被销毁。 （这是您想要在工作线程而不是主线程中运行的服务的示例。）

#### IntentService

Most started services don't need to handle multiple requests simultaneously, and if they did, it could be a complex and error-prone multi-threading scenario. For these reasons, the IntentService class is a useful subclass of Service on which to base your service:

大多数启动的服务不需要同时处理多个请求，如果这样做，则可能是一个复杂且容易出错的多线程场景。 由于这些原因，IntentService 类是 Service 的一个有用的子类，您的服务基于它：

- IntentService automatically provides a worker thread to handle your Intent. IntentService 自动提供一个工作线程来处理您的 Intent。
- IntentService handles some of the boilerplate code that regular services need (such as starting and stopping the service). IntentService 处理常规服务所需的一些样板代码（例如启动和停止服务）。
- IntentService can create a work queue that passes one intent at a time to your onHandleIntent() implementation, so you don't have to worry about multi-threading. IntentService 可以创建一个工作队列，一次将一个意图传递给 onHandleIntent() 实现，因此您不必担心多线程。

Note:IntentService is subject to the new restrictions on background services in Android 8.0 (API 26). For this reason, Android Support Library 26.0.0 introduces a new JobIntentService class, which provides the same functionality as IntentService but uses jobs instead of services when running on Android 8.0 or higher.

注意：IntentService 受 Android 8.0 (API 26) 中`后台服务的新限制约束`。 因此，Android 支持库 26.0.0 引入了一个新的 JobIntentService 类，该类提供与 IntentService 相同的功能，但在 Android 8.0 或更高版本上运行时使用作业而不是服务。

To implement IntentService:

- Provide a small constructor for the service.
- Create an implementation of onHandleIntent() to do the work that the client provides.

Here's an example implementation of IntentService:

```java
public class HelloIntentService extends IntentService {
  /**
   * A constructor is required, and must call the 
   * super IntentService(String) constructor with a name 
   * for the worker thread.
   */
  public HelloIntentService() {
      super("HelloIntentService");
  }

  /**
   * The IntentService calls this method from the default 
   * worker thread with the intent that started the service. 
   * When this method returns, IntentService stops the service, 
   * as appropriate.
   */
  @Override
  protected void onHandleIntent(Intent intent) {
      // Normally we would do some work here, like download a file.
      // For our sample, we just sleep for 5 seconds.
      try {
          Thread.sleep(5000);
      } catch (InterruptedException e) {
          // Restore interrupt status.
          Thread.currentThread().interrupt();
      }
  }
}
```

## Bound services

A service is "bound" when an app component binds to it by calling bindService(). A bound service offers a client-server interface that allows components to interact with the service, send requests, and get results, sometimes using interprocess communication (IPC) to send and receive information across processes. A bound service runs only as long as another app component is bound to it. Multiple components can bind to the service at once, but when all of them unbind, the service is destroyed.

当应用程序组件通过调用bindService() 绑定到服务时，服务就被“绑定”了。 绑定服务提供客户端-服务器接口，允许组件与服务交互、发送请求并获取结果，有时使用进程间通信 (IPC) 跨进程发送和接收信息。 绑定服务仅在另一个应用程序组件绑定到它时才运行。 多个组件可以同时绑定到服务，但是当所有组件都解除绑定时，服务就会被销毁。

A bound service generally does not allow components to start it by calling startService().

绑定服务通常不允许组件通过调用 startService() 来启动它。

#### Implementing a bound service

To implement a bound service, define the interface that specifies how a client can communicate with the service. This interface, which your service returns from the onBind() callback method, must be an implementation of IBinder.

要实现绑定服务，请定义指定客户端如何与服务通信的接口。 您的服务从 onBind() 回调方法返回的该接口必须是 IBinder 的实现。

To retrieve the IBinder interface, a client app component calls bindService(). Once the client receives the IBinder, the client interacts with the service through that interface.

为了检索 IBinder 接口，客户端应用程序组件调用bindService()。 一旦客户端收到IBinder，客户端就通过该接口与服务进行交互。

There are multiple ways to implement a bound service, and the implementation is more complicated than a started service. For complete details about bound services, see Bound Services.

绑定服务的实现方式有多种，实现起来比启动服务要复杂。 有关绑定服务的完整详细信息，请参阅绑定服务。

#### Binding to a service

To bind to a service that is declared in the manifest and implemented by an app component, use bindService() with an explicit Intent.

要绑定到清单中声明并由应用程序组件实现的服务，请`使用带有显式意图的bindService()`。

Caution: Do not use an implicit intent to bind to a service. Doing so is a security hazard, because you can't be certain what service will respond to your intent, and the user can't see which service starts. Beginning with Android 5.0 (API level 21), the system throws an exception if you call bindService() with an implicit Intent .

注意：`不要使用隐式意图来绑定到服务`。 这样做存在安全隐患，因为您无法确定哪个服务将响应您的意图，并且用户无法看到哪个服务启动。 从 Android 5.0（API 级别 21）开始，如果您使用隐式 Intent 调用 bindService() ，系统会抛出异常。

## Service lifecycle

The lifecycle of a service is simpler than the Activity lifecycle. However, it's even more important that you pay close attention to how your service is created and destroyed. Because a service has no UI, services can continue to run in the background with no way for the user to know, even if the user switches to another app. This situation can potentially consume resources and drain the device battery.

服务的生命周期比 Activity 的生命周期更简单。 然而，更重要的是您`要密切关注服务的创建和销毁方式`。 由于服务没有 UI，因此即使用户切换到另一个应用程序，服务也可以继续在后台运行，而用户无法知道。 这种情况可能会消耗资源并耗尽设备电池。

Similar to an Activity, a service has lifecycle callback methods that you can implement to monitor changes in the service's state and perform work at the appropriate times. The following skeleton service implementation demonstrates each of the lifecycle methods:

与活动类似，服务具有生命周期回调方法，您可以实现这些方法来监视服务状态的变化并在适当的时间执行工作。 以下骨架服务实现演示了每个生命周期方法：

```java
public class ExampleService extends Service {
    // indicates how to behave if the service is killed.
    int mStartMode;  

    // interface for clients that bind.
    IBinder mBinder;

    // indicates whether onRebind should be used   
    boolean mAllowRebind; 

    @Override
    public void onCreate() {
        // The service is being created.
    }

    @Override
    public int onStartCommand(Intent intent, 
        int flags, int startId) {
        // The service is starting, due to a call to startService().
        return mStartMode;
    }

    @Override
    public IBinder onBind(Intent intent) {
        // A client is binding to the service with bindService().
        return mBinder;
    }

    @Override
    public boolean onUnbind(Intent intent) {
        // All clients have unbound with unbindService()
        return mAllowRebind;
    }

    @Override
    public void onRebind(Intent intent) {
        // A client is binding to the service with bindService(),
        // after onUnbind() has already been called
    }

    @Override
    public void onDestroy() {
        // The service is no longer used and is being destroyed
    }
}
```

#### Lifecycle of started services vs. bound services

A bound service exists only to serve the app component that's bound to it, so when no more components are bound to the service, the system destroys it. Bound services don't need to be explicitly stopped the way started services do (using stopService() or stopSelf()).

绑定服务的存在只是为了为与其绑定的应用程序组件提供服务，因此当没有更多组件绑定到该服务时，系统会销毁它。 绑定服务不需要像启动服务那样显式停止（使用 stopService() 或 stopSelf()）。

The diagram below shows a comparison between the started and bound service lifecycles.

下图显示了启动服务生命周期和绑定服务生命周期之间的比较。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d6f925cc4ca426baa49689671ec1d61~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=452&h=482&s=113260&e=png&a=1&b=f9e2bf)

## Foreground services

While most services run in the background, some run in the foreground. A foreground service is a service that the user is aware is running. Although both Activities and Services can be killed if the system is low on memory, a foreground service has priority over other resources.

虽然大多数服务在后台运行，但有些服务在前台运行。 前台服务是用户知道正在运行的服务。 尽管如果系统内存不足，“活动”和“服务”都可能被终止，但前台服务的优先级高于其他资源。

For example, a music player that plays music from a service should be set to run in the foreground, because the user is aware of its operation. The notification in the status bar might indicate the current song and allow the user to launch an Activity to interact with the music player.

例如，从服务播放音乐的音乐播放器应设置为在前台运行，因为用户知道其操作。 状态栏中的通知可能会指示当前歌曲，并允许用户启动 Activity 来与音乐播放器交互。

To request that a service run in the foreground, call startForeground() instead of startService(). This method takes two parameters: an integer that uniquely identifies the notification and the Notification object for the status bar notification. This notification is ongoing, meaning that it can't be dismissed. It stays in the status bar until the service is stopped or removed from the foreground.

要请求服务在前台运行，请调用 startForeground() 而不是 startService()。 该方法有两个参数：唯一标识通知的整数和状态栏通知的Notification对象。 此通知正在进行中，这意味着它无法被忽略。 它会保留在状态栏中，直到服务停止或从前台删除。

For example:

```java
Intent notificationIntent = new Intent(this, ExampleActivity.class);
PendingIntent pendingIntent =
        PendingIntent.getActivity(this, 0, notificationIntent, 0);

Notification notification =
          new Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
    .setContentTitle(getText(R.string.notification_title))
    .setContentText(getText(R.string.notification_message))
    .setSmallIcon(R.drawable.icon)
    .setContentIntent(pendingIntent)
    .setTicker(getText(R.string.ticker_text))
    .build();

startForeground(ONGOING_NOTIFICATION_ID, notification);
```

Note: The integer ID for the notification you give to startForeground() must not be 0.

To remove the service from the foreground, call stopForeground(). This method takes a boolean, indicating whether to remove the status bar notification. This method doesn't stop the service. However, if you stop the service while it's still running in the foreground, then the notification is also removed.

要从前台删除服务，请调用 stopForeground()。 该方法接受一个布尔值，指示是否删除状态栏通知。 此方法不会停止服务。 但是，如果您在服务仍在前台运行时停止该服务，则通知也会被删除。

## Background services and API 26

Services running in the background can consume device resources, potentially using device battery and resulting in a worse user experience. To mitigate this problem, the system now applies limitations on started services running in the background, beginning with Android version 8.0 (Oreo, API 26) or higher. These limitations don't affect foreground services or bound services.

Here are a few of the specific changes:

- The startService() method now throws an IllegalStateException if an app targeting API 26 tries to use that method in a situation when it isn't permitted to create background services.
- The new startForegroundService() method starts a foreground service from an app component. The system allows apps to call this method even while the app is in the background. However, the app must call that service's startForeground() method within five seconds after the service is created.

While an app is in the foreground, it can create and run both foreground and background services freely. When an app goes into the background, it has several minutes in which it is still allowed to create and use services. At the end of that time, the app is considered to be idle. The system stops the app's background services, just as if the app had called the services' Service.stopSelf() methods.

Android 8.0 and higher does not allow a background app to create a background service. Android 8.0 introduces the new method startForegroundService() to start a new service in the foreground. After the system has created the service, the app has five seconds to call the service's startForeground() method to show the new service's user-visible notification. If the app does not call startForeground() within the time limit, the system stops the service and declares the app to be ANR (Application Not Responding).

For more information on these changes, see Background Execution Limits.

## Scheduled services

For API level 21 and higher, you can launch services using the JobScheduler API, and with the background service restrictions in API 26 this may be a better alternative to services altogether. To use JobScheduler, you need to register jobs and specify their requirements for network and timing. The system schedules jobs for execution at appropriate times.

The JobScheduler interface provides many methods to define service-execution conditions. For details, see the JobScheduler reference.

Note: JobScheduler is only available on devices running API 21+, and is not available in the support library. If your app targets devices with earlier API levels, look into the backwards compatible WorkManager, a new API currently in alpha, that allows you to schedule background tasks that need guaranteed completion (regardless of whether the app process is around or not). WorkManager provides JobScheduler-like capabilities to API 14+ devices, even those without Google Play Services.

## 探究Service

## 10.1 Service是什么

Service是Android中实现程序后台运行的解决方案，它非常适合执行那些不需要和用户交互而 且还要求长期运行的任务。

Service并不是运行在一个独立的进程当中的，而是依赖于创建Service时所在的应用程序进程。当某个应用程序进程被杀掉时，所有依赖于该进程的Service也会停止运行。

实际上Service并不会自动开启线程，所有的代码 都是默认运行在主线程当中的。也就是说，我们需要在Service的内部手动创建子线程，并在这 里执行具体的任务，否则就有可能出现主线程被阻塞的情况。

## 10.2 Android多线程编程

#### 10.2.1 线程的基本用法

```kotlin
class MyThread : Runnable {
    override fun run() {
        // 编写具体的逻辑 
    }
}

val myThread = MyThread()
Thread(myThread).start()
```

简化写法:

```kotlin
Thread {
    // 编写具体的逻辑
}.start()

import kotlin.concurrent.thread
thread {
    // 编写具体的逻辑
}
```

#### 10.2.2 在子线程中更新UI

和许多其他的GUI库一样，Android的UI也是线程不安全的

如果想要更新应用程序里的UI元素，必须在主线程中进行，否则就会出现异常。

有些时候，我们必须在子线程里执行一些耗时任务，然后根据任务的执行结果来更新相应的UI控件; 对于这种情况，Android提供了一套异步消息处理机制，完美地解决了在子线程中进行UI操作的问题。

MainActivity

```kotlin
import android.os.Bundle
import android.os.Handler
import android.os.Looper
import android.os.Message
import android.widget.Button
import androidx.appcompat.app.AppCompatActivity
import kotlin.concurrent.thread

class MainActivity : AppCompatActivity() {
    val updateText = 1
    val handler = object : Handler(Looper.getMainLooper()) {
        val textView = findViewById<Button>(R.id.button_count)
        override fun handleMessage(msg: Message) {
            // 在这里可以进行UI操作
            when (msg.what) {
                updateText -> textView.text = "Nice to meet you"
            }
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        val changeTextBtn = findViewById<Button>(R.id.button_count)
        changeTextBtn.setOnClickListener {
            thread {
                val msg = Message()
                msg.what = updateText
                handler.sendMessage(msg) // 将Message对象发送出去
            }
        }
    }
}
```

注意此时 handleMessage() 方法中的代码就是在主线程当中运行的了

#### 10.2.3 解析异步消息处理机制

Android中的异步消息处理主要由4个部分组成:Message、Handler、MessageQueue和 Looper。

1. Message

Message是在线程之间传递的消息，它可以在内部携带少量的信息，用于在不同线程之间 传递数据。

上面的案例中使用到了Message的what字段，除此之外还可以使用arg1和 arg2字段来携带一些整型数据，使用obj字段携带一个Object对象

2. Handler

Handler顾名思义也就是处理者的意思，它主要是用于发送和处理消息的。发送消息一般 是使用Handler的sendMessage()方法、post()方法等，而发出的消息经过一系列地辗 转处理后，最终会传递到Handler的handleMessage()方法中

3. MessageQueue

MessageQueue是消息队列的意思，它主要用于存放所有通过Handler发送的消息。这部 分消息会一直存在于消息队列中，等待被处理。每个线程中只会有一个MessageQueue对 象。

4. Looper

Looper是每个线程中的MessageQueue的管家，调用Looper的loop()方法后，就会进入 一个无限循环当中，然后每当发现MessageQueue中存在一条消息时，就会将它取出，并 传递到Handler的handleMessage()方法中。每个线程中只会有一个Looper对象。

首先需要在主线程当中创建一个Handler对象，并重写 handleMessage()方法。然后当子线程中需要进行UI操作时，就创建一个Message对象，并 通过Handler将这条消息发送出去。之后这条消息会被添加到MessageQueue的队列中等待被 处理，而Looper则会一直尝试从MessageQueue中取出待处理消息，最后分发回Handler的 handleMessage()方法中。

由于Handler的构造函数中我们传入了 Looper.getMainLooper()，所以此时handleMessage()方法中的代码也会在主线程中运 行，于是我们在这里就可以安心地进行UI操作了。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/240e218a839742ebbeb9d8d81df8392b~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=1296&h=948&s=145373&e=png&b=fdfdfd)

一条Message经过以上流程的辗转调用后，也就从子线程进入了主线程，从不能更新UI变成了可以更新UI

#### 使用AsyncTask

AsyncTask 也可以帮助我们在子线程中对UI进行操作。借助AsyncTask，即使你对异步消息处理机制完全不了解，也可以十分简单地从 子线程切换到主线程。当然，AsyncTask背后的实现原理也是基于异步消息处理机制的，只是 Android帮我们做了很好的封装而已。

```kotlin
class MyTask : AsyncTask<Params, Progress, Result>() {
    //...
}
```
1. onPreExecute()

这个方法会在后台任务开始执行之前调用，用于进行一些界面上的初始化操作

2. doInBackground(Params...)

这个方法中的所有代码都会在子线程中运行，我们应该在这里去处理所有的耗时任务。任 务一旦完成，就可以通过return语句将任务的执行结果返回，如果AsyncTask的第三个泛 型参数指定的是Unit，就可以不返回任务执行结果。

注意，在这个方法中是不可以进行UI 操作的，如果需要更新UI元素，比如说反馈当前任务的执行进度，可以调用 publishProgress(Progress...) 方法来完成。

3. onProgressUpdate(Progress...)

当在后台任务中调用了publishProgress(Progress...)方法后， onProgressUpdate (Progress...)方法就会很快被调用，该方法中携带的参数就是 在后台任务中传递过来的。在这个方法中可以对UI进行操作，利用参数中的数值就可以对 界面元素进行相应的更新。

4. onPostExecute(Result)

当后台任务执行完毕并通过return语句进行返回时，这个方法就很快会被调用。返回的数 据会作为参数传递到此方法中，可以利用返回的数据进行一些UI操作，比如说提醒任务执 行的结果，以及关闭进度条对话框等。

```kotin
class DownloadTask : AsyncTask<Unit, Int, Boolean>() {
    override fun onPreExecute() {
        progressDialog.show() // 显示进度对话框
    }

    override fun doInBackground(vararg params: Unit?) = try {
        while (true) {
            val downloadPercent = doDownload() // 这是一个虚构的方法 
            publishProgress(downloadPercent)
            if (downloadPercent >= 100) {
                break
            }
        }
        true
    } catch (e: Exception) {
        false
    }

    override fun onProgressUpdate(vararg values: Int?) {
        // 在这里更新下载进度 
        progressDialog.setMessage("Downloaded ${values[0]}%")
    }

    override fun onPostExecute(result: Boolean) {
        progressDialog.dismiss()// 关闭进度对话框 // 在这里提示下载结果
        if (result) {
            Toast.makeText(context, "Download succeeded", Toast.LENGTH_SHORT).show()
        } else {
            Toast.makeText(context, " Download failed", Toast.LENGTH_SHORT).show()
        }
    }
}
```

使用AsyncTask的诀窍就是，在doInBackground()方法中执行具体的耗时任务， 在onProgressUpdate()方法中进行UI操作，在onPostExecute()方法中执行一些任务的收 尾工作。

```kotin
DownloadTask().execute()
```

当然，也可以给execute()方法传入任意数量的参数，这些参数将会传递到DownloadTask 的doInBackground()方法当中。

## 10.3 Service的基本用法

#### 10.3.1 定义一个Service

```kotlin
import android.app.Service
import android.content.Intent
import android.os.IBinder

class MyService : Service() {
    override fun onBind(intent: Intent): IBinder {
        TODO("Return the communication channel to the service.")
    }

    override fun onCreate() {
        super.onCreate()
    }

    override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
        return super.onStartCommand(intent, flags, startId)
    }

    override fun onDestroy() {
        super.onDestroy()
    }
}
```

onCreate()方法会在Service创建的 时候调用，
onStartCommand()方法会在每次Service启动的时候调用，
onDestroy()方法 会在Service销毁的时候调用。

通常情况下，如果我们希望Service一旦启动就立刻去执行某个动作，就可以将逻辑写在 onStartCommand()方法里。而当Service销毁时，我们又应该在onDestroy()方法中回收 那些不再使用的资源。

onCreate()方法是在Service第一次创建的时候调用的，而onStartCommand()方法则 在每次启动Service的时候都会调用。

在AndroidManifest.xml文件中进行注册

```xml
<service
    android:name=".MyService"
    android:enabled="true"
    android:exported="true">
</service>
```

#### 10.3.2 启动和停止Service

动和停止的方法主要是借助Intent来实现的。

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        startServiceBtn.setOnClickListener {
            // 启动Service 
            val intent = Intent(this, MyService::class.java)
            startService(intent) 
        }
        stopServiceBtn.setOnClickListener {
            // 停止Service
            val intent = Intent(this, MyService::class.java)
            stopService(intent) 
        }
    }
}
```

另外，Service也可以自我停止运行，只需要在Service内部调用stopSelf()方法即 可。

但是从Android 8.0系统开始，应用的后台功能被大 幅削减。现在只有当应用保持在前台可见状态的情况下，Service才能保证稳定运行，一旦应用 进入后台之后，Service随时都有可能被系统回收。之所以做这样的改动，是为了防止许多恶意的应用程序长期在后台占用手机资源，从而导致手机变得越来越卡。当然，如果你真的非常需 要长期在后台执行一些任务，可以使用前台Service或者WorkManager，前台Service我们待 会马上就会学到，而WorkManager将会在第13章中进行学习。

#### 10.3.3 Activity和Service进行通信

我们在Activity里调用了startService()方法来启动MyService，然后 MyService的onCreate()和onStartCommand()方法就会得到执行。之后Service会一直处 于运行状态，但具体运行的是什么逻辑，Activity就控制不了了。

可不可以让Activity和Service的关系更紧密一些呢?例如在Activity中指挥Service去干什 么，Service就去干什么。当然可以，需要 onBind() 方法

```kotlin
class MyService2 : Service() {
    private val mBinder = DownloadBinder()

    class DownloadBinder : Binder() {
        fun startDownload() {
            Log.d("MyService", "startDownload executed")
        }

        fun getProgress(): Int {
            Log.d("MyService", "getProgress executed")
            return 0
        }
    }

    override fun onBind(intent: Intent): IBinder {
        return mBinder
    }
    //    ... 
}

class MainActivity2 : AppCompatActivity() {
    lateinit var downloadBinder: MyService2.DownloadBinder
    private val connection = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName, service: IBinder) {
            downloadBinder = service as MyService2.DownloadBinder
            downloadBinder.startDownload()
            downloadBinder.getProgress()
        }

        override fun onServiceDisconnected(name: ComponentName) {
        }
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        bindServiceBtn.setOnClickListener {
            val intent = Intent(this, MyService::class.java)
            bindService(intent, connection, Context.BIND_AUTO_CREATE) // 绑定Service
        }
        unbindServiceBtn.setOnClickListener {
            unbindService(connection) // 解绑Service
        }
    }
}
```

这里我们首先创建了一个ServiceConnection的匿名类实现，并在里面重写了 onServiceConnected()方法和onServiceDisconnected()方法。

onServiceConnected()方法方法会在Activity与Service成功绑定的时候调用，而 onServiceDisconnected()方法只有在Service的创建进程崩溃或者被杀掉的时候才会调 用

那么在onServiceConnected()方法中，我们又通过向下转型得到 了DownloadBinder的实例，有了这个实例，Activity和Service之间的关系就变得非常紧密 了。现在我们可以在Activity中根据具体的场景来调用DownloadBinder中的任何public方 法，即实现了指挥Service干什么Service就去干什么的功能。

当然，现在Activity和Service其实还没进行绑定呢，这个功能是在 bindService()方 法将MainActivity和MyService进行绑定。bindService()方法接收3个参数，第一个参数就 是刚刚构建出的Intent对象，第二个参数是前面创建出的ServiceConnection的实例，第三 个参数则是一个标志位，这里传入BIND_AUTO_CREATE表示在Activity和Service进行绑定后 自动创建Service。这会使得MyService中的onCreate()方法得到执行，但 onStartCommand()方法不会执行。

另外需要注意，任何一个Service在整个应用程序范围内都是通用的，即MyService不仅可以和 MainActivity绑定，还可以和任何一个其他的Activity进行绑定，而且在绑定完成后，它们都可 以获取相同的DownloadBinder实例。

## 10.4 Service的生命周期

一旦在项目的任何位置调用了Context的startService()方法，相应的Service就会启动， 并回调onStartCommand()方法。如果这个Service之前还没有创建过，onCreate()方法会 先于onStartCommand()方法执行。

Service启动了之后会一直保持运行状态，直到 stopService()或stopSelf()方法被调用，或者被系统回收。

另外，还可以调用Context的bindService()来获取一个Service的持久连接，这时就会回调 Service中的onBind()方法。如果这个Service之前还没有创建过，onCreate()方 法会先于onBind()方法执行。

当调用了startService()方法后，再去调用stopService()方法。这时Service中的 onDestroy()方法就会执行，表示Service已经销毁了。类似地，当调用了bindService() 方法后，再去调用unbindService()方法，onDestroy()方法也会执行

需要注意，我们是完全有可能对一个Service既调用了startService()方法，又 调用了bindService()方法的，在这种情况下该如何让Service销毁呢?根据Android系统的 机制，一个Service只要被启动或者被绑定了之后，就会处于运行状态，必须要让以上两种条件 同时不满足，Service才能被销毁。所以，这种情况下要同时调用stopService()和 unbindService()方法，onDestroy()方法才会执行。

## 10.5 Service的更多技巧

#### 10.5.1 使用前台Service

从Android 8.0系统开始，只有当应用保持在前台可见状态的情况下，Service 才能保证稳定运行，一旦应用进入后台之后，Service随时都有可能被系统回收。而如果你希望 Service能够一直保持运行状态，就可以考虑使用前台Service。前台Service和普通Service最 大的区别就在于，它一直会有一个正在运行的图标在系统的状态栏显示，下拉状态栏后可以看 到更加详细的信息，非常类似于通知的效果

由于状态栏中一直有一个正在运行的图标，相当于我们的应用以另外一种形式保持在前台可见 状态，所以系统不会倾向于回收前台Service。另外，用户也可以通过下拉状态栏清楚地知道当 前什么应用正在运行，因此也不存在某些恶意应用长期在后台偷偷占用手机资源的情况。

创建一个前台Service

```kotlin
class MyService3 : Service() {
    override fun onCreate() {
        super.onCreate()
        Log.d("MyService", "onCreate executed")
        val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "my_service",
                "前台Service通知",
                NotificationManager.IMPORTANCE_DEFAULT
            )
            manager.createNotificationChannel(channel)
        }
        val intent = Intent(this, MainActivity::class.java)
        val pi = PendingIntent.getActivity(this, 0, intent, PendingIntent.FLAG_IMMUTABLE)
        val notification = NotificationCompat.Builder(this, "my_service")
            .setContentTitle("This is content title")
            .setContentText("This is content text")
            .setSmallIcon(R.drawable.ic_launcher_foreground)
            .setLargeIcon(BitmapFactory.decodeResource(resources, R.drawable.ic_launcher_background))
            .setContentIntent(pi)
            .build()
        startForeground(1, notification)
    }

    override fun onBind(intent: Intent?): IBinder? {
        TODO("Not yet implemented")
    }
}
```

使用前台Service必须在AndroidManifest.xml文件中进行权 限声明才行

```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
```

#### 10.5.2 使用IntentService

在本章一开始的时候我们就已经知道，Service中的代码都是默认运行在主线程当中 的，如果直接在Service里处理一些耗时的逻辑，就很容易出现ANR(Application Not Responding)的情况。这个时候就需要用到Android多线程编程的技术了


一个比较标准的Service就可以写 成如下形式:
```
class MyService : Service() {
    //...
    override fun onStartCommand(intent: Intent, flags: Int, startId: Int): Int {
        thread {
            // 处理具体的逻辑 

            stopSelf()
        }
        return super.onStartCommand(intent, flags, startId)
    }
}
```

虽说这种写法并不复杂，但是总会有一些程序员忘记开启线程，或者忘记调用stopSelf()方 法。为了可以简单地创建一个异步的、会自动停止的Service，Android专门提供了一个 IntentService类

```kotlin
class MyIntentService : IntentService("MyIntentService") {
    override fun onHandleIntent(intent: Intent?) {
        // 打印当前线程的id
        Log.d("MyIntentService", "Thread id is ${Thread.currentThread().name}")
    }
    override fun onDestroy() {
        super.onDestroy()
        Log.d("MyIntentService", "onDestroy executed")
    } 
}
```

实现onHandleIntent()这个抽象方法，这个方法中可以 处理一些耗时的逻辑，而不用担心ANR的问题，因为这个方法已经是在子线程中运行的了


