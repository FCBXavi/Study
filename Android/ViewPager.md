ViewPager
=============================
使用PagerAdapter更新数据时，调用notifyDataSetChanged方法，会遍历自己的Observer对象，在ViewPager setAdapter时，会向Adapter注册一个PagerObserver对象，因此这里会调用ViewPager的dataSetChanged方法，这个方法中会判断Adapter的getItemPosition的返回值，如果是POSITION\_UNCHANGED，则什么都不做，如果是POSITION\_NONE，则会调用PagerAdapter的destroyItem方法移除该对象，然后调用instantiateItem来生成新对象，因此，为了让PagerAdapter的notifyDataSetChanged方法生效，需要让getItemPosition返回POSITION\_NONE。


与Fragment一起使用时，可以使用FragmentPagerAdapter和FragmentStatePagerAdapter

FragmentPagerAdapter会把所生成的Fragmnet对象通过FragmentManager缓存起来，在instantiateItem()方法中通过attach方法显示，而FragmentStatePagerAdapter的instantiateItem()方法每次都会创建一个新的Fragment，然后通过FragmentManager add进来

FragmentPagerAdapter的destroyItem()方法调用的是FragmentManager的detach方法，Fragment所占资源不会被释放，而FragmentStatePagerAdapter的destroyItem()方法调用的是FragmentManager的remove方法，Fragment所占资源被释放。所以FragmentStatePagerAdapter适用于拥有大量页面的Viewpager