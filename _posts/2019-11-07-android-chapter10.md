---
layout: post
title: "《第一行代码》Chapter 10"
subtitle: '探究服务'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 目录

1.[服务](#服务)

2.[Android多线程编程](#android多线程编程)

3.[在新线程更新UI](#在新线程更新ui)

- [异步消息处理](#异步消息处理)
- [解析异步消息处理机制](#解析异步消息处理机制)
- [runOnUiThread(Runnable)](#runonuithread)
- [AsyncTask](#asynctask)
    - [更多关于AsyncTask](#更多关于asynctask)

4.[服务的基本用法](#服务的基本用法)
- [启动服务](#启动服务)
    - [启动服务继承自Service](#启动服务继承自service)
    - [启动服务继承自IntentService](#启动服务继承自intentservice)
- [绑定服务](#绑定服务)
    - [创建IBinder的3个方法](#创建ibinder的3个方法)
    - [继承Binder类来创建Binder的具体步骤](#继承binder类来创建binder的具体步骤)
    - [使用Messenger来创建Binder的具体步骤](#使用messenger来创建binder的具体步骤)
    - [更多关于绑定服务](#更多关于绑定服务)
- [前台服务](#前台服务)

5.[更多关于服务的生命周期](#更多关于服务的生命周期)

6.[服务的最佳实践-下载文件](#服务的最佳实践-下载文件)

## 正文

---
##### 注1：本文提到的新线程指非主线程或非界面线程的线程
##### 注2：本文提到的客户端指向服务发出请求（启动请求或绑定请求）的组件

---
#### 服务

---
##### 1. 概念：即使用户未与应用交互也可在后台运行的组件。服务可由其他组件启动，也可通过绑定与之交互，甚至是执行进程间通信（IPC）。

---
##### 2. 服务和线程之间的选择

> 只有在确实需要服务的时候才使用服务。如果必须在主线程之外执行操作，但是只在用户与应用交互时才执行，那么应该使用线程。如：在运行Activity时 播放一些音乐，那么可以在onCreate()创建线程，在onStart()启动线程，在onStop()停止线程，可以考虑使用AsyncTask、HandlerThread。

> ==因为默认情况下服务运行在主线程，不得不使用服务时，若服务执行耗时任务，那么应该将服务放入新线程执行，以避免阻塞主线程。==

---
##### 3. 类型：前台、后台、绑定。

> 前台服务执行一些用户能够注意到的操作，如：音乐播放器播放音乐。前台服务必须显示通知，即使用户停止与应用交互，前台服务仍会继续运行。

> 后台服务执行用户不会注意到的操作，如：后台压缩存储空间。==从API26开始==，当应用未在前台运行时，系统会对后台服务施加限制，建议采用计划作业（scheduling）

> 应用组件通过bindService()绑定到服务时，服务即处于绑定状态。绑定服务会提供客户端-服务器接口，以便组件与服务进行交互、发送请求、接收结果，甚至是利用进程间通信（IPC）跨进程执行这些操作。多个组件可同时绑定到一个服务，当全部取消绑定后，该服务即被销毁。

---
##### 4. 服务运行方式：启动服务、绑定服务。服务可以同时以这两种状态运行。

> 启动服务：组件通过调用startService()启动服务（这会引起onStartCommand()方法的调用）。启动服务后，其生命周期独立于启动它的组件，即使组件已销毁，服务也可以在后台无限期运行，直到服务自己调用了stopSelf()或其他组件调用stopService()方法将其停止为止。

> 绑定服务：组件通过调用bindService()创建服务，若没有调用onStartCommand()，则服务在组件与其绑定时运行。在服务与所有组件取消绑定时，由系统负责销毁。

---
##### 5. ==服务生命周期==

> onCreate()：首次创建服务时，系统会调用此方法。

> onStartCommand()：当组件通过startService()方法启动服务时，系统调用此方法。

> onBind()：当组件通过bindService()方法绑定服务时，系统调用此方法。

> onUnbind()：当所有绑定该服务的组件都调用了unbindService()后，系统调用此方法。若返回true，表示下一次调用bindService()时，系统调用onReBind()方法；返回false，表示下一次调用bindService()时，系统调用onBind()方法

> onRebind()：当系统调用了onUnbind()方法之后，有一个客户端调用了bindService()方法

> onDestroy()：当销毁服务时，系统调用此方法。在此方法中应清理与服务相关的资源，如：线程、监听器、接收器等。

![启动服务和绑定服务各自的生命周期](img/in-post/android/service_lifecycle)
![既是启动也是绑定的服务的生命周期](img/in-post/android/service_binding_tree_lifecycle)

---
##### 6. 服务的注意事项

> ==默认情况下，服务与声明服务的APP运行在同一个进程，且运行在APP的主线程中。所以，如果服务执行耗时任务时，应该开启新线程运行服务，以避免阻塞主线程，以及可能导致的ANR（应用无响应）。==

> 任何组件可以像使用Activity一样，通过Intent使用服务（即使服务来自另一个APP）。为了安全性，最好使用显式Intent启动服务，且==从Android5.0（API21）开始==，使用隐式Intent调用bindService()会抛出异常

> 可以通过添加android:exported="false"，确保服务不可以被其他APP使用。可以通过添加android:description=""，告诉用户该服务的作用和好处。

---
##### 7. 服务的终止

> 只有在内存过低且必须回收系统资源以供拥有用户焦点的Activity使用时，Android系统才会停止服务。

> 前台服务几乎不会被终止；当服务绑定的Activity拥有用户焦点时，不太可能被终止；随着服务运行时间的增加，系统会逐渐降低其在后台任务列表中的位置，服务被终止的可能性会逐渐增大。

> 若系统终止服务，那么在资源可用时会立即重启服务。若服务是启动服务，则需要妥善设计，以便重启时恢复工作

---
##### 更多关于服务的内容
- [这里](https://developer.android.google.cn/guide/components/services)

---
#### Android多线程编程

##### 线程的基本用法

- 继承Thread类

```
class MyThread extends Thread{
    @Override
    public void run(){
        //新线程任务
    }
}
//启动新线程
new MyThread().start();
```

- 实现Runnable接口

```
class MyRunnable implements Runnable{
    @Override
    public void run(){
        //新线程任务
    }
}
//启动新线程
new Thread(new MyRunnable()).start();
```

- 匿名类

```
new Thread(new Runnable(){
    @Override
    public void run(){
        //新线程任务
    }
}).start();
```

---
#### 在新线程更新UI

> 上一章提到过，由于Android中显示用户界面的活动只有单个线程（如果不主动开启新线程），若以60帧/秒的速度绘制界面，意味着每17毫秒要绘制一帧，所以在主线程不应该执行耗时任务，如：联网等。

> 推荐的做法是开启新线程执行耗时任务，但是这带来一个问题：新线程中需要更新UI时，不能直接更新UI，否则程序会崩溃，抛出CallFromWrongThreadException异常。因为Android中的UI是线程不安全的，必须要在主线程中更新UI。

---
##### 异步消息处理
- 使用

```
//消息类型
public static final int MSG_UPDATE_TEXT=1;

//主线程中，新建Handler对象，重写handleMessage方法，接收消息，在该方法内可以更新UI
Handler handler=new Handler(){
    @Override
    public void handleMessage(Message msg){
        switch(msg.what){
            case MSG_UPDATE_TEXT:
                //可以更新UI
                break;
            default:
                break;
        }
    }
}

//在主线程开启的新线程中，通过handler发送消息
@Override
public void onClick(View view){
    switch(view.getItemId()){
        case R.id.xxx:
            new Thread(new Runnable() {
                @Override
                public void run() {
                    Message message=new Message();
                    message.what=MSG_UPDATE_TEXT;
                    handler.sendMessage(message);
                }
            }).start();
            break;
        default:
            break;
    }
}
```

---
##### 解析异步消息处理机制

- 异步消息处理主要由4部分组成：Message、Handler、MessageQueue、Looper

> 1. Message：在进程间传递的消息，可携带少量信息。
> 2. handler：处理者，发送和处理消息，通过Handler.sendMessage(Message)方法发送消息，通过重写的handleMessage(Message msg)方法处理消息
> 3. MessageQueue：主要存放从Handler发送的等待被处理的消息的队列。==每个线程中只会有一个MessageQueue对象。==
> 4. Looper：管理MessageQueue，调用Looper的loop()方法之后，会进入一个无限循环中，每当MessageQueue中有消息时，就取出，并传递给Handler的handleMessage()方法中。==同样的，每个线程中只会有一个Looper对象。==

- 异步消息处理的整个流程

> 主线程中，创建Handler对象，并重写handleMessage()方法，每当新线程需要更新UI，创建一个Message对象，通过Handler发送出去。

> 发送出去的Message会进入MessageQueue中等待被处理，Looper不断从MessageQueue中取出Message，并调用dispatchMessage()方法将Message传递给Handler的handleMessage()方法。

```
sequenceDiagram
Handler->>MessageQueue: sendMessage(Message)
MessageQueue->>Looper: Message
Looper->>Handler: dispatchMessage(Message)
Handler->>handleMessage: Message
```

---
##### runOnUiThread
- runOnUiThread(Runnable)背后的原理是异步消息处理机制

```
//新线程中调用runOnUiThread即可以更新UI
new Thread(new Runnable(){
    @Override
    public void run(){
        //新线程的任务
        new runOnUiThread(new Runnable(){
            @Override
            public void run(){
                //可以更新UI
            }
        });
    }
}).start();
```

---
##### AsyncTask

- AsyncTask背后的原理也是异步消息处理机制。

> 1. AsyncTask允许我们运行后台操作并把结果发送给UI线程而不需要管理Thread或Handler。

> 2. AsyncTask不支持长时间的后台操作（一般几秒），若需要运行长时间的后台任务，可以使用[Executor](https://developer.android.google.cn/reference/java/util/concurrent/Executor.html)，[ThreadPoolExecutor](https://developer.android.google.cn/reference/java/util/concurrent/ThreadPoolExecutor.html)或[FutureTask](https://developer.android.google.cn/reference/java/util/concurrent/FutureTask.html)。

> 3. AsyncTask接收3个参数，主要重写的方法有4个方法。

> - 3个参数：第一个参数Params是传入的数据的类型，第二个参数Progress是进度条参数，第三个参数Result是返回的数据的类型。

> - 4个方法：onPreExecute()、doInBackground()、onProgressUpdate()、onPostExecute()。

> onPreExecute()：执行后台任务前进行界面的初始化工作，如：显示进度条

> doInBackground()：在后台线程执行任务，通过return语句将执行结果传给onPostExecute()。此方法内不应该执行UI更新（因为不是主线程或界面线程）。

> onProgressUpdate()：更新任务进度，需要在doInBackground()方法中使用publishProgress()方法传入当前进度信息，该进度信息会传递给onProgressUpdate()方法。该方法内可以更新UI

> onPostExecute()：根据doInBackground()返回的结果执行任务，可以更新UI

- 事实上，由于AsyncTask适合处理几秒钟的耗时任务，onProgressUpdate()不太有必要使用，一般通过控制ProgressBar的VISIBILITY显示执行情况即可。

- AyncTask的创建和使用举例

```
class MyAsyncTask extends AsyncTask<Void,Integer,Boolean>{

    @Override
    protected void onPreExecute() {
        //可以执行UI界面的初始化工作
    }

    @Override
    protected Boolean doInBackground(Void... voids) {
        //执行后台任务，成功返回true，失败返回false
        //记得调用publishProgress()方法传递进度信息给onProgressUpdate()
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        //更新UI界面中任务的进度
    }

    @Override
    protected void onPostExecute(Boolean aBoolean) {
        //得到任务执行结果，可以更新UI
    }
}

//不重写onProgressUpdate，xml布局资源文件中，添加ProgressBar，visibility设为INVISIBLE
ProgressBar mTaskProgressBar=findViewById(R.id.xxx);
class MyAsyncTask extends AsyncTask<Void,Integer,Boolean>{

    @Override
    protected void onPreExecute() {
        //可以执行UI界面的初始化工作
        mTaskProgressBar.setVisibility(View.VISIBLE);
    }

    @Override
    protected Boolean doInBackground(Void... voids) {
        //执行后台任务，成功返回true，失败返回false
    }

    @Override
    protected void onPostExecute(Boolean aBoolean) {
        //得到任务执行结果，可以更新UI
        mTaskProgressBar.setVisibility(View.INVISIBLE);
    }
}

//在需要执行AsyncTask的地方，实例化AsyncTask并调用execute()方法
new MyAsyncTask().execute();
```

---
##### 更多关于AsyncTask
- [这里](https://developer.android.google.cn/reference/android/os/AsyncTask.html)

---
### 服务的基本用法

---
#### 启动服务

- 启动服务由另一个组件通过调用startService()启动，startService()会回调onStartCommand()方法
- 无论其他组件向一个启动服务发出多少启动请求，==只要调用一次stopSelf()或者stopService()方法，服务即停止。==
- 一般可以继承两个类来创建启动服务

> 1. Service：适用于所有服务的基类，可一次处理多个启动服务请求。但可能需要你手动创建（可能多个）新线程用于执行服务，以免阻塞主线程

> 2. IntentService：Service的子类，一次处理一个启动服务请求，IntentService会执行以下操作
>> 1. 创建新线程执行服务，所以不需要你创建新线程
>> 2. 创建存放Intent队列，逐一按顺序将Intent传递给onHandleIntent()执行，不必担心多线程问题
>> 3. 处理完所有Intent之后停止服务，所以不需要你调用stopSelf()
>> 4. 提供onStartCommand(的默认实现，将Intent依次发送到工作队列和onHandleIntent()
>> 5. 提供onBind()的默认实现，返回null

```
//继承自Service
public class HelloService extends Service{
    @Override
    public void onCreate(){
        //创建服务时的准备工作
    }
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId){
        //执行任务，耗时任务需要手动开启新线程
        
        //也可以在耗时任务执行结束之后，调用stopSelf()来停止服务
        stopSelf();
    }
    
    @Override
    public IBinder onBind(Intent intent) {
        // 如果服务不提供绑定，返回null即可
        return null;
    }

    
    @Override
    public void onDestroy(){
        //释放服务相关资源
    }
}

//启动服务
Intent intent=new Intent(this, HelloService.class);
startService(intent);
//停止服务
Intent intent=new Intent(this, HelloService.class);
stopService(intent);


//继承自IntentService
public class HelloIntentService extends IntentService{
    //构造函数是必须的
    public HelloIntentService(){
        //一般新线程的名字是类名即可，这里是HelloIntentService
        super("HelloIntentService");
    }
    
    //重写onHandleIntent()
    @Override
    protected void onHandleIntent(Intent intent){
        //要执行的任务
    }
    
    //如果想重写其他回调方法（onCreate()、onStartCommand()、onDestroy()等），一定要确保使用超类(super.xxx)实现。onBind()是重写时唯一不需要调用超类的方法，而且只有在服务需要绑定时，才需要实现该方法。
    //举例：重写onStartCommand()
    @Override
    public int onStartCommand(Intent intent, int flags, int startId){
        return super.onStartCommand(intent,flags,startId);
    }
}

//启动服务，不需要终止，因为IntentService已经实现了
Intent intent=new Intent(this, HelloIntentService.class);
startService(intent);

```

---
##### 注意：
- onStartCommand()方法返回的必须是以下三种常量之一：

> 1. START_NOT_STICKY：除非有待传递的PendingIntent，否则不会重建服务
> 2. START_STICKY：重建服务，但是不会重新传递上一个Intent给onStartCommand()，但是如果上一个Intent是PendingIntent，那就会重新传递geionStartCommand()。适用于不需要执行命令、无限期运行并等待作业的多媒体播放器（或类似的服务）。
> 3. START_REDELIVER_INTENT：重建服务，并且重新传递上一次的Intent给onStartCommand()。适用于正在执行作业的、需要立刻恢复的服务，如：下载文件。

- 如果服务同时处理多个对onStartCommand()的请求，不应该在处理第一个请求后就停止服务（停止服务会终止第二个请求），应该停止最近一次处理完的请求，通过调用stopSelf(int)方法实现。

---
##### 启动服务继承自Service
- [查看更多](https://developer.android.google.cn/guide/components/services#ExtendingService)

##### 启动服务继承自IntentService
- [查看更多](https://developer.android.google.cn/guide/components/services#ExtendingIntentService)

---
#### 绑定服务
- 绑定服务与启动服务的区别在于：
>> 1. 绑定服务必须重写onBind()方法，在该方法中返回IBinder，这样当其他组件提供bindService()绑定服务时返回的就是IBinder，通过IBinder的方法进行通信。如果绑定服务不提供启动状态，可以不用重写onStartCommand()方法。
>> 2. 其他组件通过bindService()方法创建绑定服务，而不是startService()方法。而且，调用时，其他组件必须提供ServiceConnection的实现，用以监控与服务的连接。
>> 3. 其他组件通过调用unbindService()与绑定服务解除绑定。
>> 4. 绑定服务只在为其他组件提供服务时处于活动状态，不会无限期在后台运行。
>> 5. 绑定服务的停止不需要像启动服务那样需要调用stopSelf()或stopService()方法来停止服务。==当没有组件与其绑定时，系统会销毁绑定服务。==

---
##### 创建IBinder的3个方法

> 1. 继承Binder类
>> ==服务仅供自己应用使用，且无需跨进程工作（与客户端处于同一应用和进程内）==，如：那么可以选择继承Binder类创建IBinder。客户端可通过Binder直接访问Binder内的方法和Service中可用的公共方法。如：需要将活动绑定到在后台播放音乐的服务的APP。
> 2. 使用Messenger
>> ==服务需要跨进程==使用，那么可以使用Messenger创建IBinder，Messenger是执行进程间通信（IPC）最简单的方法，这是因为Messenger会在单个线程中创建包含所有绑定请求的队列，这样服务就一次接收一个请求，而AIDL接口会同时向服务发送多个请求，服务随后必须执行多线程处理。而且使用Messenger你不需要对服务进行线程安全设计。 
> 3. 使用AIDL
>> AIDL可让服务==同时处理多个请求==，但大多数应用==不应该使用AIDL==创建绑定服务，因为AIDL需要多线程处理能力，这可能导致更为复杂的实现，同时AIDL需要保证达到线程安全的要求。

- 如果确实需要直接使用AIDL，参阅[这里](https://developer.android.google.cn/guide/components/aidl.html)

---
##### 继承Binder类来创建Binder的具体步骤

> 1. 在您的服务中，创建可执行以下某种操作的 Binder 实例：
>> 1. 包含客户端可调用的公共方法。
>> 2. 返回当前的 Service 实例，该实例中包含客户端可调用的公共方法。
>> 3. 返回由服务承载的其他类的实例，其中包含客户端可调用的公共方法。
> 2. 从 onBind() 回调方法返回此 Binder 实例。
> 3. 在客户端中，从 onServiceConnected() 回调方法接收 Binder，并使用提供的方法调用绑定服务。

```
//1. 继承Binder类，实现访问Binder中和服务中的方法
//创建服务
public calss LocalService extends Service{
    private final IBibder binder = new LocalBinder();
    private final Random mRandom = new Random();
    
    public class LocalBinder extends Binder{
        //Binder内的getService()方法，返回LocalService实例，这样当客户端调用getService()方法，客户端就可以根据得到的LocalService实例访问服务中的方法
        LocalService getService(){
            return LocalService.this;
        }
        
        //还可以根据需要创建更多binder内的方法，不过有了getService()方法，在服务内创建公共方法效果一样，客户端都可以访问
    }
    
    //服务提供的可访问的公共方法
    public int getRandomNumber(){
        return mRandom.netInt(100);
    }
    
    //重写onBind()方法，返回IBinder
    @Override
    public IBinder onBind(Intent intent){
        return binder;
    }
}

//客户端绑定服务
public class BindingActivity extends AppCompatActivity{
    private LocalService mService;
    private boolean mBound;
    
    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }
    
    @Override
    protected void onStart(){
        super.onStart();
        //绑定服务
        Intent intent=new Intent(this,LocalService.class);
        bindService(intent,connection,Context.BIND_AUTO_CREATE);
    }
    
    @Override
    protected void onStop(){
        super.onStop();
        //取消绑定
        unbindService(connection);
        mBound=false;
    }
    
    //监控与服务的连接
    private ServiceConnection connection=new ServiceConnection(){
        @Override
        public void onServiceConnected(ComponentName className, IBinder service){
            //IBinder向下转型得到LocalBinder，调用LocalBinder的getService()方法得到LocalService实例
            LocalBinder mBinder=(LocalBinder)servie;
            mService=mBinder.getService();
            mBound=true;
        }
        
        @Override
        public void onServiceDisconnected(Componentname arg0){
            mBound=false;
        }
    }
    
    //点击按钮时，调用服务的getRandomNumber()方法。按钮在xml布局资源文件中通过android:onClick属性创建按钮的点击事件处理方法
    public void onButtonClick(View v){
        if(mBound)
            Toast.makeText(this,"number: "+mService.getRandomNumber(), Toast.LENGTH_LONG).show();
    }
}

```

---
##### 使用Messenger来创建Binder的具体步骤

> 1. 服务实现继承自Handler的内部类，由该类为每个客户端调用接收回调。
> 2. 服务使用Handler的实例来创建Messenger对象。
> 3. Messenger创建一个IBinder，服务通过onBinde()返回个客户端。
> 4. 客户端使用ServiceConnection的onServiceConnection()方法传入的IBinder，创建Messenger对象，通过Messenger对象发送Message给服务。
> 5. 服务在继承自Handler内部类重写的handleMessage()方法中接收并处理Message。

```
//通过Messenger创建服务
public class MessengerService extends Service{
    
    Messenger mMessenger;
    public static final int MSG_SAY_HELLO=1;
    
    //1. 服务实现继承自Handler的内部类，由该类为每个客户端调用接收回调。
    static class MessengerHandler extends Handler{
        private Context applicationContext;
        
        MessengerHandler(Context context){
            applicationContext=context.getApplicationContext();
        }
        
        //5. 服务在继承自Handler内部类重写的handleMessage()方法中接收并处理Message。
        @Override
        public void handleMessage(Message msg){
            switch(msg.what()){
                case MSG_SAY_HELLO:
                    Toast.makeText(applicationContext,"hello~",Toast.LENGTH_LONG).show();
                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }
    
    @Override
    public IBinder onBind(){
        //2. 服务使用Handler的实例来创建Messenger对象。
        mMessenger=new Messenger(new MessengerHandler(this));
        //3. Messenger创建一个IBinder，服务通过onBinde()返回个客户端。
        return mMessenger.getBinder();
    }
    
}


//客户端使用绑定服务
public class MessengerActivity extends AppCompatActivity{
    private Messenger mMessenger=null;
    private boolean mBound;

    @Override
    protected void onCreate(Bundle savedInstanceState){
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
    }
    
    @Override
    protected void onStart(){
        super.onStart();
        //绑定服务
        Intent intent=new Intent(this,MessengerService.class);
        bindService(intent,connection,Context.BIND_AUTO_CREATE);
    }
    
    @Override
    protected void onStop(){
        super.onStop();
        //取消绑定
        if(mBound){
            unbindService(connection);
            mBound=false;
        }
    }
    
    //4. 客户端使用ServiceConnection的onServiceConnection()方法传入的IBinder，创建Messenger对象
    private ServiceConnection connection=new ServiceConnection(){
        @Override
        public void onServiceConnected(ComponentName className,IBinder service){
            mMessenger=new Messenger(service);
            mBound=true;
        }
        
        @Override
        public void onServiceDisconnected(ComponentName className){
            mMessenger=null;
            mBound=false;
        }
    }
    
    //4. 通过Messenger对象发送Message给服务
    public void sayHello(View v){
        if(!mBound)
            return;
        Message msg=new Message.obtain(null,MessengerService.MSG_SAY_HELLO,0,0);
        try{
            mMessenger.send(msg);
        }catch(RemoteException e){
            e.printStackTrace();
        }
    }
}

```

- 注意：Messenger的上述示例，没有说明服务如何对客户端作出响应，如果需要服务对客户端作出响应，[看这里（服务）](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerService.java)和[这里（客户端）](https://android.googlesource.com/platform/development/+/master/samples/ApiDemos/src/com/example/android/apis/app/MessengerServiceActivities.java)

- 注意：==只有Activity、Service和ContentProvider可以绑定到服务，BroadcastReceiver无法绑定到服务。==

---
##### 更多关于绑定服务
- [看这里](https://developer.android.google.cn/guide/components/bound-services.html)

---
##### 注意
- 如果==服务同时具有启动和绑定状态==，那么在没有组件绑定服务时，该服务不会被销毁，这意味着，只有所有组件调用unbindService()方法之后，而且服务自身调用了stopSelf()或其他组件调用了stopService()，该服务才会停止。

---
#### 前台服务
- 前台服务与启动、绑定服务的区别
> 1. 前台服务是用户主动意识到服务，在低内存时系统也不会考虑杀掉它
> 2. 前台服务必须在状态栏提供一个通知，除非服务被停止或从其他移除。
> 3. 尽量不使用前台服务。只有当你的APP执行的任务需要让用户知道（即使用户没有与APP交互），你才应该使用前台服务。否则，建议使用安排作业

---
##### 注意
- 面向Android 9(API28)及以上，使用前台服务需要声明==FOREGROUND_SERVICE==权限，该权限是普通权限。若没有声明，则系统会抛出SecurityException异常。

```
Intent intent=new Intent(this,ExampleActivity.class);
PendingIntent pendingIntent=PendingIntent.getActivity(this,0,intent,0);

//Android8.0开始Notification.Builder需要传入Channel参数
Notification notification=new Notification.Builder(this, CHANNEL_DEFAULT_IMPORTANCE)
    .setContentTitle(getText(R.string.notification_title))
    .setContentText(getText(R.string.notification_message))
    .setSmallIcon(R.drawable.icon)
    .setContentIntent(pendingIntent)
    .setTicker(getText(R.string.ticker_text))
    .build();

//启动前台服务
startForeground(ID_FOREGROUND_NOTIFICATION, notification);
```

---
##### 注意
- 用于标识通知的ID_FOREGROUND_NOTIFICATION==不能为0==。
- 通过调用==stopForeground()可以移除服务==，该方法需要传入一个boolean值，可以指示是否同时移除通知，但是不会停止服务。而服务停止时，通知也会移除。

---
#### 更多关于服务的生命周期
- [启动服务和绑定服务各自的生命周期](https://developer.android.google.cn/guide/components/services#LifecycleCallbacks)
- [即是启动又是绑定的服务的生命周期](https://developer.android.google.cn/guide/components/bound-services#Lifecycle)

---
#### 服务的最佳实践-下载文件
- 实现的功能：暂停下载、取消下载，显示下载进度，支持断点下载
- 技术细节：使用AsyncTask执行下载任务，使用回调机制监听下载状态，使用前台服务显示下载进度，下载位置为SD卡的Download文件夹，文件名定为Url的最后一个"/"接着的路径名。

```
//添加依赖库
implementation 'com.squareup.okhttp3:okhttp:4.2.2'

//定义回调接口，创建AsyncTask
//新建class，名为DownloadUtils，定义回调接口
public class DownloadUtils {

    public static final int BYTE_SIZE = 1024;
    public static final int PROGRESS_START = 0;
    public static final int PROGRESS_MAX = 100;
    public static final int PROGRESS_NONE = -1;
    public static final int ID_PROGRESS_NOTIFICATION = 1;

    public interface DownloadCallback {
        void onProgress(int progress);

        void onSuccess();

        void onFailure();

        void onPause();

        void onCancel();
    }

}

//新建class，名为DownloadTask，创建AsyncTask，传入下载的url字符串，进度条参数为整型，返回下载状态（用整型表示）
public class DownloadTask extends AsyncTask<String, Integer, Integer> {

    private static final int STATUS_SUCCESS = 1;
    private static final int STATUS_FAILURE = 2;
    private static final int STATUS_PAUSE = 3;
    private static final int STATUS_CANCEL = 4;

    private boolean isPaused = false;
    private boolean isCanceled = false;

    private int lastProgress = 0;
    private DownloadCallback mDownloadListener;

    DownloadTask(DownloadCallback listener) {
        mDownloadListener = listener;
    }

    @Override
    protected Integer doInBackground(String... strings) {
        //已下载文件大小
        long fileDownloadSize = 0;
        //需要下载的文件的大小
        long fileContentLength;
        //下载资源的url
        String urlString = strings[0];
        //读写文件
        InputStream in = null;
        RandomAccessFile randomAccessFile = null;

        //下载位置在SD卡的Download文件夹里，文件名为urlString的最后一个/后面的字符串
        File file = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).getAbsolutePath() +
                urlString.substring(urlString.lastIndexOf("/")));
        if (file.exists())
            fileDownloadSize = file.length();
        fileContentLength = getUrlContentLength(urlString);
        if (fileContentLength == 0)
            return STATUS_FAILURE;
        else if (fileDownloadSize == fileContentLength)
            return STATUS_SUCCESS;

        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
            //开始访问url并下载
            try {
                URL url = new URL(urlString);
                HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                connection.setRequestProperty("RANGE", "bytes=" + fileDownloadSize + "-");
                in = connection.getInputStream();

                randomAccessFile = new RandomAccessFile(file, "rw");
                randomAccessFile.seek(fileDownloadSize);

                byte[] b = new byte[BYTE_SIZE];
                int len;
                int total = 0;
                while ((len = in.read(b)) != -1) {
                    if (isPaused)
                        return STATUS_PAUSE;
                    else if (isCanceled)
                        return STATUS_CANCEL;

                    randomAccessFile.write(b, 0, len);
                    total += len;
                    int progress = (int) ((total + fileDownloadSize) * 100 / fileContentLength);
                    mDownloadListener.onProgress(progress);
                }
                connection.disconnect();
                return STATUS_SUCCESS;
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                try {
                    if (in != null)
                        in.close();
                    if (randomAccessFile != null)
                        randomAccessFile.close();
                    if (isCanceled)
                        file.delete();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        } else {
            //OkHttp需要Android5.0及以上
            OkHttpClient client = new OkHttpClient();
            Request request = new Request.Builder()
                    //断点下载，从上一次下载的位置开始
                    .addHeader("RANGE", "bytes=" + fileDownloadSize + "-")
                    .url(urlString)
                    .build();
            try {
                Response response = client.newCall(request).execute();
                in = response.body().byteStream();

                randomAccessFile = new RandomAccessFile(file, "rw");
                //从上一次位置开始下载
                randomAccessFile.seek(fileDownloadSize);
                byte[] b = new byte[BYTE_SIZE];
                int len;
                int total = 0;
                while ((len = in.read(b)) != -1) {

                    if (isPaused)
                        return STATUS_PAUSE;
                    else if (isCanceled)
                        return STATUS_CANCEL;

                    randomAccessFile.write(b, 0, len);
                    total += len;
                    int progress = (int) ((total + fileDownloadSize) * 100 / fileContentLength);
                    publishProgress(progress);
                }
                response.close();
                return STATUS_SUCCESS;
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                try {
                    if (in != null)
                        in.close();
                    if (randomAccessFile != null)
                        randomAccessFile.close();
                    if (isCanceled)
                        file.delete();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        return STATUS_FAILURE;
    }

    @Override
    protected void onProgressUpdate(Integer... values) {
        int progress = values[0];
        if (progress > lastProgress) {
            mDownloadListener.onProgress(progress);
            lastProgress = progress;
        }
    }

    @Override
    protected void onPostExecute(Integer status) {
        switch (status) {
            case STATUS_SUCCESS:
                mDownloadListener.onSuccess();
                break;
            case STATUS_FAILURE:
                mDownloadListener.onFailure();
                break;
            case STATUS_PAUSE:
                mDownloadListener.onPause();
                break;
            case STATUS_CANCEL:
                mDownloadListener.onCancel();
                break;
            default:
                throw new IllegalArgumentException("Unknown status");
        }
    }

    /**
     * @param urlString：需要下载的文件的url
     * @return 需要下载的文件的大小
     **/
    private long getUrlContentLength(String urlString) {
        long fileSize = 0;
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.LOLLIPOP) {
            try {
                URL url = new URL(urlString);
                HttpURLConnection connection = (HttpURLConnection) url.openConnection();
                fileSize = (long) connection.getContentLength();
                connection.disconnect();
            } catch (Exception e) {
                e.printStackTrace();
            }
        } else {
            //OkHttp需要Android5.0及以上
            OkHttpClient client = new OkHttpClient();
            Request request = new Request.Builder().url(urlString).build();
            try {
                Response response = client.newCall(request).execute();
                fileSize = response.body().contentLength();
                response.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return fileSize;
    }

    //DownloadTask提供的公共方法
    void pauseDownload() {
        isPaused = true;
    }

    void cancelDownload() {
        isCanceled = true;
    }

}

//创建前台服务
public class DownloadService extends Service {

    private String fileUrlString;
    private DownloadTask mDownloadTask;
    private DownloadBinder mBinder = new DownloadBinder();


    //实现DownloadCallback的五个抽象方法的具体功能，用以实例化DownloadTask
    private DownloadCallback downloadListener = new DownloadCallback() {
        @Override
        public void onProgress(int progress) {
            NotificationManagerCompat.from(getApplicationContext()).notify(ID_PROGRESS_NOTIFICATION, getNotification("Downloading...", progress));
        }

        @Override
        public void onSuccess() {
            mDownloadTask = null;
            //停止前台服务，同时移除通知
            stopForeground(true);
            //显示下载成功的通知
            NotificationManagerCompat.from(getApplicationContext()).notify(ID_PROGRESS_NOTIFICATION, getNotification("Download Success", PROGRESS_NONE));
        }

        @Override
        public void onFailure() {
            mDownloadTask = null;
            //停止前台服务，同时移除通知
            stopForeground(true);
            //显示下载失败的通知
            NotificationManagerCompat.from(getApplicationContext()).notify(ID_PROGRESS_NOTIFICATION, getNotification("Download Failed", PROGRESS_NONE));
        }

        @Override
        public void onPause() {
            mDownloadTask = null;
            NotificationManagerCompat.from(getApplicationContext()).notify(ID_PROGRESS_NOTIFICATION, getNotification("Download Paused", PROGRESS_NONE));
        }

        @Override
        public void onCancel() {
            mDownloadTask = null;
            //停止前台服务，同时移除通知
            stopForeground(true);
            NotificationManagerCompat.from(getApplicationContext()).notify(ID_PROGRESS_NOTIFICATION, getNotification("Download Canceled", PROGRESS_NONE));
        }
    };

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }

    private Notification getNotification(String title, int progress) {

        Intent intent = new Intent(getApplicationContext(), MainActivity.class);
        PendingIntent pendingIntent = PendingIntent.getActivity(getApplicationContext(), 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);

        Notification.Builder notification = new Notification.Builder(getApplicationContext())
                .setContentTitle(title)
                .setContentIntent(pendingIntent)
                .setSmallIcon(R.drawable.ic_file_download_24dp)
                .setLargeIcon(BitmapFactory.decodeResource(getResources(), R.drawable.ic_file_download_24dp))
                .setAutoCancel(true);
        if (progress >= 0) {
            notification.setContentText(progress + "%");
            notification.setProgress(PROGRESS_MAX, progress, false);
        }
        return notification.build();
    }

    //新建类继承自Binder，用以创建IBinder
    class DownloadBinder extends Binder {

        void startDownload(String urlString) {
            fileUrlString = urlString;
            if (mDownloadTask == null) {
                mDownloadTask = new DownloadTask(downloadListener);
                mDownloadTask.execute(fileUrlString);
                //启动前台服务
                startForeground(ID_PROGRESS_NOTIFICATION, getNotification("Downloading...", PROGRESS_START));
            }
        }

        void pauseDownload() {
            if (mDownloadTask != null)
                mDownloadTask.pauseDownload();
        }

        void cancelDownload() {
            if (mDownloadTask != null)
                mDownloadTask.cancelDownload();
            else if (TextUtils.isEmpty(fileUrlString)) {
                File file = new File(Environment.getExternalStoragePublicDirectory(Environment.DIRECTORY_DOWNLOADS).getPath() +
                        fileUrlString.substring(fileUrlString.lastIndexOf("/")));
                if (file.exists())
                    file.delete();
                NotificationManagerCompat.from(getApplicationContext()).cancel(ID_PROGRESS_NOTIFICATION);
                stopForeground(true);
            }
        }
    }
}

//创建界面布局
//xml布局资源文件
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="startDownload"
        android:text="@string/start_download"
        android:textAllCaps="false"/>

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="pauseDownload"
        android:text="@string/pause_download"
        android:textAllCaps="false"/>

    <Button
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:onClick="cancelDownload"
        android:text="@string/cancel_download"
        android:textAllCaps="false"/>

</LinearLayout>

//MainActivity需要申请权限，启动和绑定服务，以及处理按钮的点击事件
public class MainActivity extends AppCompatActivity {

    private DownloadBinder mBinder;
    private boolean mBound;
    //监控与服务的连接
    ServiceConnection connection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mBinder = (DownloadBinder) service;
            mBound = true;
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            mBound = false;
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //申请写SD卡的危险权限
        if (ContextCompat.checkSelfPermission(getApplicationContext(), Manifest.permission.WRITE_EXTERNAL_STORAGE)
                != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(MainActivity.this, new String[]{Manifest.permission.WRITE_EXTERNAL_STORAGE}, PermissionUtils.CODE_WRITE_EXTERNAL_STORAGE);
        }

        //启动和绑定服务
        Intent intent = new Intent(this, DownloadService.class);
        startService(intent);
        bindService(intent, connection, BIND_AUTO_CREATE);
    }

    //重写申请权限的回调方法onRequestPermissionsResult，根据申请结果处理事务
    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        switch (requestCode) {
            case PermissionUtils.CODE_WRITE_EXTERNAL_STORAGE:
                if (grantResults.length > 0 && grantResults[0] != PackageManager.PERMISSION_GRANTED) {
                    Toast.makeText(this, "You deny the write external storage permission", Toast.LENGTH_LONG).show();
                    finish();
                }
        }
    }


    //按钮点击事件
    //开始下载
    public void startDownload(View view) {
        if (mBinder != null)
            mBinder.startDownload("http://dldir1.qq.com/weixin/android/weixin708android1540.apk");
    }

    //暂停下载
    public void pauseDownload(View view) {
        if (mBinder != null) {
            mBinder.pauseDownload();
        }
    }

    //取消下载
    public void cancelDownload(View view) {
        if (mBinder != null) {
            mBinder.cancelDownload();
        }
    }


    @Override
    protected void onDestroy() {
        super.onDestroy();
        //解除绑定
        unbindService(connection);
    }
}


//最后不要忘记注册服务和申请权限
//注册服务
<service android:name=".DownloadService"
            android:exported="false"/>
//申请权限
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```
