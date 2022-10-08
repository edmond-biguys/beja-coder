
[数据绑定DataBinding官方入口](https://developer.android.com/topic/libraries/data-binding?hl=zh-cn "https://developer.android.com/topic/libraries/data-binding?hl=zh-cn")

在[viewBinding](https://github.com/edmond-biguys/beja-coder/tree/main/jetpack/viewbinding.md)文章最后，已经提过viewBinding和DataBinding优缺点，使用场景。

这里我们记录下DataBinding的使用。

首先在build.gradle中加入如下，

```gradle
    buildFeatures {
        viewBinding = true
        dataBinding = true
    }
```

需要针对你的xml文件，做简单修改，如下，

最外层使用layout 标签，使用layout标签后，这个xml对应的activity（比如MainActivity）生成的类ActivityMainBinding会继承ViewDataBinding；如果没有使用layout标签，生成的类ActivityMainBinding，则会继承ViewBinding。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    >
    <data>
        <variable
            name="viewModel"
            type="com.xinzailingtech.daggersample.MainViewModel" />
        <variable
            name="user"
            type="com.xinzailingtech.daggersample.User" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        tools:context=".MainActivity">

        <TextView
            android:id="@+id/textView_1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="@{viewModel.userNameLiveData.userName}"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:textColor="@color/black"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent" />

        <TextView
            android:id="@+id/textView_2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginTop="24dp"
            android:text='@{user.userName == null ? "" : user.userName}'
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toBottomOf="@+id/textView_1" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>

```

在MainActivity中，有不同的方法得到bingding实例，如下3种，

1. 使用ActivityMainBinding.inflate

```kotlin
    private lateinit var binding: ActivityMainBinding
    private val viewModel by viewModels<MainViewModel>()
  
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = ActivityMainBinding.inflate(layoutInflater).also {
            setContentView(it.root)
        }
        binding.lifecycleOwner = this
        binding.viewModel = viewModel
}
```


2. 使用DataBindingUtil

```kotlin
    private lateinit var binding: ActivityMainBinding
    private val viewModel by viewModels<MainViewModel>()
  
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)

        binding.lifecycleOwner = this
        binding.viewModel = viewModel
}
```

3. 对dataBinding进行简单封装。

```kotlin
    private val binding by contentView<MainActivity, ActivityMainBinding>(
        R.layout.activity_main
    )
    private val viewModel by viewModels<MainViewModel>()
  
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding.lifecycleOwner = this
        binding.viewModel = viewModel
}
```



这里 binding.lifecycleOwner = this ，需要注意添加，非则liveData无法正常使用。

binding.viewModel = viewModel，是用来给xml种定义的数据赋值的。


对DataBinding进行简单封装的类。

```kotlin
import android.app.Activity
import androidx.annotation.LayoutRes
import androidx.databinding.DataBindingUtil
import androidx.databinding.ViewDataBinding
import kotlin.reflect.KProperty

/**
 * A delegate who lazily inflates a data binding layout, calls [Activity.setContentView] and returns
 * the binding.
 */
class ContentViewBindingDelegate<in R : Activity, out T : ViewDataBinding>(
    @LayoutRes private val layoutRes: Int
) {

    private var binding: T? = null

    operator fun getValue(activity: R, property: KProperty<*>): T {
        if (binding == null) {
            binding = DataBindingUtil.setContentView(activity, layoutRes)
        }
        return binding!!
    }
}

fun <R : Activity, T : ViewDataBinding> contentView(
    @LayoutRes layoutRes: Int
): ContentViewBindingDelegate<R, T> = ContentViewBindingDelegate(layoutRes)
```

在activity中使用

```kotlin
private val binding by contentView<ShotActivity, ActivityDribbbleShotBinding>(
        R.layout.activity_dribbble_shot
    )
```
