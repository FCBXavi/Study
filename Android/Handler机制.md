#Handler机制
<a href = "http://blog.csdn.net/lmj623565791/article/details/38377229" >http://blog.csdn.net/lmj623565791/article/details/38377229 </a>
###1、首先Looper.prepare()在本线程中保存一个Looper实例，然后该实例中保存一个MessageQueue对象；因为Looper.prepare()在一个线程中只能调用一次，所以MessageQueue在一个线程中只会存在一个。
###2、Looper.loop()会让当前线程进入一个无限循环，不端从MessageQueue的实例中读取消息，然后回调msg.target.dispatchMessage(msg)方法。
###3、Handler的构造方法，会首先得到当前线程中保存的Looper实例，进而与Looper实例中的MessageQueue想关联。
###4、Handler的sendMessage方法，会给msg的target赋值为handler自身，然后加入MessageQueue中。
###5、在构造Handler实例时，我们会重写handleMessage方法，也就是msg.target.dispatchMessage(msg)最终调用的方法。


Handler的post(Runnable r)方法，实际上是生成一个Message对象，将Message对象的callback属性赋值为r，然后放入消息队列，从消息队列中取出message后，如果message的callback不为空，直接执行callback的run方法。


##Handler导致的内存泄漏
当一个Handler在主线程进行了初始化之后，我们发送一个target为这个Handler的消息到Looper处理的消息队列时，实际上已经发送的消息已经包含了一个Handler实例的引用，只有这样Looper在处理到这条消息时才可以调用Handler#handleMessage(Message)完成消息的正确处理。在Java中，非静态的内部类和匿名内部类都会隐式地持有其外部类的引用，Handler持有Activity的引用，导致内存泄漏。   
要解决这种问题，思路就是避免使用非静态内部类，继承Handler时，要么是放在单独的类文件中，要么就是使用静态内部类。因为静态的内部类不会持有外部类的引用，所以不会导致外部类实例的内存泄露。当你需要在静态内部类中调用外部的Activity时，我们可以使用弱引用来处理。另外关于同样也需要将Runnable设置为静态的成员属性。   
注意：一个静态的匿名内部类实例不会持有外部类的引用。 修改后不会导致内存泄露的代码如下

<pre>
public class SampleActivity extends Activity {
  /**
   * Instances of static inner classes do not hold an implicit
   * reference to their outer class.
   */
  private static class MyHandler extends Handler {
    private final WeakReference<SampleActivity> mActivity;
    public MyHandler(SampleActivity activity) {
      mActivity = new WeakReference<SampleActivity>(activity);
    }
    @Override
    public void handleMessage(Message msg) {
      SampleActivity activity = mActivity.get();
      if (activity != null) {
        // ...
      }
    }
  }
  private final MyHandler mHandler = new MyHandler(this);
  /**
   * Instances of anonymous classes do not hold an implicit
   * reference to their outer class when they are "static".
   */
  private static final Runnable sRunnable = new Runnable() {
      @Override
      public void run() { /* ... */ }
  };
  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    // Post a message and delay its execution for 10 minutes.
    mHandler.postDelayed(sRunnable, 1000 * 60 * 10);
    // Go back to the previous Activity.
    finish();
  }
}
</pre>

其实在Android中很多的内存泄露都是由于在Activity中使用了非静态内部类导致的，就像本文提到的一样，所以当我们使用时要非静态内部类时要格外注意，如果其实例的持有对象的生命周期大于其外部类对象，那么就有可能导致内存泄露。个人倾向于使用文章的静态类和弱引用的方法解决这种问题。


##HandlerThread
* HandlerThread本质上是一个线程类，它继承了Thread；
* HandlerThread有自己的内部Looper对象，可以进行looper循环；
* 通过获取HandlerThread的looper对象传递给Handler对象，可以在handleMessage方法中执行异步任务。
* 创建HandlerThread后必须先调用HandlerThread.start()方法，Thread会先调用run方法，创建Looper对象。