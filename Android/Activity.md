Activity
Activity生命周期中的7个方法，四大状态（可见、可见不可操作、不可见、销毁）。

onCreat() -> onStart() -> [可见不可操作] -> onResume() -> [可操作] -> onPause() -> [可见不可操作] -> onStop() -> [不可见] -> onDestroy() -> [销毁]
如何判断activity是否被销毁

	if(activity == null || activity.isFinishing || activity.isDestroyed) {
    	return;
	}


正常情况下ActivityA启动ActivityB生命周期流程         
A:onPause         
B:onCreate->onStart->onResume         
A:onStop        
按下返回键       
B:onPause       
A:onRestart->onStart->onResume       
B:onStop->onDestroy    

A的launchMode为SingleTop且A在栈顶打开A时      
onPause->onNewIntent->onResume           

A的launchMode为SingleTask且栈中已有一个A的实例，从B打开A时     
B:onPause       
A:onNewIntent->onRestart->onStart->onResume      
B:onStop->onDestroy      
  
onCreate中finish        
onCreate()->onDestroy()       

onStart中finish       
oncreate->onStart->onStop->onDestroy      

onResume中finish  
oncreate->onStart->onResume->onPause->onStop->onDestroy           

onPause、onStop、onRestart中finish，生命周期执行顺序不变           

	之所以是这样，源码中给出了解释，activity会判断状态，只有没有被finish才会执行下一个生命周期。
	mInstrumentation.callActivityOnCreate(activity, r.state) // 函数中会判断：
	if (!r.activity.mFinished) {
		activity.performStart();
		r.stopped = false;
	}
	/**执行完 onCreate()后，判断这时activity有没有finish，没有就会接着执行 onStart()，否则会调用destory()
	执行完onStart()后会执行handleResumeActivity函数,其中performResumeActivity 函数中：*/
	if (r != null && !r.activity.mFinished) {
		r.activity.performResume();
	}
	/**会调用 onResume 如果此时finish，就不会执行finish()，
	会调用ActivityManagerNative.getDefault().finishActivity(token, Activity.RESULT_CANCELED, null);执行销毁 */

 
* 异常情况下的生命周期

1. 资源相关的系统配置发生改变导致Activity被杀死重建     
	activity被销毁后会调用onPause，onStop，onDestroy方法，在onStop之前会调用onSaveInstanceState方法保存View的状态（并不一定能保证在onPause方法之前或之后调用），onSaveInstanceState只有在Activity异常情况下被终止才会被调用，正常情况下是不会被调用的。创建时会把保存状态的bundle传给oncreate方法和onRestoreInstanceState方法，onRestoreInstanceState在onStart之后调用
	
	如何让系统资源配置发生改变时不重新创建Activity？     
	在AndroidManifest文件中activity节点下增加android:configChanges设置某些属性发生变化时activity不重建。在onConfigurationChanged(Configuration newConfig)方法中可以接收到configChange发生变化的情况。
	
2. 内存不足导致Activity被杀死           
 activity优先级：      
 a. 前台Activity——正在和用户交互的Activity，优先级最高     
 b. 可见但非前台Activity——比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法与用户直接交互。      
 c. 后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低。       
 
 activity数据存储和恢复过程与上述一样         
 
 
 
 View的生命周期     
 -----------------------------    
 onFinishInflate(Activity的OnCreate之后)->(Activity的onResume之后)onAttachedToWindow->onMeasure->onSizeChanged->onLayout->onDraw->(Activity的onDestroy之后)onDetachToWindow
 

	