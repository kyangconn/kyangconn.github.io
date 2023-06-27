---
layout: post
title: "App for calculate marks"
time:  2023-06-27 14:00:00 0800
catagory: Readme
---

# **这是什么？**

这是一个学校要求完成的安卓程序，它被要求具有输入成绩，清空成绩，显示统计数据。其中统计要求有数量，总分，最高分，最低分，平均分五个数据。

# **我的想法**

其实最开始我没有任何想法，因为我知道这些要求时处于对Android一无所知~~，甚至Android Studio都没有配置~~，但好在互联网能够告诉我如何去做。功能很简单，一个`.size()`可以获取总数，`total += marks.get(i)`可以获取总分，最大和最小都有写好的代码，平均数更是`ave = total/size`就可以，但对我来说难点显而在于如何使用安卓，对于一个完全的新手，显然读官方文档不够一个短学期去完成一个原生小程序，更别说还有第二个。所以我就~~心安理得~~去找了GPT。

## 首页界面

很快啊，我就知道功能哪里写界面哪里写了，那么新问题来了，这个xml怎么写，有哪些属性，我该怎么去达到我要的效果。于是我又开始问GPT和搜索引擎。~~很快啊~~一个星期了，我终于完成了我想要的第一个页面。

[![activity_main](https://github.com/kyangconn/input-marks-app/raw/master/pictures/activity_main.png)](https://github.com/kyangconn/input-marks-app/blob/master/app/src/main/res/layout/activity_main.xml)

做成这样是因为要求弹出弹窗输入成绩，所以使用了3个按钮，并且使用了自带的初始化toolbar，为了均匀分布则在按钮上使用了：

```xml
    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintVertical_bias="0.5"
```

通过调整3个按钮的垂直方向的偏移（`Vertical_bias`）来使3个按钮显示在一条竖线上。

## 首页功能

马上，新问题就来了，这些画好的控件怎么把它功能写出来，该用什么。因为一开始我用的是`basic activity`，这里面的'Activity_main.java'文件中有大量的初始化代码，我根本不知道该在哪里写。  
所以实际上最开始的代码完全依赖GPT写，直到后来我看懂了各个区块才改成现在这样。

于是我把初始的java代码拿给了GPT，让它帮我改好，可是还是有很多不相干的代码，我只能手动safe delete，然后依据GPT的说法去`onCreate()`函数里一大段一大段地加代码。那么新问题出现了，弹窗怎么办？

因为据我搜索得知，这个是可以通过调用Android自带的来实现，可是一个弹窗那么多属性，我要设置哪些？

于是我又去找了GPT，告诉它我需要一个输入框，两个按钮的弹窗，很快啊，我就得到了一个xml文件和一部分代码：

```xml
<!-- dialog_custom_input.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <EditText
        android:id="@+id/dialog_input"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="请输入数字"
        android:inputType="numberDecimal" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="end">

        <Button
            android:id="@+id/dialog_button1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="按钮1" />

        <Button
            android:id="@+id/dialog_button2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="按钮2" />
    </LinearLayout>
</LinearLayout>
```

以及

```java
Enter_mark.setOnClickListener(view -> {
    // 加载自定义布局
    LayoutInflater inflater = getLayoutInflater();
    View dialogView = inflater.inflate(R.layout.dialog_custom_input, null);

    // 创建弹窗并设置自定义布局
    AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
    builder.setView(dialogView);

    // 获取输入框和按钮的引用
    EditText dialogInput = dialogView.findViewById(R.id.dialog_input);
    Button dialogButton1 = dialogView.findViewById(R.id.dialog_button1);
    Button dialogButton2 = dialogView.findViewById(R.id.dialog_button2);

    // 创建弹窗
    AlertDialog dialog = builder.create();

    // 设置按钮1的点击监听器
    dialogButton1.setOnClickListener(v -> {
        // 在这里处理按钮1的点击事件，例如获取输入框的值
        float input;
        try {
            input = Float.parseFloat(dialogInput.getText().toString());
        } catch (NumberFormatException e) {
            input = 0;
            Toast.makeText(MainActivity.this, "无效输入", Toast.LENGTH_SHORT).show();
        }

        // ...处理输入值
        // 关闭对话框
        dialog.dismiss();
    });

    // 设置按钮2的点击监听器
    dialogButton2.setOnClickListener(v -> {
        // 在这里处理按钮2的点击事件
        // 关闭对话框
        dialog.dismiss();
    });

    // 显示弹窗
    dialog.show();
});
```

## 数据传递

好，那么现在在我看来最重量级的问题都解决了，想必接下来的也没问题！  
那么接下来开始写把这个数组传递给第二页吧！  
等等，该怎么写？函数传参？调用final对象？想了想都不是，那么只好去问万能的GPT了！

然后我就又花了3天时间去研究到底是怎么传的以及为什么会有各种各样的报错，包括但不限于传了个空的导致闪退，编译说这个类型错了，写 不 进 去！

这3天里我用各种报错和描述与GPT疯狂互动，最后得出了两段能跑，但被警告的代码：

```java
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
intent.putExtra("inputMarks", inputMarks);
startActivity(intent);
```

```java
Intent intent = getIntent();
ArrayList<Float> inputMarks = (ArrayList<Float>) intent.getSerializableExtra("inputMarks");
```

当然最后我还是优化掉了这些报错，现在我还没那个能力~~GPT也不行~~。

## 第二页布局

在第二页的要求里，需要显示5个值，并且要有相应的文字对应，听起来很简单，但是如何做好看是个问题，我首先想到用约束套线性，但如何给这些控件赋值是我的盲区，于是我又去找了GPT去寻找该如何设计。

很快啊，我就得到了超长的xml文件，使用了约束套滚动套竖向线性套横向线性，听着就晕；´д｀）ゞ。但没事，放在Android Studio里一预览，懂了。

[![activity_second](https://github.com/kyangconn/input-marks-app/raw/master/pictures/activity_second.png)](https://github.com/kyangconn/input-marks-app/blob/master/app/src/main/res/layout/activity_second.xml)

好，虽然看着很头疼，但之后就不需要看了，只需要去java代码里就行了，写算数不是简简单单！  
事实上确实没什么难度，大部分时间我都在解决各种各样的报错和闪退。

## 第二页功能

5个值，让程序员为我写23行代码！

因为每个文本框，都需要去找一次对象，导致这里看起来很复杂，但我已经尽力换行了。

首先就是总数，这个简单，每个都自带：

```java
tmpString = String.valueOf(inputMarks.size());
TextView1.setText(tmpString);
```

然后是总分，这个也简单，加起来就行：

```java
for(int i = 0; i < inputMarks.size(); i++)
  tmpFloat += inputs.get(i);
TextView2.setText(String.valueOf(tmpFloat));
```

接下来是平均值：

```java
tmpFloat /= inputs.size();
TextView3.setText(String.valueOf(tmpFloat));
```

最后是最高和最低，这个正好学过：

```java
tmpFloat = inputs.get(0);
for (int i = 0; i < inputMarks.size(); i++) {
  if(inputMarks.get(i) > tmpFloat)
    tmpFloat = inputs.get(i);
}
TextView4.setText(String.valueOf(tmpFloat));
```

两个差不多，起手赋值一个临时变量，假定它是最小，然后进循环，与每一个判断大小，比临时大那就当前最大，出循环了临时值就是最大。最小也一样，只不过是判断比临时小。

```java
tmpFloat = inputs.get(0);
for (int i = 0; i < inputMarks.size(); i++) {
  if(inputMarks.get(i) < tmpFloat)
    tmpFloat = inputs.get(i);
}
TextView5.setText(String.valueOf(tmpFloat));
```

到这里，恭喜我自己！基本功能做完了，程序能跑了！
