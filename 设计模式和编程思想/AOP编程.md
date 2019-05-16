AOP编程
===================
AOP是Aspect Oriented Program的首字母缩写，面向切面编程。在运行时，动态地将代码切入到类的制定方法、指定位置上的编程思想。              
一般而言，我们管切入到指定类指定方法的代码片段称为切面，而切入到哪些类、哪些方法则叫切入点。有了AOP，我们就可以把几个类共有的代码，抽取到一个切片中，等到需要时再切入对象中去，从而改变其原有的行为。这样看来，AOP其实只是OOP的补充而已。OOP从横向上区分出一个个的类来，而AOP则从纵向上向对象中加入特定的代码。有了AOP，OOP变得立体了。如果加上时间维度，AOP使OOP由原来的二维变为三维了，由平面变成立体了。从技术上来说，AOP基本上是通过代理机制实现的AOP在编程历史上可以说是里程碑式的，对OOP编程是一种十分有益的补充。         

AspectJ
-----------------------
AOP编程思想的一个实践，不是新语言，是一个代码编译器(ajc)，可以编译Java代码，在编译器将开发者编写的Aspect程序编织到目标程序中，对目标程序做了重构，建立目标程序与Aspect程序的连接。          
1. Join Point(连接点)             
是AspectJ的核心思想之一，把程序的整个执行过程切成一段段不同的部分。如构造方法调用、调用方法、方法执行、异常等等。指插入代码的地方。         
2. Pointcuts(切入点)         
告诉代码注入工具，在什么地方注入一段特定代码的表达式。可简单理解为带条件的Join Points，作为我们需要的代码切入点。        
3. Advice(通知)      
如何注入class文件中的代码，有before、after和around，分别指在目标方法执行前、执行后和完全替换。        
4. Aspect(切面)      
PointCut和Advice的组合。          
5. Weaving(织入)       
注入代码(advices)到目标位置(join points)的过程。      

	@Before("execution(* android.app.Activity.on*(android.os.Bundle))")
	public void onActivityMethodBefore(JoinPoint joinPoint) throws Throwable {
		String key = joinPoint.getSignature().toString();
		Log.d(TAG, "onActivityMethodBefore: " + key);
	}
	@After("execution(* android.app.Activity.on*(android.os.Bundle))")
	public void onActivityMethodAfter(JoinPoint joinPoint) throws Throwable {
		String key = joinPoint.getSignature().toString();
		Log.d(TAG, "onActivityMethodAfter: " + key);
	}
	//Around的用法
	@Around("execution(* com.example.myaspectjapplication.activity.RelativeLayoutTestActivity.testAOP())")
	public void onActivityMethodAround(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
		String key = proceedingJoinPoint.getSignature().toString();
		Log.d(TAG, "onActivityMethodAroundFirst: " + key);
		proceedingJoinPoint.proceed();
		Log.d(TAG, "onActivityMethodAroundSecond: " + key);
	}

