---
layout: post
title: "《第一行代码》Chapter 02"
subtitle: '探究活动'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 目录

1.[@+id](#正文)

2.[R类](#r类)

3.[label](#label)

4.[主活动](#主活动)

5.[重写方法的快捷键](#重写方法的快捷键)

6.[Intent](#intent)

7.[返回数据给上一个活动](#返回数据给上一个活动)

8.[重写back键逻辑](#重写back键逻辑)

9.[返回栈](#返回栈)

10.[活动状态](#活动状态)

11.[活动生存期](#活动生存期)

12.[保存活动的临时数据和状态](#保存活动的临时数据和状态)

13.[活动的启动模式](#活动的启动模式)

14.[杀掉当前APP的进程](#杀掉当前APP的进程)

15.[显示启动活动的推荐写法](#显示启动活动的推荐写法)

## 正文

---
#### @+id
> @：告诉工具，这不是字符串，而是引用资源

> +：告诉工具，若该资源不存在，则创建资源

> id：告诉工具，该资源是id，而不是其他资源（如string、bool、drawable等）

---
#### R类

> 项目中添加的任何资源都会在R类中自动生成一个相应的资源ID。

> R类由Android工具链自动生成

---
#### label

> 主活动在Manifest文件的<activity>标签中指定的**label**属性，不仅是主题栏（TitleBar）的内容，也是启动器（Launcher）中APP的名称。

---
#### 主活动

> 若APP没有主活动，依旧可以安装，但是无法在启动器（Launcher）看到或打开这个APP，这种APP一般是作为第三方服务供其他APP使用，如：支付宝快捷支付服务。

---
#### 重写方法的快捷键

> ctrl+o：打开该class可以重写的方法列表

---
#### Intent

> 每个Intent只能有一个action，但是可以有多个category。

> 隐式Intent启动前，最好做检查，以免程序崩溃

```
if (intent.resolveActivity(getPackageManager()) != null) {
        startActivity(intent);
    }

```

> 注（转自[通用 Intent](https://developer.android.google.cn/guide/components/intents-common#java)）

> 如果设备上没有可接收隐式 Intent 的应用，您的应用将在调用 startActivity() 时崩溃。如需事先验证是否存在可接收 Intent 的应用，请对 Intent 对象调用 resolveActivity()。如果结果为非空，则至少有一个应用能够处理该 Intent，并且可以安全调用 startActivity()。如果结果为空，则您不应使用该 Intent。

---
#### 返回数据给上一个活动

> 上一个活动用startActivityForResult(Intent intent, int REQUEST_CODE)启动当前活动

> 当前活动新建Intent用putExtra()方法装载要返回的数据，并调用setResult(int RESULT_CODE, Intent intent)将Intent返回给上一个活动

> 上一个活动重写onActivityResult(int requestCode, int resultCode, Intent intent)来做判断之后获取Intent中装载的数据

---
#### 重写back键逻辑

> 重写onBackPressed()方法

---
#### 返回栈

> Android使用任务（Task）管理活动

> 任务（Task）：一组存放在栈里的活动的集合

> 当启动新的活动，新活动入栈并处于栈顶。当使用finish()方法或按下back键销毁活动时，活动出栈。

> Android总是显示处于栈顶的活动

---
#### 活动状态

> 运行状态：活动处于栈顶

> 暂停状态：活动不在栈顶，但仍可见（如：活动被对话框遮挡）

> 停止状态：活动不在栈顶，而且不可见

---
#### 活动生存期

> 前台生存期：onResume到onPause

> 可见生存期：onStart到onStop

> 完整生存期：onCreate到onDestroy

---
#### 保存活动的临时数据和状态

> 重写onSaveInstanceSate()，数据保存为Bundle类型，以键值对形式保存

> 在onCreate(Bundle savedInstanceState)方法中从Bundle类参数中取出数据
 
---
#### 活动的启动模式

> 在manifest文件中<activity>标签中指定launchMode来选择启动模式

> 1. standard：

>> 启动活动时，不管返回栈有没有该活动实例，都创建一个新的活动实例

>> 优点：保持了活动入栈的顺序

>> 缺点：返回栈内存消耗大

> 2. singleTop：

>> 启动活动时，检查返回栈的栈顶，若启动的活动已经在栈顶，则直接使用该活动实例，不创建新的

>> 比standard省一点内存

> 3. singleTak：

>> 启动活动时，检查返回栈中是否已经有该活动实例，若有，则直接使用该活动实例，并且将该活动实例之上的其他活动实例出栈（如果有其他活动实例）

>> 优点：比singleTop更省内存

>> 缺点：破坏了活动入栈的顺序

> 4. singleInstance：

>> 指定为singleInstance启动模式的活动，独自放在一个与其他3种启动模式不同的返回栈中，当其他APP访问该活动时，不会创建新的活动实例，而是使用该返回栈的活动实例

>> 应用：其他APP共享当前APP的活动

---
#### 杀当前APP的进程

> android.os.Process.killProcess(android.os.Process.myPid());

---
#### 显示启动活动的推荐写法

> 将启动活动封装成函数，形参表明需要用到的参数，方便阅读理解，也方便调用。
