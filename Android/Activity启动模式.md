#Activity启动方式
设置方式：在AndroidManifest.xml文件中对应的<activity>标签设置android：lunchMode属性   

 * 可取值为

 1. standard
 
 默认模式，打开的Activity依次压栈，点back键会依照栈顺序依次推出
 2. singleTop

 如果当前Activity在栈顶，启动相同的Activity，不会创建新的Activity，而是会调用它的onNewIntent方法。
 3. singleTask

 只有一个实例，启动当前Activity时，如果栈中没有实例，将会创建一个新的Activity入栈，如果栈中有其实例，则会把栈中所有该Activity之上的其他Activity移除，并调用其onNewIntent方法。
 注：如果从其他应用打开该Activity，会新建一个task存放该Activity
 
 4. singleInstance

 只有一个实例，单独有一个栈保存其实例，栈中有且只有这一个Activity。SingleInstance模式启动的Activity在系统中具有全局唯一性。




###android : taskAffinity属性   
 standard和singleTop启动模式都是在原任务栈中新建Activity实例，不会启动新的Task，即使你指定了taskAffinity属性。
 
 singleTask启动模式启动Activity时，首先会根据taskAffinity去寻找当前是否存在一个对应名字的任务栈，如果不存在，则会创建一个新的Task，并创建新的Activity实例入栈到新创建的Task中去，如果存在，则得到该任务栈，查找该任务栈中是否存在该Activity实例，如果存在实例，则将它上面的Activity实例都出栈，然后回调启动的Activity实例的onNewIntent方法，如果不存在该实例，则新建Activity，并入栈。  
此外，我们可以将两个不同App中的Activity设置为相同的taskAffinity，这样虽然在不同的应用中，但是Activity会被分配到同一个Task中去。 