---
layout: post
title: "《第一行代码》Chapter 09"
subtitle: '网络技术'
author: "Jamke"
header-style: text
tags:
  - Android Foundation
  - AF
---

## 目录

1.[WebView](#webview)

2.[使用HTTP协议访问网络](#使用http协议访问网络)

3.[使用OkHttp访问网络](#使用okhttp访问网络)

4.[解析XML格式的数据](#解析xml格式的数据)

5.[解析JSON格式的数据](#解析json格式的数据)

6.[java回调机制](#java回调机制)

## 正文

---
#### WebView

> WebView，用于在APP内展示网页

```
//xml布局文件中，通过<WebView>标签创建一个WebView
<WebView
    android:id="@+id/web_view"
    android:layout_width="match_parent"
    android:layout_height="match_parent"/>

//活动中，得到WebView的引用，然后设置WebView
WebView webView=findViewById(R.id.web_view);
webView.getSetting.setJavaScriptEnabled(true);//支持JavaScript脚本
webView.setWbeViewClient(new WebViewClient());//当从一个网页跳转到另一个网页时，网页依旧在WebView中显示，而不是打开浏览器
webView.loadUrl("https://github.com")//需要打开的网页的URL

//manifest文件中，申请访问网络的权限
<manifest>
    ...
    <uses-permission android:name="android.permission.INTERNET"/>
    ...
</manifest>
```

---
#### 使用HTTP协议访问网络

> http协议的极简~~不靠谱~~流程：客户端发送请求，服务器收到请求后返回数据，客户端收到服务响应，客户端解析返回的数据

> Android中有两个发送http请求的方式：HttpURLConnection和HttpClient，其中HttpClient由于API过多、扩展困难等缺点，已经在Android6.0（API23）弃用。

```
private TextView mUrlResultTextView;

@Override
protected void onCreate(@Nullable Bundle savedInstanceState) {
    ...
    mUrlResultTextView=findViewById(R.id.url_result);
    ...
}

//开启子线程访问网络
//由于Android的用户界面是单线程，负责从各种传感器获取事件，并绘制要设置的下一帧，假如以60帧/秒的速度运行，那么需要确保每一帧绘制的时间要小于17毫秒。换句话说，要尽可能少地在主线程上执行任务，但是联网一般需要几秒，如果在主线程上调用网络连接，用户界面就会被冻结（无法交互），冻结5秒后，Android会提示用户关闭应用。所以需要在子线程（也可以叫辅助执行线程）中调用网络连接。
new Thread(new Runnable(){
    @Override
    public void run(){
        //构建URL
        URL url=new URL("https://github.com");
        //创建URL连接对象，还没有连接网络，这一步可以添加请求方法、连接属性、头字段等
        HttpURLConnection urlConnection=(HttpURLConnection)url.openConnection();
        //urlConnection.setRequestMethod("GET");
        //urlConnection.setConnectTimeout("8000");//设置连接超时时间8000毫秒
        //urlConnection.setReadTimeout("8000");//设置读取超时时间8000毫秒
        try{
            //从URL连接对象获取输入流
            InputStream in=urlConnection.getInputStream();
            //使用扫描器读取输入流
            Scanner scanner=new Scanner(in);
            //将分隔符设置为\A，即从输入流的起点开始分隔
            scanner.useDelimiter("\\A");
            //若扫描器有缓存的输入流，取出
            if(scanner.hasNext())
                result=scanner.next();
            else
                result=null;
            //在主线程显示结果
            showResult(result);
        }catch(IOException e){
            e.printStackTrace();
        }finally{
            //最后，释放URL连接对象
            urlConnection.disconnect();
        }
    }
}).start();

//子线程执行完，通过调用runOnUiThread()方法可以将线程切换到主线程，然后更新主线程UI
private void showResult(String result){
    runOnUiThread(new Runnable(){
       public void run(){
           mUrlResultTextView.setText(result);
       }
    });
}

//manifest文件中，申请访问网络的权限
<manifest>
    ...
    <uses-permission android:name="android.permission.INTERNET"/>
    ...
</manifest>
```

---
> [更多读取输入流的方法以及各方法的优缺点](http://stackoverflow.com/questions/309424/read-convert-an-inputstream-to-a-string%EF%BC%8C%E8%AF%A6%E7%BB%86%E4%BA%86%E8%A7%A3%E5%AE%8C%E6%88%90%E8%BF%99%E4%B8%80%E6%93%8D%E4%BD%9C%E7%9A%84%E4%B8%8D%E5%90%8C%E6%96%B9%E6%B3%95%E3%80%82)

> 若想发送数据给服务器，可以在通过URL连接对象获取输入流之前，设置请求方法和要发送的数据

```
    ...
    //设置请求方法为POST（发送数据给服务器）
    urlConnection.setRequestMethod("POST");
    //设置发送的数据内容
    DataOutputStream out=new DataOutputStream(urlConnection.getOutputStream());
    out.writeByte("userName=admin&password=1234");
    ...
```

---
#### 使用OkHttp访问网络

- [OkHttp](https://github.com/square/okhttp)，在众多Android开源的网络通信库中，是比较出色的一个。

```
//添加依赖库
implementation "com.squareup.okhttp3:okhttp:4.2.1"

//从服务器获取数据
OkHttpClient client=new OkHttpClient();
new Thread(new Runnable(){
    @Override
    public void run(){
        Request request=new Request.Builder()
            .url("https://github.com")
            .build();
        try{
            result=client.newCall(request).execute();
            showResult(result);
        }
    }
}).start();

//发送数据给服务器，区别在于，需要构建一个RequestBody，添加需要发送的数据，然后传入request的post方法，发送给服务器
OkHttpClient client=new OkHttpClient();
new Thread(new Runnable(){
    @Override
    public void run(){
        RequestBody requestBody=new FormBody.Builder()
            .add("userName","admin")
            .add("password","1234")
            .build();
        Request request=new Request.Builder()
            .url("https://github.com")
            .post(requestBody)
            .build();
        try{
            result=client.newCall(request).execute();
            showResult(result);
        }
    }
}).start();
```

---
#### 解析XML格式的数据

> 解析XML常用的方法有两种：Pull解析和SAX解析

- XML数据

```
//假设要解析的XML数据内容如下

 <apps>
    <app>
        <id>1</id>
        <name>google map</name>
        <version>1.0</version>
    </app>
    <app>
        <id>2</id>
        <name>chrome</name>
        <version>2.1</version>
    </app>
    <app>
        <id>3</id>
        <name>google play</name>
        <version>2.3</version>
    </app>
</apps>

```

- Pull解析

```

private void parseXmlWithPull(String xml){
    //获得XmlPullParserFactory实例
    XmlPullParserFactory factory=XmlPullParserFactory.newInstance();
    //通过XmlPullParserFactory获得XmlPullParser
    XmlPullParser xmlPullParser=factory.newPullParser();
    //设置要解析的数据
    xmlPullParser.setInput(new StringReader(xml));
    //每个节点内的数据
    String id="";
    String name="";
    String version="";
    //获得当前解析事件
    int eventType=xmlPullParser.getEvemtType();
    while(eventtype!=XmlPullParser.END_DOCUMENT){
        //获取当前节点名
        String nodeName=xmlPullParser.getName();
        switch(eventType){
            //开始解析某个节点
            case XmlPullParser.START_TAG:
                if("id".equals(nodeName))
                    id=xmlPullParser.nextText();
                else if("name".equals(nodeName))
                    name=xmlPullParser.nextText();
                else if("version".equals(nodeName))
                    version=xmlPullParser.nextText();
                break;
            //完成解析某个节点
            case XmlPullParser.END_TAG:
                break;
            default:
                break;
        }
        //下一个节点
        eventType=xmlPullParser.next();
    }
}
```

- SAX解析

```
//新建class继承自DefaultHandler，构建SAX解析需要的Handler
public class MySAXHandler extends DefaultHandler{
    private String nodeName;
    private StringBuilder id;
    private StringBuilder name;
    private StringBuilder version;
    
    //开始解析XML
    @Override
    public void startDocument() throws SAXException {
        id=new StringBuilder();
        name=new StringBuilder();
        version=new StringBuilder();
    }

    //开始解析某个节点
    @Override
    public void startElement(String uri, String localName, String qName, Attributes attributes) throws SAXException {
        //保存节点名
        nodeNmae=localName;
    }

    //解析节点内容
    @Override
    public void characters(char[] ch, int start, int length) throws SAXException {
        if("id".equals(nodeNmae))
            id.append(ch,start,length);
        else if("name".equals(nodeName))
            name.append(ch,start,length);
        else if("version".equals(nodeName))
            version.append(ch,start,length);
    }

    //完成解析某个节点
    @Override
    public void endElement(String uri, String localName, String qName) throws SAXException {
        if("app".equals(nodeName)){
            //将StringBuilder清空，以便处理下一个节点
            id.setLength(0);
            name.setLength(0);
            version.setLength(0);
        }
    }

    //完成解析XML
    @Override
    public void endDocument() throws SAXException {
        super.endDocument();
    }
}

//使用SAX方法解析XML
private void parseXMLWithSAX(String xml){
    try{
        //获得SAXParserFactory实例
        SAXParserFactory factory=SAXParserFactory.newInstance();
        //通过SAXParserFactory获得XMLReader对象
        XMLReader xmlReader=factory.newSAXParser().getXMLReader();
        //通过XMLReader设置ContentHandler
        xmlReader.setContentHandler(new MySAXHandler());
        //xml数据传入xmlReader的parse方法，开始解析
        xmlReader.parse(new InputSource(new StringReader(xml)));
    }catch(Exception e){
        e.printStackTrace();
    }
}
```

---
> 除了上述两种方法，还有一种常用的方法：DOM解析

---
#### 解析JSON格式的数据

> 解析JSON的常用方法：JSONObject、GSON、Jackson、FastJSON等

- JSON数据

```
//假设要解析的JSON数据内容如下
{
  "data": {
    "address": "广东省 广州市",
    "forecasts": [
      {
        "date": "2019-11-04",
        "dayOfWeek": "1",
        "dayWeather": "晴",
        "nightWeather": "晴",
        "dayTemp": "29℃",
        "nightTemp": "19℃",
        "dayWindDirection": "北",
        "nightWindDirection": "北",
        "dayWindPower": "4级",
        "nightWindPower": "4级"
      }
    ]}
}
```

- JSONObject

```
private void parseJSONWithJSONObject(String json){
    private JSONObject forecastJson=new JSONObject(json);
    private JSONObject data=forecastJson.getJSONObject("data");
    private String address=data.getString("address");
    private JSONArray forecast=data.getJSONArray("forecast");
    
    private String date=forecast.getString("date");
    private String dayOfWeek=forecast.getString("dayOfWeek");
    private String dayCondition=forecast.getString("dayWeather");
    private String nightCondition=forecast.getString("nightWeather");
    private String dayTemp=forecast.getString("dayTemp");
    private String nightTemp=forecast.getString("nightTemp");
    private String dayWindDirection=forecast.getString("dayWindDirection");
    private String nightWindDirection=forecast.getString("nightWindDirection");
    private String dayWindPower=forecast.getString("dayWindPower");
    private String nightWindPower=forecast.getString("nightWindPower");
}
```

- [GSON](https://github.com/google/gson)
> GSON将一段JSON格式的字符串自动映射成一个对象，从而不需要再手动编写代码一个个解析
```
//添加依赖库
dependencies {
  implementation 'com.google.code.gson:gson:2.8.6'
}

//假设要解析的JSON数据如下
"forecasts": [
  {
    "date": "2019-11-04",
    "dayOfWeek": "1",
    "dayWeather": "晴",
    "nightWeather": "晴",
    "dayTemp": "29℃",
    "nightTemp": "19℃",
    "dayWindDirection": "北",
    "nightWindDirection": "北",
    "dayWindPower": "4级",
    "nightWindPower": "4级"
  },
  {
    "date": "2019-11-05",
    "dayOfWeek": "2",
    "dayWeather": "晴",
    "nightWeather": "晴",
    "dayTemp": "30℃",
    "nightTemp": "17℃",
    "dayWindDirection": "无风向",
    "nightWindDirection": "无风向",
    "dayWindPower": "≤3级",
    "nightWindPower": "≤3级"
  }
]

//解析JSON数组，需要借助TypeToken解析，创建class（类似java bean）以获得TypeToken
public class Forecast{
    private String date;
    private String dayOfWeek;
    private String dayWeather;
    private String nightWeather;
    private String dayTemp;
    private String nightTemp;
    private String dayWindDirection;
    private String nightWindDirection;
    private String dayWindPower;
    private String nightWindPower;

    public String getDate() {
        return date;
    }

    public void setDate(String date) {
        this.date = date;
    }

    public String getDayOfWeek() {
        return dayOfWeek;
    }

    public void setDayOfWeek(String dayOfWeek) {
        this.dayOfWeek = dayOfWeek;
    }

    public String getDayWeather() {
        return dayWeather;
    }

    public void setDayWeather(String dayWeather) {
        this.dayWeather = dayWeather;
    }

    public String getNightWeather() {
        return nightWeather;
    }

    public void setNightWeather(String nightWeather) {
        this.nightWeather = nightWeather;
    }

    public String getDayTemp() {
        return dayTemp;
    }

    public void setDayTemp(String dayTemp) {
        this.dayTemp = dayTemp;
    }

    public String getNightTemp() {
        return nightTemp;
    }

    public void setNightTemp(String nightTemp) {
        this.nightTemp = nightTemp;
    }

    public String getDayWindDirection() {
        return dayWindDirection;
    }

    public void setDayWindDirection(String dayWindDirection) {
        this.dayWindDirection = dayWindDirection;
    }

    public String getNightWindDirection() {
        return nightWindDirection;
    }

    public void setNightWindDirection(String nightWindDirection) {
        this.nightWindDirection = nightWindDirection;
    }

    public String getDayWindPower() {
        return dayWindPower;
    }

    public void setDayWindPower(String dayWindPower) {
        this.dayWindPower = dayWindPower;
    }

    public String getNightWindPower() {
        return nightWindPower;
    }

    public void setNightWindPower(String nightWindPower) {
        this.nightWindPower = nightWindPower;
    }
}

//借助GSON解析
private void parseJSONWithGson(String json){
    GSON gson=new GSON();
    List<Forecast> data=gson.fromJson(json, new TypeToken<List<Forecast>>(){}.getType());
    for(Forecast forecast:data){
        //do something yourself
    }
}
```

---
- [关于GSON的更多用法](https://github.com/google/gson/blob/master/UserGuide.md)

---
#### java回调机制
- 应用场景：其他线程需要获得该线程的数据，如：

> 子线程调用网络连接得到数据，主线程可以通过回调机制获取数据；实现recyclerView的点击事件等。

- 将连接网络并返回响应的数据的功能封装成NetworkUtils类

```
public class NetworkUtils{
    
    //该函数内部没有开启子线程执行网络连接，所以该函数不能放在主线程中直接执行，不然会阻塞主线程，造成APP崩溃
    public static String getResponseFromHttp(String urlStr){
        URL url=new URL(urlStr);
        HttpURLConnection urlConnection=(HttpURLConnection)url.openConnection();
        try{
            InputStream in=urlConnection.getInputStream();
            Scanner scanner=new Scanner(in);
            scanner.useDelimiter("\\A");
            if(scanner.hasNext())
                return scanner.next();
            else
                return null;
        }catch(IOException e){
            e.printStackTrace();
        }
    }
    
    //如果开启子线程执行网络连接，那么就需要考虑如何返回数据给调用方。
    //因为调用方调用函数时，在函数返回数据之前，函数已经执行结束，
    //此时调用方在另一个线程，而该函数在当前子线程中，在函数执行结束的情况下，无法直接通过return语句返回数据给调用方。这时回调机制即派上用场
    
    //首先定义接口，接口中的onFinish()方法表示当服务器返回数据成功时调用，onError()方法表示当服务器返回数据失败时调用
    public interface HttpCallbackListener{
        void onFinish(String result);
        void onError(Exception e);
    }
    public static void asyncGetResponseFromHttp(final String urlStr, final HttpCallbackListener litener){
        new Thread(new Runnable(){
            URL url=new URL(urlStr);
            HttpURLConnection urlConnection=(HttpURLConnection)url.openConnection();
            try{
                InputStream in=urlConnection.getInputStream();
                Scanner scanner=new Scanner(in);
                scanner.useDelimiter("\\A");
                if(scanner.hasNext())
                    listener.onFinish(scanner.next());
                else
                    listener.onFinish(null);
            }catch(IOException e){
                listener.onError(e.printStackTrace());
            }
        }).start();
    }
    
    //OkHttp，自带了回调机制
    public static void getResponseFromOkHttp(String urlStr, okhttp3.Callback callback){
        OkHttpClient client=new OkHttpClient();
        Request request=new Request().Builder()
                .url(urlStr)
                .build();
        //与之前不同，这里调用enqueue()方法，而不是execute()方法，因为enqueue()方法会开启子线程，并将结果回调给okhttp3.Callback中
        client.newCall(request).enqueue(callback);
    }
}
```

```
//调用asyncGetResponseFromHttp
NetworUtils.asyncGetResponseFromHttp("https://github.com",new HttpCallbackListener(){
    @Override
    public void onFinish(String result){
        //对返回数据处理
    }
    
    @Override
    public void onError(Exception e){
        //对异常情况处理
    }
}); 

//调用getResponseFromOkHttp
NetworkUils.getResponseFromOkHttp("https://github.com",new okhttp3.Callback(){
    @Override
    public void onResponse(Call call, Response response) throws IOException{
        //对返回数据处理
        String data=response.body().string();
    }
    
    @Override
    public void onFailure(Call call, IOException e){
        //对异常情况处理
    }
});
```
