Activity
Activity生命周期中的7个方法，四大状态（可见、可见不可操作、不可见、销毁）。

onCreat() -> onStart() -> [可见不可操作] -> onResume() -> [可操作] -> onPause() -> [可见不可操作] -> onStop() -> [不可见] -> onDestroy() -> [销毁]
如何判断activity是否被销毁
<pre>
if(activity == null || activity.isFinishing || activity.isDestroyed) {
    return;
}
</pre>


* 异常情况下的生命周期

1. 资源相关的系统配置发生改变导致Activity被杀死重建     
	activity被销毁后会调用onPause，onStop，onDestroy方法，在onStop之前会调用onSaveInstanceState方法保存View的状态（并不一定能保证在onPause方法之前或之后调用），onSaveInstanceState只有在Activity异常情况下被终止才会被调用，正常情况下是不会被调用的。创建时会把保存状态的bundle传给oncreate方法和onRestoreInstanceState方法，onRestoreInstanceState在onStart之后调用
	
	如何让系统资源配置发生改变时不重新创建Activity？     
	在AndroidManifest文件中activity节点下增加android:configChanges设置某些属性发生变化时activity不重建。在onConfigurationChanged(Configuration newConfig)方法中可以接收到configChange发生变化的情况。
	
2. 内存不足导致Activity被杀死			
activity优先级：
 1. 前台Activity——正在和用户交互的Activity，优先级最高
 2. 可见但非前台Activity——比如Activity中弹出了一个对话框，导致Activity可见但是位于后台无法与用户直接交互。
 3. 后台Activity——已经被暂停的Activity，比如执行了onStop，优先级最低。
 
 activity数据存储和恢复过程与上述一样

	