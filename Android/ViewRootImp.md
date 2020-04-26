ViewRootImp
-------------------------
在ActivityThread的handleResumeActivity中，拿到Activity的windowManager对象，调用了addView方法，把Activity的PhoneWindow中的DecorView传进去，这个windowManager的实例是WindowManagerImpl，代理了WindowManagerGlobal，相当于调用了WindowManagerGlobal的addView方法，这是一个单例类，内部维护了三个列表
	
	public final class WindowManagerGlobal {
		// 这三个列表对应的下标一一对应
		private final ArrayList<View> mViews = new ArrayList<View>();
    	private final ArrayList<ViewRootImpl> mRoots = new ArrayList<ViewRootImpl>();
    	private final ArrayList<WindowManager.LayoutParams> mParams =
            new ArrayList<WindowManager.LayoutParams>();
            
       public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow) {
       	  root = new ViewRootImpl(view.getContext(), display);
       	  view.setLayoutParams(wparams);
       	  mViews.add(view);
       	  mRoots.add(root);
       	  mParams.add(wparams);     
       	  // 调用ViewRootImp的setView方法
       	  root.setView(view, wparams, panelParentView);
       }
	}
	
addView方法会new出ViewRootImpl对象，然后调用该对象的setView方法，把view传入
	
	public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
		mView = view;
		//...
		// ViewRootImpl内部调用了requestLayout方法
		requestLayout();
		//...
	}
	
	public void requestLayout() {
        if (!mHandlingLayoutInLayoutRequest) {
        	// 这里	判断了当前线程是不是主线程，如果不是，会抛出异常
            checkThread();
            mLayoutRequested = true;
            scheduleTraversals();
        }
    }
    
    void scheduleTraversals() {
        if (!mTraversalScheduled) {
            //...异步发送一个消息，执行mTraversalRunnable
            mChoreographer.postCallback(
                    Choreographer.CALLBACK_TRAVERSAL, mTraversalRunnable, null);
            //...
        }
    }
    
    final class TraversalRunnable implements Runnable {
        @Override
        public void run() {
        	// Runnable里执行了该方法
            doTraversal();
        }
    }
    
    void doTraversal() {
        if (mTraversalScheduled) {
			// ...
            performTraversals();
			// ...
        }
    }
    
    private void performTraversals() {
    	// view的attachInfo在此赋值,mAttachInfo在ViewRootImpl构造方法中初始化,会遍历给所有的子View都赋值为同一个AttachInfo对象
    	final View host = mView;
    	host.dispatchAttachedToWindow(mAttachInfo, 0);
    	//...
    	// 该方法执行了view的measure方法
    	int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
      	int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
    	performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
    	//...
    	// 该方法执行了view的layout方法
    	performLayout(lp, mWidth, mHeight);
    	//...
    	// 该方法执行了view的draw方法
    	performDraw();
    	//...
    }
至此，完成了view的绘制流程。         

viewRootImp的setView方法调用了view的assignParent方法   

	public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
    	//...
    	// 这里的view是DecorView
    	view.assignParent(this);
    	//...
	}
	看一下view的assignParent方法：
	void assignParent(ViewParent parent) {
        if (mParent == null) {
            mParent = parent;
        } else if (parent == null) {
            mParent = null;
        } else {
            throw new RuntimeException("view " + this + " being added, but"
                    + " it already has a parent");
        }
    }
    将ViewRootImp设置为DecorView的mParent       
    
Choreographer
----------------
在ViewRootImp的scheduleTraversals方法中，调用了

	mChoreographer.postCallback(Choreographer.CALLBACK_TRAVERSAL, 
		mTraversalRunnable, null);
	// 看一下这个方法	
	public void postCallback(int callbackType, Runnable action, Object token) {
        postCallbackDelayed(callbackType, action, token, 0);
    }
    
    public void postCallbackDelayed(int callbackType,
            Runnable action, Object token, long delayMillis) {
        //...
        postCallbackDelayedInternal(callbackType, action, token, delayMillis);
    }
    
    private void postCallbackDelayedInternal(int callbackType,
            Object action, Object token, long delayMillis) {
        synchronized (mLock) {
            final long now = SystemClock.uptimeMillis();
            final long dueTime = now + delayMillis;
            // action是从ViewRootImp中传入的Runnable对象，放在了这个队列中
            mCallbackQueues[callbackType].addCallbackLocked(dueTime, action, token);

            if (dueTime <= now) {
                scheduleFrameLocked(now);
            } else {
            	// 通过发送message，最后也是执行了scheduleFrameLocked方法
                Message msg = mHandler.obtainMessage(MSG_DO_SCHEDULE_CALLBACK, action);
                msg.arg1 = callbackType;
                msg.setAsynchronous(true);
                mHandler.sendMessageAtTime(msg, dueTime);
            }
        }
    }
    
    private void scheduleFrameLocked(long now) {
        if (!mFrameScheduled) {
            //...
            scheduleVsyncLocked();
            //...
        }
    }
    
    private void scheduleVsyncLocked() {
        mDisplayEventReceiver.scheduleVsync();
    }
    // 看一下这个类里的scheduleVsync方法
    public void scheduleVsync() {
        if (mReceiverPtr == 0) {
            Log.w(TAG, "Attempted to schedule a vertical sync pulse but the display event "
                    + "receiver has already been disposed.");
        } else {
        // 执行了native方法，注册了一个监听器，监听底层的VSYNC绘制信号
            nativeScheduleVsync(mReceiverPtr);
        }
    }
    
    // 这个Receiver的代码
    private final class FrameDisplayEventReceiver extends DisplayEventReceiver
            implements Runnable {
        private boolean mHavePendingVsync;
        private long mTimestampNanos;
        private int mFrame;

        public FrameDisplayEventReceiver(Looper looper, int vsyncSource) {
            super(looper, vsyncSource);
        }
        
        // 接收到VSYNC信号后会回调该方法
        @Override
        public void onVsync(long timestampNanos, long physicalDisplayId, int frame) {
            long now = System.nanoTime();
            if (timestampNanos > now) {
                timestampNanos = now;
            }

            if (mHavePendingVsync) {
            } else {
                mHavePendingVsync = true;
            }
            
            mTimestampNanos = timestampNanos;
            mFrame = frame;
            // message的callback对象是当前对象，send完从消息队列里取出的时候，会执行run方法，也就是doFrame方法
            Message msg = Message.obtain(mHandler, this);
            msg.setAsynchronous(true);
            mHandler.sendMessageAtTime(msg, timestampNanos / TimeUtils.NANOS_PER_MS);
        }

        @Override
        public void run() {
            mHavePendingVsync = false;
            doFrame(mTimestampNanos, mFrame);
        }
    }
    
    
    void doFrame(long frameTimeNanos, int frame) {
    	//...
     	doCallbacks(Choreographer.CALLBACK_TRAVERSAL, frameTimeNanos);
     	//...
    }
    
    void doCallbacks(int callbackType, long frameTimeNanos) {
        CallbackRecord callbacks;
        synchronized (mLock) {
            //...
            // 从队列中取出之前传入的Runnable封装对象
            callbacks = mCallbackQueues[callbackType].extractDueCallbacksLocked(
                    now / TimeUtils.NANOS_PER_MS);
            //...        
        }
        // 执行run方法，实际上执行的是ViewRootImp传入的Runnable对象，这里就执行了ViewRootImp的doTraversals方法
        c.run(frameTimeNanos);
    }


1. 我们知道一个 View 发起刷新的操作时，最终是走到了 ViewRootImpl 的 scheduleTraversals() 里去，然后这个方法会将遍历绘制 View 树的操作 performTraversals() 封装到 Runnable 里，传给 Choreographer，以当前的时间戳放进一个 mCallbackQueue 队列里，然后调用了 native 层的方法向底层注册监听下一个屏幕刷新信号事件。          

2. 当下一个屏幕刷新信号发出的时候，如果我们 app 有对这个事件进行监听，那么底层它就会回调我们 app 层的 onVsync() 方法来通知。当 onVsync() 被回调时，会发一个 Message 到主线程，将后续的工作切到主线程来执行。           

3. 切到主线程的工作就是去 mCallbackQueue 队列里根据时间戳将之前放进去的 Runnable 取出来执行，而这些 Runnable 有一个就是遍历绘制 View 树的操作 performTraversals()。在这次的遍历操作中，就会去绘制那些需要刷新的 View。        

4. 所以说，当我们调用了 invalidate()，requestLayout()，等之类刷新界面的操作时，并不是马上就会执行这些刷新的操作，而是通过 ViewRootImpl 的 scheduleTraversals() 先向底层注册监听下一个屏幕刷新信号事件，然后等下一个屏幕刷新信号来的时候，才会去通过 performTraversals() 遍历绘制 View 树来执行这些刷新操作。     


在一次绘制信号到来的时候，也就是16.6ms内，如果有多个绘制请求发起，只会执行一次performTraversal       

	 void scheduleTraversals() {
	     // 这个变量是false，才会把Runnable传给Choreographer
        if (!mTraversalScheduled) {
            mTraversalScheduled = true;
            // 发送同步屏障，Message的next发现队头是同步内存屏障时，就会遍历整个队列，只找出设置了异步标志的消息，拿出来执行，否则next()会进入阻塞状态，next在阻塞状态时，主线程是空闲的。默认的消息都是同步的，在Choreographer里的message都设置了异步标志位，所以doTraversal()可以得到执行，这样就保证了view的绘制操作优先得到处理。
            // 
            mTraversalBarrier = mHandler.getLooper().getQueue().postSyncBarrier();
            //...
        }
    }
    
    void doTraversal() {
        if (mTraversalScheduled) {
            // Runnable中执行了doTraversal方法，这个方法得到执行时，才会把mTraversalScheduled这个变量重新设置成false
            mTraversalScheduled = false;
            // 移除同步屏障
            mHandler.getLooper().getQueue().removeSyncBarrier(mTraversalBarrier);
        }
    }  

    