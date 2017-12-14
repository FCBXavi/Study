#Activity启动方式
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

 只有一个实例，单独有一个栈保存其实例，栈中有且只有这一个Activity。SingleInstance模式启动的Activity在系统中具有全局唯一性。
 
 
可在intent中通过setFlag设置即将启动的Activity的模式，优先级高于AndroidManifest.xml文件中定义的启动模式。

 * FLAG\_ACTIVITY\_NEW\_TASK	   
 与singleTask相同
 * FLAG\_AVTIVITY\_SINGLE\_TOP    
 与singleTop相同
 * FLAG\_ACTIVITY\_CLEAR\_TOP    
 如果启动的Activity在栈中已经存在，则会把栈中所有该Activity之上的其他Activity移除，如果没有设置FLAG\_AVTIVITY\_SINGLE\_TOP且启动的Activity的模式是standard，该Activity也会被移除，然后创建一个新的实例，否则会调用其onNewIntent方法。(只判断启动该Activity的Activity所在task)
 
 Intent.FLAG\_ACTIVITY\_REORDER\_TO\_FRONT




###android : taskAffinity属性   
 standard和singleTop启动模式都是在原任务栈中新建Activity实例，不会启动新的Task，即使你指定了taskAffinity属性。
 
 activity没有指定taskAffinity属性时，默认为父标签application的taskAffinity属性，当application也没指定时，默认为包名
 
 singleTask启动模式启动Activity时，首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈，如果不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去，如果存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例，如果存在实例，则将它上面的Activity实例都出栈，然后回调启动的Activity实例的onNewIntent方法，如果不存在该实例，则新建Activity，并入栈。  
此外，我们可以将两个不同App中的Activity设置为相同的taskAffinity，这样虽然在不同的应用中，但是Activity会被分配到同一个Task中去。 