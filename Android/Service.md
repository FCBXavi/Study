#Service  
####启动的两种方式 startService和bindService   
分别会调用Service的 **onStartCommand**方法和**onBind**方法   
对应的关闭的两种方式 **stopService**和**unBindService**      
**startService**无法让**activity**和**Service**交互，但**bindService**可以，
**bindService(Intent service, ServiceConnection conn, int flags)**
**ServiceConnection**是一个接口，自己实现，实现**onServiceConnect(ComponentName name, IBinder service)**方法，这里的**service**参数就是**Service**中**onBind**方法返回的**IBinder**对象，然后可以和**Service**进行交互
一个**Service**必须要在既没有和任何**Activity**关联又处于停止状态的时候才会被销毁    
Service运行在**主线程**，不可以进行耗时操作，处理耗时操作需要在Service中**新建子线程**

IntentService 处理异步请求
实现步骤：  
步骤1：定义IntentService的子类：传入线程名称、复写onHandleIntent()方法  
步骤2：在Manifest.xml中注册服务   
步骤3：在Activity中开启Service服务   

本质是使用Handler&HanglerThread实现的  
通过HandlerThread单独开启一个名为IntentService的线程，创建一个名叫ServiceHandler的内部Handler   
把内部Handler与HandlerThread所对应的子线程进行绑定  
通过onStartCommand()传递给服务intent，依次插入到工作队列中，并逐个发送给onHandleIntent()  
通过onHandleIntent()来依次处理所有Intent请求对象所对应的任务     
因此我们通过复写方法onHandleIntent()，再在里面根据Intent的不同进行不同的线程操作就可以了    
多次调用startService(Intent intent)方法(也就是多次调用onStartCommand方法)时,会将消息加入到消息队列中执行，因此事件是按照顺序执行的    
与后台线程相比，IntentService是一种后台服务，优势是：优先级高（不容易被系统杀死），从而保证任务的执行


[ Android中的Service：Binder，Messenger，AIDL](http://blog.csdn.net/luoyanglizi/article/details/51594016)