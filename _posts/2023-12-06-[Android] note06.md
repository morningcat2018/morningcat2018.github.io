---
layout:     post
title:      android 笔记六
subtitle:   AsyncTask and AsyncTaskLoader
date:       2023-12-06
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Android
---

[Android developer document - 7.1: AsyncTask and AsyncTaskLoader](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-3-working-in-the-background/lesson-7-background-tasks/7-1-c-asynctask-and-asynctaskloader/7-1-c-asynctask-and-asynctaskloader.html)

There are several ways to do background processing in Android. Two of those ways are:

- 您可以使用 AsyncTask 类直接进行后台处理。
- 您可以使用 Loader 框架然后使用 AsyncTaskLoader 类间接进行后台处理。

In most situations the Loader framework is a better choice, but it's important to know how AsyncTask works. 在大多数情况下，Loader 框架是更好的选择，但了解 AsyncTask 的工作原理很重要。

In this chapter you learn why it's important to process some tasks in the background, off the UI thread. You learn how to use AsyncTask, when not to use AsyncTask, and the basics of using loaders.

## The UI thread

When an Android app starts, it creates the main thread, which is often called the UI thread. The UI thread dispatches events to the appropriate user interface (UI) widgets. The UI thread is where your app interacts with components from the Android UI toolkit (components from the android.widget and android.view packages).

当 Android 应用程序启动时，它会创建`主线程`，通常称为 `UI 线程`。 UI 线程将事件分派到适当的用户界面 (UI) 小部件。 UI 线程是您的应用程序与 Android UI 工具包中的组件（来自 android.widget 和 android.view 包的组件）进行交互的地方。

Android's thread model has two rules:

- Do not block the UI thread.
- Do UI work only on the UI thread. 仅在 UI 线程上执行 UI 工作。

The UI thread needs to give its attention to drawing the UI and keeping the app responsive to user input. If everything happened on the UI thread, long operations such as network access or database queries could block the whole UI. From the user's perspective, the app would appear to hang. Even worse, if the UI thread were blocked for more than a few seconds (about 5 seconds currently) the user would be presented with the "application not responding" (ANR) dialog. The user might decide to quit your app and uninstall it.

UI 线程`需要关注绘制 UI 并保持应用程序响应用户输入`。 如果一切都发生在 UI 线程上，那么网络访问或数据库查询等长时间操作可能会阻塞整个 UI。 从用户的角度来看，应用程序似乎挂起。 更糟糕的是，如果 UI 线程被阻塞超过几秒（当前约为 5 秒），用户将看到“应用程序未响应”(ANR) 对话框。 用户可能决定退出您的应用程序并将其卸载。

To make sure your app doesn't block the UI thread:

- Complete all work in less than 16 ms for each UI screen. 每个 UI 屏幕在 16 毫秒内完成所有工作。
- Don't run asynchronous tasks and other long-running tasks on the UI thread. Instead, implement tasks on a background thread using AsyncTask (for short or interruptible tasks) or AsyncTaskLoader (for tasks that are high-priority, or tasks that need to report back to the user or UI). 不要在 UI 线程上运行异步任务和其他长时间运行的任务。 相反，使用 AsyncTask（对于短任务或可中断任务）或 AsyncTaskLoader（对于高优先级任务，或者需要向用户或 UI 报告的任务）在后台线程上实现任务。

Conversely, don't use a background thread to manipulate your UI, because the Android UI toolkit is not thread-safe.

相反，不要使用后台线程来操作您的 UI，因为 Android UI 工具包不是线程安全的。


## AsyncTask

A worker thread is any thread which is not the main or UI thread. Use the AsyncTask class to implement an asynchronous, long-running task on a worker thread. AsyncTask allows you to perform background operations on a worker thread and publish results on the UI thread without needing to directly manipulate threads or handlers.

工作线程是除主线程或 UI 线程之外的任何线程。 使用 AsyncTask 类在工作线程上实现异步、长时间运行的任务。 AsyncTask 允许您在工作线程上执行后台操作并在 UI 线程上发布结果，而无需直接操作线程或处理程序。

When AsyncTask is executed, it goes through four steps:

1. `onPreExecute()` is invoked on the UI thread before the task is executed. This step is normally used to set up the task, for instance by showing a progress bar in the UI. onPreExecute() 在执行任务之前在 UI 线程上调用。 此步骤通常用于设置任务，例如通过在 UI 中显示进度条。

2. doInBackground(Params...) is invoked on the background thread immediately after onPreExecute() finishes. This step performs a background computation, returns a result, and passes the result to onPostExecute(). The doInBackground() method can also call publishProgress(Progress...) to publish one or more units of progress. onPreExecute() 完成后立即在后台线程上调用 doInBackground(Params...)。 此步骤执行后台计算，返回结果，并将结果传递给 onPostExecute()。 doInBackground() 方法还可以调用publishProgress(Progress...) 来发布一个或多个进度单位。

3. onProgressUpdate(Progress...) runs on the UI thread after publishProgress(Progress...) is invoked. Use onProgressUpdate() to report any form of progress to the UI thread while the background computation is executing. For instance, you can use it to pass the data to animate a progress bar or show logs in a text field. onProgressUpdate(Progress...) 在调用publishProgress(Progress...) 后在UI 线程上运行。 在执行后台计算时，使用 onProgressUpdate() 向 UI 线程报告任何形式的进度。 例如，您可以使用它来传递数据以动画进度条或在文本字段中显示日志。

4. onPostExecute(Result) runs on the UI thread after the background computation has finished. The result of the background computation is passed to this method as a parameter. onProgressUpdate(Progress...) 在调用publishProgress(Progress...) 后在UI 线程上运行。 在执行后台计算时，使用 onProgressUpdate() 向 UI 线程报告任何形式的进度。 例如，您可以使用它来传递数据以动画进度条或在文本字段中显示日志。

For complete details on these methods, see the AsyncTask reference. Below is a diagram of their calling order.

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9294c92f5ea14c4cbb3bf45074c35f65~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=894&h=255&s=40922&e=png&a=1&b=00858f)

## AsyncTask usage

To use the AsyncTask class, define a subclass of AsyncTask that overrides the doInBackground(Params...) method (and usually the onPostExecute(Result) method as well). This section describes the parameters and usage of AsyncTask, then shows a complete example.

要使用 AsyncTask 类，请定义 AsyncTask 的子类，该子类重写 doInBackground(Params...) 方法（通常还重写 onPostExecute(Result) 方法）。 本节介绍 AsyncTask 的参数和用法，然后展示一个完整的示例。

#### AsyncTask parameters

In your subclass of AsyncTask, provide the data types for three kinds of parameters:

在 AsyncTask 的子类中，提供三种参数的数据类型：

- "Params" specifies the type of parameters passed to doInBackground() as an array. 指定传递给 doInBackground() 的参数类型为数组。
- "Progress" specifies the type of parameters passed to publishProgress() on the background thread. These parameters are then passed to the onProgressUpdate() method on the main thread. 指定传递给后台线程上的publishProgress()的参数类型。 然后这些参数被传递给主线程上的 onProgressUpdate() 方法。
- "Result" specifies the type of parameter that doInBackground() returns. This parameter is automatically passed to onPostExecute() on the main thread. 指定 doInBackground() 返回的参数类型。 该参数会自动传递给主线程上的onPostExecute()。

Specify a data type for each of these parameter types, or use Void if the parameter type will not be used. For example:

为每个参数类型指定一个数据类型，或者如果不使用该参数类型，则使用 Void。 例如：

```java
public class MyAsyncTask extends AsyncTask <String, Void, Bitmap>{

}
```

In this class declaration:

- The "Params" parameter type is String, which means that MyAsyncTask takes one or more strings as parameters in doInBackground(), for example to use in a query. 在 doInBackground() 中采用一个或多个字符串作为参数，例如在查询中使用。
- The "Progress" parameter type is Void, which means that MyAsyncTask won't use the publishProgress() or onProgressUpdate() methods. 这意味着 MyAsyncTask 不会使用publishProgress() 或onProgressUpdate() 方法。
- The "Result" parameter type is Bitmap. MyAsyncTask returns a Bitmap in doInbackground(), which is passed into onPostExecute(). 它被传递到 onPostExecute() 中。

## Example of an AsyncTask

```java
private class DownloadFilesTask extends AsyncTask<URL, Integer, Long> {
     protected Long doInBackground(URL... urls) {
         int count = urls.length;
         long totalSize = 0;
         for (int i = 0; i < count; i++) {
             totalSize += Downloader.downloadFile(urls[i]);
             publishProgress((int) ((i / (float) count) * 100));
             // Escape early if cancel() is called
             if (isCancelled()) break;
         }
         return totalSize;
     }

     protected void onProgressUpdate(Integer... progress) {
         setProgressPercent(progress[0]);
     }

     protected void onPostExecute(Long result) {
         showDialog("Downloaded " + result + " bytes");
     }
 }
```

上面的示例经历了四个基本AsyncTask步骤中的三个：

- doInBackground()下载内容，一项长期运行的任务。它计算从循环索引下载的文件的百分比for并将其传递给publishProgress(). isCancelled()循环内部的检查for可确保如果任务已取消，系统不会等待循环完成。
- onProgressUpdate()更新百分比进度。每次publishProgress()在 inside 调用该方法时都会调用它doInBackground()，从而更新百分比进度。
- doInBackground()计算下载的总字节数并将其返回。onPostExecute()接收返回的结果并将其传递到onPostExecute()，并在对话框中显示。

## Executing an AsyncTask 执行异步任务

定义 AsyncTask 的子类后，在 UI 线程上实例化它。然后调用execute()实例，传入任意数量的参数。

```java
new DownloadFilesTask().execute(url1, url2, url3);
```

## Cancelling an AsyncTask 取消异步任务

您可以随时从任何线程通过调用该cancel()方法来取消任务。

- 如果任务无法取消，该cancel()方法将返回 false，通常是因为任务已经正常完成。否则，cancel()返回true。
- 要查明任务是否已取消，请定期从 doInBackground(Object[]) 检查 isCancelled() 的返回值，例如从循环内部检查，如下例所示。 如果任务在正常完成之前被取消，则 isCancelled() 方法返回 true。
- AsyncTask 任务取消后，doInBackground() 返回后不会调用 onPostExecute()。 相反，会调用 onCancelled(Object)。 onCancelled(Object) 的默认实现调用 onCancelled() 并忽略结果。
- 默认情况下，允许完成进程中的任务。 要允许 cancel() 中断正在执行任务的线程，请为 mayInterruptIfRunning 的值传递 true。

## Limitations of AsyncTask 局限性

AsyncTask对于某些用例是不切实际的：

- 对设备配置的更改会导致问题。
    - 当设备配置在AsyncTask运行时更改时（例如，如果用户更改屏幕方向），创建AsyncTask的 activity 将被销毁并重新创建。AsyncTask无法访问新创建的 activity，并且不会发布AsyncTask的结果。
- 旧的AsyncTask对象仍然存在，您的应用程序可能会耗尽内存或崩溃。
    - 如果`创建AsyncTask的 activity 被销毁，则Async任务不会与其一起销毁`。例如，如果您的用户在AsyncTask启动后退出应用程序，则除非您调用cancel（），否则AsyncTask将继续使用资源。

何时使用AsyncTask：

- 短时间或可中断的任务。
- 不需要向UI或用户报告的任务。
- 可以不完成的低优先级任务。

For all other situations, use `AsyncTaskLoader`

## Loaders

后台任务通常用于加载预测报告或电影评论等数据。加载数据可能是内存密集型的，并且即使设备配置发生更改，您也希望数据可用。对于这些情况，请使用 Loaders，这是便于将数据加载到 activity 中的类。

Loaders 使用LoaderManager类来管理一个或多个 Loaders 。LoaderManager包括一组回调，用于何时创建加载程序、何时完成加载数据以及何时重置。

#### Starting a loader

Use the LoaderManager class to manage one or more Loader instances within an activity or fragment. Use initLoader() to initialize a loader and make it active. Typically, you do this within the activity's onCreate() method. For example:

```java
// Prepare the loader. Either reconnect with an existing one,
// or start a new one.
getLoaderManager().initLoader(0, null, this);
```

If you're using the `Support Library`, make this call using `getSupportLoaderManager()` instead of `getLoaderManager()`. For example:

```java
getSupportLoaderManager().initLoader(0, null, this);
```

The initLoader() method takes three parameters:

- A unique ID that identifies the loader. This ID can be whatever you want. 标识加载程序的唯一 ID。 这个 ID 可以是任何你想要的。
- Optional arguments to supply to the loader at construction, in the form of a Bundle. If a loader already exists, this parameter is ignored. 在构造时以捆绑包的形式提供给加载器的可选参数。 如果加载程序已经存在，则忽略此参数。
- `A LoaderCallbacks implementation`, which the LoaderManager calls to report loader events. In this example, the local class implements the LoaderManager.LoaderCallbacks interface, so it passes a reference to itself, `this`. LoaderCallbacks 实现，LoaderManager 调用它来报告加载器事件。 在此示例中，本地类实现了 LoaderManager.LoaderCallbacks 接口，因此它传递了对其自身的引用，即 this。

The initLoader() call has two possible outcomes: 有两种可能的结果：

- If the loader specified by the ID already exists, the last loader created using that ID is reused. 使用该 ID 创建的最后一个加载器将被重用。
- If the loader specified by the ID doesn't exist, initLoader() triggers the onCreateLoader() method. This is where you implement the code to instantiate and return a new loader. initLoader() 触发 onCreateLoader() 方法。 这是您实现实例化并返回新加载程序的代码的地方。

注意：无论 initLoader() 创建新的加载程序还是重用现有的加载程序，给定的 LoaderCallbacks 实现都会与加载程序相关联，并在加载程序的状态更改时调用。 如果请求的加载器存在并且已经生成数据，那么系统会立即调用 onLoadFinished() （在 initLoader() 期间），因此请做好发生这种情况的准备。 将对 initLoader() 的调用放在 onCreate() 中，以便当配置更改时 Activity 可以重新连接到同一个加载器。 这样，加载器就不会丢失已经加载的数据。

#### Restarting a loader

When initLoader() reuses an existing loader, it doesn't replace the data that the loader contains, but sometimes you want it to. For example, when you use a user query to perform a search and the user enters a new query, you want to reload the data using the new search term. In this situation, use the restartLoader() method and pass in the ID of the loader you want to restart. This forces another data load with new input data.

About the restartLoader() method:

- restartLoader() uses the same arguments as initLoader().
- restartLoader() triggers the onCreateLoader() method, just as initLoader() does when creating a new loader.
- If a loader with the given ID exists, restartLoader() restarts the identified loader and replaces its data.
- If no loader with the given ID exists, restartLoader() starts a new loader.


#### LoaderManager callbacks

The LoaderManager object automatically calls onStartLoading() when creating the loader. After that, the LoaderManager manages the state of the loader based on the state of the activity and data, for example by calling onLoadFinished() when the data has loaded.

To interact with the loader, use one of the LoaderManager callbacks in the activity where the data is needed:

- Call onCreateLoader() to instantiate and return a new loader for the given ID.
- Call onLoadFinished() when a previously created loader has finished loading. This is typically the point at which you move the data into activity views.
- Call onLoaderReset() when a previously created loader is being reset, which makes its data unavailable. At this point your app should remove any references it has to the loader's data.

The subclass of the Loader is responsible for actually loading the data. Which Loader subclass you use depends on the type of data you are loading, but one of the most straightforward is AsyncTaskLoader, described next. AsyncTaskLoader uses an AsyncTask to perform tasks on a worker thread.

## AsyncTaskLoader

AsyncTaskLoader 是相当于 AsyncTask 的加载器。 AsyncTaskLoader 提供了一个方法 loadInBackground()，该方法在单独的线程上运行。 loadInBackground() 的结果会通过 LoaderManager 回调 onLoadFinished() 自动传递到 UI 线程。

## AsyncTaskLoader usage

To define a subclass of AsyncTaskLoader, create a class that extends `AsyncTaskLoader<D>`, where D is the data type of the data you are loading. For example, the following AsyncTaskLoader loads a list of strings:

```java
public static class StringListLoader extends AsyncTaskLoader<List<String>> {

}
```

Next, implement a constructor that matches the superclass implementation:

- Your constructor takes the application context as an argument and passes it into a call to super().
- If your loader needs additional information to perform the load, your constructor can take additional arguments.

In the example shown below, the constructor takes a query term.

```java
public StringListLoader(Context context, String queryString) {
   super(context);
   mQueryString = queryString;
}
```

To perform the load, use the loadInBackground() override method, the corollary to the doInBackground() method of AsyncTask. For example:

```java
@Override
public List<String> loadInBackground() {
    List<String> data = new ArrayList<String>;
    //TODO: Load the data from the network or from a database
    return data;
}
```

#### Implementing the callbacks

Use the constructor in the onCreateLoader() LoaderManager callback, which is where the new loader is created. For example, this onCreateLoader() callback uses the StringListLoader constructor defined above:

```java
@Override
public Loader<List<String>> onCreateLoader(int id, Bundle args) {
   return new StringListLoader(this, args.getString("queryString"));
}
```

The results of loadInBackground() are automatically passed into the onLoadFinished() callback, which is where you can display the results in the UI. For example:

```java
public void onLoadFinished(Loader<List<String>> loader, List<String> data) {
    mAdapter.setData(data);
}
```

The onLoaderReset() callback is only called when the loader is being destroyed, so you can leave onLoaderReset() blank most of the time, because you won't try to access the data after the loader is destroyed.

When you use AsyncTaskLoader, your data survives device-configuration changes. If your activity is permanently destroyed, the loader is destroyed with it, with no lingering tasks that consume system resources.

Loaders have other benefits too, for example, they let you monitor data sources for changes and reload the data if a change occurs. You learn more about the specifics of loaders in a future lesson.
