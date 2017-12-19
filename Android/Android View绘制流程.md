#Android View绘制流程
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
父View的MeasureSpec传递给子View，结合子View的LayoutParams 一起再算出子View的MeasureSpec，然后继续传给子View，不断计算每个View的MeasureSpec，子View有了MeasureSpec才能更测量自己和自己的子View。

1. 父View的MeasureSpec是EXACTLY		
 * 子View的layout_xxx是match_parent，size=父View的size，mode是EXACTLY
 * 子View的layout_xxx是wrap_content，size暂定为父View的大小（最大值），mode是AT_MOST
 * 子View的layout_xxx是固定值，size固定，mode是EXACTLY
2. 父View的MeasureSpec是AT_MOST			
 * 子View的layout_xxx是match_parent，size暂定为父View的大小（最大值），mode是AT_MOST
 * 子View的layout_xxx是wrap_content，size暂定为父View的大小（最大值），mode是AT_MOST
 * 子View的layout_xxx是固定值，size固定，mode是EXACTLY
3. 父View的MeasureSpec是UNSPECFIED			
 * 子View的layout_xxx是match_parent，size=0，mode是UNSPECFIED
 * 子View的layout_xxx是wrap_content，size=0，mode是UNSPECFIED
 * 子View的layout_xxx是固定值，size固定，mode是EXACTLY

子View得到MeasureSpec后，如果MODE是UNSPECFIED，View的高度（宽度）由设置的最小高度（最小宽度）和view的background一起决定，不是的话，就取MeasureSpec的SIZE。


父View等所有子View都测量结束后，再测量自己。最外层DecorView的MeasureSpec的mode是EXACTLY，size是DecorView的size。


###layout过程
根据子视图的大小以及布局参数将View树放到合适的位置上。

根据设置的Gravity及其他参数和MeasuredHeight、MeasuredWidth 一起来确定子View在父视图的具体位置的。但是可以自己指定layout的四个参数，这时getMeasuredWidth() 和getWidth() 就很有可能不是同一个值


###draw过程
1. 绘制背景；
2. 如果有必要，保存画布的layer准备fading;
3. 绘制View内容;
4. 绘制子View;
5. 如果需要，绘制fading edges，恢复layer;
6. 绘制滚动条




invalidate重新绘制，只能在主线程调用，postInvalidate可以在子线程调用，当View的大小形状或者位置发生改变时，调用requestLayout。