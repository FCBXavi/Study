Activity
Activity生命周期中的7个方法，四大状态（可见、可见不可操作、不可见、销毁）。
onCreat() -> onStart() -> [可见不可操作] -> onResume() -> [可操作] -> onPause() -> [可见不可操作] -> onStop() -> [不可见] -> onDestroy() -> [销毁]
如何判断activity是否被销毁
<pre>
if(activity == null || activity.isFinishing || activity.isDestroyed) {
    return;
}
</pre>