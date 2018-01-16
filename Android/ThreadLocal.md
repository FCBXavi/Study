#ThreadLocal		
作用域：线程内部。       
生命周期：伴随线程执行始终，线程结束，变量生命结束。      
共享性：多个线程之间不共享。      
ThreadLocal解决了变量在同一个线程内部之间的传递。     

原理：ThreadLocal有一个ThreadLocalMap内部类，Thread类有一个ThreadLocal.ThreadLocalMap threadLocals成员变量，这个ThreadLocalMap对象将当前ThreadLocal为key，要保存的变量为value进行存储。      

ThreadLocal的get方法，会首先获取到当前线程，然后得到当前线程的ThreadLocalMap对象，如果不为空，就从map对象中获取以当前ThreadLocal对象为key的值，如果为空，就生成一个ThreadLocalMap对象，key是当前ThreadLocal对象，value是initialValue()的返回值，将其赋值给Thread对象的ThreadLocalMap对象。