## Technology_3

## 概述

Android线程从用途分，可以分为主线程跟子线程，主线程往往处理和界面相关的事情，而子线程则往往用于执行耗时操作。由于Android的特性，如果在主线程中执行耗时操作那么就会导致程序无法及时的响应，因此耗时操作必须放在子线程中去执行。

除了Thread以外，在Android中扮演线程程序的还有很多，比如IntentService(IntentService 受Android 8.0（API 级别 26）施加的所有[后台执行限制的](https://developer.android.com/reference/kotlin/preview/features/background.html)约束。考虑使用 androidx.work.WorkManager 或 androidx.core.app.JobIntentService，它在 Android 8.0 或更高版本上运行时使用作业而不是服务)，HandlerThread，AsyncTask(api30即Android11被废除)

HandlerThread是一种具有消息循环的线程，在它的内部可以使用Handler。

IntentService是一个服务，系统对其进行了封装使其可以更方便地执行后台任务，IntentService内部采用HandlerThread来执行任务，当任务执行完毕后IntentService会自动退出

线程的创建和销毁都有相应的开销，因此采用线程池的做法，通过线程池可以避免因为频繁创建和销毁线程所带来的系统开销。Android中的线程池来源于Java，主要通过Executor来派生特定类型的线程池，不同种类的线程池有不同的特性。



## UI线程

在Java中默认情况下一个进程只有一个线程，这个线程就是主线程，也叫UI线程。主线程的作用是运行四大组件以及处理它们和用户的交互，而子线程的作用是执行耗时任务，比如网络请求、I/O操作等。从Android开始系统要求网络访问必须在子线程中进行，否则网络访问将会失败并抛出NetWorkOnMainThreadException异常，这样做是为了避免主线程由于被耗时操作所阻塞产生ANR现象。

## HandlerThread

### 概述

HandlerThread继承了Thread，它是一种可以使用Handler的Thread，它的实习也很简单，就是在run方法中通过Looper.prepare()来创建消息须暖，这样在实际的使用中就允许在HandlerThread中创建Handler了。

### Sample

```
DownloadThread

class DownloadThread(name: String?): HandlerThread(name) {

    val TAG = "DownloadThread"
    val TYPE_START = 2
    val TYPE_FINISHED = 3

    lateinit var handler: Handler

    override fun onLooperPrepared() {
        super.onLooperPrepared()
        Log.d(TAG,"downloadthread prepare")
    }

    fun setUIHandler( UIHandler:Handler){
        handler= UIHandler
        Log.d(TAG,"主线程的handler传入downloadthread")
    }

    fun startDownload(){
        Log.d(TAG,"开始下载")
        handler.sendEmptyMessage(TYPE_START)

        Log.d(TAG,"下载中")
        SystemClock.sleep(2000)

        Log.d(TAG,"下载结束")
        handler.sendEmptyMessage(TYPE_FINISHED)
    }
}
```

```
MainActivity

class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    private lateinit var mdownloadThread: DownloadThread
    private lateinit var mhandler: Handler

    private val TAG = "MainActivity"
    val TYPE_START = 2
    val TYPE_FINISHED = 3



    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        mdownloadThread = DownloadThread("mHandlerThread")
        mdownloadThread.start()
        mhandler = object : Handler(mdownloadThread.getLooper()) {
            override fun handleMessage(msg: Message) {
                //判断mHandlerThread里传来的msg，根据msg进行主页面的UI更改
                when (msg.what) {
                   TYPE_START ->                         //不是在这里更改UI哦，只是说在这个时间，你可以去做更改UI这件事情，改UI还是得在主线程。
                        Log.e(TAG, "4.主线程知道Download线程开始下载了...这时候可以更改主界面UI")
                    TYPE_FINISHED -> Log.e(TAG, "7.主线程知道Download线程下载完成了...这时候可以更改主界面UI，收工")
                    else -> {
                    }
                }
                super.handleMessage(msg)
            }
        }

        mdownloadThread.setUIHandler(mhandler)
        mdownloadThread.startDownload()

    }

    override fun onDestroy() {
        mdownloadThread.quitSafely()
        super.onDestroy()

    }
}
```



### 总结

- 如果经常要开启线程，接着又是销毁线程，这是很耗性能的，`HandlerThread `很好的解决了这个问题；
- `HandlerThread `由于异步操作是放在 `Handler `的消息队列中的，所以是串行的，但只适合并发量较少的耗时操作。

- `HandlerThread `用完记得调用退出方法。
- 注意使用 handler 避免出现内存泄露



## WorkManager

### 概述

WorkManager是什么？官方给的解释是：它对可延期任务操作非常简单，同时稳定性非常强，对于异步任务，即使App退出运行或者设备重启，它都能够很好的保证任务的顺利执行。

所以关键点是简单与稳定性。

对于平常的使用，如果一个后台任务在执行的过程中，app突然退出或者手机断网，这时后台任务将直接终止。

典型的场景是：App的关注功能。如果用户在弱网的情况下点击关注按钮，此时用户由于某种原因马上退出了App，但关注的请求并没有成功发送给服务端，那么下次用户再进入时，拿到的还是之前未关注的状态信息。这就产生了操作上的bug，降低了用户的体验，增加了用户不必要的操作。

使用workmanager就能简单实现

workmanager的使用过程

1. 构建Work
2. 配置WorkRequest
3. 添加到WorkContinuation中
4. 获取响应结果

### 构建Work

```
class CleanUpWorker(context: Context,workerParameters: WorkerParameters) : Worker(context,workerParameters) {
    override fun doWork(): Result {
        val outputDir = File(applicationContext.filesDir,"~/temp")
        if (outputDir.exists()){
            val fileLists = outputDir.listFiles()
            for(file in fileLists){
                val fileName = file.name
                if (!TextUtils.isEmpty(fileName) && fileName.endsWith(".png")) {
                    file.delete()
                }
            }
        }
        return Result.success()
    }
}
```



所有代码都在doWork中，实现逻辑也非常简单：找到相关目录，然后逐一判断目录中的文件是否为.png图片，如果是就删除。

以上是逻辑代码，关键点是返回值Result.success()，它是一个Result类型，可用值有三个

1. Result.success(): 成功
2. Result.failure(): 失败
3. Result.retry(): 重试

对于success与failure，它还支持传递Data类型的值，Data内部是一个Map来管理的，所以对于kotlin可以直接使用workDataOf

```
return Result.success(workDataOf(Constants.KEY_IMAGE_URI to outputFileUri.toString()))
```

它传递的值将放入OutputData中，可以在链式请求中传递，与最终的响应结果获取。其实本质是WorkManager结合了Room，将数据保存在数据库中。

### 配置workrequest

WorkManager主要是通过WorkRequest来配置任务的，而它的WorkRequest种类包括：

1. OneTimeWorkRequest
2. PeriodicWorkRequest

#### OneTimeWorkRequest

简单构建

```
val cleanUpRequest = OneTimeWorkRequestBuilder<CleanUpWorker>().build()
```

添加配置

```
val constraint = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()
 
val blurRequest = OneTimeWorkRequestBuilder<BlurImageWorker>()
        .setInputData(workDataOf(Constants.KEY_IMAGE_RES_ID to R.drawable.yaodaoji))
        .addTag(Constants.TAG_BLUR_IMAGE)
        .setConstraints(constraint)
        .build()
```

添加tag是为了打上标签，以便后续获取结果；传入的inputData可以在BlurImageWork中获取传入的值；添加网络连接constraint约束条件，代表只有在网络连接的状态下才会触发该WorkRequest。

#### PeriodicWorkRequest

PeriodicWorkRequest是可以周期性的执行任务，它的使用方式与配置和OneTimeWorkRequest一致。

```
val constraint = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()
 
// at least 15 minutes
mPeriodicRequest = PeriodicWorkRequestBuilder<DataSourceWorker>(15, TimeUnit.MINUTES)
        .setConstraints(constraint)
        .addTag(Constants.TAG_DATA_SOURCE)
        .build()
```



需要注意的是：它的周期间隔最少为15分钟。

### 添加到WorkContinuation中

对于单个的WorkRequest，可以直接通过WorkManager的enqueue方法

```
private val mWorkManager: WorkManager = WorkManager.getInstance(application)
 
mWorkManager.enqueue(cleanUpRequest)
```

如果想使用链式工作，只需调用beginWith或者beginUniqueWork方法即可。其实它们本质都是实例化了一个WorkContinuationImpl，只是调用了不同的构造方法。而最终的构造方法为

```
    WorkContinuationImpl(@NonNull WorkManagerImpl workManagerImpl,
            String name,
            ExistingWorkPolicy existingWorkPolicy,
            @NonNull List<? extends WorkRequest> work,
            @Nullable List<WorkContinuationImpl> parents) { }
```

其中beginWith方法只需传入WorkRequest

```
val workContinuation = mWorkManager.beginWith(cleanUpWork)
```

beginUniqueWork允许我们创建一个独一无二的链式请求。使用也很简单

```
val workContinuation = mWorkManager.beginUniqueWork(Constants.IMAGE_UNIQUE_WORK, ExistingWorkPolicy.REPLACE, cleanUpWork)
```

其中第一个参数是设置该链式请求的name；第二个参数ExistingWorkPolicy是设置name相同时的表现，它三个值，分别为：

1. REPLACE: 当有相同name且未完成的链式请求时，将原来的进度取消并删除，重新加入新的链式请求
2. KEEP: 当有相同name且未完成的链式请求时，链式请求保持不变
3. APPEND: 当有相同name且未完成的链式请求时，将新的链式请求追加到原来的子队列中，即当原来的链式请求全部执行后才开始执行。

而不管是beginWith还是beginUniqueWork，它都会返回WorkContinuation对象，通过该对象我们可以将后续任务加入到链式请求中。例如将上面的cleanUpRequest(清除)、blurRequest(图片模糊处理)与saveRequest(保存)串行起来执行，实现如下：

```
val cleanUpRequest = OneTimeWorkRequestBuilder<CleanUpWorker>().build()
val workContinuation = mWorkManager.beginUniqueWork(Constants.IMAGE_UNIQUE_WORK, ExistingWorkPolicy.REPLACE, cleanUpRequest)
 
val blurRequest = OneTimeWorkRequestBuilder<BlurImageWorker>()
        .setInputData(workDataOf(Constants.KEY_IMAGE_RES_ID to R.drawable.yaodaoji))
        .addTag(Constants.TAG_BLUR_IMAGE)
        .build()
 
val saveRequest = OneTimeWorkRequestBuilder<SaveImageToMediaWorker>()
        .addTag(Constants.TAG_SAVE_IMAGE)
        .build()
 
workContinuation.then(blurRequest)
        .then(saveRequest)
        .enqueue()
```

除了串行执行，还支持并行。例如将cleanUpRequest与blurRequest并行处理，完成之后再与saveRequest串行

```
val left = mWorkManager.beginWith(cleanUpRequest)
val right = mWorkManager.beginWith(blurRequest)
 
WorkContinuation.combine(arrayListOf(left, right))
        .then(saveRequest)
        .enqueue()
```

需要注意的是：如果你的WorkRequest是PeriodicWorkRequest类型，那么它不支持建立链式请求，这一点需要注意了。简单的理解，周期性的任务原则上是没有终止的，是个闭环，也就不存在所谓的链了。

### 获取响应结果

WorkManager支持两种方式来获取响应结果

1. Request.id: WorkRequest的id
2. Tag.name: WorkRequest中设置的tag

同时返回的WorkInfo还支持LiveData数据格式。

例如，现在我们要监听上述blurRequest与saveRequest的状态，使用tag来获取

```
// ViewModel
internal val blurWorkInfo: LiveData<List<WorkInfo>>
get() = mWorkManager.getWorkInfosByTagLiveData(Constants.TAG_BLUR_IMAGE)
 
internal val saveWorkInfo: LiveData<List<WorkInfo>>
get() = mWorkManager.getWorkInfosByTagLiveData(Constants.TAG_SAVE_IMAGE)
 
// Activity
private fun addObserver() {
    vm.blurWorkInfo.observe(this, Observer {
        if (it == null || it.isEmpty()) return@Observer
        with(it[0]) {
            if (!state.isFinished) {
                vm.processEnable.value = false
            } else {
                vm.processEnable.value = true
                val uri = outputData.getString(Constants.KEY_IMAGE_URI)
                if (!TextUtils.isEmpty(uri)) {
                    vm.blurUri.value = Uri.parse(uri)
                }
            }
        }
    })
 
    vm.saveWorkInfo.observe(this, Observer {
        saveImageUri = ""
        if (it == null || it.isEmpty()) return@Observer
        with(it[0]) {
            saveImageUri = outputData.getString(Constants.KEY_SHOW_IMAGE_URI)
            vm.showImageEnable.value = state.isFinished && !TextUtils.isEmpty(saveImageUri)
        }
    })
 
    ......
     ......
}
```

通过id获取

```
    // ViewModel
    internal val dataSourceInfo: MediatorLiveData<WorkInfo> = MediatorLiveData()
  
    private fun addSource() {
        val periodicWorkInfo = mWorkManager.getWorkInfoByIdLiveData(mPeriodicRequest.id)
        dataSourceInfo.addSource(periodicWorkInfo) {
            dataSourceInfo.value = it
        }
    }
    
    // Activity
    private fun addObserver() {
        vm.dataSourceInfo.observe(this, Observer {
            if (it == null) return@Observer
            with(it) {
                if (state == WorkInfo.State.ENQUEUED) {
                    val result = outputData.getString(Constants.KEY_DATA_SOURCE)
                    if (!TextUtils.isEmpty(result)) {
                        Toast.makeText(this@OtherWorkerActivity, result, Toast.LENGTH_LONG).show()
                    }
                }
            }
        })
    }
```



## JobIntentService

Android8.0对资源的管控更加严格，添加了后台限制规则。



### 使用注意

manifast里

```
    <uses-permission android:name="android.permission.WAKE_LOCK" />//needed
	<service
            android:name=".TestJobIntentService"
            android:permission="android.permission.BIND_JOB_SERVICE"//needed
            android:exported="false"></service>
```



### sample

```
public class TestJobIntentService extends JobIntentService {
    public static final int JOB_ID = 1;

    public static void enqueueWork(Context context, Intent work) {
        enqueueWork(context, TestJobIntentService.class, JOB_ID, work);
        }

    @Override
    protected void onHandleWork(@NonNull Intent intent) {
        Log.d("houson", "onHandleWork: "+intent.getStringExtra("work"));

    }


}
```



```
class MainActivity : AppCompatActivity() {
    lateinit var binding: ActivityMainBinding
    private val TAG = "MainActivity"
    var num = 0



    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        binding.button.setOnClickListener {
            val workIntent = Intent()
            num++
            workIntent.putExtra("work","work num : ${num}")
            TestJobIntentService.enqueueWork(applicationContext,workIntent)
        }

    }

    override fun onDestroy() {
        super.onDestroy()
    }
}
```



### 总结

IntentService是Service的子类，在独立的handler线程里处理异步任务请求。

IntentService中HandlerThread线程类，开启了一个HandlerThread线程实例，这个实例做一件就是开辟一个线程，并创建Looper循环器和消息队列MessageQueue。最后在IntentService里，通过HandlerThread线程实例获得Looper，然后IntentService里的ServiceHandler实例将与此Looper进行绑定。

所以当Clients通过startService(Intent)的方式开启服务并发送任务请求时，会执行IntentService类的onCreate方法，最终在onStartCommand里将任务送入消息队列，尔后，再startService时，都只会执行onStartCommand把任务放入消息队列中，而不会再执行onCreate。

IntentService在工作线程里运行，将依次处理每一个Intent。当执行完所有Intent时，它就会关闭服务。所有的请求都会在一个单例工作线程中被处理，每一时刻只能处理一个请求。这个工作线程不会阻塞应用的主线程中的主循环。

使用时，只需要继承IntentService类，并实现onHandleIntent(Intent)方法。

在Android 8.0 (API level 26)或以上，IntentService的所有后台执行会受到限制约束。所以在Android 8.0或更高的平台上，最好使用android.support.v4.app.JobIntentService。

## 线程池

线程池的优点

1. 重用线程池里的线程，避免因为线程的创建和销毁所带来的性能开销
2. 能有效控制线程池的最大并发数，避免大量的线程之间相互争抢系统资源而导致的阻塞现象
3. 能够对线程进行简单的管理，并提供定时执行以及指定间隔循环执行等功能

Android中的线程池的概念来源于Java中的Executor，Executor是一个接口，真正的线程池的实现为ThreadPoolExecutor。ThreadPoolExecutor提供了一系列参数来配置线程池，通过不同的参数可以创建不同的线程池，从线程池的功能特性上来说，Android线程池分为四类，这四类线程池尅通过Executors所提供的工厂方法来得到。

### ThreadPoolExecutor

ThreadPoolExecutor是线程池的真正实现，它的构造方法提供了一系列参数来配置线程池

```
public ThreadPoolExecutor(int corePoolSize,
						  int maxinumPoolSize,
						  long keepAliveTime,
						  TimeUnit unit,
						  BlockingQueue<Runnalbe> workQueue,
						  ThreadFactory threadFactory)
```

#### corePoolSize

线程池的核心线程数，默认情况下，核心线程会在线程池子中一直存活，即使它们处于闲置状态。

如果将ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，那么闲置的核心线程在等待新任务到来时会有超时策略，这个时间间隔由keepAliveTime所指定，当等待时间超过keepAliveTime所指定的时长后，核心线程就会被终止。

#### maxinumPoolSize

线程池所能容纳的最大线程数，当活动的线程数达到这个数值后，后续的新任务将会被阻塞。

#### keepAliveTime

非核心线程闲置时的超时时长，超过这个时长，非核心线程就会被回收。当ThreadPoolExecutor的allowCoreThreadTimeOut属性设置为true，keepAliveTime同样会作用于核心线程。

#### unit

用于指定keepAliveTime参数的时间单位，这是一个枚举，常用的有TimeUnit.MILLISECONDS/SECONDES/MINUTES

#### workQueue

线程池中的任务队列，通过线程池的execute方法提交的Runnable对象会存储在这个参数中。

#### threadFactory

线程工厂，为线程池提供创建新线程的功能。ThreadFactory是一个接口，它只有一个方法：Thread newThread(Runnable r)

### 线程池的分类

#### FixedThreadPool

Executor.newFixedThreadPool(integer_nTheads)

是一种线程数量固定的线程池，当线程处于空闲状态时，它们并不会被回收，除非线程池被关闭。当所有的线程都处于活动状态时，新任务都处于等待状态，直到有线程空闲出来。

由于FixedThreadPool只有核心线程并且这些核心线程不会被回收，这意味着它能够更加快速地响应外界的请求。

通过它的实现方法可以看出，它只有核心线程并且核心线程没有超时机制，另外任务队列也没有大小限制。

#### CachedThreadPool

Executor.newCachedThreadPool()

它是一种线程数量不定的线程池，它只有非核心线程，并且其最大线程数为Integer.MAX_VALUE。由于这个数很大，实际上就相当于最大线程数可以任意大。

当线程池中的线程都处于活动状态时，线程池会创建新的线程来处理新任务，否则就利用空闲的线程来处理新任务。

线程池中都有超时机制，这个超时时长为60s，超过60s就会被回收。

从它的特性看，这类线程比较适合执行大量的耗时较少的任务。当整个线程池都处于闲置状态时，线程池中的线程都会超时而被停止，这个时候CachedThreadPool之中实际上没有任何线程的，它几乎不占有任何系统资源

#### ScheduledThreadPool

Executor.newScheduledThreadPool(integer_corePoolSize)

它的核心线程数固定，而非核心线程数是没有限制的，并且当非核心线程闲置时会被立即回收。ScheduledThreadPool这类线程池主要用于执行定时任务和具有固定周期的重复任务

#### SingleThreadExecutor

Executor.newSingleThreadExecutor()

这类线程池内部只有一个核心线程，它确保所有的任务都在同一个线程中按顺序执行。SingleThreadExecutor的意义在于统一所有的外界任务到一个线程中，这使得在这些任务之间不需要处理线程同步的问题。



