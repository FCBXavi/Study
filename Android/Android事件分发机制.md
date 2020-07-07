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

onTouchEvent() 处理事件，返回true代表消费事件，事件停止传递，返回false代表不消费事件，事件继续传递给上层的onTouchEvent处理，<font color=red>如果在down事件中返回false，当前View不会接收到此次事件序列的其他事件</font>。    

ViewGroup的dispatchTouchEvent()方法：在down事件时会遍历所有的子View，判断哪个按下坐标在哪个子View的范围内，然后调用该子View的dispatchTouchEvent方法，如果返回true，表示该View处理事件，就把View加在mFirstTouchTarget链表头部(倒序添加)。如果子View的dispatchTouchEvent返回false，表示子View不消费，就不会加在mFirstTouchTarget链表中，同时交给super.dispatchTouchEvent(event)也就是父类View来处理。      
后续事件来的时候，如果mFirstTouchTarget不为null，而且不拦截，直接遍历mFirstTouchTarget链表，执行对应View的dispatchTouchEvent方法。



###Touch事件的后续传递（MOVE，UP）
dispatchTouchEvent消费事件(返回true)后，后续还能接收到其他事件		
onTouchEvent消费事件(返回true)后，后续的事件到达此View后不会再向下传递，直接交给该View的onTouchEvent方法并结束本次事件传递过程

onInterceptTouchEvent一旦返回一次true，就再也不会再被调用了，<font color=red>onInterceptTouchEvent返回true，会清空mFirstTouchTarget链表，下次在dispatchTouchEvent中的临时变量intercepted直接为true</font>     
如果ViewGroup拦截了一个半路的事件（如MOVE），这个事件将会被系统变成一个CANCEL事件传递给之前处理该事件的子VIEW，并不会直接传递给ViewGroup的onTouchEvent，只有后续再到来的事件才会传递给ViewGroup的onTouchEvent




###onTouch和onClick
onTouch()的执行高于onClick()
onTouch()返回true，会让View的dispatchTouchEvent方法直接返回true，不会调用onTouchEvent()，onClick也不会执行，onTouch如果返回false，就会在dispatchTouchEvent中执行onTouchEvent，如果注册了OnClickListener，就会在onTouchEvent中调用performClick，在里面回调onClick


###滑动冲突解决     

外部拦截法：当事件传递到父容器时，通过父容器去判断自己是否需要此事件，若需要则拦截事件，不需要则不拦截事件，将事件传递给子View
	
	public boolean onInterceptTouchEvent(MotionEvent ev) {
        boolean isIntercept = false;
        switch (ev.getAction()) {
            case MotionEvent.ACTION_DOWN:
                //DOWN事件不能拦截，否则事件将无法分发到子View
                isIntercept = false;
                break;
            case MotionEvent.ACTION_MOVE:
                //根据条件判断是否拦截事件
                isIntercept = needThisEvent();
                break;
            case MotionEvent.ACTION_UP:
                //一旦父容器拦截了UP事件，子View将无法触发点击事件
                isIntercept = false;
                break;
            default:
                break;
        }
        return isIntercept;
    }
	
 内部拦截法：ViewGroup默认情况下不拦截事件，由子View去控制事件的处理，若子View需要此事件，则自己处理，否则交由父容器处理。       

	// 子View
	@Override
	public boolean dispatchTouchEvent(MotionEvent ev) {
	    switch (ev.getAction()) {
	        case MotionEvent.ACTION_DOWN:
	            //禁止父容器拦截事件
	            getParent().requestDisallowInterceptTouchEvent(true);
	            break;
	        case MotionEvent.ACTION_MOVE:
	            if (当前View不需要此事件) {
	                // 允许父容器拦截事件
	                getParent().requestDisallowInterceptTouchEvent(false);
	            }
	            break;
	        default:
	            break;
	    }
	    return super.dispatchTouchEvent(ev);
	    }
	
	// ViewGroup
	@Override
	public boolean onInterceptTouchEvent(MotionEvent ev) {
	    switch (ev.getAction()) {
	        case MotionEvent.ACTION_DOWN:
	            return false;
	        default:
	            return true;
	    }
	}