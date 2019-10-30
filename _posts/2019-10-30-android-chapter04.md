---
layout: post
title: "《第一行代码》Chapter 04"
subtitle: '探究碎片'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 正文
#### 碎片（Fragment）
- 一种可以嵌入活动当中的UI片段，能让程序更加合理使用大屏幕的空间
- 在平板上使用广泛
- 本质是自适应，即使不是平板，当屏幕旋转时，也可以考虑使用Fragment管理UI

#### 碎片的使用
1. 创建碎片：
    1. 创建xml布局资源文件，根布局没有要求
    2. 创建class继承自`Fragment`或Fragment的子类（如`PreferenceFragmentCompat`）
    3. 在class中重写`onCreateView`方法，使用形参inflater（数据类型是LayoutInflater）的inflate方法关联xml布局资源文件
2. 使用碎片：
    1. 创建需要使用Fragment的xml布局，通过给`<fragment>`标签中的`android:name`属性赋值：完整的包名+class的路径，引入碎片
    2. 创建class继承自`AppCompatActivity`，重写`onCreate`方法，调用`setContentView`方法，关联xml布局资源文件

#### 活动与碎片的通信
1. 活动中与碎片通信：

`(xxxFragment)getSupportFragmentManager().findFragmentById(int xmlResId);`
2. 碎片中与活动通信：

`(xxxActivity)getActivity();`

3. 碎片中与碎片通信
 
`(xxxFragment)getFragmentManager().findFragmentById(int xmlResId);`

#### 动态添加碎片

```
getSupportFragmentManager().beginTransaction().replace(int layoutResId, Fragment fragment).commit();
```
##### 注：由于Fragment并没有类似活动的系统提供的任务栈（即，当按下返回键时，并不会返回上一个Fragment，而是返回上一个活动或退出APP） ，所以若想实现类似效果，需要调用FragmentTransaction的addToBackStack(String state);方法，举例如下：

```
getSupportFragmentManager().beginTransaction().replace(int layoutResId, Fragment fragment).addToBackStack(null).commit();
```

#### 碎片的生命周期
- 碎片的状态和回调
    1. 状态：
        1. 运行状态：活动可见且可交互时（`onResume()`），碎片处于运行状态
        2. 暂停状态：活动可见但不可交互时（`onPause()`），碎片处于暂停状态
        3. 停止状态：活动不可见（`onStop()`），或碎片被`replace()`或`remove()`时有`addToBackStack()`，碎片处于停止状态
        4. 销毁状态：活动被销毁（`onDestroy()`），或碎片被`replace()`或`remove()`时没有`addToBackStack()`，碎片处于销毁状态
    2. 回调
        1. `onAttach()`：当碎片与活动关联时调用
        2. `onCreateView()`：当碎片与xml布局资源关联时的时候调用
        3. `onActivityCreated()`：与碎片相关联的活动创建完成时调用
        4. `onDestroyView()`：与碎片关联的xml布局资源文解除关联时调用
        5. `onDetach()`：当碎片与活动解除关联时调用
- 碎片的生命周期
1. `onAttach()`
2. `onCreate()`
3. `onCreateView()`
4. `onStart()`
5. `onResume()`
6. `onPause`
7. `onStop`
8. `onDestroyView()`
    1. 当碎片有`addToBackStack()`时，碎片从返回栈回到`onCreateView()`
    2. 当碎片没有`addToBackStack()`时，执行9和10
9. `onDestroy()`
10. `onDetach()`

#### 使用Bundle保存Fragment的临时数据和状态
- 保存（store）：重写`onSaveInstanceState()`
- 取出（restore）：重写`onCreate()`、`onCreateView()`、`onActivityCreated()`三个方法任一一个均可看情况选择

#### 动态加载布局（自适应）
- 使用资源限定符（qualifier），常用的包括：
1. `-port`：竖屏
2. `-land`：横屏
3. `-swxxxdp`：最小屏幕宽度，一般600dp，720dp等
4. `-xx`：各国语言的字符串
