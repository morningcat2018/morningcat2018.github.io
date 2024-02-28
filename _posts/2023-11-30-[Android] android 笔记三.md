---
layout:     post
title:      Activity lifecycle and state
subtitle:   
date:       2023-11-30
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Android
---

[Android developer document - 2.2: Activity lifecycle and state](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-1-get-started/lesson-2-activities-and-intents/2-2-c-activity-lifecycle-and-state/2-2-c-activity-lifecycle-and-state.html)


## Introduction

## About the Activity lifecycle

The activity lifecycle is the set of states an activity can be in during its entire lifetime, from the time it's created to when it's destroyed and the system reclaims its resources. As the user interacts with your app and other apps on the device, activities move into different states.

activity 生命周期是一个 activity 在其整个生命周期中可以处于的状态集，从它被创建到它被销毁以及系统回收其资源。 当用户与您的应用程序和设备上的其他应用程序交互时，活动会进入不同的状态。

- When you start an app, the app's main activity is started, comes to the foreground, and receives the user focus. 当您启动应用程序时，应用程序的 Main Activity 将启动，进入前台并接收用户焦点。
- When you start a second activity, a new activity is created and started, and the main activity is stopped. 当您启动第二个 Activity 时，将创建并启动一个新 Activity，并停止 Main Activity。
- When you're done with the Activity 2 and navigate back, Activity 1 resumes(继续). Activity 2 stops(停止) and is no longer needed.
- If the user doesn't resume Activity 2, the system eventually destroys(销毁) it.


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ffb2f73def1d4a57a43454f64341f53d~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=666&h=334&s=34739&e=png&a=1&b=f1f1f1)

## Activity states and lifecycle callback methods

When an Activity transitions into and out of the different lifecycle states as it runs, the Android system calls several lifecycle callback methods at each stage. All of the callback methods are hooks that you can override in each of your Activity classes to define how that Activity behaves when the user leaves and re-enters the Activity. Keep in mind that the lifecycle states (and callbacks) are per Activity, not per app, and you may implement different behavior at different points in the lifecycle of each Activity.

当 Activity 在运行时进入和退出不同的生命周期状态时，Android 系统会在每个阶段调用多个生命周期回调方法。 所有回调方法都是钩子，您可以在每个 Activity 类中重写它们，以定义当用户离开并重新进入 Activity 时该 Activity 的行为方式。 请记住，生命周期状态（和回调）是针对每个 Activity 的，而不是针对每个应用程序的，并且您可以在每个 Activity 生命周期的不同点实现不同的行为。

This figure shows each of the Activity states and the callback methods that occur as the Activity transitions between different states:

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f8ba4dbda8b4c1281e09ee4297dba9d~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=662&h=295&s=65593&e=png&a=1&b=dfe36c)

Depending on the complexity of your Activity, you probably don't need to implement all the lifecycle callback methods in an Activity. However, it's important that you understand each one and implement those that ensure your app behaves the way users expect. Managing the lifecycle of an Activity by implementing callback methods is crucial to developing a strong and flexible app.

可能不需要在活动中实现所有生命周期回调方法。然而，了解每一项并实施这些措施以确保您的应用程序按照用户期望的方式运行非常重要。

#### Activity created: the onCreate() method

```java
@Override
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // The activity is being created.
}
```

Your Activity enters into the created state when it is started for the first time. When an Activity is first created, the system calls the onCreate() method to initialize that Activity. For example, when the user taps your app icon from the Home screen to start that app, the system calls the onCreate() method for the Activity in your app that you've declared to be the "launcher" or "main" Activity. In this case the main Activity onCreate() method is analogous to the main() method in other programs.

当您的 Activity 第一次启动时，它会进入已创建状态。 当一个Activity第一次被创建时，系统会调用onCreate()方法来初始化该Activity。 例如，当用户从主屏幕点击您的应用程序图标来启动该应用程序时，系统会为您声明为“启动器”或“主Activity” 的应用程序中的 Activity 调用 onCreate() 方法。 在这种情况下，主 Activity 的 onCreate() 方法类似于其他程序中的 main() 方法。

Similarly, if your app starts another Activity with an Intent (either explicit or implicit), the system matches your Intent request with an Activity and calls onCreate() for that new Activity.

同样，如果您的应用程序使用 Intent（显式或隐式）启动另一个 Activity，系统会将您的 Intent 请求与 Activity 进行匹配，并为该新 Activity 调用 onCreate() 。

The onCreate() method is the only required callback you must implement in your Activity class. In your onCreate() method you perform basic app startup logic that should happen only once, such as setting up the user interface, assigning class-scope variables, or setting up background tasks.

onCreate() 方法是您必须在 Activity 类中实现的`唯一必需回调`。 在 onCreate() 方法中，您执行`只应发生一次的`基本应用程序启动逻辑，例如设置用户界面、分配类范围变量或设置后台任务。

Created is a transient state; the Activity remains in the created state only as long as it takes to run onCreate(), and then the Activity moves to the started state.

创建是一个短暂的状态； Activity `仅在运行 onCreate() 时保持已创建状态`，然后 Activity 就会转至`启动状态`。


#### Activity started: the onStart() method (启动状态)

```java
@Override
protected void onStart() {
    super.onStart();
    // The activity is about to become visible.
}
```

After your Activity is initialized with onCreate(), the system calls the onStart() method, and the Activity is in the started state. The onStart() method is also called if a stopped Activity returns to the foreground, such as when the user clicks the Back button or the Up button to navigate to the previous screen. While onCreate() is called only once when the Activity is created, the onStart() method may be called many times during the lifecycle of the Activity as the user navigates around your app.

当你的Activity用onCreate()初始化后，系统会调用onStart()方法，此时Activity就处于启动状态。 如果停止的 Activity 返回前台，例如当用户单击“后退”按钮或“向上”按钮导航到上一屏幕时，也会调用 onStart() 方法。 虽然 onCreate() 在创建 Activity 时仅调用一次，但当用户在应用程序中导航时，`onStart() 方法可能会在 Activity 的生命周期内被调用多次`。

When an Activity is in the started state and visible on the screen, the user cannot interact with it until onResume() is called, the Activity is running, and the Activity is in the foreground.

当 Activity `处于启动状态并且在屏幕上可见时，用户无法与其交互`，直到调用 onResume()、Activity 正在运行且 Activity 位于前台。

Typically you implement onStart() in your Activity as a counterpart to the onStop() method. For example, if you release hardware resources (such as GPS or sensors) when the Activity is stopped, you can re-register those resources in the onStart() method.

通常，您在 Activity 中实现 onStart() 作为 onStop() 方法的对应方法。 例如，如果您在 Activity 停止时释放了硬件资源（例如 GPS 或传感器），则可以在 onStart() 方法中`重新注册这些资源`。

Started, like created, is a transient state. After starting, the Activity moves into the resumed (running) state.

`启动状态`和`创建状态`一样，是一个`短暂`的状态。 启动后，活动进入`恢复（运行）状态`。

#### Activity resumed/running: the onResume() method   (运行/恢复状态)

```java
@Override
protected void onResume() {
    super.onResume();
    // The activity has become visible (it is now "resumed").
}
```

Your Activity is in the resumed state when it is initialized, visible on screen, and ready to use. The resumed state is often called the running state, because it is in this state that the user is actually interacting with your app.

当您的 Activity 初始化、在屏幕上可见且可供使用时，它就处于`恢复状态`。 恢复状态通常称为`运行状态`，因为在这种状态下用户实际上正在与您的应用程序进行交互。

The first time the Activity is started the system calls the onResume() method just after onStart(). The onResume() method may also be called multiple times, each time the app comes back from the paused state.

第一次启动 Activity 时，系统会在 onStart() 之后调用 onResume() 方法。 onResume() 方法也`可以在每次应用程序从暂停状态返回时被调用多次`。

As with the onStart() and onStop() methods, which are implemented in pairs, you typically only implement onResume() as a counterpart to onPause(). For example, if in the onPause() method you halt any animations, you would start those animations again in onResume().

与成对实现的 onStart() 和 onStop() 方法一样，您通常只实现 onResume() 作为 onPause() 的对应项。 例如，如果在 onPause() 方法中停止任何动画，则可以在 onResume() 中再次启动这些动画。

The Activity remains in the resumed state as long as the Activity is in the foreground and the user is interacting with it. From the resumed state the Activity can move into the paused state.

只要 Activity 位于前台并且用户正在与其交互，该 Activity 就会保持在已恢复状态。 

Activity 可以从恢复状态进入暂停状态。


#### Activity paused: the onPause() method 暂停状态

```java
@Override
protected void onPause() {
    super.onPause();
    // Another activity is taking focus 
    // (this activity is about to be "paused").
}
```

The paused state can occur in several situations: 暂停状态可能在多种情况下发生：

- The Activity is going into the background, but has not yet been fully stopped. This is the first indication that the user is leaving your Activity.
    - Activity 正在进入后台，但尚未完全停止。 这是用户离开您的活动的第一个迹象。
- The Activity is only partially visible on the screen, because a dialog or other transparent Activity is overlaid on top of it.
    - 该活动在屏幕上仅部分可见，因为对话框或其他透明活动覆盖在其顶部。
- In multi-window or split screen mode (API 24), the Activity is displayed on the screen, but some other Activity has the user focus.
    - 在多窗口或分屏模式（API 24）下，Activity 显示在屏幕上，但其他一些 Activity 具有用户焦点。

The system calls the onPause() method when the Activity moves into the paused state. Because the onPause() method is the first indication you get that the user may be leaving the Activity, you can use onPause() to stop animation or video playback, release any hardware-intensive resources, or commit unsaved Activity changes (such as a draft email).

当 Activity 进入暂停状态时，系统会调用 onPause() 方法。 因为 onPause() 方法是您获得的用户可能要离开 Activity 的第一个指示，所以您可以使用 onPause() 停止动画或视频播放、释放任何硬件密集型资源或提交未保存的 Activity 更改（例如 电子邮件草稿）。

The onPause() method should execute quickly. Don't use onPause() for CPU-intensive operations such as writing persistent data to a database. The app may still be visible on screen as it passes through the paused state, and any delays in executing onPause() can slow the user's transition to the next Activity. Implement any heavy-load operations when the app is in the stopped state instead.

onPause() 方法`应该快速执行`。 不要将 onPause() 用于 CPU 密集型操作，例如将持久数据写入数据库。 当应用程序经过暂停状态时，它在屏幕上可能仍然可见，并且执行 onPause() 时的任何延迟都会减慢用户到下一个 Activity 的转换速度。 当应用程序处于停止状态时实施任何重负载操作。

Note that in multi-window mode (API 24), your paused Activity may still fully visible on the screen. In this case you do not want to pause animations or video playback as you would for a partially visible Activity. You can use the inMultiWindowMode() method in the Activity class to test whether your app is running in multi-window mode.

请注意，在多窗口模式（API 24）下，暂停的 Activity 可能仍然在屏幕上完全可见。 在这种情况下，您不想像部分可见的 Activity 那样暂停动画或视频播放。 您可以使用 Activity 类中的 inMultiWindowMode() 方法来测试您的应用程序是否在多窗口模式下运行。

Your Activity can move from the paused state into the resumed state (if the user returns to the Activity) or to the stopped state (if the user leaves the Activity altogether).

您的 Activity 可以从暂停状态进入恢复状态（如果用户返回到 Activity）或停止状态（如果用户完全离开 Activity）。

#### Activity stopped: the onStop() method 停止状态

```java
@Override
protected void onStop() {
    super.onStop();
    // The activity is no longer visible (it is now "stopped")
}
```

An Activity is in the stopped state when it's no longer visible on the screen. This is usually because the user started another activity or returned to the home screen. The Android system retains the activity instance in the back stack, and if the user returns to the activity, the system restarts it. If resources are low, the system might kill a stopped activity altogether.

当 Activity 在屏幕上不再可见时，它就处于停止状态。 这通常是因为用户开始了另一个活动或返回到主屏幕。 Android系统将Activity实例保留在返回栈中，如果用户返回到该Activity，系统会重新启动它。 如果资源不足，系统可能会完全终止已停止的活动。

The system calls the onStop() method when the activity stops. Implement the onStop() method to save persistent data and release resources that you didn't already release in onPause(), including operations that may have been too heavyweight for onPause().

当 Activity 停止时，系统会调用 onStop() 方法。 实现 onStop() 方法来保存持久数据并释放尚未在 onPause() 中释放的资源，包括对于 onPause() 来说可能`过于繁重的操作`。

#### Activity destroyed: the onDestroy() method

```java
@Override
protected void onDestroy() {
    super.onDestroy();
    // The activity is about to be destroyed.
}
```

When your Activity is destroyed it is shut down completely, and the Activity instance is reclaimed by the system. This can happen in several cases:

当您的 Activity 被销毁时，它会完全关闭，并且 Activity 实例将被系统回收。 这可能在多种情况下发生：

- You call finish() in your Activity to manually shut it down.
您可以在 Activity 中调用 `finish()` 来手动关闭它。
- The user navigates back to the previous Activity.
用户`导航回上一个 Activity`。
- The device is in a low memory situation where the system reclaims any stopped Activity to free more resources.
`设备内存不足，系统会回收所有已停止的 Activity` 以释放更多资源。
- A device configuration change occurs.
设备配置发生更改。

Use onDestroy() to fully clean up after your Activity so that no component (such as a thread) is running after the Activity is destroyed.

使用 onDestroy() 在您的 Activity 后进行完全清理，以便在 Activity 销毁后没有任何组件（例如线程）在运行。

Note that there are situations where the system will simply kill the hosting process for the Activity without calling this method (or any others), so you should not rely on onDestroy() to save any required data or Activity state. Use onPause() or onStop() instead.

请`注意`，在某些情况下，系统将简单地终止 Activity 的托管进程，而`不调用此方法`（或任何其他方法），因此您`不应依赖 onDestroy()` 来保存任何所需的数据或 Activity 状态。 请改用 onPause() 或 onStop()。

#### Activity restarted: the onRestart() method

```java
@Override
protected void onRestart() {
    super.onRestart();
    // The activity is about to be restarted.
}
```

The restarted state is a transient state that only occurs if a stopped Activity is started again. In this case the onRestart() method is called between onStop() and onStart(). If you have resources that need to be stopped or started you typically implement that behavior in onStop() or onStart() rather than onRestart().

重新启动状态是一种暂时状态，仅在已停止的 Activity 再次启动时才会发生。 在这种情况下，onRestart() 方法在 onStop() 和 onStart() 之间调用。 如果您有需要停止或启动的资源，您通常在 onStop() 或 onStart() 而不是 onRestart() 中实现该行为。

## Configuration changes and Activity state

Earlier in the section on onDestroy() you learned that your Activity may be destroyed when the user navigates back, or when your code executes the finish() method, or when the system needs to free resources. Another way an Activity can be destroyed is when the device undergoes a configuration change.

在前面关于 onDestroy() 的部分中，您了解到，当用户导航回来时，或者当您的代码执行 finish() 方法时，或者当系统需要释放资源时，您的 Activity 可能会被销毁。 Activity 被销毁的另一种方式是`当设备发生配置更改时`。

Configuration changes occur on the device, in runtime, and invalidate the current layout or other resources in your Activity. The most common form of a configuration change is when the device is rotated. When the device rotates from portrait to landscape, or from landscape to portrait, the layout for your app needs to change. The system recreates the Activity to help that Activity adapt to the new configuration by loading alternative resources (such as a landscape-specific layout).

配置更改发生在运行时的设备上，并使 Activity 中的当前布局或其他资源无效。 配置更改的`最常见形式是旋转设备`。 当设备从纵向旋转到横向，或从横向旋转到纵向时，应用程序的布局需要更改。 系统重新创建 Activity，以通过加载替代资源（例如特定于景观的布局）来帮助该 Activity 适应新配置。

Other configuration changes can include a change in locale (the user chooses a different system language), or the user enters multi-window mode (Android 7). In multi-window mode, if you have configured your app to be resizeable, Android recreates the Activity to use a layout definition for the new, smaller size.

其他配置更改可以包括区域设置的更改（用户选择不同的系统语言），或者用户进入多窗口模式（Android 7）。 在多窗口模式下，如果您已将应用程序配置为可调整大小，Android 会重新创建 Activity 以使用新的较小尺寸的布局定义。

When a configuration change occurs, the Android system shuts down your activity, calling onPause(), onStop(), and onDestroy(). Then the system restarts the activity from the beginning, calling onCreate(), onStart(), and onResume().

当配置发生更改时，Android 系统会调用 onPause()、onStop() 和 onDestroy() 来关闭您的 Activity。 然后系统从头开始重新启动 Activity，调用 onCreate()、onStart() 和 onResume()。

#### Activity instance state 实例状态

When an Activity is destroyed and recreated, there are implications for the runtime state of that Activity. When an Activity is paused or stopped, the state of the Activity is retained because that Activity is still held in memory. When an Activity is recreated, the state of the Activity and any user progress in that Activity is lost, with these exceptions:

当一个 Activity 被销毁并重新创建时，该 Activity 的运行时状态会受到影响。 当 Activity 暂停或停止时，该 Activity 的状态将被保留，因为该 Activity 仍保存在内存中。 重新创建 Activity 时，该 Activity 的状态以及该 Activity 中的任何用户进度都会丢失，但以下情况除外：

Some Activity state information is automatically saved by default. The state of View elements in your layout with a unique ID (as defined by the android:id attribute in the layout) are saved and restored when an Activity is recreated. In this case, the user-entered values in EditText elements are usually retained when the Activity is recreated.

默认情况下会自动保存一些 Activity 状态信息。 重新创建 Activity 时，布局中具有唯一 ID（由布局中的 android:id 属性定义）的 View 元素的状态将被保存和恢复。 在这种情况下，重新创建 Activity 时通常会保留 EditText 元素中用户输入的值。

The Intent that was used to start the Activity, and the information stored in the data or extras for that Intent, remains available to that Activity when it is recreated.

用于启动 Activity 的 Intent 以及存储在该 Intent 的数据或附加内容中的信息在该 Activity 重新创建时仍然可用。

The Activity state is stored as a set of key/value pairs in a Bundle object called the Activity instance state . The system saves default state information to instance state Bundle just before the Activity is stopped, and passes that Bundle to the new Activity instance to restore.

Activity 状态作为一组键/值对存储在称为 Activity 实例状态的 Bundle 对象中。 系统在 Activity 停止之前将默认状态信息保存到实例状态 Bundle，并将该 Bundle 传递给新的 Activity 实例进行恢复。

You can add your own instance data to the instance state Bundle by overriding the onSaveInstanceState() callback. The state Bundle is passed to the onCreate() method, so you can restore that instance state data when your Activity is created. There is also a corresponding onRestoreInstanceState() callback you can use to restore the state data.

您可以通过重写 onSaveInstanceState() 回调将自己的实例数据添加到实例状态 Bundle。 状态 Bundle 被传递给 onCreate() 方法，因此您可以在创建 Activity 时恢复该实例状态数据。 还有一个相应的 onRestoreInstanceState() 回调可用于恢复状态数据。

Test that your Activity behaves correctly when the user rotates the device, because device rotation is a common use case. Implement instance state if you need to.

测试当用户旋转设备时您的 Activity 行为是否正确，因为设备旋转是常见的用例。 如果需要，请实现实例状态。

Note: The Activity instance state is particular to a specific instance of an Activity, running in a single task. If the user force-quits the app or reboots the device, or if the Android system shuts down the app process to preserve memory, the Activity instance state is lost. To keep state changes across app instances and device reboots, you need to write that data to shared preferences. You learn more about shared preferences in another chapter.

注意：Activity 实例状态特定于在单个任务中运行的特定 Activity 实例。 如果用户强制退出应用程序或重新启动设备，或者 Android 系统关闭应用程序进程以保留内存，则 Activity 实例状态将丢失。 要在应用程序实例和设备重新启动之间保持状态更改，您需要将该数据写入共享首选项。 您将在另一章中了解有关共享偏好的更多信息。

#### Saving Activity instance state

To save information to the instance state Bundle, use the onSaveInstanceState() callback. This is not a lifecycle callback method, but it is called when the user is leaving your Activity (sometime before the onStop() method).

要将信息保存到实例状态 Bundle，请使用 onSaveInstanceState() 回调。 这不是生命周期回调方法，但它会在用户离开 Activity 时调用（有时在 onStop() 方法之前）。

```java
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    super.onSaveInstanceState(savedInstanceState);
    // save your state data to the instance state bundle
}
```

The onSaveInstanceState() method is passed a Bundle object (a collection of key/value pairs) when it is called. This is the instance state Bundle to which you will add your own Activity state information.

调用 onSaveInstanceState() 方法时，会向其传递一个 Bundle 对象（键/值对的集合）。 这是实例状态包，您将向其中添加您自己的活动状态信息。

You learned about Bundle in a previous chapter when you added keys and values to the Intent extras. Add information to the instance state Bundle in the same way, with keys you define and the various "put" methods defined in the Bundle class:

在2.1章中向 Intent extras 添加键和值时了解了 Bundle。 使用您定义的键和 Bundle 类中定义的各种“put”方法，以相同的方式将信息添加到实例状态 Bundle：

```java
@Override
public void onSaveInstanceState(Bundle savedInstanceState) {
    super.onSaveInstanceState(savedInstanceState);

    // Save the user's current game state
    savedInstanceState.putInt("score", mCurrentScore);
    savedInstanceState.putInt("level", mCurrentLevel);
}
```

Don't forget to call through to the superclass, to make sure the state of the View hierarchy is also saved to the Bundle.

不要忘记调用超类，以确保 View 层次结构的状态也保存到 Bundle 中。

#### Restoring Activity instance state

Once you've saved the Activity instance state, you also need to restore it when the Activity is recreated. You can do this one of two places:

保存 Activity 实例状态后，您还需要在重新创建 Activity 时恢复它。 您可以在以下两个位置之一执行此操作：

- The onCreate() callback method, which is called with the instance state Bundle when the Activity is created.
- The onRestoreInstanceState() callback, which is called after onStart() after the Activity is created.

Most of the time the better place to restore the Activity state is in onCreate(), to ensure that your UI, including the state, is available as soon as possible.

大多数时候，恢复 Activity 状态的更好位置是在 onCreate() 中，以确保您的 UI（包括状态）尽快可用。

To restore the saved instances state in onCreate(), test for the existence of a state Bundle before you try to get data out of it. When your Activity is started for the first time there will be no state and the Bundle will be null.

要恢复 onCreate() 中保存的实例状态，请在尝试从中获取数据之前测试状态束是否存在。 当您的 Activity 第一次启动时，不会有任何状态，并且 Bundle 将为 null。

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
    // Always call the superclass first
    super.onCreate(savedInstanceState); 

    // Check if recreating a previously destroyed instance.
    if (savedInstanceState != null) {
        // Restore value of members from saved state.
        mCurrentScore = savedInstanceState.getInt("score");
        mCurrentLevel = savedInstanceState.getInt("level");
    } else {
        // Initialize members with default values for a new instance.
        // ...
    }
    // ... Rest of code
}
```