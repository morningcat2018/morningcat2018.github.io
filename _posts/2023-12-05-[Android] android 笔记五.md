---
layout:     post
title:      android 笔记五
subtitle:   Layouts and resources for the UI
date:       2023-12-05
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Android
---

[Android developer document - 1.2: Layouts and resources for the UI](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-1-get-started/lesson-1-build-your-first-app/1-2-c-layouts-and-resources-for-the-ui/1-2-c-layouts-and-resources-for-the-ui.html)

user interface (UI) layout 用户界面布局

## Views

The UI consists of a hierarchy of objects called views — every element of the screen is a View. The View class represents the basic building block for all UI components, and the base class for classes that provide interactive UI components such as buttons, checkboxes, and text entry fields.

UI 由称为视图的对象层次结构组成 - 屏幕的每个元素都是一个视图。 View 类代表所有 UI 组件的基本构建块，以及提供交互式 UI 组件（例如按钮、复选框和文本输入字段）的类的基类。

A View has a location, expressed as a pair of left and top coordinates, and two dimensions, expressed as a width and a height. The unit for location and dimensions is the density-independent pixel (dp).

视图具有一个位置（表示为一对左坐标和顶部坐标）和两个维度（表示为宽度和高度）。 位置和尺寸的单位是密度无关像素 (dp)。

The Android system provides hundreds of predefined View subclasses. Commonly used View subclasses described over several lessons include:

- `TextView` for displaying text 用于显示文本
- `EditText` to enable the user to enter and edit text 使用户能够输入和编辑文本
- `Button` and other clickable elements (such as `RadioButton`, `CheckBox`, and `Spinner`) to provide interactive behavior 提供交互行为
- `ScrollView` and `RecyclerView` to display scrollable items 显示可滚动项目
- `ImageView` for displaying images
- `ConstraintLayout` and `LinearLayout` for containing other views and positioning them 用于包含其他视图并定位它们

You can define a View to appear on the screen and respond to a user tap. A View can also be defined to accept text input, or `to be invisible until needed`.

You can specify View elements in layout resource files. Layout resources are written in XML and listed within the layout folder in the res folder in the Project > Android pane.

#### ViewGroup groups

The following are commonly used ViewGroup groups:

- `ConstraintLayout`: A group that places UI elements (child View elements) using constraint connections to other elements and to the layout edges (parent View). 使用与其他元素和布局边缘（父视图）的约束连接来放置 UI 元素（子视图元素）的组。
- `ScrollView`: A group that contains one other child View element and enables scrolling the child View element. 包含另一个子 View 元素并允许滚动子 View 元素的组。
- `RecyclerView`: A group that contains a list of other View elements or ViewGroup groups and enables scrolling them by adding and removing View elements dynamically from the screen. 包含其他 View 元素或 ViewGroup 组的列表，并通过在屏幕上动态添加和删除 View 元素来启用滚动它们的组。

#### Layout ViewGroup groups

Some ViewGroup groups are designated as layouts because they organize child View elements in a specific way and are typically used as the root ViewGroup. Some examples of layouts are:

某些 ViewGroup 组被指定为布局，因为它们以特定方式组织子 View 元素，并且通常用作根 ViewGroup。 布局的一些示例是：

- `ConstraintLayout`: A group of child View elements using constraints, edges, and guidelines to control how the elements are positioned relative to other elements in the layout. ConstraintLayout was designed to make it easy to click and drag View elements in the layout editor. 一组子 View 元素，使用约束、边缘和辅助线来控制元素相对于布局中其他元素的定位方式。 ConstraintLayout 的设计目的是让您可以轻松地在布局编辑器中单击并拖动视图元素。
- `LinearLayout`: A group of child View elements positioned and aligned `horizontally` or `vertically`. 一组水平或垂直定位和对齐的子视图元素。
- `RelativeLayout`: A group of child View elements in which each element is positioned and aligned relative to other elements within the ViewGroup. In other words, the positions of the child View elements can be described in relation to each other or to the parent ViewGroup. 一组子 View 元素，其中每个元素相对于 ViewGroup 内的其他元素进行定位和对齐。 换句话说，子 View 元素的位置可以相对于彼此或相对于父 ViewGroup 进行描述。
- `TableLayout`: A group of child View elements arranged into rows and columns. 一组按行和列排列的子视图元素。
- `FrameLayout`: A group of child View elements in a stack. FrameLayout is designed to block out an area on the screen to display one View. Child View elements are drawn in a stack, with the most recently added child on top. The size of the FrameLayout is the size of its largest child View element. 堆栈中的一组子 View 元素。 FrameLayout 的设计目的是在屏幕上划出一块区域来显示一个 View。 子视图元素在堆栈中绘制，最近添加的子元素位于顶部。 FrameLayout 的大小是其最大子 View 元素的大小。
- `GridLayout`: A group that places its child View elements in a rectangular grid that can be scrolled. 将其子 View 元素放置在可滚动的矩形网格中的组。

Tip: You can explore the layout hierarchy of your app using Hierarchy Viewer. It shows a tree view of the hierarchy and lets you analyze the performance of View elements on an Android-powered device.

## The layout editor

The layout editor shows a visual representation of XML code. You can drag View elements into the design or blueprint pane and arrange, resize, and specify attributes for them. You immediately see the effect of changes you make.

#### Layout editor toolbars

#### Using ConstraintLayout

#### Constraining a UI element

例如，创建以下 XML 代码，将元素的顶部约束到其父元素的顶部：

```xml
app:layout_constraintTop_toTopOf="parent"
```

#### Using a baseline constraint

You can align one UI element that contains text, such as a TextView or Button, with another UI element that contains text. A baseline constraint lets you constrain the elements so that the text baselines match. Select the UI element that has text, and then hover your pointer over the element until the baseline constraint button  appears underneath the element.

#### Using the Attributes pane

#### Creating layout variants for orientations and devices

## Editing XML directly

To view and edit the XML code, open the XML layout file. The layout editor appears with the Design tab at the bottom highlighted. Click the Text tab to see the XML code. The following shows the XML code for a LinearLayout with two Button elements with a TextView in the middle:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.example.android.hellotoast.MainActivity">

    <Button
        android:id="@+id/button_toast"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:background="@color/colorPrimary"
        android:onClick="showToast"
        android:text="@string/button_label_toast"
        android:textColor="@android:color/white" />

    <TextView
        android:id="@+id/show_count"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:layout_marginBottom="8dp"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:background="#FFFF00"
        android:text="@string/count_initial_value"
        android:textAlignment="center"
        android:textColor="@color/colorPrimary"
        android:textSize="160sp"
        android:textStyle="bold"
        android:layout_weight="1"/>

    <Button
        android:id="@+id/button_count"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="8dp"
        android:layout_marginEnd="8dp"
        android:layout_marginStart="8dp"
        android:background="@color/colorPrimary"
        android:onClick="countUp"
        android:text="@string/button_label_count"
        android:textColor="@android:color/white" />
</LinearLayout>
```

#### XML attributes (view properties)

Views have properties that define where a view appears on the screen, its size, how the view relates to other views, and how it responds to user input. When defining views in XML or in the layout editor's Attributes pane, the properties are referred to as attributes.

定义视图在屏幕上显示的位置、其大小、视图与其他视图的关系以及它如何响应用户输入的属性。

For example, in the following XML description of a TextView, the android:id, android:layout_width, android:layout_height, android:background, are XML attributes that are translated automatically into the TextView properties:

```xml
<TextView
       android:id="@+id/show_count"
       android:layout_width="match_parent"
       android:layout_height="wrap_content"
       android:background="@color/myBackgroundColor"
       android:textStyle="bold"
       android:text="@string/count_initial_value" />
```

Attributes generally take this form:
属性通常采用以下形式：

```
android:attribute_name="value"
```

The attribute_name is the name of the attribute. The value is a string with the value for the attribute. For example: attribute_name 是属性的名称。 该值是一个包含属性值的字符串。 例如：

```
android:textStyle="bold"
```

If the value is a resource, such as a color, the @ symbol specifies what kind of resource. For example: 如果该值是资源，例如颜色，则 @ 符号指定资源类型。 例如：

```
android:background="@color/myBackgroundColor"
```

---

Every View and ViewGroup supports its own variety of XML attributes:

每个 View 和 ViewGroup 都支持自己的各种 XML 属性：

- 某些属性特定于 View 子类。 例如，TextView 子类支持textSize 属性。 任何扩展 TextView 子类的元素都会继承这些子类特定的属性。
- 有些属性是所有 View 元素所共有的，因为它们是从根 View 类继承的。 android:id 属性就是一个例子。

有关特定属性的描述，[请参阅 View 类文档的概述部分](https://developer.android.com/reference/android/view/View.html)。

#### Identifying a View

To uniquely identify a View and reference it from your code, you must give it an id. The android:id attribute lets you specify a unique id—a resource identifier for a View.

```
android:id="@+id/button_count"
```

The @+id/button_count part of the attribute creates an id called button_count for a Button (a subclass of View). You use the plus (+) symbol to indicate that you are creating a new id.

#### Referencing a View 引用视图

To refer to an existing resource identifier, omit the plus (+) symbol. For example, to refer to a View by its id in another attribute, such as android:layout_toLeftOf (described in the next section) to control the position of a View, you would use:

要引用现有资源标识符，请省略加号 (+)。 例如，要通过另一个属性中的 id 引用视图，例如 android:layout_toLeftOf 来控制视图的位置，您可以使用：

```
android:layout_toLeftOf="@id/show_count"
```

#### Positioning a View

Some layout-related positioning attributes are required for a View or a ViewGroup, and automatically appear when you add the View or ViewGroup to the XML layout.

###### LinearLayout positioning

LinearLayout is required to have these attributes set:

- android:layout_width
- android:layout_height
- android:orientation

The `android:layout_width` and `android:layout_height` attributes can take one of three values:

- `match_parent` expands the UI element to fill its parent by width or height. When the LinearLayout is the root ViewGroup, it expands to the size of the device screen. For a UI element within a root ViewGroup, it expands to the size of the parent ViewGroup. 扩展 UI 元素以按宽度或高度填充其父元素。 当 LinearLayout 是根 ViewGroup 时，它会扩展到设备屏幕的大小。 对于根 ViewGroup 中的 UI 元素，它会扩展到父 ViewGroup 的大小。
- `wrap_content` shrinks the UI element to the size of its content. If there is no content, the element becomes invisible. 将 UI 元素缩小到其内容的大小。 如果没有内容，该元素将变得不可见。
- Use a fixed number of dp (density-independent pixels) to specify a fixed size, adjusted for the screen size of the device. For example, 16dp means 16 density-independent pixels. 使用固定数量的 dp（与密度无关的像素）指定固定尺寸，并根据设备的屏幕尺寸进行调整。 例如，16dp 表示 16 个与密度无关的像素。 

The `android:orientation` can be:

- `horizontal`: Views are arranged from left to right. 水平的
- `vertical`: Views are arranged from top to bottom.

Other layout-related attributes include:

- android:layout_gravity: This attribute is used with a UI element to control where the element is arranged within its parent. For example, the following attribute centers the UI element horizontally within the parent ViewGroup: 此属性与 UI 元素一起使用，以控制该元素在其父元素中的排列位置。 例如，以下属性使 UI 元素在父 ViewGroup 中水平居中：

```
android:layout_gravity="center_horizontal"
```

- Padding is the space, measured in density-independent pixels, between the edges of the UI element and the element's content. 填充是 UI 元素边缘和元素内容之间的空间，以与密度无关的像素为单位测量。

The size of a View includes its padding. The following are commonly used padding attributes:

- `android:padding`: Sets the padding of all four edges. 设置所有四个边缘的填充。
- `android:paddingTop`: Sets the padding of the top edge.
- `android:paddingBottom`: Sets the padding of the bottom edge. 设置底部边缘的填充。
- `android:paddingLeft`: Sets the padding of the left edge.
- `android:paddingRight`: Sets the padding of the right edge.
- `android:paddingStart`: Sets the padding of the start of the view, in pixels. Used in place of the padding attributes listed above, especially with views that are long and narrow. 设置视图开头的填充（以像素为单位）。 用于代替上面列出的填充属性，尤其是对于又长又窄的视图。
- `android:paddingEnd`: Sets the padding of the end edge of the view, in pixels. Used along with android:paddingStart. 设置视图结束边缘的填充（以像素为单位）。 与 android:paddingStart 一起使用。

Tip: To see all of the XML attributes for a LinearLayout, see the Summary section of the [LinearLayout](https://developer.android.com/reference/android/widget/LinearLayout.html) class definition.

###### RelativeLayout Positioning

Another useful Viewgroup for layout is RelativeLayout, which you can use to position child View elements relative to each other or to the parent. The attributes you can use with RelativeLayout include the following:

另一个有用的布局视图组是RelativeLayout，您可以使用它来相对于彼此或父级定位子View 元素。 您可以与RelativeLayout一起使用的属性包括以下内容：

`android:layout_toLeftOf`: Positions the right edge of this View to the left of another View (identified by its ID). 将此视图的右边缘定位到另一个视图的左侧
`android:layout_toRightOf`: Positions the left edge of this View to the right of another View (identified by its ID).
`android:layout_centerHorizontal`: Centers this View horizontally within its parent. 将此视图在其父视图中水平居中。
`android:layout_centerVertical`: Centers this View vertically within its parent. 将此视图在其父视图中垂直居中。
`android:layout_alignParentTop`: Positions the top edge of this View to match the top edge of the parent. 定位此视图的上边缘以匹配父级的上边缘。
`android:layout_alignParentBottom`: Positions the bottom edge of this View to match the bottom edge of the parent. 定位此视图的底部边缘以匹配父级的底部边缘。

For a complete list of attributes for View and View subclass elements in a RelativeLayout, see [RelativeLayout.LayoutParams](https://developer.android.com/reference/android/widget/RelativeLayout.LayoutParams.html).

#### Style-related attributes

您可以指定样式属性来自定义视图的外观。 没有样式属性（例如 android:textColor、android:textSize 和 android:background）的视图采用应用程序`主题`中定义的样式。

The following are style-related attributes used in lesson on using the layout editor:

- `android:background`: Specifies a color or drawable resource to use as the background. 指定用作背景的颜色或可绘制资源。
- `android:text`: Specifies text to display in the view.
- `android:textColor`: Specifies the text color.
- `android:textSize`: Specifies the text size.
- `android:textStyle`: Specifies the text style, such as bold. 指定文本样式，例如粗体。

## Resource files

Resource files are a way of separating static values from code so that you don't have to change the code itself to change the values. You can store all the strings, layouts, dimensions, colors, styles, and menu text separately in resource files.

资源文件是一种将静态值与代码分离的方法，这样您就不必更改代码本身来更改值。 您可以将所有字符串、布局、尺寸、颜色、样式和菜单文本单独存储在资源文件中。

Resource files are stored in folders located in the res folder when viewing the Project > Android pane. These folders include:

- `drawable`: For images and icons
- `layout`: For layout resource files
- `menu`: For menu items
- `mipmap`: For pre-calculated, optimized collections of app icons used by the Launcher 用于启动器使用的预先计算、优化的应用程序图标集合
- `values`: For colors, dimensions, strings, and styles (theme attributes) 对于颜色、尺寸、字符串和样式（主题属性）

在 XML 布局中引用资源的语法如下：

```
@package_name:resource_type/resource_name
```

- `package_name` is the name of the package in which the resource is located. The package name is not required when you reference resources that are stored in the res folder of your project, because these resources are from the same package. 资源所在的包的名称。 当您引用存储在项目的 res 文件夹中的资源时，不需要包名称，因为这些资源来自同一个包。
- `resource_type` is the R subclass for the resource type. See Resource Types for more about the resource types and how to reference them.
- `resource_name` is either the resource filename without the extension, or the android:name attribute value in the XML element.

For example, the following XML layout statement sets the android:text attribute to a string resource:

```
android:text="@string/button_label_toast"
```

- No package_name is included, because the resource is stored in the strings.xml file in the project.
- The resource_type is `string`.
- The resource_name is `button_label_toast`.

Another example: this XML layout statement sets the android:background attribute to a color resource, and since the resource is defined in the project (in the colors.xml file), the package_name is not specified:

```
android:background="@color/colorPrimary"
```

In the following example, the XML layout statement sets the android:textColor attribute to a color resource. However, the resource is not defined in the project but supplied by Android, so you need to specify the package_name, which is android, followed by a colon:

在以下示例中，XML 布局语句将 android:textColor 属性设置为颜色资源。 但是，该资源不是在项目中定义的，而是由Android提供的，因此需要指定package_name，即android，后面跟一个冒号：

```
android:textColor="@android:color/white"
```

- [Android提供的color](https://developer.android.com/reference/android/R.color.html)

#### Values resource files

例如，必须将字符串保存在单独的资源文件中以用于翻译和本地化应用程序，以便您可以为每种语言创建字符串资源文件而无需更改代码。 图像、颜色、尺寸和其他属性的资源文件可以方便地开发针对不同设备屏幕尺寸和方向的应用程序。

#### Strings

String resources are located in the strings.xml file. You can edit this file directly by opening it in the editor pane:

```xml
<resources>
    <string name="app_name">Hello Toast</string>
    <string name="button_label_count">Count</string>
    <string name="button_label_toast">Toast</string>
    <string name="count_initial_value">0</string>
</resources>
```

The name (for example, button_label_count) is the resource name you use in your XML code, as in the following attribute:

```
android:text="@string/button_label_count"
```

该名称的字符串值是 `<string></string>` 标记内包含的单词 (`Count`)。 （除非引号是字符串值的一部分，否则不要使用引号。）

#### Extracting strings to resources

您还应该将 XML 布局文件中的硬编码字符串提取到字符串资源。

To extract a hard-coded string in an XML layout, follow these steps, as shown in the figure above:

- Click the hard-coded string and press `Alt-Enter` in Windows, or `Option-Return` in Mac OS X.
- Select `Extract string resource`.
- Edit the `Resource name` for the string value.

#### Colors

#### Dimensions

To make dimensions easier to manage, you should separate the dimensions from your code, especially if you need to adjust your layout for devices with different screen densities. Keeping dimensions separate from code also makes it easy to have consistent sizing for UI elements, and to change the size of multiple elements by changing one dimension resource.

为了使尺寸更易于管理，您应该将尺寸与代码分开，特别是当您需要为具有不同屏幕密度的设备调整布局时。 将维度与代码分开还可以轻松地使 UI 元素的大小保持一致，并通过更改一个维度资源来更改多个元素的大小。

Dimension resources are located in the dimens.xml file (inside res > values in the Project > Android pane). The dimens.xml file can actually be a folder holding more than one dimens.xml file—one for each device screen resolution. You can edit each dimens.xml file directly:

维度资源位于dimens.xml 文件中（在项目> Android 窗格中的res > 值内）。 dimens.xml 文件实际上可以是一个包含多个 dimens.xml 文件的文件夹 - 每个设备屏幕分辨率都有一个文件。 您可以直接编辑每个dimens.xml文件：

```xml
<resources>
    <!-- Default screen margins, per the Android Design guidelines. -->
    <dimen name="activity_horizontal_margin">16dp</dimen>
    <dimen name="activity_vertical_margin">16dp</dimen>
    <dimen name="my_view_width">300dp</dimen>
    <dimen name="count_text_size">200sp</dimen>
    <dimen name="counter_height">300dp</dimen>
</resources>
```

[Density-independent pixels (dp)](https://developer.android.com/training/multiscreen/screendensities.html) are independent of screen resolution. For example, 10px (10 fixed pixels) look a lot smaller on a higher resolution screen, but Android scales 10dp (10 device-independent pixels) to look right on different resolution screens. Text sizes can also be set to look right on different resolution screens using scaled-pixel (sp) sizes.

密度无关像素 (dp) 与屏幕分辨率无关。 例如，10px（10 个固定像素）在较高分辨率的屏幕上看起来要小得多，但 Android 会缩放 10dp（10 个与设备无关的像素）以在不同分辨率的屏幕上看起来正确。 还可以使用缩放像素 (sp) 大小将文本大小设置为在不同分辨率的屏幕上看起来正确。

#### Styles

A style is a resource that specifies common attributes such as height, padding, font color, font size, background color. Styles are meant for attributes that modify the look of the view.

Styles are defined in the styles.xml file (inside res > values in the Project > Android pane). You can edit this file directly. Styles are covered in a later chapter, along with the Material Design Specification.

样式是指定常见属性（例如高度、填充、字体颜色、字体大小、背景颜色）的资源。 样式用于修改视图外观的属性。

样式在 styles.xml 文件中定义（在项目 > Android 窗格中的 res > 值内）。 

#### Other resource files

Android Studio defines other resources that are covered in other chapters:

- `Images and icons`: The drawable folder provides icon and image resources. If your app does not have a drawable folder, you can manually create it inside the res folder. For more information about drawable resources, see Drawable Resources in the App Resources section of the Android Developer's Guide.
- `Optimized icons`: The mipmap folder typically contains pre-calculated, optimized collections of app icons used by the Launcher. Expand the folder to see that versions of icons are stored as resources for different screen densities.
- `Menus`: You can use an XML resource file to define menu items and store them in your project in the menu folder. Menus are described in a later chapter.

## Responding to View clicks

A click event occurs when the user taps or clicks a clickable View, such as a Button, ImageButton, ImageView, or FloatingActionButton. When such an event occurs, your code performs an action. In order to make this pattern work, you have to:

当用户点击或单击可单击视图（例如 Button、ImageButton、ImageView 或 FloatingActionButton）时，会发生单击事件。 当此类事件发生时，您的代码将执行一个操作。 为了使该模式发挥作用，您必须：

- Write a Java method that performs the specific action you want the app to do when this event occurs. This method is typically referred to as an event handler. 编写一个 Java 方法，用于执行您希望应用程序在此事件发生时执行的特定操作。 此方法通常称为事件处理程序。
- Associate this event-handler method to the View, so that the method executes when the event occurs. 将此事件处理程序方法关联到视图，以便在事件发生时执行该方法。

#### The onClick attribute

use the android:onClick attribute in the XML layout.

For example, the following XML attribute sets a Button to be clickable, and sets showToast() as the event handler:

```xml
<Button
    android:id="@+id/button_toast"
    android:onClick="showToast"/>
```

When the user taps the button_toast Button, the button's android:onClick attribute calls the showToast() method. In order to work with the android:onClick attribute, the showToast() method must be public and return void. To know which View called the method, the showToast() method must require a view parameter.

当用户点击button_toast按钮时，按钮的android:onClick属性会调用showToast()方法。 为了使用 android:onClick 属性，showToast() 方法必须是公共的并返回 void。 要知道哪个 View 调用了该方法，showToast() 方法必须需要一个视图参数。

在对应activity.java下:

```java
public void showToast(View view) {
        // Do something in response to the button click.
}
```

#### Updating a View

To update a View, for example to replace the text in a TextView, your code must first instantiate an object from the View. Your code can then update the object, which updates the screen.

要更新视图，例如替换 TextView 中的文本，您的代码必须首先从视图实例化一个对象。 然后，您的代码可以更新该对象，从而更新屏幕。

To refer to the View in your code, use the findViewById() method of the View class, which looks for a View based on the resource id. For example, the following statement sets mShowCount to be the TextView in the layout with the resource id show_count:

```java
mShowCount = (TextView) findViewById(R.id.show_count);
```

From this point on, your code can use mShowCount to represent the TextView, so that when you update mShowCount, the TextView is updated.

从这时起，你的代码就可以使用mShowCount来表示TextView了，这样当你更新mShowCount时，TextView也会更新。

For example, when the following Button with the android:onClick attribute is tapped, onClick calls the countUp() method:

```xml
<Button
    android:id="@+id/button_count"
    android:onClick="countUp"/>
```

You can implement countUp() to increment the count, convert the count to a string, and set the string as the text for the mShowCount object:

```java
public void countUp(View view) {
        mCount++;
        if (mShowCount != null)
            mShowCount.setText(Integer.toString(mCount));
}
```

Since you had already associated mShowCount with the TextView for displaying the count, the mShowCount.setText() method updates the TextView on the screen.

---
---
---


[Android developer document - 1.3: Text and scrolling views](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-1-get-started/lesson-1-build-your-first-app/1-3-c-text-and-scrolling-views/1-3-c-text-and-scrolling-views.html)

This chapter describes one of the most often used View subclasses in apps: the TextView, which shows textual content on the screen. A TextView can be used to show a message, a response from a database, or even entire magazine-style articles that users can scroll. This chapter also shows how you can create a scrolling view of text and other elements.

本章介绍应用程序中最常用的 View 子类之一：TextView，它在屏幕上`显示文本内容`。 TextView 可用于显示消息、数据库的响应，甚至用户可以滚动浏览整篇杂志风格的文章。 本章还介绍了如何创建文本和其他元素的滚动视图。

## TextView

If you want to allow users to edit the text, use EditText, a subclass of TextView that allows text input and editing. 

如果要允许用户编辑文本，请使用 EditText，它是 TextView 的子类，允许文本输入和编辑。

#### TextView attributes

您可以使用 TextView 的 XML 属性来控制：

- TextView 在布局中的位置（与任何其他视图一样）
- TextView 本身的显示方式，例如背景颜色
- 文本在 TextView 中的外观，例如初始文本及其样式、大小和颜色

You can extract the text string into a string resource (perhaps called hello_world) that's easier to maintain for multiple-language versions of the app, or if you need to change the string in the future. 

您可以将文本字符串提取到字符串资源（可能称为 hello_world）中，这样对于应用程序的多语言版本或将来需要更改字符串时更容易维护。

```xml
<TextView
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:text="@string/hello_world"
   <!-- more attributes -->
/>
```

In addition to android:layout_width and android:layout_height (which are required for a TextView), the most often used attributes with TextView are the following:

之外，TextView 最常用的属性如下：

- `android:text`: Set the text to display.
- `android:textColor`: Set the color of the text. You can set the attribute to a color value, a predefined resource, or a theme. 设置文本的颜色。 您可以将该属性设置为颜色值、预定义资源或主题。
- `android:textAppearance`: The appearance of the text, including its color, typeface, style, and size. You set this attribute to a predefined style resource or theme that already defines these values. 文本的外观，包括颜色、字体、样式和大小。 您可以将此属性设置为已定义这些值的预定义样式资源或主题。
- `android:textSize`: Set the text size (if not already set by android:textAppearance). Use sp (scaled-pixel) sizes such as 20sp or 14.5sp, or set the attribute to a predefined resource or theme. 设置文本大小（如果尚未通过 android:textAppearance 设置）。 使用 sp（缩放像素）大小，例如 20sp 或 14.5sp，或将属性设置为预定义的资源或主题。
- `android:textStyle`: Set the text style (if not already set by android:textAppearance). Use normal, bold, italic, or bold|italic. 设置文本样式（如果尚未通过 android:textAppearance 设置）。 使用正常、粗体、斜体或粗体|斜体。
- `android:typeface`: Set the text typeface (if not already set by android:textAppearance). Use normal, sans, serif, or monospace. 设置文本字体（如果尚未通过 android:textAppearance 设置）。 使用普通、无衬线、衬线或等宽字体。
- `android:lineSpacingExtra`: Set extra spacing between lines of text. Use sp (scaled-pixel) or dp (device-independent pixel) sizes, or set the attribute to a predefined resource or theme. 设置文本行之间的额外间距。 使用 sp（缩放像素）或 dp（与设备无关的像素）大小，或将属性设置为预定义的资源或主题。
- `android:autoLink`: Controls whether links such as URLs and email addresses are automatically found and converted to clickable (touchable) links. 控制是否自动查找 URL 和电子邮件地址等链接并将其转换为可单击（可触摸）的链接。

Use one of the following with android:autoLink:

- none: Match no patterns (default).
- web: Match web URLs.
- email: Match email addresses.
- phone: Match phone numbers.
- map: Match map addresses.
- all: Match all patterns (equivalent to web|email|phone|map).

For example, to set the attribute to match web URLs, use `android:autoLink="web"`.

#### Using embedded tags in text

To properly display in a text view, text must be formatted following these rules:

要在文本视图中正确显示，文本的格式必须遵循以下规则：

- Enter \n to represent the end of a line, and another \n to represent a blank line. You need to add end-of-line characters to keep paragraphs from running into each other.
- If you have an apostrophe (') in your text, you must escape it by preceding it with a backslash (\'). If you have a double-quote in your text, you must also escape it (\"). You must also escape any other non-ASCII characters. See the "Formatting and Styling" section of String Resources for more details.
- Enter the HTML and `</b>` tags around words that should be in bold.
- Enter the HTML and `</i>` tags around words that should be in italics. Note, however, that if you use curled apostrophes within an italic phrase, you should replace them with straight apostrophes.
- You can combine bold and italics by combining the tags, as in ... words...`</i></b>`. Other HTML tags are ignored.
- To create a long string of text in the strings.xml file, enclose the entire text within `<string name="your_string_name"></string>` (your_string_name is the name you provide the string resource, such as article_text).
- As you enter or paste text in the strings.xml file, the text lines don't wrap around to the next line—they extend beyond the right margin. This is the correct behavior—each new line of text starting at the left margin represents an entire paragraph.

#### Referring to a TextView in code

To refer to a TextView in your Java code, use its resource id. 

Find the TextView and assign it to a variable. You use the findViewById() method of the View class, and refer to the view you want to find using this format:

In which view_id is the resource identifier for the view (such as show_count) :

```java
//  R.id.view_id
TextView mShowCount = (TextView) findViewById(R.id.show_count);
```

After retrieving the View as a TextView member variable, you can then set the text to new text (in this case, mCount_text) using the setText() method of the TextView class:

```java
mShowCount.setText(mCount_text);
```

## Scrolling views

If the information you want to show in your app is larger than the device's display, you can create a scrolling view that the user can scroll vertically by swiping up or down, or horizontally by swiping right or left.

如果您想要在应用程序中显示的信息大于设备的显示屏，您可以创建一个滚动视图，用户可以通过向上或向下滑动来垂直滚动，或者通过向右或向左滑动来水平滚动。

You would typically use a scrolling view for news stories, articles, or any lengthy text that doesn't completely fit on the display. You can also use a scrolling view to combine views (such as a TextView and a Button) within a scrolling view.

还可以使用滚动视图在滚动视图中组合视图（例如 TextView 和 Button）。

#### Creating a layout with a ScrollView

The ScrollView class provides the layout for a vertical scrolling view. (For horizontal scrolling, you would use HorizontalScrollView.) ScrollView is a subclass of FrameLayout, which means that you can place only one View as a child within it; that child contains the entire contents to scroll.

ScrollView 类提供垂直滚动视图的布局。 （对于水平滚动，您可以使用 HorizontalScrollView。）ScrollView 是 FrameLayout 的子类，这意味着您`只能将一个 View 作为子级放置在其中`； 该子项包含要滚动的全部内容。

Even though you can place only one child View inside a ScrollView, the child View can be a ViewGroup with a hierarchy of child View elements, such as a LinearLayout. A good choice for a View within a ScrollView is a LinearLayout that is arranged in a vertical orientation.

即使您只能在 ScrollView 中放置一个子 View，该子 View 也可以是具有子 View 元素层次结构的 ViewGroup，例如 LinearLayout。 对于 ScrollView 中的 View 来说，一个不错的选择是沿垂直方向排列的 LinearLayout。

#### ScrollView and performance 性能

All of the contents of a ScrollView (such as a ViewGroup with View elements) occupy memory and the view hierarchy even if portions are not displayed on screen. This makes ScrollView useful for smoothly scrolling pages of free-form text, because the text is already in memory. However, a ScrollView with a ViewGroup with View elements can use up a lot of memory, which can affect the performance of the rest of your app.

即使部分内容未在屏幕上显示，ScrollView 的所有内容（例如具有 View 元素的 ViewGroup）也会占用内存和视图层次结构。 这使得 ScrollView 对于平滑滚动自由格式文本的页面非常有用，因为文本已经在内存中。 但是，带有 ViewGroup 和 View 元素的 ScrollView 可能会`占用大量内存`，这可能会影响应用程序其余部分的性能。

Using nested instances of LinearLayout can also lead to an excessively deep view hierarchy, which can slow down performance. Nesting several instances of LinearLayout that use the android:layout_weight attribute can be especially expensive as each child View needs to be measured twice. Consider using flatter layouts such as RelativeLayout or GridLayout to improve performance.

使用 LinearLayout 的嵌套实例还可能导致视图层次结构过深，从而降低性能。 嵌套使用 android:layout_weight 属性的 LinearLayout 的多个实例可能会特别昂贵，因为每个子 View 都需要测量两次。 `考虑使用更扁平的布局（例如RelativeLayout 或GridLayout）来提高性能`。

Complex layouts with ScrollView may suffer performance issues, especially with images. We recommend that you not use images within a ScrollView. To display long lists of items, or images, consider using a RecyclerView, which is covered in another lesson.

使用 ScrollView 的复杂布局可能会遇到性能问题，尤其是图像。 我们`建议您不要在 ScrollView 中使用图像`。 要`显示长列表或图像，请考虑使用` [RecyclerView](https://developer.android.com/reference/android/support/v7/widget/RecyclerView.html)。

#### ScrollView with a TextView

To display a scrollable magazine article on the screen, you might use a RelativeLayout that includes a separate TextView for the article heading, another for the article subheading, and a third TextView for the scrolling article text (see the figure below), set within a ScrollView. The only part of the screen that would scroll would be the ScrollView with the article text.

要在屏幕上显示可滚动的杂志文章，您可以使用relativelayout，其中包含一个单独的textview用于文章标题，另一个用于文章副标题，第三个textview用于滚动文章文本（参见下图），设置在 滚动视图。 屏幕上唯一会滚动的部分是带有文章文本的 ScrollView。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f32a5126e97b4bf4a5e8055fddd7ed41~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=670&h=458&s=74769&e=png&b=fafafa)

#### ScrollView with a LinearLayout

A ScrollView can contain only one child View; however, that View can be a ViewGroup that contains several View elements, such as LinearLayout. You can nest a ViewGroup such as LinearLayout within the ScrollView, thereby scrolling everything that is inside the LinearLayout.

一个ScrollView只能包含一个子View； 但是，该 View 可以是包含多个 View 元素的 ViewGroup，例如 LinearLayout。 您可以在 ScrollView 中嵌套 ViewGroup（例如 LinearLayout），从而滚动 LinearLayout 内的所有内容。

For example, if you want the subheading of an article to scroll along with the article even if they are separate TextView elements, add a LinearLayout to the ScrollView as a single child View as shown in the figure below, and then move the TextView subheading and article elements into the LinearLayout. The user scrolls the entire LinearLayout which includes the subheading and the article.

例如，如果您希望文章的副标题与文章一起滚动，即使它们是单独的 TextView 元素，也可以将 LinearLayout 作为单个子 View 添加到 ScrollView 中，如下图所示，然后将 TextView 副标题移动到滚动视图中： 将文章元素放入 LinearLayout 中。 用户滚动整个 LinearLayout，其中包括副标题和文章。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c597b535b344c71944e63cff26cdd08~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=670&h=458&s=78527&e=png&b=fafafa)

When adding a LinearLayout inside a ScrollView, use match_parent for the LinearLayout android:layout_width attribute to match the width of the parent ScrollView, and use wrap_content for the LinearLayout android:layout_height attribute to make it only large enough to enclose its contents.

在 ScrollView 中添加 LinearLayout 时，使用 LinearLayout android:layout_width 属性的 match_parent 来匹配父 ScrollView 的宽度，并使用 LinearLayout android:layout_height 属性的 `wrap_content` 使其足够大以包围其内容。

Since ScrollView only supports vertical scrolling, you must set the LinearLayout orientation attribute to vertical (android:orientation="vertical"), so that the entire LinearLayout will scroll vertically. For example, the following XML layout scrolls the article TextView along with the article_subheading TextView:

由于ScrollView只支持垂直滚动，所以必须将LinearLayout的orientation属性设置为vertical（`android:orientation="vertical"`），这样整个LinearLayout才会垂直滚动。 例如，以下 XML 布局将文章 TextView 与article_subheading TextView 一起滚动：

```xml
<ScrollView
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:layout_below="@id/article_heading">

   <LinearLayout
      android:layout_width="match_parent"
      android:layout_height="wrap_content"
      android:orientation="vertical">

      <TextView
         android:id="@+id/article_subheading"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:padding="@dimen/padding_regular"
         android:text="@string/article_subtitle"
         android:textAppearance=
                       "@android:style/TextAppearance.DeviceDefault" />

      <TextView
         android:id="@+id/article"
         android:layout_width="wrap_content"
         android:layout_height="wrap_content"
         android:autoLink="web"
         android:lineSpacingExtra="@dimen/line_spacing"
         android:padding="@dimen/padding_regular"
         android:text="@string/article_text" />
   </LinearLayout>

</ScrollView>
```


