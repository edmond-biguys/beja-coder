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




**与数据绑定的对比**
***视图绑定***和***数据绑定***均会生成可用于直接引用视图的绑定类。但是，视图绑定旨在处理更简单的用例，与数据绑定相比，具有以下优势：

* 更快的编译速度：视图绑定不需要处理注释，因此编译时间更短。
* 易于使用：视图绑定不需要特别标记的 XML 布局文件，因此在应用中采用速度更快。在模块中启用视图绑定后，它会自动应用于该模块的所有布局。

反过来，与数据绑定相比，视图绑定也具有以下限制：

* 视图绑定不支持布局变量或布局表达式，因此不能用于直接在 XML 布局文件中声明动态界面内容。
* 视图绑定*不支持双向数据*绑定。

考虑到这些因素，在某些情况下，最好在项目中同时使用视图绑定和数据绑定。您可以在需要高级功能的布局中使用数据绑定，而在不需要高级功能的布局中使用视图绑定。


总结，不管在哪种界面种使用viewbinding，viewBinding会为你新建的xml文件，增加一个binding类，通过binding类，获取到view即可。


关于[数据绑定](https://github.com/edmond-biguys/beja-coder/tree/main/jetpack/databinding.md)，我们会在另一篇中说明。
