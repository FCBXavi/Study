Synchronized
A. 无论synchronized关键字加在方法上还是对象上，如果它作用的对象是非静态的，则它取得的锁是对象；如果synchronized作用的对象是一个静态方法或一个类，则它取得的锁是对类。 
B. 每个对象只有一个锁（lock）与之相关联，谁拿到这个锁谁就可以运行它所控制的那段代码。 
C. 实现同步是要很大的系统开销作为代价的，甚至可能造成死锁，所以尽量避免无谓的同步控制。

调用一个Object的wait与notify/notifyAll的时候，必须保证调用代码对该Object是同步的，也就是说必须在作用等同于synchronized(obj){......}的内部才能够去调用obj的wait与notify/notifyAll三个方法，否则就会报错：
  java.lang.IllegalMonitorStateException:current thread not owner
wait():
调用任意对象的 wait() 方法导致该线程阻塞，该线程不可继续执行，并且该对象上的锁被释放。
notify():
唤醒在等待该对象同步锁的线程(只唤醒一个,如果有多个在等待),注意的是在调用此方法的时候，并不能确切的唤醒某一个等待状态的线程，而是由JVM确定唤醒哪个线程，而且不是按优先级。调用任意对象的notify()方法则导致因调用该对象的 wait()方法而阻塞的线程中随机选择的一个解除阻塞（但要等到获得锁后才真正可执行）。
notifyAll():
唤醒所有等待的线程,注意唤醒的是notify之前wait的线程,对于notify之后的wait线程是没有效果的。

谈一下synchronized和wait()、notify()等的关系:
1.有synchronized的地方不一定有wait,notify
2.有wait,notify的地方必有synchronized.这是因为wait和notify不是属于线程类，而是每一个对象都具有的方法，而且，这两个方法都和对象锁有关，有锁的地方，必有synchronized。
另外，注意一点：如果要把notify和wait方法放在一起用的话，必须先调用notify后调用wait，因为如果调用完wait，该线程就已经不是currentthread了。