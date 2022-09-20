详细资料请查看官方文档
https://developer.android.com/topic/libraries/view-binding

以下是使用过的获取控件手段，
findViewById, butterknife, databinding, kotlin-android-extensions, viewBinding

一直很好用的kotlin-android-extensions也被谷歌废弃了，使用android studio build时，会提示如下内容，
Warning: The 'kotlin-android-extensions' Gradle plugin is deprecated，官方已不在建议使用此插件。

官方主推viewBinding，使用比较简单，按如下步骤

1. build.gradle中添加如下内容

```
android {
    ...
    viewBinding {
        enabled = true
    }
}
```

系统会为每个布局文件生成一个binding类，生成规则为，如你的xml文件名为result_profile.xml，则生成的binding类文件名为ResultProfileBinding，binding文件在build/generated/data_binding_base_class_source_out/xxx...package name/databinding该目录下，可自行查看自动生成代码。
如果不希望该xml生成binding类，可以在根目录中加入tools:viewBindingIgnore="true"，如下

```
  <LinearLayout
        ...
            tools:viewBindingIgnore="true" >
        ...
    </LinearLayout>
```

activity中，使用方法如下

```kotlin
private lateinit var binding: ResultProfileBinding

    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)
        binding = ResultProfileBinding.inflate(layoutInflater)
        val view = binding.root
        setContentView(view)
    }

   binding.name.text = viewModel.name
    binding.button.setOnClickListener { viewModel.userClicked() }
```

注意在xml中include一个 layout（LinearLayout、FrameLayout等布局，layout_include.xml）时，必须带id，否则报错，如下，

```xml
<include
        android:id="@+id/includeLayout"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        layout="@layout/layout_include"/>
```

在xml中include一个layout（merge为根布局，layout_include_merge.xml）时，必须不带id，否则报错，如下

```
<include
        android:layout_width="match_parent"
        android:layout_height="48dp"
        layout="@layout/layout_include_merge"/>

```

layout_include_merge.xml文件内容如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="48dp">
    <TextView
        android:id="@+id/tvIncludeMerge"
        android:layout_width="match_parent"
        android:layout_height="48dp"
        android:text="I am in include layout, i am a merge"
        app:layout_constraintTop_toBottomOf="@+id/includeLayout"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        />
</merge>
```

关于include普通布局文件和include一个merge为根布局的文件，不同的原因，可以查看自动生成的binding类，LayoutIncludeBinding（layout_include.xml）中的bind方法，传入的rootview为如下内容，

```
     id = R.id.includeLayout;
       View includeLayout = rootView.findViewById(id);
      if (includeLayout == null) {
        break missingId;
      }
      LayoutIncludeBinding binding_includeLayout = LayoutIncludeBinding.bind(includeLayout);
```

因为layout_include.xml此layout中是有实际root布局文件的。

我们再来看LayoutIncludeMergeBinding（layout_include_merge.xml）中的bind方法，也需要传入一个root view，但是merge标签是没有实际布局文件的，所以他的rootview是include他的布局文件，所以，在使用merge导入时，我们需要在代码中这么使用，如下

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityViewBindingTestBinding.inflate(layoutInflater)
        setContentView(binding.root)

     
		//merge 情况下需要这么使用，感觉有点啰嗦
        val mergeBinding: LayoutIncludeMergeBinding = LayoutIncludeMergeBinding.bind(binding.root)
        mergeBinding.tvIncludeMerge.text = "this is include merge layout"


		//include 情况下使用
binding.includeLayout.tvInclude.text = "this is include layout"

	
		//include 情况下，模仿merge方式的用法
        val includeBinding: LayoutIncludeBinding = LayoutIncludeBinding.bind(binding.includeLayout.root)
        includeBinding.tvInclude.text = "this is include layout"
    }
```

fragment中，在fragment中使用时，注意需要在onDestroyView中释放，使用方法如下，

```kotlin
    private var _binding: ResultProfileBinding? = null
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        _binding = ResultProfileBinding.inflate(inflater, container, false)
        val view = binding.root
        return view
    }
  
    override fun onDestroyView() {
        super.onDestroyView()
	_binding = null
    }
  
  
```

adapter中使用
adapter中使用viewbinding没有太大区别，如下

```kotlin
class Adapter(val items: List<String>): RecyclerView.Adapter<Adapter.ViewHolder>() {

        class ViewHolder(binding: LayoutItemViewbindingInAdapterBinding, itemView: View):
            RecyclerView.ViewHolder(itemView) {
            val textView: TextView = binding.textView
        }

        override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
            val binding = LayoutItemViewbindingInAdapterBinding.inflate(LayoutInflater.from(parent.context), parent, false)
//            val view = LayoutInflater.from(parent.context).inflate(R.layout.layout_item_viewbinding_in_adapter, parent, false)
            return ViewHolder(binding, binding.root)
        }

        override fun onBindViewHolder(holder: ViewHolder, position: Int) {
            holder.textView.text = items[position]
        }

        override fun getItemCount(): Int {
            return items.size
        }
    }
}
```

在自定义view中使用viewBinding

```kotlin
    private lateinit var binding: InfoViewTiwaipingBinding
    constructor(context: Context) : super(context) {
        val root = inflate(context, R.layout.info_view_tiwaiping, this)
	//root_tiwaiping 为info_view_tiwaiping.xml根布局的id
        binding = InfoViewTiwaipingBinding.bind(root.root_tiwaiping)
    }

    constructor(context: Context, attrs: AttributeSet?) : super(context, attrs) {
    }

    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(
        context,
        attrs,
        defStyleAttr
    ) {
    }
```



对ViewBinding进行简单封装。

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

总结，不管在哪种界面种使用viewbinding，viewBinding会为你新建的xml文件，增加一个binding类，通过binding类，获取到view即可。
