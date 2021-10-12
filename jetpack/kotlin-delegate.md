
# 关于kotlin的delegate

## 属性委托
属性代理的实现方法如下，

```
var prop: String by Delegate()
```

```class Delegate {

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you delegate '${property.name}' to me"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef")
    }
}
```
同时kotlin也提供了更方便的接口实现方法，val/var可以分别实现ReadOnlyProperty，ReadWriteProperty接口来实现属性委托。

```
val prop1: String by Delegate1()
var prop2: Int by Delegate2()
    
class Delegate1(): ReadOnlyProperty<Any, String> {
    override fun getValue(thisRef: Any, property: KProperty<*>): String {
        return "use readOnlyProperty interface"
    }
}

class Delegate2(): ReadWriteProperty<Any, Int> {
    override fun getValue(thisRef: Any, property: KProperty<*>): Int {
        return 10
    }

    override fun setValue(thisRef: Any, property: KProperty<*>, value: Int) {
        println("use readWriteProperty interface $value")
    }

}
```

kotlin已经帮你封装好了集中属性委托，如下

### lazy properties
使用比较简单，记录下
```
val lazyProp: String by lazy {
    println("do this println only first time")
    "ok"
}
```
如果连续使用3次lazyProp属性，只有第一次会打印“do this println only first time”
同时lazy还支持输入参数，如下
```
val lazyProp: String by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
        println("do this println only first time")
        "ok"
    }
```

参数有3种，如果默认为SYNCHRONIZED，其他的参数如下

```
public enum class LazyThreadSafetyMode {

    /**
     * Locks are used to ensure that only a single thread can initialize the [Lazy] instance.
     */
    SYNCHRONIZED,

    /**
     * Initializer function can be called several times on concurrent access to uninitialized [Lazy] instance value,
     * but only the first returned value will be used as the value of [Lazy] instance.
     */
    PUBLICATION,

    /**
     * No locks are used to synchronize an access to the [Lazy] instance value; if the instance is accessed from multiple threads, its behavior is undefined.
     *
     * This mode should not be used unless the [Lazy] instance is guaranteed never to be initialized from more than one thread.
     */
    NONE,
}
```


### observable properties

observable properties 可以监听属性变更。
```
var observableProp: String by Delegates.observable("default value") {
        property, oldValue, newValue ->
            println("${property.name} , $oldValue, $newValue")

    }
```

### vetoable properties 
vetoable properties 和 observable properties比较像，不同在于，vetoable可以在监听时进行判断，如果返回false，则不会赋值。

```
var vetoableProp: Int by Delegates.vetoable(0) {
        _, oldValue, newValue ->
        newValue > oldValue
    }
```

### storing properties
storing properties用于保存数据，使用方法如下
```
class User(val map: Map<String, Any>) {
    val name: String by map
    val age:  Int    by map
}

val user = User(mapOf(
    "name" to "tom",
    "age"  to 20
))
```

以上就是常见属性委托使用。
