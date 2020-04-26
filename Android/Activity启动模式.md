Activity启动模式
================================
设置方式：在AndroidManifest.xml文件中对应的<activity>标签设置android：lunchMode属性   

 * 可取值为

1. standard
	默认模式，打开的Activity依次压栈，点back键会依照栈顺序依次推出
2. singleTop
	如果当前Activity在栈顶，启动相同的Activity，不会创建新的Activity，而是会调用它的onNewIntent方法。
3. singleTask 
	首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈，如果不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去，如果存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例，如果存在实例，则将它上面的Activity实例都出栈，然后回调启动的Activity实例的onNewIntent方法，如果不存在该实例，则新建Activity，并入栈。
 注：尽管Activity在新task中启动，但按返回按钮还是会返回上一个Activity
4. singleInstance
	只有一个实例，单独有一个栈保存其实例，栈中有且只有这一个Activity。SingleInstance模式启动的Activity在系统中具有全局唯一性。SingleInstance Activity启动其他Activity时，会在intent中加上FLAG\_ACTIVITY\_NEW\_TASK标志
 
 
可在intent中通过setFlag设置即将启动的Activity的模式，优先级高于AndroidManifest.xml文件中定义的启动模式。

 * FLAG\_ACTIVITY\_NEW\_TASK	   
 目标Activity和启动Activity的taskAffinity相同，该属性不起作用       
 目标Activity和启动Activity的taskAffinity不同，会在目标栈中查看是否有该Activity的实例，如果有，无法启动，如果没有，才会启动并压入栈中
 * FLAG\_AVTIVITY\_SINGLE\_TOP    
 与singleTop相同
 * FLAG\_ACTIVITY\_CLEAR\_TOP    
 如果启动的Activity在栈中已经存在，则会把栈中所有该Activity之上的其他Activity移除，如果没有设置FLAG\_AVTIVITY\_SINGLE\_TOP且启动的Activity的模式是standard，该Activity也会被移除，然后创建一个新的实例，否则会调用其onNewIntent方法。(只判断启动该Activity的Activity所在task)    
 * FLAG\_ACTIVITY\_CLEAR\_TASK       
  目标Activity和启动Activity的taskAffinity相同，该属性不起作用    
  目标Activity和启动Activity的taskAffinity不同，和FLAG\_ACTIVITY\_NEW\_TASK一起使用，如果启动的Activity不在栈中，直接启动，如果已经在栈中，会把当前栈中的所有Activity先移除掉，然后将该Activity放入新的栈中



android : taskAffinity属性   
-------------------------------------
 standard和singleTop启动模式都是在原任务栈中新建Activity实例，不会启动新的Task，即使你指定了taskAffinity属性。
 
 activity没有指定taskAffinity属性时，默认为父标签application的taskAffinity属性，当application也没指定时，默认为包名
 
 singleTask启动模式启动Activity时，首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈，如果不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去，如果存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例，如果存在实例，则将它上面的Activity实例都出栈，然后回调启动的Activity实例的onNewIntent方法，如果不存在该实例，则新建Activity，并入栈。  
此外，我们可以将两个不同App中的Activity设置为相同的taskAffinity，这样虽然在不同的应用中，但是Activity会被分配到同一个Task中去。                 


TaskAffinity还有一个功能，就是和allowTaskReparenting结合：
------------------------------------

allowTaskReparenting：官方定义是“Whether or not the activity can move from the task that started it to the task it has an affinity for when that task is next brought to the front — "true" if it can move, and "false" if it must remain with the task where it started.”简单翻译就是是否允许该Activity从启动他的任务（可以理解为activity栈）转移到与他有亲密关系（affinity）的任务中,当有亲密关系的任务再次启动时。 

举例说明：

ActvityA是应用1的主Actvity，ActivityB和ActvityC属于应用2，B为主Activity。

操作路径：A启动C-->点击Home键盘到Launcher->点击launcher上的应用2

情况1.ActivityC的allowTaskReparenting属性为false，此时会进入ActvityB     
典型案例：将文字文件等，分享到短信

原因：当前任务栈1为AC，此时启动应用2，会启动任务栈2，然后将主ActivityB放入任务栈2中

情况2：ActivityC的allowTaskReparenting属性为true，此时会进入ActvityC     
典型案例：将文字文件等，分享到微信

原因：当前任务栈1为AC，此时启动应用2，会启动任务栈2，然后系统发现C的taskAffinity属性任务栈2已经创建，就把C从任务栈1转移过来，这就是allowTaskReparenting的功能。