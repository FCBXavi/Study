GC机制
======================================
垃圾回收主要针对的是堆区的回收，因为栈区的内存是随着线程而释放的  
     
* 年轻代（Young Generation）(由一个Eden区和俩个survivor区组成)
	停止-复制（Stop-and-copy）停止是指在回收内存时，需要暂停其他所有线程的执行                         
	新创建的对象都在年轻代的Eden区，当Eden区满的时候，执行GC，存活下来的会被复制到Survivor0区，清空Eden区，下次Eden区满了，执行GC，存活下来的对象复制到Survivor1区，清除Eden区，把Survivor0区中存活的对象复制到Survivor1区，清空Survivor0区，保证Survivor区总有一个为空，两个Survivor区切换几次后，存活的对象被复制到年老代  
* 年老代（Old Generation）       
	使用标记-整理（Mark-Sweep）法            
	先遍历一遍找出存活的对象，然后将所有存活的对象向一端移动，清理剩余的区域。    的回收。
* 永久代（Permanent Generation，也就是方法区）。
	回收内容：常量池中的常量，无用的类信息。常量没有引用就可以被回收，对于类来说回收要保证三点：1.类的所有实例已经被回收2.加载类的ClassLoader已经被回收，3.类对象的Class对象没有引用
	
	
Java引用类型
-----------------------------------
1. StrongReference强引用     
	强引用指向的对象永远不会被回收，可能引起内存泄漏       
2. SoftReference软引用         
	系统内存不足的时候会进行回收，初始化时可以注册传一个引用队列的参数，一旦被垃圾回收器回收，便会被加入到注册的引用队列中
3. WeakReference弱引用           
	下次gc一定会被回收
4. PhantomReference虚引用    
	持有虚引用的对象和没有引用几乎是一样的，随时都可能被垃圾回收器回收，使用get()方法获得强引用时总是会失败，必须和引用队列一起使用，它的作用在于跟踪垃圾回收过程。在对象被gc是受到系统通知
	
	
GCRoot
--------------------------------------
Traceing GC的思路：给定一个集合的引用作为root，通过引用关系遍历对象图，能被遍历到的判定为存活，没有被遍历到的判定为死亡。            
GC Root指一组必须活跃的引用         
1. 所有Java线程当前活跃栈帧里指向GC堆里对象的引用       
2. VM一些静态数据结构里指向GC对象的引用      
3. JNI handles，包括global handles和local handles        
4. 所有当前被加载的java类       
5. java类的引用和静态变量      
6. 常量池里的引用类型常量(String或Class类型)       
7. String常量池里的引用   

1. Class - 由系统类加载器(system class loader)加载的对象，这些类是不能够被回收的，他们可以以静态字段的方式保存持有其它对象。我们需要注意的一点就是，通过用户自定义的类加载器加载的类，除非相应的java.lang.Class实例以其它的某种（或多种）方式成为roots，否则它们并不是roots。     
2. Thread - 活着的线程         
3. Stack Local - Java方法的local变量或参数         
4. JNI Local - JNI方法的local变量或参数        
5. JNI Global - 全局JNI引用          
6. Monitor Used - 用于同步的监控对象           
7. Held by JVM - 用于JVM特殊目的由GC保留的对象，但实际上这个与JVM的实现是有关的。可能已知的一些类型是：系统类加载器、一些JVM知道的重要的异常类、一些用于处理异常的预分配对象以及一些自定义的类加载器等。然而，JVM并没有为这些对象提供其它的信息，因此需要去确定哪些是属于"JVM持有"的了。
        
	