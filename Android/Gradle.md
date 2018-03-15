Gradle
==============================
Gradle中两个关键的概念：项目和任务
-------------------------
每个build.gradle构建脚本代表一个项目project             
任务task定义在构建脚本里                
每次构建至少包括一个项目，每个项目里至少包括一个任务                     


构建生命周期
---------------------------------
1. 初始化
   读取settings.gradle中的include信息，决定哪些工程加入构建，创建project实例，如果项目里有多个module，或者以来多个library，并且它们都有对应的build.gradle文件，就会创建多个项目实例。
2. 配置          
   在这个阶段构建脚本被执行，执行所有project的build.gradle脚本，配置project对象，一个project对象有多个task组成，此阶段也会去创建、配置task及相关信息。         
3. 执行
   根据gradle传来的task名称，执行相关依赖任务。
   
用Gradle输出HelloWorld     
          
    //在配置阶段打印出来      
	task hello {
		println 'hello'
	}
	//在运行阶段打印出来
	task hello{
    	doLast{
        	println 'Hello world!'
    	}
	}
   
   
   
[十分钟理解Gradle](https://www.cnblogs.com/Bonker/p/5619458.html)                   

[Gradle生命周期](http://blog.csdn.net/u013626215/article/details/51490643)