
[kotlin flow 官方文档](https://developer.android.com/kotlin/flow?hl=zh-cn)

### flow操作符  
1. filter  
```
/**
 * Returns a flow containing only values of the original flow that match the given [predicate].
 */
public inline fun <T> Flow<T>.filter(crossinline predicate: suspend (T) -> Boolean): Flow<T> = transform { value ->
    if (predicate(value)) return@transform emit(value)
}
```



