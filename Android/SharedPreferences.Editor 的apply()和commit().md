#SharedPreferences.Editor 的apply()和commit()
apply方法在官方SDK说明如下：
Commit your preferences changes back from this Editor to the SharedPreferences object it is editing. This atomically performs the requested modifications, replacing whatever is currently in the SharedPreferences.
Note that when two editors are modifying preferences at the same time, the last one to call apply wins.
Unlike commit, which writes its preferences out to persistent storage synchronously, apply commits its changes to the in-memory SharedPreferences immediately but starts an asynchronous commit to disk and you won’t be notified of any failures. If another editor on this SharedPreferences does a regular commit while a apply is still outstanding, the commit will block until all async commits are completed as well as the commit itself.
As SharedPreferences instances are singletons within a process, it’s safe to replace any instance of commit with apply if you were already ignoring the return value.
You don’t need to worry about Android component lifecycles and their interaction with apply() writing to disk. The framework makes sure in-flight disk writes from apply() complete before switching states.
The SharedPreferences.Editor interface isn’t expected to be implemented directly. However, if you previously did implement it and are now getting errors about missing apply(), you can simply call commit from apply().


这两个方法的区别在于： 

1. apply没有返回值而commit返回boolean表明修改是否提交成功 
2. apply是将修改数据原子提交到内存, 而后异步真正提交到硬件磁盘, 而commit是同步的提交到硬件磁盘，因此，在多个并发的提交commit的时候，他们会等待正在处理的commit保存到磁盘后在操作，从而降低了效率。而apply只是原子的提交到内存，后面有调用apply的函数的将会直接覆盖前面的内存数据，这样从一定程度上提高了很多效率。 
3. apply方法不会提示任何失败的提示。 
由于在一个进程中，sharedPreference是单实例，一般不会出现并发冲突，如果对提交的结果不关心的话，建议使用apply，当然需要确保提交成功且有后续操作的话，还是需要用commit的。