ANR
===============================
全名为Application Not Responding，即应用无响应。    
产生原因：  	       
1. 5s内无法响应用户输入事件。				
2. BroadcastReceiver在10s内无法结束。      

ANR分析
------------------------------
ANR发生时，系统会生成一个traces.txt的文件存放在/data/anr下，可以导出到本地

	$adb pull data/anr/traces.txt.
	
主要类型：           
1. 普通阻塞导致的ANR。           
2. CPU满负荷，一般是由频繁的文件读写或数据库读写操作放在了主线程。        
3. 内存原因，但更多的是会造成OOM。   
解决方法一般是开辟独立子线程来处理耗时任务。       

运行在主线程的方法：        
1. Activity所有生命周期的回调。		
2. Service。            
3. BroadcastReceiver的onReceiver方法              
4. 没有使用子线程的looper的Handler的handleMessage, post(Runnable)是执行在主线程的.          
5. AsyncTask的回调中除了doInBackground, 其他都是执行在主线程的.           
6. View的post(Runnable)是执行在主线程的.        

使用子线程的方式：         
1. 启动Thread。      
2. 使用AsyncTask。       
3. HandlerThread。       
4. IntentService。    


监测工具：ANRWatchDog           
原理：创建一个监测线程，该线程不断往UI线程post一个任务，然后睡眠5s，等该线程重新起来后检测之前post的任务是否执行了，如果任务未被执行，则生成ANRError,并终止进程。
	

