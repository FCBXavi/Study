#Callable、Future和FutureTask
<http://www.cnblogs.com/dolphin0520/p/3949310.html>

在Java多线程中，继承Thread和实现Runnable接口都可以实现多线程，但是没办法获取返回的结果，Future可以实现获取结果。

Callable，接口，有一个call方法；
Future，接口，有cancel，isCanceled，isDone，get等方法；
FutureTask，实现了RunnableFuture接口，RunnableFuture接口继承了Future和Runnable接口。
Future，Callable一般和ExecutorService一起使用，ExecutorService.submit(...)方法会返回Future对象。
	
	public interface Runnable {
    	public abstract void run();
	}
	
	public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
	}

Future就是对于具体的Runnable或者Callable任务的执行结果进行取消、查询是否完成、获取结果。必要时可以通过get方法获取执行结果，该方法会阻塞直到任务返回结果。	

	public interface Future<V> {
   		boolean cancel(boolean mayInterruptIfRunning);
   		boolean isCancelled();
   		boolean isDone();
    	V get() throws InterruptedException, ExecutionException;
    	V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
	}


FutureTask是Future接口的一个唯一实现类。
   
	public class FutureTask<V> implements RunnableFuture<V> {}
	public interface RunnableFuture<V> extends Runnable, Future<V> 	{
    	/**
    	 * Sets this Future to the result of its computation
    	 * unless it has been cancelled.
    	 */
    	void run();
	}	
	
