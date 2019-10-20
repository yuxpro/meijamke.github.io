---
layout: post
title: "《第一行代码》Chapter 03"
subtitle: 'UI开发'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

##### 注：这里使用视图来代替原文的控件这一词
## 正文
#### gravity与layout_gravity属性
- `gravity`：视图内容相对于视图的位置关系
- `layout_gravity`：视图相对于布局的位置关系

#### inputType和maxLines属性
- `inputType`：EditText中，决定了输入时弹出的键盘类型
- `maxLines`：决定了显示文本时最大的行数（EditText中，超出自动滚动；TextView中，超出以...显示）

#### src和scaleType
- `src`：指定要显示的图片
- `scaleType`：指定如何裁剪图片
 
#### setVisibility
- `VISIBLE`：可见
- `INVISIBLE`：不可见，但是存在在屏幕上
- `GONE`：不可见，也不存在在屏幕上

#### 按钮点击事件监听注册的3种方法
1. 获取按钮实例，采用匿名类注册监听器
```java
button.setOnClickListenner(new View.OnClickListenner(){
    @Override
    public void onClick(View v){
        //do something
    }
});
```
2. 获取按钮实例，实现接口（`implements View.OnClickListenner`）注册监听器
```
button.setOnClickListenner(Context);
@Override
public void onClick(View v){
    switch(v.getId){
        case R.id.xxx:
            //do something
            break;
        default:
            break;
    }
}
```
3. 在布局xml文件中添加按钮的android:onClick="xxx"属性，根布局要加上`tools:context=".XXXActivity"`和`xmlns:tools="http://schemas.android.com/tools"`
，然后只需要在XXXActivity中，添加以下代码即可
```
public void xxx(View view) {
    }
```

#### AlertDialog
AlertDialog的类型是AlertDialog.Builder（可能AlertDialog类型也可以，还没有试过）

#### FrameLayout（帧布局）
- 视图默认放在布局的左上角，若不控制位置，会发生遮挡

#### 百分比布局（感觉很少用？）
- PercentFrameLayout：增加了`layout_widthPercent`和`layout_heightPercent`
- PercentRelativeLayout：增加了`layout_widthPercent`和`layout_heightPercent`

#### 引入布局和自定义视图
1. 引入布局
- 好处：一次编写，多处使用，方便维护
2. 自定义视图
- 新建布局xxx.xml
- 新建class继承自XXXLayout，在构造函数通过`LayoutInflater.from(Context).inflate(R.layout.xxx, ParentLayout this);`加载布局，同时可以在这里处理视图事件

#### 添加依赖-RecyclerView
- `com.android.support:recyclerview-v7:28.0.0`

---
### **补充：**
#### 布局黄金规则
单个布局
- 不要包含超过**80**个视图
- 嵌套视图组（布局）不要超过**10**个
#### ConstraintLayout（约束布局）
- 相比于相对布局、百分比布局来说，更适合用于**复杂的布局**。
- 不需要过分依赖父布局和其他视图，而且在相同的比较复杂的布局下，其**布局树的深度**比相对布局低，加载渲染快，效率高。

#### [制作9-patch图片](https://romannurik.github.io/AndroidAssetStudio/nine-patches.html#&sourceDensity=320&name=example)

#### [制作镜像图片](http://ps.xunjiepdf.com/flip)
