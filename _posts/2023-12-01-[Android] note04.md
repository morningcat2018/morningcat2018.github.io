---
layout:     post
title:      android 笔记四
subtitle:   Implicit intents
date:       2023-12-01
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Android
---

[Android developer document - 2.3: Implicit intents](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-1-get-started/lesson-2-activities-and-intents/2-3-c-implicit-intents/2-3-c-implicit-intents.html)

## Introduction

In this chapter you learn how to send and receive an implicit intent. In an implicit intent, you declare a general action to perform, and the system matches your request with an activity.

在隐式 intent 中，您声明要执行的`一般操作`，系统将您的请求与 activity 相匹配。

## Understanding an implicit Intent

In an earlier chapter you learned how to start an activity from an activity by specifying the class name of the activity to start in an explicit intent. This is the most basic way to use an intent: To start an Intent or other app component and pass data to it (and sometimes receive data).

通过指定要在`显式意图`中启动的活动的类名来从活动启动另一个活动, 这是使用 Intent 的最基本方法：启动 Intent 或其他应用程序组件并向其传递数据（有时还接收数据）。

A more flexible use of an Intent is the implicit intent. You don't specify the exact activity (or other component) to run—instead, you include just enough information in the intent about the task you want to perform. The Android system matches the information in your request intent with any activity available on the device that can perform that task. If there's only one activity that matches, that activity is launched. If more than one activity matches the intent, the user is presented with an app chooser and picks which app they would like to perform the task.

Intent 更灵活的用法是隐式意图。 您无需指定要运行的确切活动（或其他组件） - 相反，您`只需在意图中包含有关您要执行的任务的足够信息`即可。 `Android 系统将您的请求意图中的信息与设备上可以执行该任务的任何可用活动进行匹配`。 如果只有一项活动匹配，则启动该活动。 如果多个活动符合意图，则会向用户显示一个应用程序选择器，并选择他们想要执行该任务的应用程序。

For example, you have an app that lists available snippets of video. If the user touches an item in the list, you want to play that video snippet. Rather than implementing an entire video player in your own app, you can launch an Intent that specifies the task as "play an object of type video." The Android system then matches your request with an Activity that has registered itself to play objects of type video.

例如，您有一个列出可用视频片段的应用程序。 如果用户触摸列表中的某个项目，您想要播放该视频片段。 您可以启动一个 Intent，`将任务指定为“播放视频类型的对象”`，而不是在自己的应用程序中实现整个视频播放器。 然后，Android 系统将您的请求与已注册以播放视频类型对象的 Activity 进行匹配。

An Activity registers itself with the system as being able to handle an implicit Intent with Intent filters, declared in the AndroidManifest.xml file. For example, the main Activity (and only the main Activity) for your app has an Intent filter that declares it the main Activity for the launcher category. This Intent filter is how the Android system knows to start that specific Activity in your app when the user taps the icon for your app on the device home screen.

Activity 将自身注册到系统，以便能够使用 AndroidManifest.xml 文件中声明的 Intent 过滤器处理隐式 Intent。 例如，应用程序的主 Activity（并且仅是主 Activity）有一个 Intent 过滤器，该过滤器将其声明为启动器类别的主 Activity。 当用户在设备主屏幕上点击您的应用程序的图标时，此 Intent 过滤器是 Android 系统如何知道在您的应用程序中启动特定 Activity 的方式。

#### Intent actions, categories, and data

An implicit Intent, like an explicit Intent, is an instance of the Intent class. In addition to the parts of an Intent you learned about in an earlier chapter (such as the Intent data and extras), these fields are used by an implicit Intent:

隐式 Intent 与显式 Intent 一样，都是 Intent 类的实例。 除了您在前面的章节中了解的 Intent 部分（例如 Intent data 和 extras）之外，隐式 Intent 使用这些字段：

`The Intent action`, which is the generic action the receiving Activity should perform. The available Intent actions are defined as constants in the Intent class and begin with the word ACTION_. A common Intent action is ACTION_VIEW, which you use when you have some information that an Activity can show to the user, such as a photo to view in a gallery app, or an address to view in a map app. You can specify the action for an Intent in the Intent constructor, or with the setAction() method.

`Intent action` , 这是接收 Activity 应该执行的通用操作。 可用的 Intent 操作在 Intent 类中定义为常量，并以单词 `ACTION_` 开头。 常见的 Intent 操作是 ACTION_VIEW，当您有 Activity 可以向用户显示的某些信息（例如要在图库应用程序中查看的照片或要在地图应用程序中查看的地址）时，可以使用它。 您可以在 Intent 构造函数中或使用 setAction() 方法指定 Intent 的操作。

`An Intent category,` which provides additional information about the category of component that should handle the Intent. Intent categories are optional, and you can add more than one category to an Intent. Intent categories are also defined as constants in the Intent class and begin with the word CATEGORY_. You can add categories to the Intent with the addCategory() method.

Intent category , 它提供了有关应处理 Intent 的组件类别的附加信息。 Intent 类别是可选的，您可以向一个 Intent 添加多个类别。 Intent 类别也在 Intent 类中定义为常量，并以单词 `CATEGORY_` 开头。 您可以使用 addCategory() 方法将类别添加到 Intent。

`The data type`, which indicates the MIME type of data the Activity should operate on. Usually, the data type is inferred from the URI in the Intent data field, but you can also explicitly define the data type with the setType() method.

data type , 它指示 Activity 应操作的`数据的 MIME 类型`。 通常，数据类型是从 Intent 数据字段中的 URI 推断出来的，但您也可以使用 setType() 方法显式定义数据类型。

Intent actions, categories, and data types are used both by the Intent object you create in your sending Activity, as well as in the Intent filters you define in the AndroidManifest.xml file for the receiving Activity. The Android system uses this information to match an implicit Intent request with an Activity or other component that can handle that Intent.

Intent 操作、类别和数据类型由您在发送活动中创建的 Intent 对象以及您在 AndroidManifest.xml 文件中为接收活动定义的 Intent 过滤器中使用。 Android 系统使用此信息将隐式 Intent 请求与可以处理该 Intent 的 Activity 或其他组件进行匹配。

## Sending an implicit Intent

Starting an Activity with an implicit Intent, and passing data from one Activity to another, works much the same way as it does for an explicit Intent:

使用隐式 Intent 启动 Activity，并将数据从一个 Activity 传递到另一个 Activity，其工作方式与显式 Intent 的工作方式大致相同：

- In the sending Activity, create a new Intent object.
- Add information about the request to the Intent object, such as data or extras.
- Send the Intent with startActivity() (to just start the Activity) or startActivityforResult() (to start the Activity and expect a result back).

When you create an implicit Intent object, you:

- Do not specify the specific Activity or other component to launch. 不要指定要启动的特定活动或其他组件。
- Add an Intent action or Intent categories (or both). 添加 Intent action 或 Intent categories（或两者）。
- Resolve the Intent with the system before calling startActivity() or startActivityforResult(). 在调用 startActivity() 或 startActivityforResult() 之前，先与系统解析 Intent。
- Show an app chooser for the request (optional). 显示请求的应用程序选择器（可选）。

#### Create implicit Intent objects

To use an implicit Intent, create an Intent object as you did for an explicit Intent, only without the specific component name.
You can also create the Intent object with a specific action:

要使用隐式 Intent，请像创建显式 Intent 一样创建一个 Intent 对象，只是没有特定的组件名称。

```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_VIEW);
```

您还可以使用特定操作创建 Intent 对象：

```java
Intent sendIntent = new Intent(Intent.ACTION_VIEW);
```

Once you have an Intent object you can add other information (category, data, extras) with the various Intent methods. For example, this code creates an implicit Intent object, sets the Intent action to ACTION_SEND, defines an Intent extra to hold the text, and sets the type of the data to the MIME type "text/plain".

一旦有了 Intent 对象，您就可以使用各种 Intent 方法添加其他信息（类别、数据、附加信息）。 例如，此代码创建一个隐式 Intent 对象，将 Intent 操作设置为 ACTION_SEND，定义一个 Intent extra 来保存文本，并将数据类型设置为 MIME 类型“text/plain”。

```java
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");
```

#### Resolve the Activity before starting it

When you define an implicit Intent with a specific action and/or category, there is a possibility that there won't be any Activity on the device that can handle your request. If you just send the Intent and there is no appropriate match, your app will crash.

当您使用特定操作和/或类别定义隐式 Intent 时，设备上可能没有任何 Activity 可以处理您的请求。 如果您只是发送 Intent 并且`没有合适的匹配项`，您的应用程序将会崩溃。

To verify that an Activity or other component is available to receive your Intent, use the resolveActivity() method with the system package manager like this:

要`验证` Activity 或其他组件是否可用于接收您的 Intent，请使用系统包管理器的 resolveActivity() 方法，如下所示：

```java
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```

If the result of resolveActivity() is not null, then there is at least one app available that can handle the Intent, and it's safe to call startActivity(). Do not send the Intent if the result is null.

如果resolveActivity()的结果不为null，那么至少有一个应用程序可以处理该Intent，并且可以安全地调用startActivity()。 如果结果为空，则不发送 Intent。

If you have a feature that depends on an external Activity that may or may not be available on the device, a best practice is to test for the availability of that external Activity before the user tries to use it. If there is no Activity that can handle your request (that is, resolveActivity() returns null), disable the feature or provide the user an error message for that feature.

如果您的某个功能依赖于设备上可能可用或不可用的外部活动，则最佳实践是在用户尝试使用该外部活动之前测试该外部活动的可用性。 如果没有可以处理您的请求的 Activity（即，resolveActivity() 返回 null），请禁用该功能或向用户提供该功能的错误消息。

#### Show the app chooser

To find an Activity or other component that can handle your Intent requests, the Android system matches your implicit Intent with an Activity whose Intent filters indicate that they can perform that action. If there are multiple apps installed that match, the user is presented with an app chooser that lets them select which app they want to use to handle that Intent.

为了找到可以处理您的 Intent 请求的 Activity 或其他组件，Android 系统会将您的隐式 Intent 与其 Intent 过滤器表明它们可以执行该操作的 Activity 进行匹配。 如果安装了多个匹配的应用程序，则会向用户显示一个应用程序选择器，让他们选择要使用哪个应用程序来处理该 Intent。

In many cases the user has a preferred app for a given task, and they will select the option to always use that app for that task. However, if multiple apps can respond to the Intent and the user might want to use a different app each time, you can choose to explicitly show a chooser dialog every time. For example, when your app performs a "share this" action with the ACTION_SEND action, users may want to share using a different app depending on the current situation.

在许多情况下，用户对于给定任务有一个首选应用程序，并且他们将选择始终使用该应用程序来执行该任务的选项。 但是，如果多个应用程序可以响应 Intent 并且用户可能希望每次使用不同的应用程序，则您可以选择每次都显式显示选择器对话框。 例如，当您的应用通过 ACTION_SEND 操作执行“共享此”操作时，用户可能希望根据当前情况使用不同的应用进行共享。

To show the chooser, you create a wrapper Intent for your implicit Intent with the createChooser() method, and then resolve and call startActivity() with that wrapper Intent. The createChooser() method also requires a string argument for the title that appears on the chooser. You can specify the title with a string resource as you would any other string.

要显示选择器，您可以使用 createChooser() 方法为隐式 Intent 创建一个包装器 Intent，然后使用该包装器 Intent 解析并调用 startActivity()。 createChooser() 方法还需要一个字符串参数作为显示在选择器上的标题。 您可以像使用任何其他字符串一样使用字符串资源指定标题。

For example:

```java
// The implicit Intent object
Intent sendIntent = new Intent(Intent.ACTION_SEND);
// Always use string resources for UI text.
String title = getResources().getString(R.string.chooser_title);
// Create the wrapper intent to show the chooser dialog.
Intent chooser = Intent.createChooser(sendIntent, title);
// Resolve the intent before starting the activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(chooser);
}
```

## Receiving an implicit Intent

If you want an Activity in your app to respond to an implicit Intent (from your own app or other apps), declare one or more Intent filters in the AndroidManifest.xml file. Each Intent filter specifies the type of Intent it accepts based on the action, data, and category for the Intent. The system will deliver an implicit Intent to your app component only if that Intent can pass through one of your Intent filters.

如果您希望应用程序中的 Activity 响应隐式 Intent（来自您自己的应用程序或其他应用程序），请在 AndroidManifest.xml 文件中声明一个或多个 Intent 过滤器。 每个 Intent 过滤器根据 Intent 的操作、数据和类别指定它接受的 Intent 类型。 仅当 Intent 可以通过您的 Intent 过滤器之一时，系统才会向您的应用程序组件传递隐式 Intent。 ???

Note: An explicit Intent is always delivered to its target, regardless of any Intent filters the component declares. Conversely, if an Activity does not include Intent filters, it can only be launched with an explicit Intent.

注意：无论组件声明任何 Intent 过滤器，显式 Intent 始终会传递到其目标。 相反，如果一个 Activity 不包含 Intent 过滤器，则只能使用显式 Intent 来启动它。

Once your Activity is successfully launched with an implicit Intent, you can handle that Intent and its data the same way you did an explicit Intent, by:

使用隐式 Intent 成功启动 Activity 后，您可以像处理显式 Intent 一样处理该 Intent 及其数据，方法是：

- Getting the Intent object with getIntent().
- Getting Intent data or extras out of that Intent.
- Performing the task the Intent requested. 执行意图请求的任务。
- Returning data to the calling Activity with another Intent, if needed.

#### Intent filters

Define Intent filters with one or more `<intent-filter>` elements in the AndroidManifest.xml file, nested in the corresponding `<activity>` element. Inside `<intent-filter>`, specify the type of intent your activity can handle. The Android system matches an implicit intent with an activity or other app component only if the fields in the Intent object match the Intent filters for that component.



An Intent filter may contain the following elements, which correspond to the fields in the Intent object described above:

- `<action>`: The Intent action that the activity accepts.
- `<data>`: The type of data accepted, including the MIME type or other attributes of the data URI (such as scheme, host, port, and path).
- `<category>`: The Intent category.

For example, the main Activity for your app includes this `<intent-filter>` element, which you saw in an earlier chapter:

```xml
<intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
```

This Intent filter has the action MAIN and the category LAUNCHER. The `<action>` element specifies that this is the app's "main" entry point. The `<category>` element specifies that this activity should be listed in the system's app launcher (to allow users to launch the activity). Only the main activity for your app should have this Intent filter.

Here's another example for an implicit Intent that shares a bit of text. This Intent filter matches the implicit Intent example from the previous section:

```xml
<activity android:name="ShareActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <data android:mimeType="text/plain"/>
    </intent-filter>
</activity>
```

You can specify more than one action, data, or category for the same Intent filter, or have multiple Intent filters per Activity to handle each different kind of Intent.

The Android system tests an implicit Intent against an Intent filter by comparing the parts of that Intent to each of the three Intent filter elements (action, category, and data). The Intent must pass all three tests or the Android system won't deliver the Intent to the component. However, because a component may have multiple Intent filters, an Intent that does not pass through one of a component's filters might make it through on another filter.

#### Actions

An Intent filter can declare zero or more `<action>` elements for the Intent action. The action is defined in the name attribute, and consists of the string "android.intent.action." plus the name of the Intent action, minus the `ACTION_` prefix. So, for example, an implicit Intent with the action ACTION_VIEW matches an Intent filter whose action is android.intent.action.VIEW.

For example, this Intent filter matches either ACTION_EDIT and ACTION_VIEW:

```xml
<intent-filter>
    <action android:name="android.intent.action.EDIT" />
    <action android:name="android.intent.action.VIEW" />
</intent-filter>
```

To get through this filter, the action specified in the incoming Intent object must match at least one of the actions. You must include at least one Intent action for an incoming implicit Intent to match.

#### Categories

An Intent filter can declare zero or more `<category>` elements for Intent categories. The category is defined in the name attribute, and consists of the string "android.intent.category." plus the name of the Intent category, minus the CATEGORY prefix.

For example, this Intent filter matches either CATEGORY_DEFAULT and CATEGORY_BROWSABLE:

```xml
<intent-filter>
    <category android:name="android.intent.category.DEFAULT" />
    <category android:name="android.intent.category.BROWSABLE" />
</intent-filter>
```

Note that any Activity that you want to accept an implicit Intent must include the android.intent.category.DEFAULT Intent filter. This category is applied to all implicit Intent objects by the Android system.

#### Data

An Intent filter can declare zero or more `<data>` elements for the URI contained in the Intent data. As the Intent data consists of a URI and (optionally) a MIME type, you can create an Intent filter for various aspects of that data, including:

- URI Scheme
- URI Host
- URI Path
- Mime type

For example, this Intent filter matches any data Intent with a URI scheme of http and a MIME type of either "video/mpeg" or "audio/mpeg".

```xml
<intent-filter>
    <data android:mimeType="video/mpeg" android:scheme="http" />
    <data android:mimeType="audio/mpeg" android:scheme="http" />
</intent-filter>
```

## Sharing data with ShareCompat.IntentBuilder

Share actions are an easy way for users to share items in your app with social networks and other apps. Although you can build a share action in your own app using an implicit Intent with the ACTION_SEND action, Android provides the ShareCompat.IntentBuilder helper class to easily implement sharing in your app.

Note: For apps that target Android releases after API 14, you can use the ShareActionProvider class for share actions instead of ShareCompat.IntentBuilder. The ShareCompat class is part of the V4 support library, and allows you to provide share actions in apps in a backward-compatible fashion. ShareCompat provides a single API for sharing on both old and new Android devices. You learn more about the Android support libraries in another chapter.

With the ShareCompat.IntentBuilder class you do not need to create or send an implicit Intent for the share action. Use the methods in ShareCompat.IntentBuilder to indicate the data you want to share as well as any additional information. Start with the from() method to create a new Intent builder, add other methods to add more data, and end with the startChooser() method to create and send the Intent. You can chain the methods together like this:

```java
ShareCompat.IntentBuilder
    .from(this)         // information about the calling activity
    .setType(mimeType)  // mime type for the data
    .setChooserTitle("Share this text with: ") //title for the app chooser
    .setText(txt)       // intent data
    .startChooser();    // send the intent
```

## Managing tasks

In a previous chapter you learned about tasks and the back stack. The task for your app contains its own stack that contains each Activity the user has visited while using your app. As the user navigates around your app, Activity instances for that task are pushed and popped from the stack for that task.

Most of the time the user's navigation from one Activity to another Activity and back again through the stack is straightforward. Depending on the design and navigation of your app there may be complications, especially with an Activity started from another app and other tasks.

For example, say you have an app with three Activity objects: A, B, and C. A launches B with an Intent, and B launches C. C, in turn sends an Intent to launch A. In this case the system creates a second instance of A on the top of the stack, rather than bringing the already-running instance to the foreground. Depending on how you implement each Activity, the two instances of A can get out of sync and provide a confusing experience for a user navigating back through the stack.

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c67f4bed777749328cd0e09f566ae11e~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=731&h=420&s=50379&e=png&a=1&b=eeeeee)

Or, say your Activity C can be launched from a second app with an implicit Intent. The user runs the second app, which has its own task and its own back stack. If that app uses an implicit Intent to launch your Activity C, a new instance of C is created and placed on the back stack for that second app's task. Your app still has its own task, its own back stack, and its own instance of C.

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87914724007d4fcfb3f75c1321eae5c9~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=730&h=337&s=41601&e=png&a=1&b=ededed)

Much of the time the Android's default behavior for tasks works fine and you don't have to worry about how each Activity is associated with tasks, or how they exist in the back stack. If you want to change the normal behavior, Android provides a number of ways to manage tasks and each Activity within those tasks, including:

- Activity launch modes, to determine how an Activity should be launched.
- Task affinities, which indicate which task a launched Activity belongs to.

## Activity launch modes

Use Activity launch modes to indicate how each new Activity should be treated when launched—that is, if the Activity should be added to the current task, or launched into a new task. Define launch modes for the Activity with attributes on the `<activity>` element of the AndroidManifest.xml file, or with flags set on the Intent that starts that Activity.

#### Activity attributes

To define a launch mode for an Activity add the android:launchMode attribute to the `<activity>` element in the AndroidManifest.xml file. This example uses a launch mode of "standard", which is the default.

```java
<activity
   android:name=".SecondActivity"
   android:label="@string/activity2_name"
   android:parentActivityName=".MainActivity"
   android:launchMode="standard">
   <!-- More attributes ... -->
</activity>
```

There are four launch modes available as part of the `<activity>` element:

- "standard" (the default): A new Activity is launched and added to the back stack for the current task. An Activity can be instantiated multiple times, a single task can have multiple instances of the same Activity, and multiple instances can belong to different tasks.
- "singleTop": If an instance of an Activity exists at the top of the back stack for the current task and an Intent request for that Activity arrives, Android routes that Intent to the existing Activity instance rather than creating a new instance. A new Activity is still instantiated if there is an existing Activity anywhere in the back stack other than the top.
- "singleTask": When the Activity is launched the system creates a new task for that Activity. If another task already exists with an instance of that Activity, the system routes the Intent to that Activity instead.
- "singleInstance": Same as single task, except that the system doesn't launch any other Activity into the task holding the Activity instance. The Activity is always the single and only member of its task.

The vast majority of apps will only use the standard or single top launch modes. See the android:launchMode attribute for more detailed information on launch modes.

#### Intent flags

Intent flags are options that specify how the activity (or other app component) that receives the intent should handle that intent. Intent flags are defined as constants in the Intent class and begin with the word FLAG_. You add Intent flags to an Intent object with setFlag() or addFlag().

Three specific Intent flags are used to control activity launch modes, either in conjunction with the launchMode attribute or in place of it. Intent flags always take precedence over the launch mode in case of conflicts.

- `FLAG_ACTIVITY_NEW_TASK`: start the Activity in a new task. This behavior is the same as for singleTask launch mode.
- `FLAG_ACTIVITY_SINGLE_TOP`: if the Activity to be launched is at the top of the back stack, route the Intent to that existing Activity instance. Otherwise create a new Activity instance. This is the same behavior as the singleTop launch mode.
- `FLAG_ACTIVITY_CLEAR_TOP`: If an instance of the Activity to be launched already exists in the back stack, destroy any other Activity on top of it and route the Intent to that existing instance. When used in conjunction with FLAG_ACTIVITY_NEW_TASK, this flag locates any existing instances of the Activity in any task and brings it to the foreground.

See the Intent class for more information about other available Intent flags.

#### Handle a new Intent

When the Android system routes an Intent to an existing Activity instance, the system calls the onNewIntent() callback method (usually just before the onResume() method). The onNewIntent() method includes an argument for the new Intent that was routed to the Activity. Override the onNewIntent() method in your class to handle the information from that new Intent.

Note that the getIntent() method—to get access to the Intent that launched the Activity—always retains the original Intent that launched the Activity instance. Call setIntent() in the onNewIntent() method:

```java
@Override 
public void onNewIntent(Intent intent) { 
    super.onNewIntent(intent); 
    // Use the new intent, not the original one
    setIntent(intent); 
}
```

Any call to getIntent() after this returns the new Intent.

## Task affinities






