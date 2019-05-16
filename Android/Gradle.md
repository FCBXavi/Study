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
   
   
根目录的build.gradle
------------------------------

	// 定义gradle脚本执行所需依赖
	buildscript {
		// 定义仓库，代表依赖包的来源
		repositories {
			jcenter()
		}
		dependencies {
			classpath 'com.android.tools.build:gradle:1.2.3'
		}
	}
	// 所有项目需要的依赖
	allprojects {
     	repositories {
          jcenter() 
     	}
	}
	
	
模块内的build.gradle	      
---------------------------
	
	// 表示是Android应用插件，提供Android应用和依赖库的构建、打包和测试
	apply plugin: 'com.android.application'
	android {
       compileSdkVersion 22
       buildToolsVersion "22.0.1"
       // app核心属性，会重写AndroidManifest.xml中的对应属性
       defaultConfig {
       	applicationId "com.gradleforandroid.gettingstarted"
         	minSdkVersion 14
         	targetSdkVersion 22
         	// 版本号标识
         	versionCode 1
         	versionName "1.0"
         	}
      	// 如何构建不同版本的app
      	buildTypes {
      		release {
          	minifyEnabled false
            	proguardFiles getDefaultProguardFile
                ('proguard-android.txt'), 'proguard-rules.pro'
            	}
      	} 
	}
	// gradle默认属性
	dependencies {
     	compile fileTree(dir: 'libs', include: ['*.jar'])
     	compile 'com.android.support:appcompat-v7:22.2.0'
     	}
     	
   
tasks
----------------
android插件依赖于Java插件，Java插件依赖于base插件             
base插件定义了assemble和clean任务，Java插件定义了check和build任务        

assemble：集合所有的output         
clean：清除所有的output        
check：执行所有的checks检查，通常是unit测试和instrumentation测试         
build：执行所欲哦的assemble和check             

Android插件继承了这些基本的tasks，并实现了自己的行为       
assemble：针对每个版本创建一个apk         
clean：删除所有的构建任务，包括apk文件          
check：执行Lint检查，检测到错误后停止执行脚本       
build：执行assmeble和check      

  
全局设置
-------------------
在根目录的gradle文件中
       
	ext {
		compileSdkVersion = 22
		buildToolsVersion = "22.0.1"
	}
	
在子模块的gradle文件中 

	android {
     	compileSdkVersion rootProject.ext.compileSdkVersion
     	buildToolsVersion rootProject.ext.buildToolsVersion
    }
  

仓库
-------------------
Gradle支持三种不同的仓库，分别是：Maven和Ivy以及文件夹。依赖包会在你执行build构建的时候从这些远程仓库下载，当然Gradle会为你在本地保留缓存，所以一个特定版本的依赖包只需要下载一次。        

声明依赖，一个依赖有三个元素，group，name和version

	dependencies {
       compile 'com.google.code.gson:gson:2.3'
       compile 'com.squareup.retrofit:retrofit:1.9.0'
	}
	
上述的代码是基于groovy语法的，其完整表述是：

	dependencies {
      compile group: 'com.google.code.gson', name: 'gson', version:'2.3'
      compile group: 'com.squareup.retrofit', name: 'retrofit'
           version: '1.9.0'
     }

  
   
[十分钟理解Gradle](https://www.cnblogs.com/Bonker/p/5619458.html)                   

[Gradle生命周期](http://blog.csdn.net/u013626215/article/details/51490643)