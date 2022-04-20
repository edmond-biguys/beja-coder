### ViewModel使用

viewModel使用时，一般会和liveData一起使用，需要导入的库如下

```
    def lifecycle_version = "2.5.0-alpha06"
    def arch_version = "2.1.0"
    // ViewModel
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version")
    // ViewModel utilities for Compose
    implementation("androidx.lifecycle:lifecycle-viewmodel-compose:$lifecycle_version")
    // LiveData
    implementation("androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version")
    // Lifecycles only (without ViewModel or LiveData)
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:$lifecycle_version")
    // Saved state module for ViewModel
    implementation("androidx.lifecycle:lifecycle-viewmodel-savedstate:$lifecycle_version")
    // Annotation processor
//    kapt("androidx.lifecycle:lifecycle-compiler:$lifecycle_version")
    // alternately - if using Java8, use the following instead of lifecycle-compiler
    implementation("androidx.lifecycle:lifecycle-common-java8:$lifecycle_version")
    // optional - helpers for implementing LifecycleOwner in a Service
    implementation("androidx.lifecycle:lifecycle-service:$lifecycle_version")
    // optional - ProcessLifecycleOwner provides a lifecycle for the whole application process
    implementation("androidx.lifecycle:lifecycle-process:$lifecycle_version")
    // optional - ReactiveStreams support for LiveData
    implementation("androidx.lifecycle:lifecycle-reactivestreams-ktx:$lifecycle_version")
    // optional - Test helpers for LiveData
    testImplementation("androidx.arch.core:core-testing:$arch_version")

    //注意这两个库，加入后 才能使用 by viewModels()
    implementation 'androidx.activity:activity-ktx:1.4.0'
    implementation 'androidx.fragment:fragment-ktx:1.4.1'
    
    implementation 'androidx.core:core-ktx:1.7.0'
```

在androidx.activity:activity-ktx:1.4.0中，Google定义了androidx.activity.ComponentActivity.viewModels方法，

在androidx.fragment:fragment-ktx:1.4.1中，Google定义了androidx.fragment.app.Fragment.viewModels方法，

这两个库用来```Use the 'by viewModels()' Kotlin property delegate```，否则你需要如下处理，
```
import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.lifecycle.ViewModelProvider

//这里注意需要继承androidx.activity.ComponentActivity
class MainActivity : ComponentActivity() {

    private lateinit var viewModel: MainActivityViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        ViewModelProvider(this)[MainActivityViewModel::class.java]
        viewModel.requestTest()
    }
}
```


1. 定义一个ViewModel类，基本使用如下，在官方androidx.lifecycle.ViewModel 注释中有使用说明，并且说明了，如何在多个fragment和activity间通讯。

```
public class UserActivity extends Activity {
  
        @Override
       protected void onCreate(Bundle savedInstanceState) {
           super.onCreate(savedInstanceState);
           setContentView(R.layout.user_activity_layout);
           final UserModel viewModel = new ViewModelProvider(this).get(UserModel.class);
           viewModel.getUser().observe(this, new Observer<User>() {
                @Override
               public void onChanged(@Nullable User data) {
                   // update ui.
               }
           });
           findViewById(R.id.button).setOnClickListener(new View.OnClickListener() {
                @Override
               public void onClick(View v) {
                    viewModel.doAction();
               }
           });
       }
   }
   
ViewModel would be:
   public class UserModel extends ViewModel {
       private final MutableLiveData<User> userLiveData = new MutableLiveData<>();
  
       public LiveData<User> getUser() {
           return userLiveData;
       }
  
       public UserModel() {
           // trigger user load.
       }
  
       void doAction() {
           // depending on the action, do necessary business logic calls and update the
           // userLiveData.
       }
   }
   
//ViewModels can also be used as a communication layer between different Fragments of an Activity. 
//Each Fragment can acquire the ViewModel using the same key via their Activity. 
//This allows communication between Fragments in a de-coupled fashion such that they never need to talk to the other Fragment directly.
   public class MyFragment extends Fragment {
       public void onStart() {
           UserModel userModel = new ViewModelProvider(requireActivity()).get(UserModel.class);
       }
   }
```

#### livedata使用
在viewModel中定义如下liveData，
```
val gitHubRepos: LiveData<List<GitHubRepo>> = liveData {

        //获取local数据
         val localRepo = repo.localRepos()
         emit(localRepo)

        //获取remote数据
        /*
         这里的执行顺序如下
         2022-04-13 14:17:22.904 ZLog -> 1 gitHubRepos Thread[main,5,main]
         2022-04-13 14:17:22.905 ZLog -> 2 in remoteRepos Thread[DefaultDispatcher-worker-1,5,main]
         2022-04-13 14:17:24.179 ZLog -> 3 after network Thread[DefaultDispatcher-worker-1,5,main]
         2022-04-13 14:17:24.186 ZLog -> 4 gitHubRepos Thread[main,5,main]
         1、4 在UI线程，2、3在异步线程，但整体代码看着是顺序执行，UI线程不会被卡住，使用协程，方便阅读。
         */
        log("1 gitHubRepos ${Thread.currentThread()}")
        val repos = repo.remoteRepos()
        log("4 gitHubRepos ${Thread.currentThread()}")
        emit(repos)
        log("gitHubRepos $repos")
    }
    
    //此方法为在repository中的方法
    suspend fun remoteRepos(): List<GitHubRepo> = withContext(Dispatchers.IO) {
        log("2 in remoteRepos ${Thread.currentThread()}")
        val repos = gitHubService.allRepos()
        log("3 after network ${Thread.currentThread()}")
        insertAll(repos)
        repos
    }    
```
同时在你的activity、fragment等中，observe这个gitHubRepos，那么 liveData{} 这个block方法会自动执行。
```
        gitHubTestViewModel.gitHubRepos.observe(this) {
            log("gitHubTestViewModel observe repos ${it.size}")
        }
```

&emsp;这样，我们就完成了，进入页面后，页面自动获取页面数据的方法。换成以前的写法就是，在viewModel或者presenter中定义一个fetchXXXData()的方法，然后通过回调拿到数据。使用liveData{}，好处1：这是订阅模式，不会因为网络延迟等问题，返回数据慢了，页面奔溃，当然这个问题，你用fetchXXXData方法，然后在这个方法中，使用liveData也能实现；好处2：不用费脑子，为fetchXXXData方法取名字。
   
      
   

&emsp;这样，我们就完成了，进入页面后，页面自动获取页面数据的方法。换成以前的写法就是，在viewModel或者presenter中定义一个fetchXXXData()的方法，然后通过回调拿到数据。使用liveData{}，好处1：这是订阅模式，不会因为网络延迟等问题，返回数据慢了，页面奔溃，当然这个问题，你用fetchXXXData方法，然后在这个方法中，使用liveData也能实现；好处2：不用费脑子，为fetchXXXData方法取名字。

   
&emsp;当然使用liveData{}这个block方法的最大好处是，看着同步的代码，其实是异步，不用再去不停的写回调，代码逻辑看着也比较清楚，官方提供了很多liveData{}使用的场景，你可以直接查看liveData{}源码的注释，不得不说，最近谷歌的源码注释都加上了，使用demo，比较人性化。


```
// a simple LiveData that receives value 3, 3 seconds after being observed for the first time.
val data : LiveData<Int> = liveData {
    delay(3000)
    emit(3)
}


// a LiveData that fetches a `User` object based on a `userId` and refreshes it every 30 seconds
// as long as it is observed
val userId : LiveData<String> = ...
val user = userId.switchMap { id ->
    liveData {
      while(true) {
        // note that `while(true)` is fine because the `delay(30_000)` below will cooperate in
        // cancellation if LiveData is not actively observed anymore
        val data = api.fetch(id) // errors are ignored for brevity
        emit(data)
        delay(30_000)
      }
    }
}

// A retrying data fetcher with doubling back-off
val user = liveData {
    var backOffTime = 1_000
    var succeeded = false
    while(!succeeded) {
        try {
            emit(api.fetch(id))
            succeeded = true
        } catch(ioError : IOException) {
            delay(backOffTime)
            backOffTime *= minOf(backOffTime * 2, 60_000)
        }
    }
}

// a LiveData that tries to load the `User` from local cache first and then tries to fetch
// from the server and also yields the updated value
val user = liveData {
    // dispatch loading first
    emit(LOADING(id))
    // check local storage
    val cached = cache.loadUser(id)
    if (cached != null) {
        emit(cached)
    }
    if (cached == null || cached.isStale()) {
        val fresh = api.fetch(id) // errors are ignored for brevity
        cache.save(fresh)
        emit(fresh)
    }
}

// a LiveData that immediately receives a LiveData<User> from the database and yields it as a
// source but also tries to back-fill the database from the server
val user = liveData {
    val fromDb: LiveData<User> = roomDatabase.loadUser(id)
    emitSource(fromDb)
    val updated = api.fetch(id) // errors are ignored for brevity
    // Since we are using Room here, updating the database will update the `fromDb` LiveData
    // that was obtained above. See Room's documentation for more details.
    // https://developer.android.com/training/data-storage/room/accessing-data#query-observable
    roomDatabase.insert(updated)
}
```

同样的，如果当你进入某个详情页面时，你需要通过一个id，来获取detail时，可以使用val detail = xxId.swichMap { id -> }，通过xxxViewModel.xxId.postValue("id")，改变id的值, 上述谷歌提供的使用样例中使用使用方法，至少可以不用写fetchXXDetail方法了。
