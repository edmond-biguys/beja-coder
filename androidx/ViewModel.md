### ViewModel使用

viewModel使用时，一般会和liveData一起使用，需要导入的库如下

```
    def lifecycle_version = "2.5.0-alpha04"
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
