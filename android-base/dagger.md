### Dagger

[about dagger](https://developer.android.com/training/dependency-injection/dagger-basics "https://developer.android.com/training/dependency-injection/dagger-basics")

#### 1. 添加使用dagger需要的依赖

1. 添加kapt插件

```gradle
plugins {
    id 'kotlin-kapt'
}

dependencies {
    //...
    //dagger
    implementation 'com.google.dagger:dagger:2.44'
    kapt 'com.google.dagger:dagger-compiler:2.44'
}
```


简单使用流程，这里只记录怎么使用，至于为什么，以后再研究。

#### 2. 在自定义的Application中修改

```kotlin
@Singleton
@Component
interface AppGraph {
    fun inject(activity: LoginActivity)
    fun inject(activity: MainActivity)
}

class App: Application() {
    val appGraph = DaggerAppGraph.create()
}
```


App中定义一个接口，因为这个接口使用了@Component注解，所以接口名称可以叫AppComponent，或者因为这相当于一个导航图，也可以叫AppGraph。

在这个接口里，我们定义两个方法，这两个方法，会在对应的activity中调用。@Singleton注解我们，表示单例，至于为啥在这里加，后边再说。

#### 3. 在activity中使用

```kotlin
class MainActivity : BaseActivity() {

    private val viewModel by viewModels<MainViewModel>()

    @Inject lateinit var loginViewModel: LoginViewModel

    @Inject lateinit var userLocalDataSource: UserLocalDataSource
    @Inject lateinit var userRemoteDataSource: UserRemoteDataSource

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        (applicationContext as App).appGraph.inject(this)

        Log.i("MainActivity", "userLocalDataSource: $userLocalDataSource ")
        Log.i("MainActivity", "userRemoteDataSource: $userRemoteDataSource ")

	loginViewModel.start()
    }
}
```

```kotlin
class LoginActivity : BaseActivity() {
    private lateinit var binding: ActivityLoginBinding

    @Inject lateinit var loginViewModel: LoginViewModel

    @Inject lateinit var userLocalDataSource: UserLocalDataSource
    @Inject lateinit var userRemoteDataSource: UserRemoteDataSource

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        (applicationContext as App).appGraph.inject(this)
        binding = ActivityLoginBinding.inflate(layoutInflater)


    }
}
```

如上代码，可以看到loginViewModel，userLocalDataSource，userRemoteDataSource几个都是通过@Inject注解，注入进来的。

如果你要在activity中使用，请在onCreate时候使用代码(applicationContext as App).appGraph.inject(this)，至于为啥，以后再研究，这么用就行了，//todo。

```kotlin
@Singleton
class LoginViewModel @Inject constructor(
    private val userRepository: UserRepository
): ViewModel() {
    companion object {
        const val TAG = "LoginViewModel"
    }
    fun start() {
        Log.i(TAG, "start: $this")
        Log.i(TAG, "start: userRepository $userRepository")
        userRepository.start()
    }
}
```


```kotlin
class UserRepository @Inject constructor(
    private val userLocalDataSource: UserLocalDataSource,
    private val userRemoteDataSource: UserRemoteDataSource
) {
    private val TAG = "UserRepository"
    fun start() {
        Log.i(TAG, "start: userLocalDataSource $userLocalDataSource")
        Log.i(TAG, "start: userRemoteDataSource $userRemoteDataSource")
    }
}
```


```kotlin
@Singleton
class UserLocalDataSource @Inject constructor() {
}
```


```kotlin
class UserRemoteDataSource @Inject constructor() {
}
```

从以上LoginViewModel, UserRepository, UserLocalDataSource, UserRemoteDataSource几个类可以看到，如果要在其他类中使用注解，需要对这个类的构造函数增加一个@Inject注解。

同时，我在LoginViewModel，UserLocalDataSource两个类上，增加了@Singleton注解，表示这两个在使用的时候为单例，这也是为什么之前要在App中的AppGraph接口增加@Singleton注解的原因，否则编译会报错，至于错误原因，以后再深究。

这对LoginViewModel使用了@Singleton注解，这样在不同的activity，或者fragment中使用时，将得到同一个LoginViewModel实例，方便使用。

至此，我们已经可以正常使用dagger注解，关于如何对dagger进行封装使用，以后再研究。