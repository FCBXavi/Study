okHttp
===============================
	private void getDataAsync() {
    	OkHttpClient client = new OkHttpClient();
    	Request request = new Request.Builder()
            	.url("http://www.baidu.com")
            	.build();
    	client.newCall(request).enqueue(new Callback() {
        	@Override
        	public void onFailure(Call call, IOException e) {
        	}
        	@Override
        	public void onResponse(Call call, Response response) throws IOException {
            	if(response.isSuccessful()){//回调的方法执行在子线程。
                	Log.d("kwwl","获取数据成功了");
                	Log.d("kwwl","response.code()=="+response.code());
                	Log.d("kwwl","response.body().string()=="+response.body().string());
            	}
        	}
    	});
	}	
	
Request        
包含一个URL、一个方法（GET或POST）、一些Http头，还kennel包含一个特定内容类型的数据类的主体部分。 

	public final class Request {
		final HttpUrl url;
		final String method;
		final Headers headers;
		final @Nullable RequestBody body;
		final Object tag;    
		} 
  		
  		
  		  
Response       
对请求的恢复，包含状态码，Http头和主体部分。      
	
	public final class Response implements Closeable {		final Request request;
		final Protocol protocol;
		final int code;
		final String message;
		final @Nullable Handshake handshake;
		final Headers headers;
		final @Nullable ResponseBody body;
		final @Nullable Response networkResponse;
		final @Nullable Response cacheResponse;
		final @Nullable Response priorResponse;
		final long sentRequestAtMillis;
		final long receivedResponseAtMillis;
		}



Call       
okHttp抽象出的一个满足请求的模型，尽管中间可能会有多个请求或响应，执行Call有两种方式，同步或者异步。     
在OkHttpClient的newCall方法中，返回一个RealCall对象，它是Call接口的实现类

	 @Override public Call newCall(Request request) {
    		return new RealCall(this, request, false /* for web socket */);
    	}
    	
RealCall的enqueue方法     
  
	@Override public void enqueue(Callback responseCallback) {
		//call只能被执行一次
    	synchronized (this) {
      		if (executed) throw new IllegalStateException("Already Executed");
      		executed = true;
    	}
    	captureCallStackTrace();
    	//client是创建call的OkHttpClient
    	//dispatcher()返回一个Dispatcher对象。
    	client.dispatcher().enqueue(new AsyncCall(responseCallback));
    }


Dispatcher       
这个类中维护着一个线程池和三个队列       
	
	public final class Dispatcher {
		private int maxRequests = 64;
		private int maxRequestsPerHost = 5;
		private @Nullable Runnable idleCallback;
		/** Executes calls. Created lazily. */
		//线程池
		private @Nullable ExecutorService executorService;
		/** Ready async calls in the order they'll be run. */
		//准备执行的异步请求 AsyncCall本质上是一个Runnable
		private final Deque<AsyncCall> readyAsyncCalls = new ArrayDeque<>();
		/** Running asynchronous calls. Includes canceled calls that haven't finished yet. */
		//正在运行的异步请求
		private final Deque<AsyncCall> runningAsyncCalls = new ArrayDeque<>();
		/** Running synchronous calls. Includes canceled calls that haven't finished yet. */
		//正在运行的同步请求
		private final Deque<RealCall> runningSyncCalls = new ArrayDeque<>();
		
		public synchronized ExecutorService executorService() {
			if (executorService == null) {
			//核心线程池数量是0，最大线程池数量为int最大值，线程最长存活时间为60s
				executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          	new SynchronousQueue<Runnable>(), Util.threadFactory("OkHttp Dispatcher", false));
    		}
    		return executorService;
    	}
    	
    	
    	synchronized void enqueue(AsyncCall call) {
    	//判断正在执行的异步请求的队列大小，如果没有超过最大值，就把请求放入正在执行的队列，然后让线程池运行这个请求,否则加入等待队列。
    		if (runningAsyncCalls.size() < maxRequests && runningCallsForHost(call) < maxRequestsPerHost) {
      			runningAsyncCalls.add(call);
      			executorService().execute(call);
    		} else {
      			readyAsyncCalls.add(call);
    		}
    	}	
    }


AsyncCall         
本质上是一个Runnable   
     
	final class AsyncCall extends NamedRunnable {
	//NamedRunnable是一个实现了Runnable的抽象类，在run方法中执行了自己的抽象方法execute
    	private final Callback responseCallback;

    	AsyncCall(Callback responseCallback) {
      		super("OkHttp %s", redactedUrl());
      		this.responseCallback = responseCallback;
    	}

    	String host() {
      		return originalRequest.url().host();
      	}

    	Request request() {
      		return originalRequest;
    	}

    	RealCall get() {
      		return RealCall.this;
    	}

    	@Override protected void execute() {
      		boolean signalledCallback = false;
      		try {
        		Response response = getResponseWithInterceptorChain();
        		if (retryAndFollowUpInterceptor.isCanceled()) {
          		signalledCallback = true;
          		//这里执行失败的回调
          		responseCallback.onFailure(RealCall.this, new IOException("Canceled"));
        		} else {
          		signalledCallback = true;
          		//这里执行成功的回调
          		responseCallback.onResponse(RealCall.this, response);
        		}
      		} catch (IOException e) {
        		if (signalledCallback) {
          		// Do not signal the callback twice!
          		Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        		} else {
          		responseCallback.onFailure(RealCall.this, e);
        		}
      		} finally {
        		client.dispatcher().finished(this);
      		}
    	}
    }