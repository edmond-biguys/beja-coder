### registerForActivityResult

startActivityForResult()已经被标记为Deprecated，google建议使用androidx中的API，registerForActivityResult，主要目的是解耦。

官方说明
> 位于 ComponentActivity 或 Fragment 中时，Activity Result API 会提供 registerForActivityResult() API，用于注册结果回调。registerForActivityResult() 接受 ActivityResultContract 和 ActivityResultCallback 作为参数，并返回 ActivityResultLauncher，供您用来启动另一个 activity。

使用方法，

1. 定义一个ActivityResultContract，一个activity回调结果的协议，定义方法如下

```
    class GoToDetailResultContract: ActivityResultContract<String, String>() {  //这里ActivityResultContract<I, O>()，第一个表示输入参数类型，第二表示回调参数类型
        override fun createIntent(context: Context, input: String): Intent {    //input为输入参数
            val intent = Intent(context, DecorativeMaterialDetailActivity::class.java)
            intent.putExtra("input", input)
            return intent
        }

        override fun parseResult(resultCode: Int, intent: Intent?): String {   //这里的返回值String为输出参数
            if (resultCode == RESULT_OK) {
                println("resultCode ok $resultCode")
            } else {
                println("resultCode $resultCode")
            }
            return "output012345"
        }

    }
```

2. 注册ActivityResultLauncher，注册需要在onResume生命周期前完成，否则会抛异常。

```
private val goToDetailLauncher = registerForActivityResult(GoToDetailResultContract()) {
        println("output $it")
    }
```

3. 启动通过ActivityResultLauncher启动activity

```
goToDetailLauncher.launch("input01")
```

通过上述3个步骤就可以完成registerForActivityResult的使用。


### ------------------------------------------- 我是分割线

上述ActivityResultContract使用时每次都需要定义一个Contract，比较麻烦，所以Google都已帮你定义好了一些常用的Contract，所有的Contract都定义在```ActivityResultContracts```类中，```A collection of some standard activity call contracts, as provided by android.```

定义好的有```StartActivityForResult```，```StartIntentSenderForResult```，```RequestMultiplePermissions```，```RequestPermission```，```TakePicturePreview```，```TakePicture```，```TakeVideo```，```CaptureVideo```，```PickContact```，```GetContent```，```GetMultipleContents```，```OpenDocument```，```OpenMultipleDocuments```，```OpenDocumentTree```，```CreateDocument```，接下来，我们一个个分析下，这些Contract都是在什么时候使用的。


### StartActivityForResult
```
An ActivityResultContract that doesn't do any type conversion, taking raw Intent as an input and ActivityResult as an output.
```
一个通用的contract，输入是一个Intent，输出是一个ActivityResult，使用方法如下
```
    //注册一个contract
    private val gotoPayedHostoryActivity =
        registerForActivityResult(
            ActivityResultContracts.StartActivityForResult()) {
                result ->
        println("resultCode: ${result.resultCode == RESULT_OK} ")
        println("resultData " +
                "${result.data?.getIntExtra(
                    ActivityResultContracts.StartActivityForResult.EXTRA_ACTIVITY_OPTIONS_BUNDLE,
                    0)}")
    }
    
    //启动相应的activity
    val intent =
    Intent(this@DecorativeMaterialDetailActivity, PayedHostoryActivity::class.java)
    val bundle = Bundle()
    bundle.putInt(
    ActivityResultContracts.StartActivityForResult.EXTRA_ACTIVITY_OPTIONS_BUNDLE, 987)
    intent.putExtras(bundle)
    gotoPayedHostoryActivity.launch(intent)
    
    //返回上一个activity
    val bundle = Bundle()
    bundle.putInt(
        ActivityResultContracts.StartActivityForResult.EXTRA_ACTIVITY_OPTIONS_BUNDLE, 11111)
    intent.putExtras(bundle)
    setResult(RESULT_OK, intent)
    finish()

```
































