---
layout:     post
title:      Activities and intents
subtitle:   
date:       2023-11-30
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Android
---

2.1: Activities and intents

[Android developer document - 2.1: Activities and intents ](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-1-get-started/lesson-2-activities-and-intents/2-1-c-activities-and-intents/2-1-c-activities-and-intents.html)

## About activities

An activity represents a single screen in your app with an interface the user can interact with.

一个 Activity 代表应用程序中的一个屏幕，其中包含用户可以与之交互的界面。

Your app is probably a collection of activities that you create yourself, or that you reuse from other apps.

您的应用程序可能是您自己创建的或从其他应用程序重用的活动的集合。

Although the activities in your app work with each other to form a cohesive user experience, each activity is independent of the others.

尽管应用程序中的活动相互协作以形成一致的用户体验，但每个活动都是独立于其他活动的。

Typically, one Activity in an app is specified as the "main" activity, for example MainActivity. The user sees the main activity when they launch the app for the first time.

通常，应用中的一个 Activity 被指定为“主”Activity，例如 MainActivity。用户首次启动应用程序时会看到“主”Activity。

Each activity can start other activities to perform different actions.

每个活动都可以启动其他活动来执行不同的操作。

Each time a new activity starts, the previous activity is stopped, but the system preserves the activity in a stack (the "back stack"). 

每次启动新活动时，前一个活动都会停止，但系统会将该活动保留在堆栈（“后退堆栈”）中。

When the user is done with the current activity and presses the Back button, the activity is popped from the stack and destroyed, and the previous activity resumes.

当用户完成当前活动并按下“后退”按钮时，该活动将从堆栈中弹出并销毁，并恢复上一个活动。

When an activity is stopped because a new activity starts, the first activity is notified by way of the activity lifecycle callback methods.

当一个活动因新活动启动而停止时，第一个活动会通过活动生命周期回调方法收到通知。

The activity lifecycle is the set of states an Activity can be in: when the activity is first created, when it's stopped or resumed, and when the system destroys it.

Activity 生命周期是 Activity 可以处于的状态集：首次创建 Activity 时、停止或恢复 Activity 时、系统销毁 Activity 时。

## Creating an Activity

To implement an Activity in your app, do the following:

- Create an `Activity` Java class.
- Implement a basic UI for the Activity in an XML layout file.
  - 例如: app/src/main/res/layout/activity_main.xml
- Declare the new Activity in the `AndroidManifest.xml` file.
  - 例如: app/src/main/AndroidManifest.xml

#### Create an Activity Java class.

When you create a new project in Android Studio and choose the `Backwards Compatibility (AppCompat)` option, the MainActivity is, by default, a subclass of the `AppCompatActivity` class.

创建新项目并选择“向后兼容性”(AppCompat) 选项时

The AppCompatActivity class lets you use up-to-date Android app features such as the app bar and Material Design, while still enabling your app to be compatible with devices running older versions of Android.

AppCompatActivity 类允许您使用最新的 Android 应用程序功能，同时仍然使您的应用程序能够与运行旧版 Android 的设备兼容。

```java
public class MainActivity extends AppCompatActivity {
   @Override
   protected void onCreate(Bundle savedInstanceState) {
       super.onCreate(savedInstanceState);
       setContentView(R.layout.activity_main);
   }
}
```

The first task for you in your Activity subclass is to implement the standard Activity lifecycle callback methods to handle the state changes for your Activity. These state changes include things such as when the Activity is created, stopped, resumed, or destroyed.

在 Activity 子类中，您的第一个任务是实现`标准 Activity 生命周期回调方法`来处理 `Activity 的状态更改`。 这些状态更改包括 Activity 何时创建、停止、恢复或销毁等。

The one required callback your app `must implement` is the `onCreate()` method. The system calls this method when it creates your Activity, and all the `essential components 基本组件` of your Activity should be initialized here. 
Most importantly, the onCreate() method calls `setContentView()` to create the `primary layout` for the Activity.

You typically define the UI for your Activity in one or more XML layout files. `When the setContentView() method is called with the path to a layout file, the system creates all the initial views from the specified layout and adds them to your Activity.` This is often referred to as inflating the layout.

通常会在一个或多个 XML 布局文件中定义 Activity 的 UI。当使用布局文件的路径调用 setContentView() 方法时，系统会根据指定的布局创建所有初始视图并将它们添加到您的 Activity 中。这通常称为膨胀布局。

You may often also want to implement the onPause() method in your Activity. The system calls this method as the first indication that the user is leaving your Activity (though it does not always mean that the Activity is being destroyed).

系统调用 onPause() 方法作为用户正在离开您的 Activity 的`第一个指示`

This is usually where you should commit any changes that should be persisted beyond the current user session (because the user might not come back).

`通常您应该在此处提交应在当前用户会话之外保留的任何更改`（因为用户可能不会回来）。

In addition to lifecycle callbacks, you may also implement methods in your Activity to handle other behavior such as user input or button clicks.

除..之外

#### Implement a basic UI for the Activity in an XML layout file

The UI for an activity is provided by a hierarchy of View elements, which controls a particular space within the activity window and can respond to user interaction.

Activity 的 UI 由 View 元素的层次结构提供，它控制活动窗口内的特定空间并可以响应用户交互。

The most common way to define a UI using View elements is with an XML layout file stored as part of your app's resources. Defining your layout in XML enables you to maintain the design of your UI separately from the source code that defines the activity behavior.

使用 View 元素定义 UI 的最常见方法是使用作为应用程序资源的一部分存储的 `XML 布局文件`。 通过在 XML 中定义布局，您可以将 UI 设计与定义活动行为的源代码分开维护。

You can also create new View elements directly in your activity code by inserting new View objects into a ViewGroup, and then passing the root ViewGroup to setContentView(). After your layout has been inflated—regardless of its source—you can add more View elements anywhere in the View hierarchy.

您还可以直接在 Activity 代码中创建新的 `View 元素`，方法是将新的 View 对象插入到 ViewGroup 中，然后将根 ViewGroup 传递给 setContentView()。
布局膨胀后（无论其来源如何），您可以在视图层次结构中的任何位置添加更多视图元素(??? 未理解)。

#### Declare the new Activity in the `AndroidManifest.xml` file

Each Activity in your app must be declared in the AndroidManifest.xml file with the `<activity>` element, inside the `<application>` section. When you create a new project or add a new Activity to your project in Android Studio, the AndroidManifest.xml file is created or updated to include skeleton declarations for each Activity.

应用程序中的每个 Activity 都必须在 AndroidManifest.xml 文件中使用 `<application>` 部分内的 `<activity> 元素`进行声明。

```xml
<!-- app/src/main/AndroidManifest.xml -->
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <application
        android:allowBackup="true"
        android:dataExtractionRules="@xml/data_extraction_rules"
        android:fullBackupContent="@xml/backup_rules"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        tools:targetApi="31">

        <!-- setContentView(R.layout.activity_main); -->
        <!-- app/src/main/res/layout/activity_main.xml -->
        <activity
            android:name=".MainActivity"
            android:exported="true"
            android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

    </application>

</manifest>
```

The `<activity>` element includes a number of attributes to define properties of the Activity such as its label, icon, or theme. __The only required attribute is `android:name`__, which specifies the class name for the Activity (such as MainActivity). 

唯一必需的属性是 android:name，它指定 Activity 的类名称

The `<activity>` element can also include declarations for Intent filters. The Intent filters specify the kind of Intent your Activity will accept.

Intent 过滤器指定您的 Activity 将接受的 Intent 类型。

```xml
<intent-filter>
   <action android:name="android.intent.action.MAIN" />
   <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

Intent filters must include at least one `<action>` element, and can also include a `<category>` and optional `<data>`. The MainActivity for your app needs an Intent filter that defines the "main" action and the "launcher" category so that the system can launch your app.

Intent 过滤器必须至少包含一个 `<action>` 元素，还可以包含 `<category>` 和 `<data>`。 您的应用程序的 MainActivity 需要一个 Intent 过滤器来定义`“主”操作`和`“启动器”类别`，以便系统可以启动您的应用程序。

The `<action>` element specifies that this is the "main" entry point to the app. The `<category>` element specifies that this Activity should be listed in the system's app launcher (to allow users to launch this Activity).

此处的, `<action>` 元素指定这是应用程序的“主”入口点。 `<category>` 元素指定此 Activity 应列在系统的应用程序启动器中（以允许用户启动此 Activity）。

Each Activity in your app can also declare Intent filters, `but only your MainActivity should include the "main" action`.

#### Add another Activity to your project

When you choose an Activity template, you see the same set of screens for creating the new activity that you did when you created the project. Android Studio provides three things for each new activity in your app:

- A Java file for the new Activity with a skeleton class definition and onCreate() method. The new Activity, like MainActivity, is a subclass of AppCompatActivity.
- An XML file containing the layout for the new activity. Note that the setContentView() method in the Activity class inflates this new layout.
- An additional `<activity>` element in the AndroidManifest.xml file that specifies the new activity. The second Activity definition does not include any Intent filters. If you plan to use this activity only within your app (and not enable that activity to be started by any other app), you do not need to add filters. 第二个 Activity 定义不包含任何 Intent 过滤器。 如果您计划仅在您的应用程序中使用此 Activity （并且不允许任何其他应用程序启动该 Activity），则无需添加 Intent 过滤器。

## About intents

Each activity is started or activated with an Intent, which is a message object that makes a request to the Android runtime to start an activity or other app component in your app or in some other app.

每个 Activity 都是通过 Intent 启动或激活的，Intent 是一个消息对象，它向 Android 运行时发出请求，以启动您的应用程序或其他应用程序中的 Activity 或其他应用程序组件。

When your app is first started from the device home screen, the Android runtime sends an Intent to your app to start your app's main activity (the one defined with the MAIN action and the LAUNCHER category in the AndroidManifest.xml file). To start another activity in your app, or to request that some other activity available on the device perform an action, you build your own intent and call the startActivity() method to send the intent.

当您的应用程序首次从设备主屏幕启动时，Android 运行时会向您的应用程序发送一个 Intent 以启动应用程序的主 Activity（使用 AndroidManifest.xml 文件中的 MAIN 操作和 LAUNCHER 类别定义的活动）。 要在应用程序中启动另一个 Activity ，或者请求设备上可用的其他一些 Activity 执行操作，您可以构建自己的 Intent 并调用 startActivity() 方法来发送该 Intent。

In addition to starting an activity, an intent can also be used to pass data between one activity and another. When you create an intent to start a new activity, you can include information about the data you want that new activity to operate on. 

除了启动一项 Activity 之外，Intent 还可以用于在一个 Activity 与另一个 Activity 之间传递数据。 当您创建启动新 Activity 的 Intent 时，您可以包含有关您希望新 Activity 操作的数据的信息。

intents can also be used to start services or broadcast receivers. Intent 也可用于启动服务或广播接收器

#### Intent types

1. Explicit intent 显式的

You specify the receiving activity (or other component) using the activity's fully qualified class name. You use explicit intents to start components in your own app (for example, to move between screens in the UI), because you already know the package and class name of that component.

您可以使用 activity 的完全限定类名来指定接收 activity（或其他组件）。 您可以使用显式 intent 来启动自己的应用程序中的组件（例如，在 UI 的屏幕之间移动），因为您已经知道该组件的包和类名称。

2. Implicit intent 隐式的

You do not specify a specific activity or other component to receive the intent. Instead, you declare a general action to perform, and the Android system matches your request to an activity or other component that can handle the requested action. You learn more about using implicit intents in another practical.

您无需指定特定活动或其他组件来接收意图。 相反，您声明要执行的一般操作，Android 系统会将您的请求与可以处理所请求操作的活动或其他组件相匹配。

#### Intent objects and fields

For an explicit Intent, the key fields include the following:

- The Activity class (for an explicit Intent). This is the class name of the Activity or other component that should receive the Intent; for example, com.example.SampleActivity.class. Use the Intent constructor or the setComponent(), setComponentName(), or setClassName() methods to specify the class. 
Activity 类（用于显式 Intent）。 这是应接收 Intent 的 Activity 或其他组件的类名； 例如，com.example.SampleActivity.class。 使用 Intent 构造函数或 setComponent()、setComponentName() 或 setClassName() 方法指定类。
- The Intent data. The Intent data field contains a reference to the data you want the receiving Activity to operate on as a Uri object.
Intent 数据字段包含对您希望接收 Activity 作为 Uri 对象进行操作的数据的引用。
- Intent extras. These are key-value pairs that carry information the receiving Activity requires to accomplish the requested action.
这些键值对携带接收活动完成请求操作所需的信息。
- Intent flags. These are additional bits of metadata, defined by the Intent class. The flags may instruct the Android system how to launch an Activity or how to treat it after it's launched.
这些是元数据的附加位，由 Intent 类定义。 这些标志可以指示 Android 系统如何启动 Activity 或启动后如何处理它。

## Starting an Activity with an explicit Intent

To start a specific Activity from another Activity, use an explicit Intent and the startActivity() method. An explicit Intent includes the fully qualified class name for the Activity or other component in the Intent object. All the other Intent fields are optional, and null by default.

要从另一个 Activity 启动特定 Activity，请使用显式 Intent 和 startActivity() 方法。 显式 Intent 包括 Intent 对象中 Activity 或其他组件的完全限定类名称。 所有其他 Intent 字段都是可选的，默认为空。

For example, if you want to start the ShowMessageActivity to show a specific message in an email app, use code like this:

```java
Intent messageIntent = new Intent(this, ShowMessageActivity.class);
startActivity(messageIntent);
```

The intent constructor takes two arguments for an explicit Intent:

- An application context. In this example, the Activity class provides the context (this).
- The specific component to start (ShowMessageActivity.class).

Use the startActivity() method with the new Intent object as the only argument. The startActivity() method sends the Intent to the Android system, which launches the ShowMessageActivity class on behalf of your app. The new Activity appears on the screen, and the originating Activity is paused.

使用 startActivity() 方法并将新的 Intent 对象作为唯一参数。 startActivity() 方法将 Intent 发送到 Android 系统，该系统代表您的应用程序启动 ShowMessageActivity 类。 新的 Activity 出现在屏幕上，并且原始 Activity 已暂停。

The started Activity remains on the screen until the user taps the Back button on the device, at which time that Activity closes and is reclaimed by the system, and the originating Activity is resumed. You can also manually close the started Activity in response to a user action (such as a Button click) with the finish() method:

启动的 Activity 保留在屏幕上，直到用户点击设备上的“后退”按钮，此时该 Activity 将关闭并被系统回收，并且原始 Activity 将恢复。 您还可以使用 finish() 方法手动关闭已启动的 Activity 以响应用户操作（例如单击按钮）：

```java
public void closeActivity (View view) {
    finish();
}
```

## Passing data from one Activity to another

In addition to simply starting one Activity from another Activity, you also use an Intent to pass information from one Activity to another. The Intent object you use to start an Activity can include Intent data (the URI of an object to act on), or Intent extras, which are bits of additional data the Activity might need.

In the first (sending) Activity, you:

- Create the Intent object.
- Put data or extras into that Intent.
- Start the new Activity with startActivity().

In the second (receiving) Activity, you:

- Get the Intent object the Activity was started with.
- Retrieve the data or extras from the Intent object.

#### When to use Intent data or Intent extras

1. use Intent data

can hold `only one piece` of information: a `URI` representing the location of the data you want to operate on. 

只能保存一条信息：表示要操作的数据位置的 URI

Use the Intent data field:
- When you only have one piece of information you need to send to the started Activity.
- When that information is a data location that can be represented by a URI.

2. Intent extras

Intent extras are for any other arbitrary data you want to pass to the started Activity. Intent extras are stored in a Bundle object as key and value pairs. A Bundle is a map, optimized for Android, in which a key is a string, and a value can be any primitive or object type (objects must implement the Parcelable interface). To put data into the Intent extras you can use any of the Intent class putExtra() methods, or create your own Bundle and put it into the Intent with putExtras().

Intent extras 用于您想要传递给启动的 Activity 的任何其他任意数据。 Intent extras 以键值对的形式存储在 Bundle 对象中。 Bundle 是一个针对 Android 进行了优化的映射，其中键是字符串，值可以是任何基元或对象类型（对象必须实现 Parcelable 接口）。 要将数据放入 Intent extras 中，您可以使用任何 Intent 类 putExtra() 方法，或者创建自己的 Bundle 并使用 putExtras() 将其放入 Intent 中。

Use the Intent extras:
- If you want to pass more than one piece of information to the started Activity.
- If any of the information you want to pass is not expressible by a URI.

Intent data 和 Intent extras 可以同时存在

#### Add data to the Intent

To add data to an `explicit Intent` from the originating Activity, create the Intent object as you did before

Use the setData() method with a Uri object to add that URI to the Intent. Some examples of using setData() with URIs

```java
Intent messageIntent = new Intent(this, ShowMessageActivity.class);

// A web page URL
messageIntent.setData(Uri.parse("http://www.google.com")); 
// a Sample file URI
messageIntent.setData(Uri.fromFile(new File("/sdcard/sample.jpg")));
// A sample content: URI for your app's data model
messageIntent.setData(Uri.parse("content://mysample.provider/data")); 
// Custom URI 
messageIntent.setData(Uri.parse("custom:" + dataID + buttonId));
// 如果多次调用 setData()，则仅使用最后一个值。

startActivity(messageIntent);
```

#### Add extras to the Intent

To add Intent extras to an explicit Intent from the originating Activity:
- Determine the keys to use for the information you want to put into the extras, or define your own. Each piece of information needs its own unique key.
- Use the putExtra() methods to add your key/value pairs to the Intent extras. Optionally you can create a Bundle object, add your data to the Bundle, and then add the Bundle to the Intent.

Intent class includes extra keys you can use, defined as constants that begin with the word `EXTRA_`. For example, you could use Intent.EXTRA_EMAIL , Intent.EXTRA_REFERRER ..

You can also define your own Intent extra keys. Conventionally you define Intent extra keys as static variables with names that begin with EXTRA_. To guarantee that the key is unique, the string value for the key itself should be prefixed with your app's fully qualified class name. For example:

您还可以定义自己的 Intent 额外键。 按照惯例需要将额外键定义为名称以 EXTRA_ 开头的静态变量。 为了保证键是唯一的，键本身的字符串值应该以应用程序的完全限定类名作为前缀。 例如：

```java
// ExtraKey.java
public final static String EXTRA_MESSAGE = "com.example.mysampleapp.MESSAGE";
public final static String EXTRA_POSITION_X = "com.example.mysampleapp.X";
public final static String EXTRA_POSITION_Y = "com.example.mysampleapp.Y";
```

```java
Intent messageIntent = new Intent(this, ShowMessageActivity.class);
messageIntent.putExtra(EXTRA_MESSAGE, "this is my message");
messageIntent.putExtra(EXTRA_POSITION_X, 100);
messageIntent.putExtra(EXTRA_POSITION_Y, 500);
startActivity(messageIntent);
```

或者

Bundle defines many "put" methods for different kinds of primitive data as well as objects that implement `Android's Parcelable` interface or `Java's Serializable`.

```java
Intent messageIntent = new Intent(this, ShowMessageActivity.class);
Bundle extras = new Bundle();
extras.putString(EXTRA_MESSAGE, "this is my message");
extras.putInt(EXTRA_POSITION_X, 100);
extras.putInt(EXTRA_POSITION_Y, 500);
messageIntent.putExtras(extras);
startActivity(messageIntent);
```

#### Retrieve the data from the Intent in the started Activity

To retrieve the Intent the Activity (or other component) was started with, use the getIntent() method

Use getData() to get the URI from that Intent

```java
Intent intent = getIntent();
Uri locationUri = intent.getData();
```

Use one of the getExtra() methods to extract extra data out of the Intent object:

```java
Intent intent = getIntent();
String message = intent.getStringExtra(MainActivity.EXTRA_MESSAGE); 
int positionX = intent.getIntExtra(MainActivity.EXTRA_POSITION_X);
int positionY = intent.getIntExtra(MainActivity.EXTRA_POSITION_Y);
```

Or you can get the entire extras Bundle from the Intent and extract the values with the various Bundle methods:

```java
Intent intent = getIntent();
Bundle extras = intent.getExtras();
String message = extras.getString(MainActivity.EXTRA_MESSAGE);
int positionX = extras.getInt(MainActivity.EXTRA_POSITION_X);
int positionY = extras.getInt(MainActivity.EXTRA_POSITION_Y);
```

## Getting data back from an Activity

#### Use startActivityForResult() to launch the Activity

#### Return a response from the launched Activity

#### Read response data in onActivityResult()

## Activity navigation

Any app of any complexity that you build will include more than one Activity. As your users move around your app and from one Activity to another, consistent navigation becomes more important to the app's user experience. Few things frustrate users more than basic navigation that behaves in inconsistent and unexpected ways. Thoughtfully designing your app's navigation will make using your app predictable and reliable for your users.

多个 Activity 一致的导航对于应用程序的用户体验很重要

Android system supports two different forms of navigation strategies for your app.

- Back (temporal) navigation, provided by the device Back button, and the back stack.
- Up (ancestral) navigation, provided by you as an option in the app bar.


#### Back navigation, tasks, and the back stack

Back navigation allows your users to return to the previous Activity by tapping the device back button. Back navigation is also called temporal navigation because the back button navigates the history of recently viewed screens, in reverse chronological order.

后退导航允许用户通过点击设备`后退按钮`返回到上一个活动。 后退导航也称为时间导航，因为后退按钮以相反的时间顺序导航最近查看的屏幕的历史记录。

The back stack is the set of each Activity that the user has visited and that can be returned to by the user with the back button. Each time a new Activity starts, it is pushed onto the back stack and takes user focus. The previous Activity is stopped but is still available in the back stack. The back stack operates on a "last in, first out" mechanism, so when the user is done with the current Activity and presses the Back button, that Activity is popped from the stack (and destroyed) and the previous Activity resumes.

返回堆栈是用户访问过的每个 Activity 的集合，用户可以使用后退按钮返回该 Activity。 每次新的 Activity 启动时，它都会被推入后台堆栈并获取用户焦点。 前一个 Activity 已停止，但在返回堆栈中仍然可用。 返回堆栈按照“后进先出”机制运行，因此当用户完成当前 Activity 并按下“后退”按钮时，该 Activity 将从堆栈中弹出（并销毁），并恢复上一个 Activity。

Because an app can start an Activity both inside and outside a single app, the back stack contains each Activity that has been launched by the user in reverse order. Each time the user presses the Back button, each Activity in the stack is popped off to reveal the previous one, until the user returns to the Home screen.

由于应用程序可以在单个应用程序内部和外部启动 Activity，因此返回堆栈包含用户按相反顺序启动的每个 Activity。 每次用户按下“后退”按钮时，堆栈中的每个 Activity 都会弹出以显示前一个 Activity，直到用户返回主屏幕。

Android provides a back stack for each task. A task is an organizing concept for each Activity the user interacts with when performing an operation, whether they are inside your app or across multiple apps. Most tasks start from the Android home screen, and tapping an app icon starts a task (and a new back stack) for that app. If the user uses an app for a while, taps home, and starts a new app, that new app launches in its own task and has its own back stack. If the user returns to the first app, that first task's back stack returns. Navigating with the Back button returns only to the Activity in the current task, not for all tasks running on the device. Android enables the user to navigate between tasks with the overview or recent tasks screen, accessible with the square button on lower right corner of the device

Android 为每个任务提供一个返回堆栈。 任务是用户在执行操作时与之交互的每个 Activity 的组织概念，无论它们是在您的应用程序内部还是跨多个应用程序。 大多数任务从 Android 主屏幕开始，点击应用程序图标即可启动该应用程序的任务（以及新的后退堆栈）。 如果用户使用某个应用程序一段时间，点击主页并启动一个新应用程序，则该新应用程序将在其自己的任务中启动并具有自己的后堆栈。 如果用户返回到第一个应用程序，则第一个任务的返回堆栈将返回。 使用“后退”按钮导航仅返回当前任务中的活动，而不返回设备上运行的所有任务。 Android 使用户能够通过概述或最近任务屏幕在任务之间导航，可通过设备右下角的方形按钮进行访问

In most cases you don't have to worry about managing either tasks or the back stack for your app—the system keeps track of these things for you, and the back button is always available on the device.

#### Up navigation

Up navigation, sometimes referred to as ancestral or logical navigation, is used to navigate within an app based on the explicit hierarchical relationships between screens. With Up navigation, each Activity is arranged in a hierarchy, and each "child" Activity shows a left-facing arrow in the app bar that returns the user to the "parent" Activity. The topmost Activity in the hierarchy is usually MainActivity, and the user cannot go up from there.

向上导航，有时称为祖先导航或逻辑导航，用于根据屏幕之间的显式层次关系在应用程序内进行导航。 通过向上导航，每个活动都按层次结构排列，每个“子”活动在应用栏中显示一个向左的箭头，使用户返回到“父”活动。 层次结构中最顶层的 Activity 通常是 MainActivity，用户无法从那里向上移动。

The behavior of the Up button is defined by you in each Activity based on how you design your app's navigation. In many cases, Up and Back navigation may provide the same behavior: to just return to the previous Activity. For example, a Settings Activity may be available from any Activity in your app, so "up" is the same as back—just return the user to their previous place in the hierarchy.

向上按钮的行为由您根据您设计应用程序导航的方式在每个活动中定义。 在许多情况下，向上和返回导航可能提供相同的行为：仅返回到上一个 Activity。 例如，设置活动可以从应用程序中的任何活动中使用，因此“向上”与返回相同 - 只需将用户返回到层次结构中的先前位置即可。

Providing Up behavior for your app is optional, but a good design practice, to provide consistent navigation for your app.

为您的应用程序提供向上行为是可选的，但这是一个很好的设计实践，可以为您的应用程序提供一致的导航。

#### Implement Up navigation with a parent Activity

With the standard template projects in Android Studio, it's straightforward to implement Up navigation. If one Activity is a child of another Activity in your app's Activity hierarchy, specify the parent of that other Activity in the AndroidManifest.xml file.

如果一个 Activity 是应用程序 Activity 层次结构中另一个 Activity 的子级，请在 AndroidManifest.xml 文件中指定该另一个 Activity 的父级。

Beginning in Android 4.1 (API level 16), declare the logical parent of each Activity by specifying the android:parentActivityName attribute in the `<activity>` element. To support older versions of Android, include `<meta-data>` information to define the parent Activity explicitly. Use both methods to be backwards-compatible with all versions of Android.

从 Android 4.1（API 级别 16）开始，通过在 `<activity>` 元素中指定 android:parentActivityName 属性来声明每个 Activity 的逻辑父级。 为了支持旧版本的 Android，请包含“<meta-data>”信息来显式定义父 Activity。 使用这两种方法可以向后兼容所有版本的 Android。

```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
    <!-- The main activity (it has no parent activity) -->
    <activity android:name=".MainActivity">
       <intent-filter>
            <action android:name="android.intent.action.MAIN" />

            <category android:name="android.intent.category.LAUNCHER" />
       </intent-filter>
    </activity>
    <!-- The child activity) -->
    <activity android:name=".SecondActivity"
       android:label = "Second Activity"
       android:parentActivityName=".MainActivity">
       <meta-data
          android:name="android.support.PARENT_ACTIVITY"
          android:value="com.example.android.twoactivities.MainActivity" />
       </activity>
</application>
```

## Learn more

- [Application Fundamentals](https://developer.android.com/guide/components/fundamentals.html)
- [Activities](https://developer.android.com/guide/components/activities/intro-activities)
- [Intents and Intent Filters](http://developer.android.com/guide/components/intents-filters.html)
- [Designing Back and Up navigation](https://developer.android.com/training/design-navigation/ancestral-temporal)
- [Activity](http://developer.android.com/reference/android/app/Activity.html)
- [Intent](http://developer.android.com/reference/android/content/Intent.html)



