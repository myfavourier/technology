## Technology_3

## 概述

Android线程从用途分，可以分为主线程跟子线程，主线程往往处理和界面相关的事情，而子线程则往往用于执行耗时操作。由于Android的特性，如果在主线程中执行耗时操作那么就会导致程序无法及时的响应，因此耗时操作必须放在子线程中去执行。

除了Thread以外，在Android中扮演线程程序的还有很多，比如IntentService，HandlerThread，AsyncTask(api30即Android11被废除)

HandlerThread是一种具有消息循环的线程，在它的内部可以使用Handler。

IntentService是一个服务，系统对其进行了封装使其可以更方便地执行后台任务，IntentService内部采用HandlerThread来执行任务，当任务执行完毕后IntentService会自动退出

线程的创建和销毁都有相应的开销，因此采用线程池的做法，通过线程池可以避免因为频繁创建和销毁线程所带来的系统开销。Android中的线程池来源于Java，主要通过Executor来派生特定类型的线程池，不同种类的线程池有不同的特性。



## UI线程

在Java中默认情况下一个进程只有一个线程，这个线程就是主线程，也叫UI线程。主线程的作用是运行四大组件以及处理它们和用户的交互，而子线程的作用是执行耗时任务，比如网络请求、I/O操作等。从Android开始系统要求网络访问必须在子线程中进行，否则网络访问将会失败并抛出NetWorkOnMainThreadException异常，这样做是为了避免主线程由于被耗时操作所阻塞产生ANR现象。


## HandlerThread

HandlerThread继承了Thread，它是一种可以使用Handler的Thread，它的实习也很简单，就是在run方法中通过Looper.prepare()来创建消息须暖，这样在实际的使用中就允许在HandlerThread中创建Handler了。

Sample

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



总结

- 如果经常要开启线程，接着又是销毁线程，这是很耗性能的，`HandlerThread `很好的解决了这个问题；
- `HandlerThread `由于异步操作是放在 `Handler `的消息队列中的，所以是串行的，但只适合并发量较少的耗时操作。

- `HandlerThread `用完记得调用退出方法。
- 注意使用 handler 避免出现内存泄露

## IntentService

IntentService是一种特殊的Service，它继承了Service并且它是一个抽象类，因此必须创建它的子类才能使用IntentService。IntentService可用于执行后台耗时的任务，当任务执行后它会自动停止，同时由于IntentService是服务的原因，这导致它的优先级比单纯的线程要高很多，所以IntentService比较适合执行一些高优先级的后台任务，因为它优先级高不容易被系统杀死。在实现上，IntentService封装了HandlerThread和Handler



## 线程池