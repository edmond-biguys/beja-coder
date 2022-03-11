### ViewModel使用

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
