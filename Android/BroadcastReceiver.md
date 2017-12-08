#BroadcastReceiver
<p>分两个角色：发送者和接收者 

使用观察者模式实现 基于消息的发布／订阅事件模型</p>
实现自定义BroadcastReceiver
#####1.继承BroadcastReceiver类
#####2.复写抽象方法onReceive()方法
onReceive方法运行在主线程，所以不能进行耗时操作
注册方法有两种，静态注册和动态注册
#####静态注册(不受任何组件生命周期的影响)：
在AndroidManifest文件中用<receiver>标签声明 <p>
在该标签下通过<intent-filter>标签中的 <action>标签确定接受什么action的广播<p>
当此App首次启动时，系统会自动实例化mBroadcastReceiver类，并注册到系统中。当应用关闭后，如果有广播信息传来Receiver也会被系统调用而自动运行
#####动态注册(跟随组件生命周期变化)：
在代码中通过调用Context方法的registerReceiver(BroadcastReceiver receiver, IntentFilter filter)方法注册
但要在相应的位置销毁广播，否则会导致内存泄漏
广播的类型：
######1.普通广播（Normal Broadcast）开发者自己定义的intent广播
<pre>
Intent intent = new Intent();
//对应BroadcastReceiver中intentFilter的action
intent.setAction(BROADCAST_ACTION);
</pre>
//发送广播
sendBroadcast(intent);
######2.系统广播（System Broadcast）系统自动发送
######3.有序广播（Ordered Broadcast）发送出去的广播被广播接受者按照先后顺序接收 有序是针对广播接收者而言的
优先级通过IntentFilter指定，数值越大，优先级越高
<intent-filter android:priority="998">  
  <action android:name="android.intent.action.MY_BROADCAST"/>  
    <category android:name="android.intent.category.DEFAULT" />    </intent-filter>
有序广播的使用过程与普通广播非常类似，差异仅在于广播的发送方式：
sendOrderedBroadcast(Intent intent, String permission);
如果为null则表示不要求接收者声明指定的权限，如果不为null，则表示接收者若要接收此广播，需声明指定权限。
可以在优先级高的Receiver中调用abortBroadcast()终止传递
######4.粘性广播（Sticky Broadcast）
在Android5.0 & API 21中已经失效
######5.App应用内广播（Local Broadcast）
* 具体使用1 - 将全局广播设置成局部广播
    1. 注册广播时将exported属性设置为false，使得非本App内部发出的此广播不被接收；
    2. 在广播发送和接收时，增设相应权限permission，用于权限验证；
    3. 发送广播时指定该广播接收器所在的包名，此广播将只会发送到此包中的App内与之相匹配的有效广播接收器中。
通过intent.setPackage(packageName)指定包名
* 具体使用2 - 使用封装好的LocalBroadcastManager类
使用方式上与全局广播几乎相同，只是注册/取消注册广播接收器和发送广播时将参数的context变成了LocalBroadcastManager的单一实例
注：对于LocalBroadcastManager方式发送的应用内广播，只能通过LocalBroadcastManager动态注册，不能静态注册
<pre>
//注册应用内广播接收器 
//步骤1：实例化BroadcastReceiver子类 & IntentFilter mBroadcastReceiver 
mBroadcastReceiver = new mBroadcastReceiver(); 
IntentFilter intentFilter = new IntentFilter();
 //步骤2：实例化LocalBroadcastManager的实例
 localBroadcastManager = LocalBroadcastManager.getInstance(this);
 //步骤3：设置接收广播的类型 
intentFilter.addAction(android.net.conn.CONNECTIVITY_CHANGE); 
//步骤4：调用LocalBroadcastManager单一实例的registerReceiver（）方法进行动态注册 
localBroadcastManager.registerReceiver(mBroadcastReceiver, intentFilter);
 //取消注册应用内广播接收器
 localBroadcastManager.unregisterReceiver(mBroadcastReceiver); 
//发送应用内广播 
Intent intent = new Intent(); 
intent.setAction(BROADCAST_ACTION); 
localBroadcastManager.sendBroadcast(intent);
</pre>


