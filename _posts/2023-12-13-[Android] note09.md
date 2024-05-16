---
layout:     post
title:      android 笔记十
subtitle:   Buttons and clickable images
date:       2023-12-13
author:     BY morningcat
header-img: img/20190213/top_2019_Space.jpg
catalog: true
tags:
    - Android
---

[Android developer document - 4.1: Buttons and clickable images](https://google-developer-training.github.io/android-developer-fundamentals-course-concepts-v2/unit-2-user-experience/lesson-4-user-interaction/4-1-c-buttons-and-clickable-images/4-1-c-buttons-and-clickable-images.html)

## Buttons and clickable images

用户点击或单击以执行操作的任何元素都称为可单击元素。

用户交互通常涉及点击、键入、使用手势或说话

如果应用使用常见手势（如双击、长按、甩动等），则可以利用该 GestureDetector 类来检测常见手势。

GestureDetectorCompat 允许您检测常见手势，而无需自行处理单个触摸事件。它使用 MotionEvent 对象检测各种手势和事件，这些对象通过手指（或鼠标、笔或轨迹球）报告移动。

- 实现 GestureDetector.OnGestureListener 接口以检测所有标准手势，
- 扩展该 GestureDetector.SimpleOnGestureListener 类，通过重写所需的方法，可以使用它来仅处理几个手势。

SimpleOnGestureListener 提供  onDown() 、 onLongPress() 、 onFling() 、 onScroll() 和 onSingleTapUp() 等方法。

若要截获触摸事件，请重写 GestureDetectorCompat 类的 onTouchEvent() 回调:

```java
@Override 
public boolean onTouchEvent(MotionEvent event){ 
      this.mDetector.onTouchEvent(event);
      return super.onTouchEvent(event);
}
```

检测所有手势

- 收集有关触摸事件的数据。
- 解释数据，查看它是否符合应用支持的任何手势的条件。

```java
// This example shows an Activity, but you would use the same approach if
// you were subclassing a View.
   @Override
   public boolean onTouchEvent(MotionEvent event){
       int action = MotionEventCompat.getActionMasked(event);
       switch(action) {
          case (MotionEvent.ACTION_DOWN) :
             Log.d(DEBUG_TAG,"Action was DOWN");
             return true;
          case (MotionEvent.ACTION_MOVE) :
             Log.d(DEBUG_TAG,"Action was MOVE");
             return true;
          case (MotionEvent.ACTION_UP) :
             Log.d(DEBUG_TAG,"Action was UP");
             return true;
          case (MotionEvent.ACTION_CANCEL) :
             Log.d(DEBUG_TAG,"Action was CANCEL");
             return true;
          case (MotionEvent.ACTION_OUTSIDE) :
             Log.d(DEBUG_TAG,"Movement occurred outside bounds " +
                    "of current screen element");
             return true;      
          default : 
             return super.onTouchEvent(event);
    }      
}
```


## 4.2: Input controls

输入控件

- EditText 字段（子类 TextView ），用于使用键盘输入文本
- SeekBar 用于向左或向右滑动到设置
- CheckBox 用于选择一个或多个选项的元素
- RadioGroup 用于选择一个选项的 RadioButton 元素
- ToggleButton 和 Switch ：打开或关闭选项
- Spinner 用于选择一个选项的下拉菜单

Android 设备使用多种输入法，包括方向键 （D-pad）、轨迹球、触摸屏、外接键盘等

#### 复选框

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="vertical"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent">

        <CheckBox android:id="@+id/checkbox1_chocolate"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/chocolate_syrup" />
        <CheckBox android:id="@+id/checkbox2_sprinkles"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/sprinkles" />
        <CheckBox android:id="@+id/checkbox3_nuts"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@string/crushed_nuts" />

</LinearLayout>
```

```java
public void onSubmit(View view) {
   StringBuffer toppings = new StringBuffer().append(getString(R.string.toppings_label));
   if  (((CheckBox) findViewById(R.id.checkbox1_chocolate)).isChecked()) {
      toppings.append(getString(R.string.checkbox1));
   }
   if  (((CheckBox) findViewById(R.id.checkbox2_sprinkles)).isChecked()) {
      toppings.append(getString(R.string.checkbox2));
   }
   if  (((CheckBox) findViewById(R.id.checkbox3_nuts)).isChecked()) {
      toppings.append(getString(R.string.checkbox3));
   }
// Code to display the result...
}
```

#### 单选按钮

每个单选按钮都是类 RadioButton 的一个实例。单选按钮通常放置在布局中的 a RadioGroup 中。当多个 RadioButton 元素位于 中 RadioGroup 时，选择一个 RadioButton 元素将清除所有其他元素。

```xml
<RadioGroup
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginLeft="24dp"
        android:layout_marginStart="24dp"
        android:orientation="vertical"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/delivery_label">

        <RadioButton
            android:id="@+id/sameday"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onRadioButtonClicked"
            android:text="Same day messenger service" />

        <RadioButton
            android:id="@+id/nextday"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onRadioButtonClicked"
            android:text="Next day ground delivery" />

        <RadioButton
            android:id="@+id/pickup"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onRadioButtonClicked"
            android:text="Pick up" />
</RadioGroup>
```

```java
public void onRadioButtonClicked(View view) {
   // Check to see if a button has been clicked.
   boolean checked = ((RadioButton) view).isChecked();
   // Check which radio button was clicked.
   switch(view.getId()) {
      case R.id.sameday:
         if (checked)
            // Code for same day service ...
            break;
      case R.id.nextday:
         if (checked)
            // Code for next day delivery ...
            break;
      case R.id.pickup:
         if (checked)
            // Code for pick up ...
            break;
   }
}
```

#### Spinner

当用户有三个以上的选择时，微调器效果很好，因为微调器会根据需要滚动，并且微调器不会占用布局中的太多空间。如果仅提供两个或三个选项，并且布局中有空间，则可能需要使用单选按钮而不是微调器。

```xml
<Spinner
   android:id="@+id/label_spinner"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>

<string-array name="labels_array">
        <item>Home</item>
        <item>Work</item>
        <item>Mobile</item>
        <item>Other</item>
</string-array>
```

```java
 @Override
 protected void onCreate(Bundle savedInstanceState) {
    // ... Rest of onCreate code ...
    // Create the spinner.
    Spinner spinner = findViewById(R.id.label_spinner);
    if (spinner != null) {
        spinner.setOnItemSelectedListener(this);
    }
    // Create ArrayAdapter using the string array and default spinner layout.

    // Create ArrayAdapter using the string array and default spinner layout.
    ArrayAdapter<CharSequence> adapter = ArrayAdapter.createFromResource(this, 
                 R.array.labels_array, android.R.layout.simple_spinner_item);
    // Specify the layout to use when the list of choices appears.

     // Specify the layout to use when the list of choices appears.
    adapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item);
    // Apply the adapter to the spinner.
    if (spinner != null) {
        spinner.setAdapter(adapter);
    }
    // ... End of onCreate code ...
}

public void onItemSelected(AdapterView<?> adapterView, View view, int i, long l) {
    String spinner_item = adapterView.getItemAtPosition(i).toString();
    // Do something with spinner_item string.
}
```

#### ToggleButton

```xml
<ToggleButton
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/my_toggle"
        android:text=""
        android:onClick="onToggleClick"/>
```

```java
public void onToggleClick(View view) {
   ToggleButton toggle = findViewById(R.id.my_toggle);
   toggle.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
      public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
         StringBuffer onOff = new StringBuffer().append("On or off? ");
         if (isChecked) { // The toggle is enabled
            onOff.append("ON ");
         } else { // The toggle is disabled
            onOff.append("OFF ");
         }
         Toast.makeText(getApplicationContext(), onOff.toString(), Toast.LENGTH_SHORT).show();
      }
   });
}
```

#### Switch

```xml
<Switch
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:id="@+id/my_switch"
    android:text="@string/turn_on_or_off"
    android:onClick="onSwitchClick"/>
```

```java
public void onSwitchClick(View view) {
   Switch aSwitch = findViewById(R.id.my_switch);
   aSwitch.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
      public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
         StringBuffer onOff = new StringBuffer().append("On or off? ");
         if (isChecked) { // The switch is enabled
            onOff.append("ON ");
         } else { // The switch is disabled
            onOff.append("OFF ");
         }
         Toast.makeText(getApplicationContext(), onOff.toString(), Toast.LENGTH_SHORT).show();
      }
   });
}
```

提示： 还可以使用该 setChecked(boolean) 方法以编程方式更改 的状态 Switch 。但请注意，在这种情况下，不会执行属性 android:onClick() 指定的方法。

#### 文本输入

```xml
<EditText
    android:id="@+id/edit_simple"
    android:layout_height="wrap_content"
    android:layout_width="match_parent"
    android:inputType="textShortMessage"
    android:inputType="textLongMessage"
    android:hint="@string/enter_text_here">
</EditText>
```

自定义 EditText 视图的属性:
- android:inputType="textCapCharacters" ：将文本条目设置为全部大写字母。
- android:inputType="textCapSentences" ：每个句子以大写字母开头。
- android:inputType="textMultiLine" ：设置文本字段以启用多行。此值可用于与其他属性结合使用。若要合并值，请使用竖线 （ | ） 字符将它们连接起来。
- android:inputType="textPassword" ：将用户输入的每个字符变成一个点以隐藏输入的密码。
- android:inputType="number" ：将文本输入限制为数字。
- android:textColorHighlight="#7cff88" ：设置所选（突出显示）文本的背景颜色。
- android:hint="@string/my_hint" ：设置文本显示在为用户提供提示的字段中，例如“输入消息”。

```java
EditText editText = findViewById(R.id.editText_main);
String showString = editText.getText().toString();

```

您可以使用竖线 | 字符（Java 按位 OR）来组合属性的 android:inputType 属性值：

```
android:inputType="textAutoCorrect|textCapSentences"
```

## 4.3: Menus and pickers

#### 菜单类型














