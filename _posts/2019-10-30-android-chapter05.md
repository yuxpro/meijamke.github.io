---
layout: post
title: "《第一行代码》Chapter 05"
subtitle: '广播机制'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 正文

#### 广播类型
- 标准广播：一（Broadcast）对多（BroadcastReceiver），且不可被截断
- 有序广播：一（Broadcast）对一（BroadcastReceiver），且可被截断（abort），可设置广播接收器（BroadcastReceiver）的优先级

---
#### 广播接收器有两种注册方法

##### 动态注册
1. 创建class继承自BroadcastReceiver，重写onReceive()方法，当接收到监听的广播时就会调用onReceive()方法，所以需要处理的事务都在onReceive()方法中执行。
2. 在需要监听某些广播的活动中注册和注销广播接收器
    1. 创建IntentFilter，确定需要监听的广播，如：
`IntentFilter intentFilter = new IntentFilter();
 intentFilter.addAction(String);`。

    2. 创建步骤1的类的实例
    3. 在onCreate()或onResume()中注册广播接收器：`registerReceiver(BroadcastReceiver, IntentFilter)`
    4. 在onDestroy()或onStop()中注销广播接收器：`unregisterReceiver(BroadcastReceiver);`

##### 静态注册
1. 创建class继承自BroadcastReceiver，重写onReceive()方法，当接收到监听的广播时就会调用onReceive()方法，所以需要处理的事务都在onReceive()方法中执行。
2. 在Manifest文件中，注册广播接收器，并且确定监听的广播类型，如：

```
<receiver name="[完整包名]+步骤1的类路径">
    <intent-filter>
        <action name=""/>
    </intent-filter>
</receiver>
```
其中receiver标签的name属性指向的就是广播接收器，action标签的name属性指向的就是监听的广播类型。

##### 注：<intent-filter>还可以说明category属性，一般不写就是默认的category。

##### 注：广播接收器的onReceive()方法中不允许开启子线程，因为onReceive()长时间运行而没有结束，就会报错。广播接收器更多的是起到打开程序的其他组件的作用，如：发送通知、开启服务等。

##### 注：一般不使用静态广播来触发后台任务，更多的使用作业管理（以前是jobService、android job，现在推荐使用workManager）。

---
#### 小结：
##### 让APP在不运行时也能执行代码
    1.尽可能使用作业管理
    	比使用静态广播接收器更好
    	作业可以添加许多约束加以管理，而且作业十分省电
    2.不得不使用静态广播接收器
    	通常是在较旧的设备上捕捉特定的广播意图
##### APP运行时获取广播意图
	1.动态广播接收器

---
#### 发送自定义广播
1. 创建Intent，作为广播发送的内容

```
Intent intent = new Intent();
intent.addAction(String);
```
2. 发送广播
    1. 发送标准广播：`sendBroadcast(Intent);`
    2. 发送有序广播：`sendOrderedBroadcast(Intent);`
#### 本地广播
- 目的：解决广播的安全性问题
- 使用：
    1. 创建class继承自BroadcastReceiver，重写onReceive()方法，当接收到监听的广播时就会调用onReceive()方法，所以需要处理的事务都在onReceive()方法中执行。
    2. 创建LocalBroadcastManager：`LocalBroadcastManager localManager=LocalBroadcastManager.getInstance(Context);`
    3. 使用LocalBroadcastManager，发送广播以及注册和注销广播：
    
```
onCreate(){
    ...
    //发送广播
    localManager.sendBroadcast(Intent);
    //注册广播
    localManager.registerBroadcast(BroadcastReceiver, IntentFilter);
}
onDestroy(){
    ...
    //注销广播
    localManager.unregisterBroadcast(BroadcastReceiver);
}
```

#### git使用
1. 创建github账号
2. 安装git
3. 配置git

`git config --user.name "xxx"`

`git config --user.email "xxx"`
4. 创建本地仓库
 
`git init`
5. 删除本地仓库：直接手动删除仓库所在文件夹
6. 添加要提交的文件
 
`git add xxx//添加xxx文件`

`git add .//添加全部文件`
7. 提交文件到本地仓库

`git commit -m "xxx"//xxx用于描述提交的内容`
