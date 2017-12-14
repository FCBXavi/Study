#Fragment
不能独立存在，只能依附于Activity

生命周期：

onAttach()->onCreate()->onCreateView()->onActivityCreated()->onStart()->onResume()->onPause()->onStop()->onDestroyView()->onDestroy()->onDetach()

与Activity通信：

1. 单向通信 Activity通过设置setArgument()方法向Fragment方法传值，Fragment通过getActivity()的方式调用Activity的方法。

2. 使用接口 Activity和Fragment分别实现接口，在Activity中获取Fragment的实例，调用接口方法，Fragment中同理；

3. 广播的方式

4. EventBus