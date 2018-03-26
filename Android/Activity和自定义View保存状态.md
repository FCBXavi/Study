Activity和自定义View保存状态
在Activity被回收之前，系统会调用onSaveInstanceState(Bundle outState)来保存View的状态，并到传入的outState对象中。     

1.保存Window
--------------------------------
在Activity被回收时，会触发一个SaveState的事件。
跟其他的事件一样，SaveState事件从Activity->Window->View传递到最大的View，然后遍历View树保存状态
状态保存在一个SparseArray中，以View的ID作为key。
自定义View可以重载onSaveInstanceState()来保存自己的状态，参考TextView的实现方法。

2.保存Fragment
------------------------------

3.调用外部注册的回调方法
------------------------------
<pre>
protected void onSaveInstanceState(Bundle outState) {
    outState.putBundle(WINDOW_HIERARCHY_TAG, mWindow.saveHierarchyState());
    Parcelable p = mFragments.saveAllState();
    if (p != null) {
        outState.putParcelable(FRAGMENTS_TAG, p);
    }
    getApplication().dispatchActivitySaveInstanceState(this, outState);
}
</pre>


onRestoreInstanceState方法
在Activity被重新创建时，会通过onCreate(Bundle savedInstanceState)和onRestoreInstanceState(Bundle savedInstanceState)传入保存的状态信息并恢复View的状态。           
1. onCreate重建Fragment       
2. onRestoreInstanceState恢复Window状态           


Window在save和restore时对View的处理

1. Save时，遍历View的树状结构调用 Parcelable onSaveInstanceState()
2. 以View的id为key在Window的SparseArray<Parcelable>中保存这些 Parcelable
3. Restore时，Window从savedInstanceState获取View的savedStates
4. 遍历View的树状结构调用 onRestoreInstanceState(Parcelable state)
5. View根据id获取自己的state并恢复

在View类中保存状态里用了三个方法，其中saveHierachyState()方法直接传递给了dispatchSaveInstanceState()方法。dispatchSaveInstanceState()用来分发保存状态的行为，这个方法的默认行为是在有ID的情况下保存自身的state，没有id的情况下什么都不做。所以ViewGroup可以选择重写这个方法，将保存状态的行为分发到子类中。onSaveInstanceState()方法用来具体地保存当前View的状态，自定义View可以选择重写这个方法。
这里需要注意一个细节：想要保存View的状态，需要在XML布局文件中提供一个唯一的ID（android:id），如果没有设置这个ID的话，View控件的onSaveInstanceState是不会被调用的。
自定义控件还要设置setSaveEnabled(true)