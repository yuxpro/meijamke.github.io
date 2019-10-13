---
layout: post
title: "《第一行代码》Chapter 1"
subtitle: '开始启程'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 前言

博客第一篇文章，本质上算是摘录型笔记（其实就是摘录_(:3」∠)_）

之前在Udacity上学了Android基础课程，于是趁热捧起《第一行代码》来~~一发~~查漏补缺，而且作为初学者必看的经典之作，看一遍怎么够（一遍还没看的某j）。

## 正文

第一章介绍了Android的框架以及认识开发工具（Android Studio）和各种文件

---
#### Android系统架构分为4层（从下往上）：
- Linux内核层：为Android设备的各种硬件提供**驱动**，如：显示驱动、音频驱动、照相机驱动、蓝牙驱动、wifi驱动、电源管理等
- 系统运行库层：包括**C/C++库**、**Android运行时库**（提供核心库）。其中Android运行时库包含了Dalvik虚拟机（Android5.0之后改为ART运行环境），它使得每一个Android APP都能运行在独立的进程中，并且拥有一个自己的Dalvik虚拟机实例。相比于java虚拟机，Dalvic虚拟机是专门为移动设备定制的，针对手机内存、CPU性能等进行了优化。

- 应用框架层：提供构建APP时可能用到的各种**API**。

- 应用层：包含了各类手机**APP**。

---
#### Android已发布的版本
最新数据访问[这里](https://developer.android.google.cn/about/dashboards?hl=en)
- Android3.0，专门为平板电脑设计
- Android4.0，兼容手机和平板
- Android5.0，用ART运行环境替代Dalvik虚拟机，推出material design概念来优化UI，推出Android Auto、Android Wear、Android TV
- Android6.0，加入运行时权限功能
- Android7.0，加入多窗口模式功能
---
#### Android4大组件
- 活动：能够看到的都放在活动中
- 服务：后台运行，不需要提供用户交互界面，活动可以启动服务，而服务会继续运行，即使活动被关闭也是如此
- 广播接收器：接收系统和其他APP的广播意图（Intent）。分为静态广播和动态广播，静态广播不绑定APP的生命周期（即APP退出后依旧可以运行），动态广播绑定APP的生命周期。
- 内容提供器：实现不同APP间共享数据。

---
#### 项目目录结构
文件（夹） | 讲解
---|---
.gradle和.idea | Android Studio自动生成的文件，无需关心
app | 项目的代码和资源等内容基本在这个文件夹里
build | 编译时自动生成的文件
gradle | 包含了gradle-wrapper配置文件，会自动根据本地缓存的情况决定是否联网下载gradle，在setting里可以指定本地缓存gradle的位置，也可以选择联网下载gradle
.gitignore | 将指定的目录和文件排除在版本控制之外
build.gradle | 项目全局的gradle构建脚本，一般不需要修改
gradle.properties | 项目全局的gradle配置文件
gradlew和gradlew.bat | 用于在命令行中执行gradle命令，其中gradlew是在Linux或Mac系统使用，gradlew.bat是在Windows系统使用
.iml | IntelliJ IDEA项目自动生成的文件（Android Studio是基于IntelliJ IDEA开发的），用于标识这是一个IntelliJ IDEA项目
local.properties | 用于指定Android SDK路径，一般自动生成，若Android SDK路径改变了，在这个文件修改即可
setting.gradle | 用于指定项目引入的所有模块，一般模块的引入都是自动完成的

---
#### app目录结构
文件（夹） | 讲解
---|---
build | 和外层build目录类似，但是更多更杂，包含了编译时自动生成的文件
libs | 项目中使用到的第三方jar包都放在该目录下，该目录下的jar包会自动添加到构建路径里
androidTest | 用于编写Android Test测试用例，对项目进行自动化测试
java | 放置java代码的目录
res | 项目中使用到的所有图片、字符串、布局等资源都要放在该目录下，此目录下有很多子目录，如图片放在drawable目录下，字符串放在values目录下，布局放在layouts目录下，应用图标（icon）放在mipmap目录下
AndroidManifest.xml | 项目的配置文件，四大组件（activity、service、receiver、provider）都需要在这个文件中注册，而且权限声明也是在这个文件中声明
test | 用于编写Unit Test测试用例，对项目进行自动化测试
.gitignore | 将指定的目录和文件排除在版本控制之外
app.iml | IntelliJ IDEA项目自动生成的文件
build.gradle | app模块的gradle构建脚本
++proguard-rules.pro++ | 用于指定项目代码的混淆规则，当代码开发完成打成安装包文件，如果不希望代码被破解，就可以在该文件配置从而混淆代码

---
#### 其他
- 项目中所有的活动都必须继承自Activity或Activity的子类，一般默认继承自`AppCompatActivity`（AppCompatActivity是Activity的子类），因为AppCompatActivity可以将Activity在各个版本中的功能和特性向下兼容到**Android2.1（API 7）**
- Android程序的设计强调逻辑和视图分离，因此不推荐在活动中编写视图界面，推荐在布局文件中编写视图界面，然后在活动中引入进来
- 资源限定符（qualifier），即连字符-，针对**自适应布局**使用的，如不同分辨率（mdpi、hdpi、xhdpi、xxhdpi、xxxhdpi）下的图片，不同最小屏幕宽度（sw多少dp）下的字符串、颜色、样式、维度的配置
- 使用资源，代码中：`R.资源所在文件夹.资源id`，xml中：`@资源所在文件夹/资源id`
- 在AndroidManifest文件中，application标签中，`android:icon`属性指定应用图标，`android:label`属性指定应用名称
- Android Studio使用**Gradle**构建项目。Gradle是一种项目构建工具，使用了基于Groovy的领域特定语言（DSL）来声明项目配置。

#### build.gradle文件
---
最外层目录下的build.gradle文件，代码如下：
```
buildscript {
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.4.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    String osName = System.getProperty("os.name").toLowerCase();
    if (osName.contains("windows")) {
        buildDir = "C:/tmp/${rootProject.name}/${project.name}"
    }
    repositories {
        google()
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}
```

#### 说明
- **repositories**声明的**google()**和**jcenter()**，它们都是代码托管仓库。声明了这行配置之后，就可以在项目中轻松引用google()和jcenter()上的开源项目了。
- **dependencies**使用classpath声明了一个Gradle插件，由于Gradle并不是专门用于构建Android项目的，所以需要这个插件来构建Android项目。
- **buildDir**指定了构建的Android项目存放的位置，此处指定了当操作系统是Windows时项目存放的位置。

---
app目录下的build.gradle文件，代码如下：
```
apply plugin: 'com.android.application'

android {
    compileSdkVersion 28

    defaultConfig {
        applicationId "com.example.android.sunshine"
        minSdkVersion 14
        targetSdkVersion 28
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
			proguardFiles getDefaultProguardFile('proguard-android.txt'),'proguard-rules.pro'
        }
    }
}

dependencies {
    implementation fileTree(include: ['*.jar'], dir: 'libs')
    implementation 'com.android.support:appcompat-v7:28.0.0'
    implementation 'com.android.support:recyclerview-v7:28.0.0'
    implementation 'com.firebase:firebase-jobdispatcher:0.8.5'

    // Instrumentation dependencies use androidTestImplementation
    // (as opposed to testImplementation for local unit tests run in the JVM)
    androidTestImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support:support-annotations:28.0.0'
}
```
#### 说明
- **apply plugin**，应用插件，一般有两种值可选：`com.android.application`表示这是一个应用程序模块，`com.android.library`表示这是一个库模块。应用程序模块可以直接运行，库模块需要依附应用程序模块运行。
- **compileSdkVersion**，指定项目的编译版本
- **applicationId**，指定项目包名，可以在此修改项目包名
- **minSdkVersion**，项目兼容的最低Android系统版本
- **targetSdkVersion**，项目已经做过充分的测试的Android系统版本
- **versionCode**，项目版本号
- **versionName**，项目版本名
- **buildTypes**闭包，一般有两个子闭包，`release`和`debug`，release用于配置正式版安装文件，debug用于配置测试版安装文件，其中debug可以忽略不写。`minifyEnabled`用于指定是否混淆项目代码。`proguardFiles`用于指定使用的混淆规则文件，第一个`proguard-android.txt`是在Android SDK目录下的，是所有项目通用的混淆规则，第二个`proguard-rules.pro`是在当前项目目录下的，可以编写当前项目使用的混淆规则
- **dependencies**闭包，用于指定当前项目的所有依赖关系，一共有3种依赖关系：本地依赖、库依赖、远程依赖。本地依赖是对本地的jar包或目录添加依赖关系，库依赖是对项目中的库模块添加依赖关系，远程依赖是对google()和jcenter()上的开源项目添加依赖关系。
    1. 本地依赖`implementation fileTree(include: ['*.jar'], dir: 'libs')`表示将libs目录下的所有jar包都添加到项目的构建路径中。
    2. 库依赖`implementation project(':helpers')`表示将helpers库模块添加到项目的构建路径中。
    3. 远程依赖`implementation 'com.android.support:appcompat-v7:28.0.0`表示将appcompat库添加到项目的构建路径中，其中`com.android.support`是域名，用于与其他公司的库区分开；`appcompat-v7`是组名称，用于与同一个不同库区分开；`28.0.0`是版本号，用于与同一个库不同版本区分开。
	
---
#### 日志
- 输入logt，按下tab键，会以当前的类名作为值自动补全成一个TAG：
```
    private static final String TAG = "DetailActivity";
```	
- 输入logv、logd、logi、logw、loge，按下tab键，自动补全一条完整的打印语句：
```
    Log.v(TAG, "onCreate: ");
    Log.d(TAG, "onCreate: ");
    Log.i(TAG, "onCreate: ");
    Log.w(TAG, "onCreate: ");
    Log.e(TAG, "onCreate: ");
```
