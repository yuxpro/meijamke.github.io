---
layout: post
title: "《第一行代码》Chapter 06"
subtitle: '持久化技术'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 目录

1.[文件存储](#文件存储)
    
- [存储数据到文件](#存储数据到文件)
- [从文件读取数据](#从文件读取数据)

2.[SharedPreferences存储](#sharedpreferences存储)
- [数据存储到SharedPreferences](#数据存储到sharedpreferences)
- [从SharedPreferences读取数据](#从sharedpreferences读取数据)

3.[SQLite数据库存储](#sqlite数据库存储)
- [创建和更新SQLite](#创建和更新sqlite)
- [SQLite的insert、delet、update、query](#sqlite的insert-delet-update-query)
 
4.[使用LitePal操作数据库](#使用litepal操作数据库)
- [配置LitePal](#配置litepal)
- [LitePal的insert、delet、update、query](#litepal的insert-delet-update-query)

## 正文

---
#### 文件存储

---
##### 存储数据到文件
- java流：
> `openFileOutput(String fileName, int MODE_PRIVATE)`->`OutputStreamWriter()`->`BufferedWriter()`

```
BufferedWriter writer=new BufferedWriter(new OutputStreamWriter(openFileOutput(String fileName, int MODE_PRIVATE)));
writer.write(String data);
writer.close();
```

---
##### 从文件读取数据
-java流：
> `openFileInput(String fileName)`->`InputStreamReader()`->`BufferedReader()`

```
BufferedReader reader=new BufferedReader(new InputStreamReader(openFileInput(String fileName)));
StringBuilder data = new StringBuilder();
String line="";
while(line=reader.readLine()!=null)
    data.append(line);
```

---
#### SharedPreferences存储

---
##### 数据存储到SharedPreferences
- 获取SharedPreferences有3个方法：

> 1. `Context.getSharedPreferences()`
> 2. `Activity.getPreferences()`
> 3. `PreferenceManager.getDefaultSharedPreferences()`

- 获取到之后调用editor()方法存储数据

```
SharedPreferences sharedPreferences=PreferenceManager.getDefaultSharedPreferences();
SharedPreferences.Editor editor=sharedPreferences.editor();
editor.putxxx(String key, xxx value)//xxx是数据类型
      .apply();
```

---
##### 从SharedPreferences读取数据
- 依旧是先获取SharedPreferences，然后调用getxxx()方法

```
SharedPreferences sharedPreferences=PreferenceManager.getDefaultSharedPreferences();
xxx data=sharedPreferences.getxxx(String key, xxx defaultValue);
```

---
#### SQLite数据库存储

---
##### 创建和更新SQLite

- 建立SQLite数据库：

> 1. 新建class，用于存储数据库的所有表格，每一个表格作为一个内部类，该内部类实现`BaseColumns`接口。虽然实现`BaseColumns`接口不是必须的，但是`BaseColumns`接口包含了两个常量：_ID（每一行的ID号）、_COUNT（表格拥有的总行数），而Android的一些Adapter（适配器）会要求表格拥有ID列。然后在内部类中，创建列名的常量和可能用到的Uri（用于内容提供器访问）

> 2. 新建class继承自`SQLiteOpenHelper`，构造函数需要传入4个参数，Context、String databaseName、SQLiteDatabase.CursorFactor factory、int databaseVersion，其中databaseName和databaseVersion可以在类内创建常量事先确定，factory允许返回一个自定义的Cursor一般为null即可，接着重写`onCreate()`和`onUpgrade()`两个方法，`onCreate()`方法创建数据库，`onUpgrade()`更新数据库。

> 3. 当调用新建class（继承自SQLiteOpenHelper）实例的`getWriteableDatabase()`或`getReadableDatabase()`方法时，会调用`onCreate()`方法创建数据库；当增大数据库的版本号（即databaseVersion），会调用`onUpgrade()`方法更新数据库。

- 创建数据库，举例如下：

```
public static final String CREATE_TABLE="create table 表格名 (列名 integer not null, 列名 real not null,列名 text not null, 列名 blob not null);";//integer指整型，real指浮点型，text指字符串类型，blob指二进制类型
SQLiteDatabase.execSQL(CREATE_TABLE);
```

- 更新数据库，举例如下：
```
SQLiteDatabase.execSQL("drop table if exists 表格名");
onCreate(Context);//先删除，后创建新的，缺点是无法保存旧的数据
```

---
##### SQLite的insert、delet、update、query
- insert：存储数据到数据库，举例如下：

```
SQLiteDatabase db=new 继承自SQLiteOpenHelper的类(Context).getWriteableDatabase();
db.insert(String tableName, null, ContentValue);//第二个参数用于指定给未赋值的列自动赋值
```
- delet：删除数据库的数据，举例如下：

```
SQLiteDatabase db=new 继承自SQLiteOpenHelper的类(Context).getWriteableDatabase();
db.delet(String tableName, String selection, String[] selectionArgs);
```
- update：更新数据库的数据，举例如下：

```
SQLiteDatabase db=new 继承自SQLiteOpenHelper的类(Context).getWriteableDatabase();
db.update(String tableName, ContentValue, String selection, String[] selectionArgs);
```
- query：查询数据库的数据，举例如下：

```
SQLiteDatabase db=new 继承自SQLiteOpenHelper的类(Context).getWriteableDatabase();
Cursor cursor=db.query(String tableName, String columnNamw, String selection, String selectionArgs, String groupBy, String having, String orderBy);
StringBuilder data=new StringBuilder();
if(cursor.moveToFirst())
    while(cursor.moveToNext()){
        xxx data1=cursor.getxxx(cursor.getColumnIndex(String columnName));
        data.append(data1);
    }
cursor.close();//不关闭会导致内存溢出（OOM）
```

---
#### 使用LitePal操作数据库

- [LitePal](https://github.com/LitePalFramework/LitePal)介绍：

> 一款开源的Android数据库框架，采用了对象关系映射（ORM）的模式，将常用的数据库功能进行了封装。

- 对象关系映射：

> 在面向对象的语言yu关系型数据库之间建立一个映射，这样就可以使用面向对象的思维而不需要使用SQL语句操作数据库

---
##### 配置LitePal
[详细步骤](https://github.com/LitePalFramework/LitePal)

---
##### 创建和升级数据库

> 1. 创建类继承自`DataSupport`，类名即表格名，类内属性即表格的列名，同时创建setter和getter方法，使用快捷键**Alt+insert**

> 2. 在litepal.xml文件的<list>标签内添加`<mapping class="完整包名+刚刚创建的类路径"/>`

> 3. 调用`LitePal.getDatabase()`方法，即可创建数据库

> 4. 更新数据库，只需要修改类内属性或增加新的类到litepal.xml文件里，再次调用`LitePal.getDatabase()`即可

---
##### LitePal的insert、delet、update、query
- insert：

> 新建创建的类的实例，调用类的`setxxx()`方法，之后调用`save()`方法

- delet：

```
DataSupport.deletAll(String 类名.class, String selection, String selectionArg1[, String selectionArg2...])
```

- update（两种方式）：

> 1. 新建创建的类的实例，调用类的set()方法，之后调用`save()`方法

> 2. 新建创建的类的实例，调用类的set方法，之后调用`updateAll(String selection, String selectionArg1[, String selectionArg2...]);`

- query：

```
list<类名> data=DataSupport.findAll(类名.class);
list<类名> data=DataSupport.findFirst(类名.class);
list<类名> data=DataSupport.findLast(类名.class);

list<类名> data=
DataSupport.select(String column1[, String column2,...])
.where(String selection,String  selectionArg1[, String selectionArg2...])
.order(String column)
.limit(int num)//限定查找的数量
.offset(int num)//查询结果的偏移量
.find(类名.class);
```
