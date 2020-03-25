view inflate     
=======================
获取LayoutInflate       
	
	LayoutInflater layoutInflater = LayoutInflate.from(context);
	相当于
	LayoutInflater layoutInflater = (LayoutInflater)context.getSystemService(Context.LAYOUT_INFLATER_SERVICE)
	
加载布局    
	
	// 调用了inflate(resourceId, root, root != null)
	layoutInflater.inflate(int resourceId, View root) 
	layoutInflater.inflate(int resourceId, View root, boolean attachToRoot)
	
inflate过程，解析传入的resource文件，判断根节点是否是merge，是的话root不能为null，attachToRoot必须是true，否则会抛出异常。 递归解析xml文件，使用反射创建view并设置属性。
对于创建出的xml文件中的最外层view，如果root不为null，如果attachToRoot是false，那么只给该view设置相应的LayoutParam属性，将来该view被add到其他布局的时候会生效，如果attachToRoot是true，则会调用root的addView方法，把该view add进来，如果root为null，inflate进来的布局是没有layoutParam的，此时attachRoot设置任何值都没有意义。      

viewGroup add view的时候，如果view被inflate的时候，root为null，那么view根布局的layout\_width和layout\height是失效的，在addView的时候会被默认指定成wrap\_content。
一个view只有处于另外一个布局的包裹之下，它的layout\_width和layout\height才会生效。
