


对DataBinding进行简单封装。

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
