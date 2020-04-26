#Android View绘制流程
流程从ViewRoot的performTraversales()开始      
过程onMeasure->onLayout->onDraw

###measure过程
onMeasure(int widthSpec, int heightSpec)    
MeasureSpec32位int，高两位代表Mode，后30位代表Size
是从父View到子View传递的值
Mode共有三种形式
>UPSPECIFIED : 父容器对于子容器没有任何限制,子容器想要多大就多大    
>EXACTLY: 父容器已经为子容器设置了尺寸,子容器应当服从这些边界,不论子容器想要多大的空间。    
>AT_MOST：子容器可以是声明大小内的任意大小

measure过程：	
    
ViewGroup有一个measureChildren方法，测量子view的大小，根据父View传给自己的measureSpec，padding，子View的LayoutParams，计算出子视图的measureSpec，调用子视图的measure方法，把该值传入

1. 父View的MeasureSpec是EXACTLY		
 * 子View的layout\_xxx是match\_parent，size是父View的size，mode是EXACTLY
 * 子View的layout\_xxx是wrap\_content，size暂定为父View的大小（最大值），mode是AT\_MOST
 * 子View的layout\_xxx是固定值，size固定，mode是EXACTLY
2. 父View的MeasureSpec是AT_MOST			
 * 子View的layout\_xxx是match\_parent，size暂定为父View的大小（最大值），mode是AT\_MOST
 * 子View的layout\_xxx是wrap\_content，size暂定为父View的大小（最大值），mode是AT\_MOST
 * 子View的layout\_xxx是固定值，size固定，mode是EXACTLY
3. 父View的MeasureSpec是UNSPECFIED			
 * 子View的layout_xxx是match\_parent，size=0，mode是UNSPECFIED
 * 子View的layout_xxx是wrap\_content，size=0，mode是UNSPECFIED
 * 子View的layout_xxx是固定值，size固定，mode是EXACTLY

子View得到MeasureSpec后，如果MODE是UNSPECFIED，View的高度（宽度）由设置的最小高度（最小宽度）和view的background一起决定，不是的话，就取MeasureSpec的SIZE。

最外层的根视图，widthMeasureSpec和heightMeasureSpec是从ViewRootImp中传入的，传入的是WindowManager.LayoutParams，都是MATCH\_PARENT，因此最外层DecorView的MeasureSpec的mode是EXACTLY，size是DecorView的size。      
   
父View等所有子View都测量结束后，再测量自己        
      
只有setMeasureDimension()调用之后，我们才能使用getMeasuredWidth和getMeasuredHeight获取View的宽高，在这之前获取都是0


###layout过程
根据子视图的大小以及布局参数将View树放到合适的位置上。ViewRoot的performTraversals方法会在measure结束后继续执行，并调用View的layout()方法来执行此过程     
    

根据设置的Gravity及其他参数和MeasuredHeight、MeasuredWidth 一起来确定子View在父视图的具体位置的。onLayout之后，就可以通过getWidth()和getHeight()来获取宽高了，getWidth()通过View右边坐标减左边坐标计算，getHeight()通过View下面坐标减上面坐标计算。可以自己指定layout的四个参数，这时getMeasuredWidth() 和getWidth() 就很有可能不是同一个值。


###draw过程
1. 绘制背景；
2. 如果有必要，保存画布的layer准备fading;
3. 绘制View内容;
4. 绘制子View;
5. 如果需要，绘制fading edges，恢复layer;
6. 绘制滚动条




invalidate重新绘制，只能在主线程调用，postInvalidate可以在子线程调用，当View的大小形状或者位置发生改变时，调用requestLayout。


onMeasure和onLayout可能会执行多次