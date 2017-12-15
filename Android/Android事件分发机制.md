#Android事件分发机制
主要发生的Touch事件有如下四种：

* MotionEvent.ACTION_DOWN：按下View（所有事件的开始）
* MotionEvent.ACTION_MOVE：滑动View
* MotionEvent.ACTION_UP：抬起View（与DOWN对应）
* MotionEvent.ACTION_CANCEL：非人为原因结束本次事件

一个完整的事件序列：从手指接触屏幕至手指离开屏幕，这个过程产生的一系列事件 
任何事件列都是以DOWN事件开始，UP事件结束，中间有无数的MOVE事件

事件传递顺序			
Activity->Window->ViewGroup->View

重要方法：		
dispatchTouchEvent() 当事件传递给当前View时会被调用，返回true代表消费事件，事件不往下传递。返回false表示本层的事件不再往下分发，事件停止传递，由上层的onTouchEvent处理，<font color=red>但当前View还会接收到此次事件序列的其他事件</font>。返回super表示事件由本层的onInterceptTouchEvent决定是否继续向下分发。

onInterceptTouchEvent() ViewGroup特有，表示是否拦截事件，返回false表示不拦截，事件交由下层的View的dispatchTouchEvent处理，<font color=red>当前View还会接收到此次事件序列的其他事件</font>，返回true表示拦截，交由本层的onTouchEvent处理。

onTouchEvent() 处理事件，返回true代表消费事件，事件停止传递，返回false代表不消费事件，事件继续传递给上层的onTouchEvent处理，<font color=red>当前View不会接收到此次事件序列的其他事件</font>。



###Touch事件的后续传递（MOVE，UP）
dispatchTouchEvent消费事件(返回true)后，后续还能接收到其他事件		
onTouch消费事件(返回true)后，后续的事件到达此View后不会再向下传递，直接交给该View的onTouchEvent方法并结束本次事件传递过程

onInterceptTouchEvent一旦返回一次true，就再也不会再被调用了
如果ViewGroup拦截了一个半路的事件（如MOVE），这个事件将会被系统变成一个CANCEL事件传递给之前处理该事件的子VIEW，并不会直接传递给ViewGroup的onTouchEvent，只有后续再到来的事件才会传递给ViewGroup的onTouchEvent




###和onTouch和onClick
onTouch()的执行高于onClick()
onTouch()返回true，会让View的dispatchTouchEvent方法直接返回true，不会调用onTouchEvent()，onClick也不会执行，onTouch如果返回false，就会在dispatchTouchEvent中执行onTouchEvent，如果注册了OnClickListener，就会在onTouchEvent中调用performClick，在里面回调onClick
