关于CoordinatorLayout可以先来看下官方的用途说明，https://developer.android.google.cn/reference/androidx/coordinatorlayout/widget/CoordinatorLayout
> CoordinatorLayout is a super-powered FrameLayout.
> 
> CoordinatorLayout is intended for two primary use cases:
> 
> As a top-level application decor or chrome layout
> 
> As a container for a specific interaction with one or more child views

CoordinatorLayout是FrameLayout的加强版，另外官方说明，主要有两个用处，
1. 做顶层布局。
2. 做一个或者多个子view的交互容器。

即然如此，我们就先把它当成FrameLayout来用，同时在此基础上，看下有哪些super-powered function.

### Behavior
先来了解下，CoordinatorLayout中的一个内部类，behavior，这是一个交互类。
先来看下效果图

<img src="https://github.com/edmond-biguys/beja-coder/blob/main/image/behavior01_720.gif" width="250"/>

如上效果图中有2个被观察者，1个观察者，观察者总是跟着被观察者运动，这里我们使用到了Behavior这个类。

这里主要原理可以这么理解，首先观察者TextView控件，是CoordinatorLayout的直接子View，同时被标记了```app:layout_behavior```，如下
```
<androidx.coordinatorlayout.widget.CoordinatorLayout
xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
xmlns:app="http://schemas.android.com/apk/res-auto"
android:layout_width="match_parent"
android:layout_height="match_parent"
tools:context="com.cym.androidx.coordinatorlayout.CoordinatorLayoutSampleActivity">
    <androidx.appcompat.widget.AppCompatTextView
        android:id="@+id/tv_observe"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/coordinator_observer"
        app:layout_behavior="com.cym.androidx.coordinatorlayout.FollowBehavior"
    />
    ...
</androidx.coordinatorlayout.widget.CoordinatorLayout>
```
同时我们自定义了一个Behavior ```com.cym.androidx.coordinatorlayout.FollowBehavior```，FollowBehavior内容如下，
```
//在这个Behavior中，我们只override重写了layoutDependsOn, onDependentViewChanged两个方法。
class FollowBehavior(
    context: Context,
    attributeSet: AttributeSet? = null
) : CoordinatorLayout.Behavior<TextView>(context, attributeSet) {

    override fun layoutDependsOn(
        parent: CoordinatorLayout,
        child: TextView,
        dependency: View
    ): Boolean {
        return dependency is Button
    }

    override fun onDependentViewChanged(
        parent: CoordinatorLayout,
        child: TextView,
        dependency: View
    ): Boolean {
        child.x = dependency.x
        child.y = dependency.y + 200
        return true
    }
}
```

翻阅CoordinatorLayout源码时，发现在CoordinatorLayout.LayoutParams中，解析属性时，可以看到对```app:layout_behavior```的解析，如下
```
    LayoutParams(@NonNull Context context, @Nullable AttributeSet attrs) {
            super(context, attrs);

            ...
            
            mBehaviorResolved = a.hasValue(
                    R.styleable.CoordinatorLayout_Layout_layout_behavior);
            if (mBehaviorResolved) {
                mBehavior = parseBehavior(context, attrs, a.getString(
                        R.styleable.CoordinatorLayout_Layout_layout_behavior));
            }
            a.recycle();
            if (mBehavior != null) {
                // If we have a Behavior, dispatch that it has been attached
                mBehavior.onAttachedToLayoutParams(this);
            }
        }
        
    //parseBehavior方法
    static Behavior parseBehavior(Context context, AttributeSet attrs, String name) {
        
        ....
        
        try {
            Map<String, Constructor<Behavior>> constructors = sConstructors.get();
            if (constructors == null) {
                constructors = new HashMap<>();
                sConstructors.set(constructors);
            }
            Constructor<Behavior> c = constructors.get(fullName);
            if (c == null) {
                final Class<Behavior> clazz =
                        (Class<Behavior>) Class.forName(fullName, false, context.getClassLoader());
                c = clazz.getConstructor(CONSTRUCTOR_PARAMS);
                c.setAccessible(true);
                constructors.put(fullName, c);
            }
            return c.newInstance(context, attrs);
        } catch (Exception e) {
            throw new RuntimeException("Could not inflate Behavior subclass " + fullName, e);
        }
    }
```
可以看到，通过反射，CoordinatorLayout获取到了他子View中被```app:layout_behavior```标记的View，同时使用了此构造函数```c.newInstance(context, attrs)```，因此在我们自定义Behavior时，必须要增加此构造函数。

上图中，可以看到，观察者总是跟着被观察者运动，同时FollowBehavior类中，增加layoutDenpendsOn方法（判断什么时候执行Behavior），增加了onDependentViewChanged方法（当dependency view，即我们定义的两个Button，变化时，当前View，即我们定义的TextView，会跟着变化方式）。我们来看下这两个方法是在什么时候被调用的，在没有头绪的情况下，我们就用最简单的方法来找，直接从调用这两个方法的地方开始找，最后发现大致是如下流程。

在CoordinatorLayout onAttachedToWindow()时，将OnPreDrawListener监听注册到ViewTreeObserver，看监听的名字可以猜测，这是一个在重绘时候会触发的回调。

注：ViewTreeObserver说明，ViewTreeObserver是监听视图变化的，同时让用户不要去实例化，它是视图层次结构提供的，可以通过View.getViewTreeObserver()来获取。
> A view tree observer is used to register listeners that can be notified of global changes in the view tree. Such global events include, but are not limited to, layout of the whole tree, beginning of the drawing pass, touch mode change.... A ViewTreeObserver should never be instantiated by applications as it is provided by the views hierarchy. Refer to View.getViewTreeObserver() for more information.


```
    @Override
    public void onAttachedToWindow() {
        ...
        if (mNeedsPreDrawListener) {
            if (mOnPreDrawListener == null) {
                mOnPreDrawListener = new OnPreDrawListener();
            }
            final ViewTreeObserver vto = getViewTreeObserver();
            vto.addOnPreDrawListener(mOnPreDrawListener);
        }
        ...
    }
```

进入OnPreDrawListener类后看到这个类实现了ViewTreeObserver.OnPreDrawListener，在onPreDraw()方法中调用了onChildViewsChanged()方法，查看onChildViewsChanged()方法，可以看到在这个方法中对所有子View进行遍历，最终调用了onDependentViewChanged()方法。

```
   class OnPreDrawListener implements ViewTreeObserver.OnPreDrawListener {
        @Override
        public boolean onPreDraw() {
            onChildViewsChanged(EVENT_PRE_DRAW);
            return true;
        }
    }
    
    final void onChildViewsChanged(@DispatchChangeEvent final int type) {
        ...
        
        //遍历子view，找到带layout_behavior属性标记的view
        for (int i = 0; i < childCount; i++) {
            // Update any behavior-dependent views for the change
            for (int j = i + 1; j < childCount; j++) {
                final View checkChild = mDependencySortedChildren.get(j);
                final LayoutParams checkLp = (LayoutParams) checkChild.getLayoutParams();
                final Behavior b = checkLp.getBehavior();

                if (b != null && b.layoutDependsOn(this, checkChild, child)) {
                    final boolean handled;
                    switch (type) {
                        case EVENT_VIEW_REMOVED:
                            // EVENT_VIEW_REMOVED means that we need to dispatch
                            // onDependentViewRemoved() instead
                            b.onDependentViewRemoved(this, checkChild, child);
                            handled = true;
                            break;
                        default:
                            // Otherwise we dispatch onDependentViewChanged()
                            
                            //这里调用onDependentViewChanged，回调我们需要的方法。
                            handled = b.onDependentViewChanged(this, checkChild, child);
                            break;
                    }
                }
            }
        }
        ...
    }
```

以上为CoordinatorLayout中Behavior执行的一些方法，主要是通过标记为```layout_behavior```，然后通过该标记，拦截相应的操作，执行 Behavior中定义的操作。Behavior中还定义了其他一些方法，包括scroll、touch等，都是类似的方式。


### AppBarLayout

目前结合CoordinatorLayout使用最多的是AppBarLayout, CollapsingToolBarLayout，






















