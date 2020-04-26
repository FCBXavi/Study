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
	
	public final class Response implements Closeable {
		final Request request;
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
    
看到AsyncCall在执行完网络请求后会在finally代码块中执行client.dispatcher().finished(this) 看一下这段代码           

	 void finished(AsyncCall call) {
    	finished(runningAsyncCalls, call, true);
    }
    //调用了以下方法
    private <T> void finished(Deque<T> calls, T call, boolean promoteCalls) {
    	int runningCallsCount;
    	Runnable idleCallback;
    	synchronized (this) {
    		//如果这个Call不在正在执行的Call队列中，抛出异常，否则从队列中移除这个Call，然后执行promoteCalls()方法，promoteCalls()方法在下面
      		if (!calls.remove(call)) throw new AssertionError("Call wasn't in-flight!");
      		if (promoteCalls) promoteCalls();
      		runningCallsCount = runningCallsCount();
      		idleCallback = this.idleCallback;
    	}
		//当dispatcher空闲，即执行请求队列为空时执行的回调，可以在代码中设置这个回调
    	if (runningCallsCount == 0 && idleCallback != null) {
      			idleCallback.run();
    		}
    }
    
    //这个方法遍历等待队列，如果满足同一主机的请求小于maxRequestPerHost时，从队列中取出请求，放入线程池执行
    private void promoteCalls() {
    	if (runningAsyncCalls.size() >= maxRequests) return; // Already running max capacity.
    	if (readyAsyncCalls.isEmpty()) return; // No ready calls to promote.

    	for (Iterator<AsyncCall> i = readyAsyncCalls.iterator(); i.hasNext(); ) {
      		AsyncCall call = i.next();

      		if (runningCallsForHost(call) < maxRequestsPerHost) {
        		i.remove();
        		runningAsyncCalls.add(call);
        		executorService().execute(call);
      		}

      		if (runningAsyncCalls.size() >= maxRequests) return; // Reached max capacity.
    	}
    }
    
    

进行网络请求是通过getResponseWithInterceptorChain()这个方法获取返回值

	Response getResponseWithInterceptorChain() throws IOException {
    	// Build a full stack of interceptors.
    	List<Interceptor> interceptors = new ArrayList<>();
    	// 这里注意一下，interceptors()的添加顺序在networkInterceptors()之前，中间有retryAndFollowUpInterceptor，这个拦截器是负责请求的重定向的，也就是说，如果有重定向情况发生，这个拦截器的不会返回response，会接着再次调用proceed方法，所以networkInterceptors中的intercept方法可能被调用多次（每次发生网络请求都会调用），而interceptors中的intercept方法只会被调用一次。
    	interceptors.addAll(client.interceptors());
    	interceptors.add(retryAndFollowUpInterceptor);
    	interceptors.add(new BridgeInterceptor(client.cookieJar()));
    	interceptors.add(new CacheInterceptor(client.internalCache()));
    	interceptors.add(new ConnectInterceptor(client));
    	if (!forWebSocket) {
      		interceptors.addAll(client.networkInterceptors());
    	}
    	interceptors.add(new CallServerInterceptor(forWebSocket));

    	Interceptor.Chain chain = new RealInterceptorChain(
        	interceptors, null, null, null, 0, originalRequest);
    	return chain.proceed(originalRequest);
    }
    
 这里增加了很多拦截器            
1. 在配置 OkHttpClient 时设置的 interceptors；         
2. 负责失败重试以及重定向的 RetryAndFollowUpInterceptor；          
3. 负责把用户构造的请求转换为发送到服务器的请求、把服务器返回的响应转换为用户友好的响应的BridgeInterceptor； 				
4. 负责读取缓存直接返回、更新缓存的 CacheInterceptor；        
5. 负责和服务器建立连接的 ConnectInterceptor；             
6. 配置 OkHttpClient 时设置的 networkInterceptors；              
7. 负责向服务器发送请求数据、从服务器读取响应数据的 CallServerInterceptor。      
OkHttp的这种拦截器链采用的是责任链模式，这样的好处是将请求的发送和处理分开，并且可以动态添加中间的处理方实现对请求的处理、短路等操作。            
不管有多少拦截器，最后都会走  

	Interceptor.Chain chain = new RealInterceptorChain(
        	interceptors, null, null, null, 0, originalRequest);
    	return chain.proceed(originalRequest);
    	
我们看一下RealInterceptorChain这个类           
	
	public final class RealInterceptorChain implements Interceptor.Chain {
		public RealInterceptorChain(List<Interceptor> interceptors, StreamAllocation streamAllocation,
        	HttpCodec httpCodec, RealConnection connection, int index, Request request) {
        	// 所有拦截器
        	this.interceptors = interceptors;
        	this.connection = connection;
        	this.streamAllocation = streamAllocation;
        	this.httpCodec = httpCodec;
        	// 当前拦截器的下标
        	this.index = index;
        	this.request = request;
        }
        	......
        @Override 
        public Response proceed(Request request) throws IOException {
       		return proceed(request, streamAllocation, httpCodec, connection);
    	}
    	public Response proceed(Request request, StreamAllocation streamAllocation, HttpCodec httpCodec, RealConnection connection) throws IOException {
    		if (index >= interceptors.size()) throw new AssertionError();

    			calls++;

    			......

    		// Call the next interceptor in the chain.
    		RealInterceptorChain next = new RealInterceptorChain(
        		interceptors, streamAllocation, httpCodec, connection, index + 1, request);
    		//获取到当前拦截器，调用其intercept方法
    		// e.g. 第一个获取的是我们自定义的Interceptor，如果没有的话，是RetryAndFollowUpInterceptor，重新new了一个RealInterceptorChain对象，把index + 1，然后调用RetryAndFollowUpInterceptor的intercept方法，把参数传进去，因此在RetryAndFollowUpInterceptor的intercept方法中，可以通过参数chain获取到最原始的request和call对象等，做一些自己的处理，然后调用proceed方法，把处理完的参数传入，获取Response对象，这时候又走到了RealInterceptorChain的proceed方法，然后获取到第二个拦截器，再执行上面的逻辑。这个方法会返回Response对象，直到最后一个Interceptor处理之前，大家都在等待这个对象。
    		// Interceptor链的最后一个Interceptor是CallServerInterceptor，它真正执行了网络请求，获取到Response后，内部不再调用chain的proceed方法，直接返回网络请求的Response，然后倒数第二个Interceptor的proceed方法接收到返回值，进行相应的处理，然后返回，倒数第三个Interceptor接收到处理...层层向上，直到最后第一个Interceptor接收到数据处理完返回，这时候RealCall的getResponseWithInterceptorChain拿到最终的返回值，网络请求完毕。
    		Interceptor interceptor = interceptors.get(index);
    		Response response = interceptor.intercept(next);
    			......

    			return response;
    	}
    	protected abstract void execute();
    }
    
Interceptor代码如下

	public interface Interceptor {
		Response intercept(Chain chain) throws IOException;
			nterface Chain {
    			Request request();
    		Response proceed(Request request) throws IOException;

    		/**
     		* Returns the connection the request will be executed on. This is only available in the chains
     		* of network interceptors; for application interceptors this is always null.
     		*/
    		@Nullable Connection connection();
    	}
    }
   