let、with、run、apply、also
=========================

都是通过一个对象调用，会形成一个临时的作用域，在作用域代码块中无需对象名称就可以访问该对象，区别是在代码块中，这个对象如何使用，以及整个表达式的返回值是什么。      
对象引用方式
-------------------
引用方式为this -> run、with、apply       
适用场景：lambda表达式中，主要对对象成员进行操作          
引用方式为it -> let、also          
适用场景：lambda表达式中，对象被作为函数调用的参数        
表达式返回值
-------------------
返回上下文对象(调用者) -> apply、also        
  
	// 可以作为辅助步骤包含在调用链中     
	var numberList = mutableListOf<Double>()
	numberList.also{ println("Populating the list")}
		.apply{
			add(2.71)
			add(3.14)
			add(1.0)
		}
		.also { println("Sorting the list")}
		.sort()
	// 也可以用在返回上下文对象的return语句中
	fun getRandomInt(): Int {
		return Random.nextInt(100).also {
			writeToLog("getRandomInt() generated value $it")
		}
	}
	
返回lambda表达式结果 -> let、run、with       

总结
----------------------

函数 | 对象引用 | 返回 | 是否是扩展函数
---- | --- | --- | ---
let | it | Lambda 表达式结果 | 是    
run | this | Lambda 表达式结果	 | 是           
run | - | Lambda 表达式结果 | 不是：调用无需上下文对象         
with  | this | Lambda 表达式结果 | 不是：把上下文对象当做参数        
apply | this | 上下文对象 | 是        
also	 | it | 上下文对象 | 是  

以下是根据预期目的选择作用域函数的简短指南：      
对一个非空（non-null）对象执行 lambda 表达式：let       
将表达式作为变量引入为局部作用域中：let      
对象配置：apply      
对象配置并且计算结果：run      
在需要表达式的地方运行语句：非扩展的 run      
附加效果：also      
一个对象的一组函数调用：with      


takeIf和takeUnless
---------------------
takeIf和takeUnless内部通过it调用对象，代码块返回一个Boolean对象，如果代码块返回true，takeIf的返回值是这个对象，否则就是null，takeUnless反之。takeIf之后调用其他方法需要用?.       

	val number = Random.nextInt(100)
	val evenOrNull = number.takeIf{ it % 2 == 0}
	val oddOrNull = number.takeUnless { it % 2 == 0}






