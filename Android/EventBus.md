#EventBus
内部有三个poster
	
	private final HandlerPoster mainThreadPoster; //前台发送者
	private final BackgroundPoster backgroundPoster; //后台发送者
	private final AsyncPoster asyncPoster;   //后台发送者(只让队列第一个待订阅者去响应)
	
每个poster内部都有一个PendingPostQueue，维护了元素入队列和出队列的方法

	class PendingPostQueue {
    	private PendingPost head;	//待发送对象队列头节点
   	 	private PendingPost tail;	//待发送对象队列尾节点
   	 	synchronized void enqueue(PendingPost pendingPost);
   	 	poll();
   	 }
   	 
 PendingPost  	 
   	 
 	final class PendingPost {
   		private final static List<PendingPost> pendingPostPool = new ArrayList<PendingPost>();//单例池，增加对象的复用性，每次用的时候先从这里取，没有的话再new

    	Object event;		//事件类型
    	Subscription subscription;	//订阅者
    	PendingPost next;	//队列下一个待发送对象
    }	
    
    
HandlerPoster继承Handler，在HanlderPoster中 enqueue方法，生成一个pendingPost并入队，根据handlerActive参数判断，如果Handler不在运行中，则发送一条空消息让handler响应，handleMessage方法中用一个循环不断从队列中取出消息，调用eventBus.invokeSubscriber(pendingPost)分发消息，并回收这个pendingPost，直到队列为空，把handlerActive参数置为false，handleMessage方法结束。

	enqueue(Subscription subscription, Object event) {
     	  PendingPost pendingPost = PendingPost.obtainPendingPost(subscription, event);
      	  synchronized (this) {
          	  queue.enqueue(pendingPost);
          	  if (!handlerActive) {
           	  	handlerActive = true;
               	if (!sendMessage(obtainMessage())) {
            	   	 	throw new EventBusException("Could not send handler message");
                }
            }
        }
    }


BackgroundPoster和AsyncPoster实现Runnable，与HanlderPoster类似，不过它们的enqueue方法是让eventbus的executorService执行自己，在run方法中取出pendingPost，不同的是BackgroundPoster会将队列的所有元素取出分发，而AsyncPoster只会取出队列的头元素分发。



调用register方法注册时

1. 首先会查找传入的Object，再SubscriberMethodFinder类中运用反射先获取到Object的所有方法，然后找以onEvent开头的方法，再跳过那些不是public，是static或者abstract的方法，再判断是不是只有一个参数的方法，再看onEvent后面有没有其他字符，后面可以有MainThread、BackgroundThread、Async修饰符，找出这些方法后，放在一个list中，然后以class的名字为key，list为value放在一个map中，方便同一个Object注册后能直接从map中取出对象。
2. 在EventBus类中，有一个Map<Class<?>, CopyOnWriteArrayList<Subscription>> subscriptionsByEventType对象，找出所有的onEvent方法后，对这些方法遍历，以方法参数的类型为key，在subscriptionsByEventType中找到所有这个参数的订阅者，把当前这个方法按照优先级生成Subscription放入CopyOnWriteArrayList中。并且以subscriber为key，所有订阅的eventType集合为value，也放入一个map中(Map<Object, List<Class<?>>> typesBySubscriber typesBySubscriber)


调用post方法
EventBus中又一个ThreadLocal对象，存储PostingThreadState对象，post方法中获取到这个对象，这个对象封装了一次事件发送过程中所需数据，在其中有一个list存储了所有post方法传入的参数，在这里进行循环，每次取出一个参数，在subscriptionsByEventType中找到订阅它的方法，执行该方法。


unregister方法
根据typesBySubscriber找到当前subscriber包含的所有eventType，然后在subscriptionsByEventType中找eventType对应的Subscription，如果是当前subscriber，删除。