---
layout: post
title: "《第一行代码》Chapter 07"
subtitle: '内容提供器'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 目录

1.[内容提供器](#内容提供器)

2.[运行时权限](#运行时权限)
- [危险权限](#危险权限)
- [运行时权限实践的建议](#运行时权限实践的建议)

3.[访问自己和其他APP的内容提供器](#访问自己和其他app的内容提供器)
- [借助ContentResolver访问内容提供器](#借助contentresolver访问内容提供器)
- [ContentResolver的insert、delet、update、query](#contentresolver的insert-delet-update-query)

4.[创建内容提供器](#创建内容提供器)
- [重写的6个方法具体如下](#重写的6个方法具体如下)

5.[APP通过内容提供器与数据库的整个交互流程如下](#app通过内容提供器与数据库的整个交互流程如下)

6.[git学习](#git学习)


## 正文

---
#### 内容提供器
- 应用场景：可控制地与其他APP共享数据

---
#### 运行时权限
- Android6.0（API23）之前，APP申请的权限是在安装APP时授予的。

> 用户安装APP时可以看到所有APP需要申请的权限，且在安装后可以在设置->权限管理界面看到APP申请的权限。

> 缺点：用户不授予APP所需的所有权限，则无法安装和使用APP

- Android6.0（API23）开始，Android将权限分为普通权限和危险权限（还有第三种特殊权限，有需要可以查找文档）。

> APP申请的普通权限不需要用户确认，但APP在运行时申请的危险权限会弹窗提醒用户是否授权

> 同样，可以在设置->权限管理界面查看APP权限，并且可以撤销授权（？待进一步确认）。

---
##### 危险权限

> 概念：可能威胁到用户隐私和设备安全的权限

> 共24个危险权限分为9组，每一组中的任一一个权限被授权，整组的权限相应地也被授权。危险权限组可能发生改变，最新的查看[这里](https://developer.android.google.cn/guide/topics/security/permissions.html?hl=zh_cn#perm-groups)

> 在Manifest文件中申请权限，无论普通权限还是危险权限都需要申请，申请权限举例如下：

```
<manifest>
    <!--其中XXX是申请的权限名-->
    <uses-permission android:name="XXX"/>
</manifest>
```

- 在运行时申请权限的实践
```
...
//检查权限是否被授权，XXX是申请的权限名
if(ContextCompat.checkSelfPermission(Activity thisActivity, Manifest.permission.XXX != PackageManager.PERMISSION_GRANTED){
    //若用户第一次拒绝，第二次申请权限时可以在申请前解释说明为什么申请权限
    if(ActivityCompat.shouldShowRequestPermissionRationale(Activity thisActivity, Manifest.permission.XXX)){
        //可以弹出一个对话框
        //然后申请权限
        ActivityCompat.requestPermissions(
        Activity thisActivity, 
        new String[]{Manifest.permission.XXX}, 
        int REQUEST_CODE);
    }
    //若用户没有拒绝过，是第一次请求授权，则申请权限
    else{
        ActivityCompat.requestPermissions(
        ACtivity thisActivity,
        new String[]{Manifest.permission.XXX},
        int REQUEST_CODE);
    }
}
...
//重写回调函数onPermissionsRequestResult，根据得到的申请权限结果处理事务
@Override
public void onRequestPermissionsResult(int requestCode,
        String[] permissions, int[] grantResults) {
    //XXX和YYY是申请的权限名
    switch (requestCode) {
        case XXX: {
            // If request is cancelled, the result arrays are empty.
            if (grantResults.length > 0
                && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
                //权限允许后执行的事务
            } else {
                //权限拒绝后执行的事务
            }
            return;
        }
        case YYY:{
            if(grantResults.length > 0 &&
            grantResults[1]==PackageManager.PERMISSION_GRANTED){
                //权限允许后执行的事务
            }
            else{
                //权限拒绝后执行的事务
            }
            return;
        }
```

---
##### 运行时权限实践的建议
- 看[这里](https://developer.android.google.cn/training/permissions/usage-notes.html?hl=zh_cn)

---
#### 访问自己和其他APP的内容提供器

---
##### 借助ContentResolver访问内容提供器
- ContentResolver借助Uri来定位资源的位置，不同于SQLite使用表格名定位数据的位置，ContentResolver的Uri由3部分组成：
> schema+authority+path

> 其中ContentResolver的schema是content://，authority一般是包名，用于和其他APP区分开，path一般是数据库的表格名。

---
##### ContentResolver的insert、delet、update、query
- insert，举例如下：

```
ContentValues values=new ContentValues();
values.put(String column1, int data1);
values.put(String column1, float data2);
values.put(String column1, String data3);
values.put(String column1, boolean data4);
...
Uri uri=Uri.parse("content://com.example.myapp/table1");
getContentResolver().insert(uri, values);
```

- delet，举例如下：

```
getContentResolver().delet(Uri uri, String selection, String[] selectionArgs);
```

- update，举例如下：

```
ContentValues values=new ContentValues();
values.put(String column1, int data);
Uri uri=Uri.parse("content://com.jamke.myapp/table2");
getContentResolver().update(uri, values, String selection, String[] selectionArgs);
```

- query，举例如下：

```
Cursor cursor=getContentResolver().query(
Uri uri,
String[]{"列名1"[,"列名2",...]}, 
String selection,//即"列名1 = ?"或者"列名1 = ? and 列名2 > ?"
String[] selectionArgs,//selection中的?相当于占位符，这里指定的是selection中的?的内容
String setOrder);
if(cursor!=null){
    while(cursor.moveToNext()){
        //取出数据
    }
    cursor.close();
}
```

---
#### 创建内容提供器
- 创建内容提供器之前，首先需要创建一个数据库，这里使用SQLiteDatabase
- 创建内容提供器：

> 1. 新建class继承自ContentProvider，重写6个方法：onCreate()、insert()、delet()、update()、query()、getType()。

> 2. 在Manifest文件中注册provider（4大组件都需要注册）

```
<provider name="[完整包名]+类路径"
    authority="com.jamke.myapp"
    enabled="true"
    exported="true">
</provider>
```

---
##### 重写的6个方法具体如下
- onCreate()

> 初始化内容提供器，当ContentResolver调用insert、delet、update时会执行这个函数初始化内容提供器。一般在该方法内创建和初始化数据库。

```
@Override
public boolean onCreate(){
    MySQLiteOpenHelper mDbHelper=new MySQLiteOpenHelper(Context);//MySQLiteOpenHelper事先创建好的继承自SQLiteOpenHelper的类
    return true;//true告诉ContentResolver数据库创建成功
}
```

> 在重写下列方法之前，应该创建一个==匹配Uri==的函数，原因是，一个表格中有很多行，我们不可能对每一个行都做一个if/switch-case判断，所以创建的这个函数，可以将单个表格映射到一个整数（一般采用100，200等整数），表格的所有行映射到另一个整数（一般采用101，102，201等整数）。举例如下：

```
publi static final int CODE_TABLE1=100;
publi static final int CODE_TABLE1_COLUMN=101;

//创建匹配Uri的函数，方式1
private static UriMatcher mUriMatcher;
public static UriMatcher buildUriMatcher(){
    UriMatcher uriMatcher=new UriMatcher(UriMatcher.NO_MATCH);
    mUriMatcher.addURI(String authority, String path, int CODE_TABLE1);
    mUriMatcher.addURI(String authority, String path+"/#", int CODE_TABLE1_COLUMN);
    return uriMatcher;
}
mUriMatcher=buildUriMatcher();

//创建匹配Uri的函数，方式2
private static UriMatcher mUriMatcher;
static{
    mUriMatcher=new UriMatcher(UriMatcher.NO_MATCH);
    mUriMatcher.addURI(String authority, String path, int CODE_TABLE1);
    mUriMatcher.addURI(String authority, String path+"/#", int CODE_TABLE1_COLUMN);
    //mUriMatcher.addURI(String authority, String path, int CODE_TABLE2);
    //mUriMatcher.addURI(String authority, String path+"/#", int CODE_TABLE2_COLUMN); 
}
```

> 注意：SQLiteDatabase支持两个通配符：#（表示任意整数）和*（表示任意字符串）

> 注意：DATE是列名，假设DATE设为主键，即key primary，且若DATE数据类型是整数，那么具体的path参数应该是path+"/#"；若DATE数据类型是字符串，那么具体的path参数应该是path+"/*"。

> 创建好之后，调用UriMatcher的match(Uri uri)方法，若Uri存在，其返回对应的整数（如：CODE_TABLE1）。

- insert()：举例如下

```
@Override
public Uri insert(Uri uri, ContentValues value){
    //mDbHelper在onCreate方法中已经创建
    SQLiteDatabase db = mDbHelper.getWriteableDatabase();
    Uri uriInserted=null;
    switch(mUriMatcher.match(uri)){
        //由于插入数据时，一般直接插入表格中，所以不考虑插入已有的行，当然也可以实现插入已有行，相当于更新
        case CODE_TABLE1:
            long id = db.insert(
            TABLE_NAME,//本质上是uri的path，可以通过uri.getPathSegment().get(0)获得
            null,//当某些列没有指定值时，指定插入null
            value);
            uriInserted = uri.buildUpon().appendPath(String.valueOf(id)).build();
            break;
        default:
            throw new IllegalArgumentException("Unknown uri");
    }
    //数据变动时，通知ContentResolver
    if(uriInserted!=null)
        getContext().getContentResolver().notifyChange(uri, null);
    return uriInserted;
}
```

- delet()：举例如下

```
@Override
public int delet(Uri uri, String selection, String[] selectionArgs){
    //mDbHelper在onCreate方法中已经创建
    SQLiteDatabase db = mDbHelper.getWriteableDatabase();
    int rowsDeleted=0;
    switch(mUriMatcher.match(uri)){
        case CODE_TABLE1:
            rowsDeleted = db.delet(
            TABLE_NAME,//本质上是uri的path，可以通过uri.getPathSegment().get(0)获得
            selection,
            selectionArgs);
            break;
        case CODE_TABLE1_COLUMN:
            rowsDeleted = db.delet(
            TABLE_NAME,//本质上是uri的path，可以通过uri.getPathSegment().get(0)获得
            "DATE = ?",//DATE是列名，假设DATE设为主键，即key primary
            new String[]{uri.getPathSegment().get(1))};//Uri的getPathSegment()方法会将path以"/"分割path
            break;
        default:
            throw new IllegalArgumentException("Unknown uri");
    }
    //数据变动时，通知ContentResolver
    if(rowsDeleted>0)
        getContext().getContenResolver().notifyChange(uri, null);
    return rowsDeleted;
}
```

- update()，举例如下

```
@Override
public int update(Uri uri, ContentValues value, String selection, String[] selectionArgs){
    //mDbHelper在onCreate方法中已经创建
    SQLiteDatabase db = mDbHelper.getWriteableDatabase();
    int rowsUpdated=0;
    switch(mUriMatcher.match(uri)){
        case CODE_TABLE1:
            rowsUpdated = db.update(
            TABLE_NAME,//本质上是uri的path，可以通过uri.getPathSegment().get(0)获得
            value,
            selection,
            selectionArgs);
            break
        case CODE_TABLE1_COLUMN:
            rowsUpdated = db.update(
            TABLE_NAME,//本质上是uri的path，可以通过uri.getPathSegment().get(0)获得
            value,
            "DATE = ?",//DATE是列名，假设DATE设为主键，即key primary
            new String[]{uri.getPathSegment().get(1))};//Uri的getPathSegment()方法会将path以"/"分割path
            break;
        default:
            throw new IllegalArgumentException("Unknown uri");
    }
    //数据变动时，通知ContentResolver
    if(rowsUpdated>0)
        getContext().getContenResolver().notifyChange(uri, null);
    return rowsUpdated;
}
```

- query()，举例如下

```
@Override
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String setOrder){
    //mDbHelper在onCreate方法中已经创建
    SQLiteDatabase db = mDbHelper.getReadableDatabase();
    Cursor cursor=null;
    switch(mUriMatcher.match(uri)){
        case CODE_TABLE1:
            cursor = db.query(
            TABLE_NAME,//本质上是uri的path，可以通过uri.getPathSegment().get(0)获得
            projection,
            selection,
            selectionArgs,
            setOrder);
            break
        case CODE_TABLE1_COLUMN:
            cursor = db.query(
            TABLE_NAME,//本质上是uri的path，可以通过uri.getPathSegment().get(0)获得
            projection,
            "DATE = ?",//DATE是列名，假设DATE设为主键，即key primary
            new String[]{uri.getPathSegment().get(1)},//Uri的getPathSegment()方法会将path以"/"分割path
            setOrder);
            break;
        default:
            throw new IllegalArgumentException("Unknown uri");
    }
    //数据变动时，通知ContentResolver
    if(cursor!=null)
        cursor.setNotificationUri(getContext().getContentResolcer(), uri);
    return cursor;
}
```

- getType()，举例如下

```
@Override
public String getType(Uri uri){
    String type=null;
    switch(mUriMatcher.match(uri)){
        case CODE_TABLE1:
            type="vnd.android.cursor.dir/"+"vnd."+AUTHORITY+"."PATH;
            break;
        case CODE_TABLE1_COLUMN:
            type="vnd.android.cursor.item/"+"vnd."+AUTHORITY+"."PATH;
            break;
        default:
            break;
    }
    return type;
}
```

---
> 事实上，还可以重写bulkInsert(Uri, ContentValues[])方法，实现批量插入数据

---
#### APP通过内容提供器与数据库的整个交互流程如下

```
sequenceDiagram
APP->>ContentResolver: insert、delet、update、query
ContentResolver->>ContentProvider: insert、delet、update、query
ContentProvider->>SQLiteDatabase:  insert、delet、update、query
SQLiteDatabase->>ContentProvider: results
ContentProvider->>ContentResolver: results
ContentResolver->>APP: results
```

---
> 实际应用中，我们只需要与ContentResolver交互即可，若需要其他APP共享该APP数据，则需要创建ContentProvider并实现6个方法

---
#### git学习
- 忽略文件

> .gitignore文件内编辑

- 查看文件修改内容

> git status //查看文件修改内容

> git diff //查看文件与修改前的不同

> git diff xxx //xxx是文件所在的具体路径，用于查看指定的文件与修改前的不同

- 撤销修改后未commit的修改

> git checkout xxx //xxx是文件所在的具体路径

- 撤销修改后commit的修改

> git reset HEAD xxx //xxx是文件所在的具体路径

> git checkout xxx //xxx是文件所在的具体路径

- 查看提交记录

> git log

> git log xxx //xxx是文件所在的具体路径
