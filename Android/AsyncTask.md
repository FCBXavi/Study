#AsnycTask

	public abstract class AsyncTask<Params, Progress, Result>

####一个异步任务使用的三种类型如下：

1. Params，the type of the parameters sent to the task upon execution.（参数的类型发送到任务执行时。）

2. Progress，the type of the progress units published during the background computation.（后台计算过程中的进度单元类型。）

3. Result，the type of the result of the background computation.（后台运行的结果类型。）


####一个异步任务的执行一般包括以下几个步骤：
1. execute(Params... params)，执行一个异步任务，需要我们在代码中调用此方法，触发异步任务的执行。
2. onPreExecute()，在execute(Params... params)被调用后立即执行，一般用来在执行后台任务前对UI做一些标记。
3. doInBackground(Params... params)，在onPreExecute()完成后立即执行，用于执行较为费时的操作，此方法将接收输入参数和返回计算结果。在执行过程中可以调用publishProgress(Progress... values)来更新进度信息。
4. onProgressUpdate(Progress... values)，在调用publishProgress(Progress... values)时，此方法被执行，直接将进度信息更新到UI组件上。
5. onPostExecute(Result result)，当后台操作结束时，此方法将会被调用，计算结果将做为参数传递到此方法中，直接将结果显示到UI组件上。


初始化的时候    

	public AsyncTask() {
		//初始化worker对象，WorkerRunnable实际上是继承Callable的一个抽象类，
		增加了一个数组存储params
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
            	//表示当前任务已经被调用过
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    postResult(result);
                }
                return result;
            }
        };

		//把mWorker作为参数传入FutureTask的构造方法,当mFuture对象被提交到AsyncTask包含的线程池执行时，mWorker的call方法会被调用
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }
    

初始化了mWorker，它是一个派生自WorkRunnable类的对象。WorkRunnable是一个抽象类，它实现了Callable<Result>接口，调用了AsyncTask对象的doInBackground方法开始执行我们所定义的后台任务，并获取返回结果存入result中。最后将任务返回结果传递给postResult方法。由此我们可以知道，实际上AsyncTask的成员mWorker包含了AyncTask最终要执行的任务（即mWorker的call方法）



内部维护两个线程池 串行线程池SERIAL\_EXECUTOR和并行线程池THREAD\_POOL\_EXECUTOR
串行线程池的作用主要是把任务加到缓存队列中，而真正执行任务的是THREAD\_POOL\_EXECUTOR

	//串行线程池
	//实际上是把任务加入缓存队列，按照队列的顺序依次执行队列中的任务
	private static volatile Executor sDefaultExecutor = SERIAL_EXECUTOR;
	public static final Executor SERIAL_EXECUTOR = new SerialExecutor();
	private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }

	//并行线程池
	//真正执行任务的地方
	private static final int CPU_COUNT = Runtime.getRuntime().availableProcessors();
    // We want at least 2 threads and at most 4 threads in the core pool,
    // preferring to have 1 less than the CPU count to avoid saturating
    // the CPU with background work
    private static final int CORE_POOL_SIZE = Math.max(2, Math.min(CPU_COUNT - 1, 4));
    private static final int MAXIMUM_POOL_SIZE = CPU_COUNT * 2 + 1;
    private static final int KEEP_ALIVE_SECONDS = 30;
    ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
    threadPoolExecutor.allowCoreThreadTimeOut(true);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
	
	

调用AsyncTask的execute方法后，会在内部调用executeOnExecutor方法，在其中首先判断状态，如果当前任务是进行中或者已完成，抛出异常，因此一个AsyncTask对象的execute方法只能调用一次，然后先执行onPreExecute方法，然后开始用sDefaultExecutor线程池开始执行mFuture，首先会加入串行线程池，从串行线程池的队列中找出下个要执行的Runnable，加入并行线程池执行。这时会执行mWorker的call方法，执行完毕后，会执行postResult方法。这里就会用handler发送事件。
	
在内部有一个Handler进行子线程和主线程的交互，然后在handleMessage方法中进行处理，对于发送来的消息根据类型判断，调用onProgressUpdate(Progress... values)方法还是finish(Result result)方法，finish方法中判断如果task没有被cancel，就调用onPostExecute(Result result)方法



####在使用的时候，有几点需要格外注意：
1. 异步任务的实例必须在UI线程中创建。
2. execute(Params... params)方法必须在UI线程中调用。(onPreExecute是在execute调用的线程执行，这个方法如果要更新UI，必须要在主线程，为了规范，则规定execute方法必须运行在主线程，其他三个回调方法是通过Handler发送消息后在Handler的handleMessage方法中执行的，一定会在主线程执行)
3. 不要手动调用onPreExecute()，doInBackground(Params... params)，onProgressUpdate(Progress... values)，onPostExecute(Result result)这几个方法。
4. 不能在doInBackground(Params... params)中更改UI组件的信息。
5. 一个任务实例只能执行一次，如果执行第二次将会抛出异常。
