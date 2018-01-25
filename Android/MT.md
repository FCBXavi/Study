MT
===================
View几个构造方法	
-----------------------

	//在代码中生成View	
	public View(Context context)
	//在xml文件中使用的时候被调用，attrs是在xml中定义的属性集合
	public View(Context context, @Nullable AttributeSet attrs)
	//defStyleAttr是定义在attrs.xml中的attribute，这个值起作用需要两个条件：值不为0且在Theme中使用(出现即可)。
	public View(Context context, @Nullable AttributeSet attrs, int defStyleAttr)
	//defStyles是在styles.xml中定义的一个style，当defStyleAttr没有起作用时，使用这个值
	public View(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes)
merge是否可以嵌套
-----------------------
   
merge标签只能作为根标签，在另一个xml文件中使用include标签引入，此时会自动忽略merge标签，merge标签里可以include其余merge标签        
当inflate以merge为根标签的布局时，必须指定父元素且attachToParent为true(inflate(int, android.view.ViewGroup, boolean) 方法)          
ViewStub不能inflate根布局是merge标签的布局(其余布局包含merge标签布局可以)                   	
Android有哪三种动画
--------------------------      
 
Android3.0(API11)之前，Android只支持2种动画，分别是Frame Animation(逐帧动画)和Tween Animation(补间动画)，在3.0后增加了一种新的动画系统，Property Animation(属性动画)		

1. Frame Animation		  
   一帧帧地播放图片，利用视觉残留带来动画的感觉

		<animation-list
			xmlns:android="http://schemas.android.com/apk/res/android"  
			android:oneshot="false">  
			<!-- oneshot 设置是否仅播放一次 -->
			<!-- 添加多个帧 -->  
			<item android:drawable="@drawable/frame01" android:duration="60" />  
			<item android:drawable="@drawable/frame01" android:duration="60" />  
    		<item android:drawable="@drawable/frame01" android:duration="60" />  
		</animation-list>  
	
	习惯上把AnimationDrawable设置为ImageView的背景        

		android:background="@anim/frame_anim"
		
	然后就可以在java代码中获取AnimationDrawable对象了      

		AnimationDrawable anim = (AnimationDrawable)imageView.getBackground();
		anim.start();//默认不播放，需要调用start方法开始播放
		anim.stop();

	也可以在java代码中直接创建AnimationDrawable对象，然后通过addFrame(Drawable frame, int duration)方法添加帧，然后start          

2. Tween Animation
	四种子类：AlphaAnimation、ScaleAnimation、TranslateAnimation、RotateAnimation。只改变view的绘画效果，不会改变view的真实属性。假设把一个Button从左边移动到右边，点击移动后位置的Button是没有反应的，只有点击移动前Button的位置才有反应，因为Button的属性没有改变
			
		<set xmlns:android="http://schemas.android.com/apk/res/android"  >  
    		<!-- 定义透明度的变换 -->  
    		<alpha   
        		android:fromAlpha="1"   
        		android:toAlpha="0"   
        		android:duration="3000"/>   
		</set>
		Animation anim = AnimationUtils.loadAnimation(context, R.anim.anim);
		anim.setFillAfter(true);//设置动画结束后保留状态
		anim.setInterpolater(interpolater);
		view.startAnimation(anim);
		
		
		直接用代码实现：
		TranslateAnimation translateAnimation =
              new TranslateAnimation(
                  Animation.RELATIVE_TO_SELF,0f,
                  Animation.RELATIVE_TO_SELF,100f,
                  Animation.RELATIVE_TO_SELF,0f,
                  Animation.RELATIVE_TO_SELF,100f);
           translateAnimation.setDuration(1000);
           view.startAnimation(translateAnimation);
 
3. Property Animation
	与Tween Animation不同的是Property Animation可以改变View的属性
	1. ValueAnimator   
		
		包含Property Animation动画的核心功能，如持续时间，开始和结束的属性值等     
		
			ValueAnimator animator = ValueAnimator.ofObject(new TypeEvaluator() {
          		@Override
            	public Object evaluate(float fraction, Object startValue, Object endValue) {
                	return null;
            	}
        	} , 0 , 100);
			animator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
        		@Override
        		public void onAnimationUpdate(ValueAnimator animation) {
          	//设置View的属性
        		}
			});
			animator.setDuration(2000);
			animator.start();
	2. ObjectAnimator
   		继承自ValueAnimator，要制定一个对象及该对象的一个属性，当属性值计算完成后自动设置为该对象的相应属性，使用时需要满足1.对象要有一个setter函数set<PropertyName>2.对于后面的可变参数，对象要有相应的getter方法get<PropertyName>，返回值的类型与setter方法的参数一致。如不满足这个条件，只能使用ValueAnimator创建动画。cancle动画立即停止，停在当前状态，end动画直接到最终状态。如果调用了cancle。再次开始必须先调用reset再调用start。                  
   		可以addListener加入AnimatorListener，在动画开始或结束或者重复或者取消的时候做一些事情。AnimationSet可以组合多个动画共同工作。       
   		
   			AnimatorSet bouncer = new AnimatorSet();  
			bouncer.play(anim1).before(anim2);  
			bouncer.play(anim2).with(anim3);  
			bouncer.play(anim2).with(anim4)  
			bouncer.play(anim5).after(amin2);  
			animatorSet.start(); 
			先播放anim1，再播放anim234，再播放anim5
		
一个Activity居中显示一个TextView，怎样布局最合适 
---------------------------
       
Activity最外层布局是一个FrameLayout，如果只显示一个TextView，可以只使用一个TextView，指定其android:layout_gravity = "center"，而不需要在TextView外嵌套任何布局。       
算法：判断对称二叉树
---------------------------
	//判断一个数是否为镜像对称：先判断根，在判断左右子树。如果左右子树都为空那就是，如果左右子树不是同时为空那就不是
    //当左右子树都存在的时候，判断他们的值是否相等,如果相等那么就递归的对他们的字节点判断（左边的左=右边的右；左边的右==右边的左）
    public bool isSymmetric(TreeNode root) {
        if (root == null)
            return true;
        return Symmetric(root.left, root.right);
    }
    public bool Symmetric(TreeNode left, TreeNode right){
        if (left == null && right == null)
            return true;
        if (left == null || right == null)
            return false;
        if (left.val == right.val){
            return (Symmetric(left.left, right.right) && Symmetric(right.left, left.right));
        }
        return false;
    }

gradle compile和provider的区别，写一个helloworld  
---------------------------
ACTION_CANCEL，CANCEL后会不会调用UP    
---------------------------
ACTION_CANCEL指手指保持按下操作，但是滑动到控件之外，表示焦点离开button，最终不会调用up
volatile和指令重排   
---------------------------
对于多个线程共享的变量，存储在主内存中，每个线程有自己独立的工作内存（如CPU寄存器），每个线程只能访问自己的工作内存，不能访问其他线程的工作内存。工作内存中保存了主内存共享变量的副本，线程操作这些共享变量通过操作工作内存中的副本来实现，操作完毕之后同步回主内存中。volatile保证可见性的原理是在每次访问变量时都会进行一次刷新，保证每次访问的都是主存中的最新版本。        
指令重排是JVM为了优化指令，提高程序运行效率，在不影响单线程程序执行结果的前提下，尽可能提高并行度。包括编译器重排和运行时重排但多线程的情况下指令重排。可能会带来问题。比如在懒加载双重判断单例模式中：        
	
	public class Singleton {

  		private static Singleton instance = null;

  		private Singleton() {}

  		public static Singleton getInstance() {
     		if(instance == null) {
        		synchronzied(Singleton.class) {
           			if(instance == null) {
               			instance = new Singleton();  //非原子操作
           			}
        		}
     		}
     		return instance;
   		}

	}
	
instance = new Singleton()不是一个原子操作，实际上是三条JVM指令：       

	memory =allocate();    //1：分配对象的内存空间 
	ctorInstance(memory);  //2：初始化对象 
	instance =memory;     //3：设置instance指向刚分配的内存地址
上面的操作2依赖于1，但3并不依赖2，所以有可能经过指令重排后执行顺序变成了132，线程A执行到3时，线程B判断instance不为null，直接使用，导致程序出错。在jdk1.5之后，使用volatile可以禁止指令重排。        
volatile和synchronized的区别：           
1. volatile不会执行加锁操作，不会使线程阻塞。          
2. volatile作用类似于同步变量读写操作。写入volatile相当于退出同步代码块，读取volatile变量相当于进入同步代码块。         
3. volatile不如synchronized安全。           
4. volatile无法同时保证内存可见性和原则性。         

使用volatile的情况：       
1. 对变量的写入操作不依赖于变量的当前值，或者你能确保只有单个线程更新变量的值。                
2. 该变量没有包含在具有其他变量的不变式中。
EventBus源码   
---------------------------
写一个单例，不能用静态内部类和枚举 
---------------------------
