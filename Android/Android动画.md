Android动画
======================
补间动画
-----------------
       
ScaleAnimation, AlphaAnimation, TranslationAnimation、RotateAnimation      
动画原理：     
           
1. view.startAnimation实际上是给view设置了动画参数，并没有马上执行动画，而是调用了view的invalidate方法，其中调用了viewParent的invalidateChild方法，在该方法中层层向上调用，不断获取parent，最终调用到了DecorView的ViewParent，也就是ViewRootImp的invalidateChild方法，该方法通过调用scheduleTraversal发起一次布局请求，当下一次绘制信号到来的时候，层层遍历View树，对需要进行绘制的view重新进行绘制。      
           
2. 从DecorView开始向下遍历，调用View的draw方法，如果view有绑定动画，就执行applyLegacyAnimation这个方法。处理动画相关的逻辑。 
                  
3. applyLegacyAnimation这个方法，调用了startAnimation传入的动画的getTransformation方法，在其内部根据当前时刻的参数执行applyTransformation方法做相应视图的改变。      
      
4. getTransformation这个方法有返回值，表示动画是否执行完成，返回true表示还未完成，在applyLegacyAnimation方法中，判断如果动画没有完成，就会再次调用parent的invalidate方法，层层向上调用，调用到ViewRootImp中，再执行一次绘制逻辑，直到动画执行完成。
5. 有一点需要注意的是，动画不是单独执行的，如果在这次绘制过程中还有其他一些view需要重绘，也是在这次遍历View树的过程中执行的，在每一帧中最多只会执行一次performTraversals方法。      
      
属性动画
--------------
