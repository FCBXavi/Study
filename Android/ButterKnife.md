#ButterKnife

	public class MainActivity extends AppCompatActivity {

    	@BindView(R.id.tv_content)
    	TextView tvContent;

    	@Override
    	protected void onCreate(Bundle savedInstanceState) {
        	super.onCreate(savedInstanceState);
        	setContentView(R.layout.activity_main);
        	ButterKnife.bind(this);
        	//new MainActivity_ViewBinding(this, getWindow().getDecorView());
    	}
	}

 以MainActivity为例，从    
 
 	ButterKnife.bind(this);
 入手，看一下源码
 
 	@NonNull @UiThread
 	public static Unbinder bind(@NonNull Activity target) {
   		View sourceView = target.getWindow().getDecorView();
    	return createBinding(target, sourceView);
  	}
  	
 返回了一个Unbinder对象，看一下createBinding这个方法
 
 	//只列出关键代码
 	private static Unbinder createBinding(@NonNull Object target, @NonNull View source) {
 	
    	Class<?> targetClass = target.getClass();
    	Constructor<? extends Unbinder> constructor = findBindingConstructorForClass(targetClass);
    	if (constructor == null) {
      		return Unbinder.EMPTY;
    	}
      	return constructor.newInstance(target, source);
    }
    
调用findBindingConstructorForClass这个方法获取到一个构造函数，然后根据这个构造函数生成对应类的一个实例返回，我们继续看一下findBindingConstructorForClass这个方法

	private static Constructor<? extends Unbinder> findBindingConstructorForClass(Class<?> cls) {
		//BINDINGS是一个以class为key，构造函数为value的map
    	Constructor<? extends Unbinder> bindingCtor = BINDINGS.get(cls);
    	if (bindingCtor != null) {
      		return bindingCtor;
    	}
    	String clsName = cls.getName();
    	if (clsName.startsWith("android.") || clsName.startsWith("java.")) {
      		return null;
    	}
    	try {
      		Class<?> bindingClass = Class.forName(clsName + "_ViewBinding");
      		//noinspection unchecked
     	 	bindingCtor = (Constructor<? extends Unbinder>) bindingClass.getConstructor(cls, View.class);
    	} catch (ClassNotFoundException e) {
     		bindingCtor = findBindingConstructorForClass(cls.getSuperclass());
    	} 
    	BINDINGS.put(cls, bindingCtor);
    	return bindingCtor;
  	}
  	
先在map中找是否已经存在这个key，即传入的Activity已经调用过bind方法，如果有的话，直接返回，没有的话，如果类是android包或者java包下的类，返回null，如果不是的话，继续往下，MainActivity_ViewBinding的类，如果找到的话，就返回这个类的构造方法，然后放在map中，返回当前构造方法，如果没有找到的话，继续从当前类的父类寻找。

也就是说，bind方法帮我们生成了一个 MainActivity_ViewBinding的实例

我们看一下这个MainActivity_ViewBinding类

	public class MainActivity_ViewBinding implements Unbinder {
  		private MainActivity target;

 	 	@UiThread
  		public MainActivity_ViewBinding(MainActivity target) {
    		this(target, target.getWindow().getDecorView());
  		}

  		@UiThread
  		public MainActivity_ViewBinding(MainActivity target, View source) {
    		this.target = target;
    		target.tvContent = Utils.findRequiredViewAsType(source, R.id.tv_content, "field 'tvContent'", TextView.class);
  		}

  		@Override
  		@CallSuper
  		public void unbind() {
    		MainActivity target = this.target;
    		this.target = null;
    		target.tvContent = null;
  		}
	}
	
构造方法我们传入了MainActivity和其View，然后调用Utils.findRequiredViewAsType方法把获取到的View赋值给tvContent，实际上在其内部调用了findViewById方法，根据Id找到了View并进行了类型转换，这里直接操作了MainActivity中的View，因此被@BindView注解的View不能是private的


那么现在还有一个问题 MainActivity_ViewBinding是如何生成的，当然是通过注解。ButterKnife使用了编译时注解。ButterKnifeProcessor继承了AnnotationProcessor，在编译时遍历所有的ButterKnife相关注解，生成了这些java文件。

