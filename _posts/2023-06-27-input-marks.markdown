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

[![activity_main](https://github.com/kyangconn/input-marks-app/blob/master/pictures/activity_main.png)](https://github.com/kyangconn/input-marks-app/blob/master/app/src/main/res/layout/activity_main.xml)


做成这样是因为要求弹出弹窗输入成绩，所以使用了3个按钮，并且使用了自带的初始化toolbar，为了均匀分布则在按钮上使用了：

    app:layout_constraintTop_toTopOf="parent"
    app:layout_constraintStart_toStartOf="parent"
    app:layout_constraintEnd_toEndOf="parent"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintVertical_bias="0.5"

通过调整3个按钮的垂直方向的偏移（`Vertical_bias`）来使3个按钮显示在一条竖线上。

## 首页功能

马上，新问题就来了，这些画好的控件怎么把它功能写出来，该用什么。因为一开始我用的是`basic activity`，这里面的'Activity_main.java'文件中有大量的初始化代码，我根本不知道该在哪里写。  
所以实际上最开始的代码完全依赖GPT写，直到后来我看懂了各个区块才改成现在这样。

于是我把初始的java代码拿给了GPT，让它帮我改好，可是还是有很多不相干的代码，我只能手动safe delete，然后依据GPT的说法去`onCreate()`函数里一大段一大段地加代码。那么新问题出现了，弹窗怎么办？


因为据我搜索得知，这个是可以通过调用Android自带的来实现，可是一个弹窗那么多属性，我要设置哪些？
