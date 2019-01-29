Zygote进程
==========================
Zygote进程, 一个在Android系统中扮演重要角色的进程. 我们知道Android系统中的两个重要服务PackageManagerService和ActivityManagerService, 都是由SystemServer进程启动的, 而这个SystemServer进程本身是Zygote进程在启动的过程中fork出来的.          

Zygote一个最主要的作用, 就是加快Android应用程序启动和运行速度. Zygote进程运行时, 会初始化Dalvik虚拟机, 并运行它. Android的应用程序是由Java编写的, 它们不能直接运行在Linux上, 只能运行在Dalvik虚拟机中. 并且, 每个应用程序都运行在各自的虚拟机中, 应用程序每次运行都要重新初始化并启动虚拟机, 这个过程会消耗相当长时间, 是拖慢应用程序的原因之一. 因此, 在Android中, 应用程序运行前, 通过Zygote进程共享已运行的虚拟机的代码与内存信息, 缩短应用程序运行所耗费的时间. 也就是说, Zygote进程会事先将应用程序要使用的Android Framework中的类与资源加载到内存中, 并组织形成所用资源的链接信息. 这样, 新运行的Android应用程序在使用所需资源时不必每次形成资源的链接信息, 这样就大大提升了程序的运行时间.Zygote进程起到了预加载资源和类到虚拟机提高应用程序提高的作用.   

zygote进程是init进程创建的. 我们知道, Android系统时基于Linux内核的, 而在Linux系统中, 所有的进程都是init进程的子孙进程.         
zygote进程是由init进程创建各种deamon后创建的, 他可以启动运行Android服务和应用程序.
zygote进程启动时会初始并运行虚拟机, 而后将所需要的类和资源加载到内存中, 新进程创建的时候可以直接使用这些类和资源, 大大加快启动运行数速度, 这就是cow技术.    
所以, Zygote进程就是init进程fork出来的. 但是, Zygote是由java编写而成的, 所以也要先初始化虚拟机, 由app_process进程装载并运行ZygoteInit类.      

app_process创建一个AppRuntime变量，然后调用它的start成员函数, 由于AppRuntime类没有重写start函数, 所以调用的是其父类AndroidRuntime中的start函数. 在这个start函数中, 它干了三件事: 一是调用函数startVM启动虚拟机，二是调用函数startReg注册运行ZygoteInit时需要调用的JNI本地方法，三是调用com.android.internal.os.ZygoteInit类的main函数.       

说白了, 以上的一切都是为了ZygoteInit类的main函数的运行, 来启动Zygote进程. 这个函数里就是Zygote进程正真做的事情, ZygoteInit类的功能.        
1. 调用registerZygoteSocket()绑定套接字, 接收新的Android应用程序运行请求, 用来和ActivityManagerServer通讯. 说的详细一点就是创建LocalServerSocket实例(也就是Socket的服务端)接收生成新Android进程的信息. 每一个新的Android应用程序进程的创建都要通过Socket请求Zygote进程(即Socket的客户端).
2. 调用preloadClasses()和preloadResource()来加载Android Application Framework使用的类与资源. 这里就是Zygote预加载资源和类, 提高Android应用程序启动速度的点.       
3. 第三步调用startSystemServer()运行SystemServer进程, 来启动各种服务.4. 最后一步调用runSelectLoopMode()来循环监听, 与第一步对应.       


借用罗老师的总结:
1. 系统启动时init进程会创建Zygote进程，Zygote进程负责后续Android应用程序框架层的其它进程的创建和启动工作。         
2. Zygote进程会首先创建一个SystemServer进程，SystemServer进程负责启动系统的关键服务，如包管理服务PackageManagerService和应用程序组件管理服务ActivityManagerService。        
3. 当我们需要启动一个Android应用程序时，ActivityManagerService会通过Socket进程间通信机制，通知Zygote进程为这个应用程序创建一个新的进程。                   