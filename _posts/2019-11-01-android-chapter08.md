---
layout: post
title: "《第一行代码》Chapter 08"
subtitle: '多媒体'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 目录

1.[通知](#通知)
- [关于通知的最新内容](#关于通知的最新内容)

2.[调用摄像头拍照](#调用摄像头拍照)
- [更多关于摄像头拍照](#更多关于摄像头拍照)

3.[从相册选择照片](#从相册选择照片)

4.[播放音频](#播放音频)
- [关于如何具体地构建一个audio app](#关于如何具体地构建一个audio-app)

5.[播放视频](#播放视频)
- [关于如何具体地构建一个video app](#关于如何具体地构建一个video-app)


## 正文

---
#### 通知

- 创建并显示通知

```
//创建通知
NotificationCompat.Builder builder=new NotificationCompat.Builder(Context,String channelId)//Android8.0(API26)及以上需要channelId，以下的不需要
.setSmallIcon(int resId)//通知显示在标题栏的图标
.setLargeIcon(Bitmap bitmap)//通知显示在下拉栏的图标
.setContentTitle("")//通知的标题
.setContentText("")//通知的内容
.setStyle(new NotificationCompat.BIgTextStyle().bigText(""))//显示通知内容的大量文本
.setStyle(new NotificationCompat.BigPictureStyle().bigPicture(Bitmap))//显示大图标
.setPriority(int PRIORITY)//通知的优先级
.setAutoCancel(true)//当用户点击通知后，通知自动消失
.setContentIntent(PendingIntent);//点击通知后，跳转到PendingIntent所指向的活动、服务或广播

//显示通知
NotificationManagerCompat notificationManager=NotificationManagerCompat.from(Context);//NotificationManagerCompat是androidx的库
notificationManager.notify(int notificationId, builder.build());
```
> Bitmap构造：BitmapFactory.decodeResource(getResources(),int R.drawable.xxx);

> [setStyle与展开式通知](https://developer.android.google.cn/training/notify-user/expanded.html)

> priority在Android8.0以下和Android8.0及以上有区别，具体请看[这里](https://developer.android.google.cn/training/notify-user/build-notification.html)

> [PendingIntent的构造](https://developer.android.google.cn/reference/android/app/PendingIntent.html)，举例如下

```
//PendingIntent.getActivity(Context context, int requestCode, Intent intent, int flag);
Intent intent = new Intent(this, xxxActivity);
PendingIntent pendingIntent = PendingIntent.getActivity(this,0,intent,FLAG_UPDATE_CURRENT);//第二个参数是requestCode，当同一时间想触发多个不同的PendingIntent时，可以用于区分不同的PendingIntent；第四个参数是flag，当同一时间只触发一个PendingIntent，可以设置为FLAG_CANCEL_CURRENT或FLAG_UPDATE_CURRENT

//PendingIntent.getService(Context context, int requestCode, Intent intent, int flag);
Intent intent = new Intent(this, xxxService);
PendingIntent pendingIntent = PendingIntent.getService(this,0,intent,FLAG_UPDATE_CURRENT);//第二个参数是requestCode，当同一时间想触发多个不同的PendingIntent时，可以用于区分不同的PendingIntent；第四个参数是flag，当同一时间只触发一个PendingIntent，可以设置为FLAG_CANCEL_CURRENT或FLAG_UPDATE_CURRENT

//PendingIntent.getBroadcast(Context context, int requestCode, Intent intent, int flag);
Intent intent = new Intent(this, xxxBroadcast);
PendingIntent pendingIntent = PendingIntent.getBroadcast(this,0,intent,FLAG_UPDATE_CURRENT);//第二个参数是requestCode，当同一时间想触发多个不同的PendingIntent时，可以用于区分不同的PendingIntent；第四个参数是flag，当同一时间只触发一个PendingIntent，可以设置为FLAG_CANCEL_CURRENT或FLAG_UPDATE_CURRENT

//PendingIntent.getActivities(Context context, int requestCode, Intent[] intents, int flag);
Intent intent1 = new Intent(this, xxxActivity);
Intent intent2 = new Intent(this, xxxActivity);
PendingIntent pendingIntent = PendingIntent.getBroadcast(this,0,new Intent[]{intent1, intent2},FLAG_UPDATE_CURRENT);//第二个参数是requestCode，当同一时间想触发多个不同的PendingIntent时，可以用于区分不同的PendingIntent；第四个参数是flag，当同一时间只触发一个PendingIntent，可以设置为FLAG_CANCEL_CURRENT或FLAG_UPDATE_CURRENT
```

---
##### 关于通知的最新内容
- 看[这里](https://developer.android.google.cn/guide/topics/ui/notifiers/notifications)
 
---
#### 调用摄像头拍照

```
private Uri imageUri=null;
private ImageView imageView；
imageView=findViewById(R.id.xxx);

//创建存储图片的文件对象
File imageFile = new File(getExternalCacheDir(), String imageName);//getExternalCacheDir()是应用关联缓存目录，不需要申请危险权限，当然可以指定其他存储位置，只是应用关联缓存目录不仅使用方便，主要也是因为这里并不需要真正的存储起来，只是缓存一下然后展示出来
try{
    if(imageFile.exists())
        imageFile.delete();
}catch(Exception e){
    e.printStackTrace();
}
//将文件对象转换成Intent可以传送的Uri，Android7.0以下及以上有差异
if(Build.VERSION.SDK_INT >= 24)
    imageUri=FileProvider.getUriForFile(Context context, String authority, File imageFile);
else
    imageUri=Uri.fromFile(imageFile);

//takePhotoIntent的requestCode
private static final int CODE_TAKE_PHOTO=1;
//启动相机，并传送Uri数据
Intent takePhotoIntent=new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
takePhotoIntent.putExtra(MediaStore.EXTRA_OUTPUT, imageUri);
startActivityForResult(takePhotoIntent, CODE_TAKE_PHOTO);

//重写onActivityResult，将存储在外部缓存的图片取出并转换成Bitmap显示在ImageView上
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    switc(requestCode){
        case CODE_TAKE_PHOTO:
            if(resultCode==RESULT_OK){
                try{
                    Bitmap bitmap=BitmapFactory.decodeStream(getContentResolver().openInputStream(imageUri));
                    imageView.setImageBitmap(bitmap);
                }catch(FileNoFoundException e){
                    e.printStackTrace();
                }
            }
    }
}

//注册provider，因为使用了FileProvider
<application>
   ...
   <provider
        android:name="android.support.v4.content.FileProvider"
        <!--这里的authorities和FileProvider.getUriForFile()的第二个参数要一致-->
        android:authorities="com.example.android.fileprovider"
        android:exported="false"
        android:grantUriPermissions="true">
        <meta-data
            android:name="android.support.FILE_PROVIDER_PATHS"
            <!--这里resource是用于指示在资源文件中配置的provider的合格路径所在位置-->
            android:resource="@xml/file_paths"></meta-data>
    </provider>
    ...
</application>
//在res文件夹的xml文件夹下创建名为file_paths文件，内容示例如下
<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-path name="my_images" path="Android/data/com.example.package.name/files/Pictures" />
</paths>
//name指明路径名，path指明真实路径，path的值会在调用getExternalFilesDir(Environment.DIRECTORY_PICTURES) 时作为返回值返回

//最后，Android4.4（API19）以下的需要声明写SD卡的权限，因为从Android4.4开始，getExternalCacheDir()即应用关联缓存目录不能被其他APP访问，而且由于运行时权限是从Android6.0（API23）才开始，所以不需要写运行时权限申请的代码。
//针对Android4.4以下的，可以这样写
<manifest>
    ...
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"
                    android:maxSdkVersion="18"/>
    ...
</manifest>
```

---
> 从Android7.0（API24）开始，不直接使用真实的Uri，因为这样会有安全隐患，而是通过FileProvider（一种特殊的内容提供器）来对真实的Uri封装，从而可以有选择性的共享给其他APP。

---
##### 更多关于摄像头拍照
- 请看[这里](https://developer.android.google.cn/training/camera/photobasics)和[CameraX（向后兼容到API21，即Android5.0）](https://note.youdao.com/)

---
#### 从相册选择照片

```
//申请写SD卡的权限
<manifest>
    ...
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    ...
</manifest>


private static final int CODE_WRITE_EXTERNAL_STORAGE=77;
private static final int CODE_OPEN_ALBUM=2;

//若写SD卡危险权限没有授权，申请写SD卡危险权限
if(ContextCompat.checkSelfPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE)!=PackageManager.PERMISSION_GRANTED){
    //若用户不是第一次拒绝，则弹窗说明申请权限的理由，然后接着继续申请权限
    if(ActivityCompat.shouldShowRequestPermissionRationale(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE)){
        AlertDialog.Builder builder=new AlertDialog.Builder(this)
                .setTitle("")
                .setMessage("");
        builder.show();
        ActivityCompat.requestPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE, CODE_WRITE_EXTERNAL_STORAGE);
    }else
        //若用户是第一次遇到，申请权限
        ActivityCompat.requestPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE, CODE_WRITE_EXTERNAL_STORAGE);
}else{
    //已经授权
    openAlbum();
}

//重写onRequestPermissionResult方法
@Override
public void onRequestPermissionResult(int requestCode, String[] permissions, int[] grantResults){
    switch(requestCode){
        case CODE_WRITE_EXTERNAL_STORAGE:
            if(grantResults.length>0 && grantResults[0]==PackageManager.PERMISSION_GRANTED)
                openAlbum();
            else
                Toast.makeText(this, "You denied the permission",Toast.LENGTH_SHORT).show();
            break;
        default:
            break;
    }
}

//打开相册，可以封装成一个函数
private void openAlbum(){
    Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
    //打开所有的图片类型
    intent.setType("image/*");
    startActivityForResult(intent, CODE_OPEN_ALBUM);
}

//重写onActivityResult，将Intent里的图片取出并解析成真实的图片路径，然后显示在ImageView上
protected void onActivityResult(int requestCode, int resultCode, Intent data){
    switc(requestCode){
        case CODE_OPEN_ALBUM:
            if(resultCode==RESULT_OK){
                //由于从Android4.4开始，图片的Uri经过provider的封装，直接拿到的不再是真实的Uri，所以需要解析出真实的Uri，然后再得到图片的路径
                if(Build。VERSION_SDK_INT >= 19){
                    //Android4.4及以上
                    openOnKitKat(data);
                }
                else{
                    //Android4.4以下
                    openBeforeKitKat(data);
                }
            }
    }
}

//Android4.4及以上，获取图片的真实路径并显示在ImageView上
@TargetApi(19)
private void openOnKitKat(Intent data){
    String imagePath=null;
    Uri imageUri=data.getData();
    
    if(DocumentsContract.isDocumentUri(this,imageUri)){
        String docId=DocumentsContract.getDocumentId(imageUri);
        if("com.android.providers.media.documents".equals(imageUri.getAuthority())){
            String id = docId.split(":")[1];
            imagePath=getImagePathFromContent(MediaStore.Images.Media.EXTRNAL_CONTENT_URI,MediaStore.Images.Media._ID+" = "+id);
        }else if("com.android.providers.downloads.documents".equals(imageUri.getAuthority())){
            imagePath=getImagePathFromContent(Uri.parse("content://downloads.public_downloads"),Long.valueOf(docId));
        }
    }else if("content".equalsIgnoreCase(imageUri.getScheme())){
        imagePath=getImagePathFromContent(imageUri,null);
    }else if("file".equalsIgnoreCase(imageUri.getScheme())){
        imagePath=imageUri.getPath();
    }
    showImage(imagePath);
}

//Android4.4以下，获取图片的真实路径并显示在ImageView上
private void openBeforeKitKat(Intent data){
    Uri imageUri=data.getData();
    String imagePath=getImagPathFromContent(imageUri,null);
    showImage(imagePath);
}

//从其他APP的内容提供器获取图片路径
private String getImagePathFromContent(Uri uri,String selection){
    String path=null;
    Cursor cursor=getContentResolver().query(
        uri,
        null,
        selection,
        null,
        null);
    if(cursor!=null&&cursor.moveToFirst())
        path=cursor.getString(cursor.getColumnIndex(MediaStore.Images.Media.DATA));
    return path;
}
//传入图片真实路径， 并显示在ImageView上
private void showImage(String path){
    if(!(TextUtils.isEmpty(path))){
        Bitmap bitmap=BitmapFactory.decodeFile(path);
        imageView.setImageBitmap(bitmap);
    }else
        throw new IllegalArgumentException("Unknown image path");
}
```

---
#### 播放音频

> 一般使用MediaPlayer类实现播放音频，MediaPlayer类常用的方法有

>> setDataSource()：设置要播放的音频文件的位置

>> prepare()：播放前调用这个方法完成准备工作

>> start()：开始或继续播放音频

>> pause()：暂停播放音频

>> reset()：将MediaPlayer对象重置到刚刚创建的状态

>> seekTo()：从指定位置开始播放

>> stop()：停止播放，调用这个方法后MediaPlayer对象无法再播放音频

>> release()：调用这个方法释放掉MediaPlayer对象的资源

>> isPlaying()：判断当前MediaPlayer对象是否在播放

>> getDuration()：获取音频文件的时长

```
private MediaPlayer mediaPlayer = new MediaPlayer();
private static final int CODE_READ_AUDIO=1;

//事先在SD卡根目录下放一个名为music.mp3的文件，读取并初始化MediaPlayer对象
private void initAudioPlayer(){
    try{
        File file = new File(Environment.getExternalStorageDirectory(),"music.mp3");
        mediaPlayer.setDataSource(file.getPath());//指定音频文件的路径
        mediaPlayer.prepare();//准备音频文件
    }catch(IOException e){
        e.printStackTrace();
    }
}

//申请写SD卡的权限
if(ContextCompat.checkSelfPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE)!=PackageManager.PERMISSION_GRANTED){
    if(ActivityCompat.shouldShowRequestPermissionRationale()){
        AlertDialog.Builder builder=new AlertDialog.Builder(Context)
                .setTitle("")
                .setMessage("");
        builder.show();
        ActivityCompat.requestPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE, CODE_READ_AUDIO);
    }else{
        ActivityCompat.requestPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE, CODE_READ_AUDIO);
    }
}else{
    initAudioPlayer();
}

//重写onRequestPermissionResult方法
@Override
public void onRequestPermissionResult(int requestCode, String[] permissions, int[] grantResults){
    switch(requestCode){
        case CODE_READ_AUDIO:
            if(grantResults.length > 0 && grantResults[0]==PackageManager.PERMISSION_GRANTED)
                initAudioPlayer();
            else{
                Toast.makeText(Context, "You denied the permission and you can't play audio~", Toast.LENGTH_LONG).show();
                finish();
            }
            break;
        deffault:
            break;
    }
}

//可以创建几个按钮测试，如play_audio、pause_audio、reset_audio
@Override
public void onClick(View view){
    switch(view.getItemId()){
        case R.id.play_audio:
            mediaPlayer.start();
            break;
        case R.id.pause_audio:
        mediaPlayer.pause();
            break;
        case R.id.reset_audio:
            mediaPlayer.reset();
            initAudioPlayer();
            break;
        default:
            break;
    }
}

//可以选择在onDestroy方法中调用MediaPlayer对象的stop()和release()方法释放资源
@Override
protected void onDestroy(){
    super.onDestroy();
    if(mediaPlayer!=null){
        mediaPlayer.stop();
        mediaPlayer.release();
    }
}

//最后，记得在manifest文件中申请权限
<manifest>
    ...
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    ...
</manifest>
```

---
##### 关于如何具体地构建一个audio app
- 看[这里](https://developer.android.google.cn/guide/topics/media-apps/audio-app/building-an-audio-app?hl=en)

---
#### 播放视频

> 主要使用VideoView类实现，VideoView的方法主要有

>> setVideoPath()：设置视频文件的路径

>> start()：开始或继续播放视频

>> pause()：暂停播放视频

>> resume()：恢复播放视频

>> seekTo()：指定视频播放位置

>> isPlaying()：判断视频是否在播放

>> getDuration()：获取视频播放时长

>> suspend()：释放视频资源

```
//xml布局资源文件中，通过<VideoView>标签创建VideoView，id设为videoView
private VideoView videoView = findViewById(R.id.videoView);
private static final int CODE_READ_VIDEO=2;

//事先在SD卡根目录下放一个名为video.mp4的文件，读取并为VideoView对象设置视频位置
private void initVideoView(){
    try{
        File file = new File(Environment.getExternalStorageDirectory(),"video.mp4");
        videoView.setVideoPath(file.getPath());//指定视频文件的路径
    }catch(IOException e){
        e.printStackTrace();
    }
}

//申请写SD卡的权限
if(ContextCompat.checkSelfPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE)!=PackageManager.PERMISSION_GRANTED){
    if(ActivityCompat.shouldShowRequestPermissionRationale()){
        AlertDialog.Builder builder=new AlertDialog.Builder(Context)
                .setTitle("")
                .setMessage("");
        builder.show();
        ActivityCompat.requestPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE, CODE_READ_VIDEO);
    }else{
        ActivityCompat.requestPermission(thisActivity, Manifest.permission.WRITE_EXTERNAL_STORAGE, CODE_READ_VIDEO);
    }
}else{
    initVideoView();
}

//重写onRequestPermissionResult方法
@Override
public void onRequestPermissionResult(int requestCode, String[] permissions, int[] grantResults){
    switch(requestCode){
        case CODE_READ_VIDEO:
            if(grantResults.length > 0 && grantResults[0]==PackageManager.PERMISSION_GRANTED)
                initVideoview();
            else{
                Toast.makeText(Context, "You denied the permission and you can't play audio~", Toast.LENGTH_LONG).show();
                finish();
            }
            break;
        deffault:
            break;
    }
}

//可以创建几个按钮测试，如play_video、pause_video、resume_video
@Override
public void onClick(View view){
    switch(view.getItemId()){
        case R.id.如play_video:
            videoView.start();
            break;
        case R.id.pause_audio:
            videoView.pause();
            break;
        case R.id.reset_audio:
            videoView.resume();
            break;
        default:
            break;
    }
}

//可以选择在onDestroy方法中调用VideoView对象的suspend()方法释放资源
@Override
protected void onDestroy(){
    super.onDestroy();
    if(videoView!=null)
        videoView.suspend();
}

//最后，记得在manifest文件中申请权限
<manifest>
    ...
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    ...
</manifest>
```

---
##### 关于如何具体地构建一个video app
- 看[这里](https://developer.android.google.cn/guide/topics/media-apps/video-app/building-a-video-app?hl=en)
