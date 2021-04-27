```text
---
title: Hello World # 标题
date: 2019/3/26 hh:mm:ss # 时间
categories: Android
- Basic
tags: Study
- A
---
```



# Android

## RecyclerView

### 四级缓存机制——缓存viewholder

第一级——缓存屏幕内的所有viewholder

第二级——缓存两个刚消失的viewholder

第三级——基本没用

第四级——缓存池子，当cacheView满了后或者adapter被更换，将cacheView中移出的ViewHolder放到Pool中，放之前会把ViewHolder数据清除掉，所以复用时需要重新bindView

预取机制——就是在滑动过程中，会把将要展示的一个元素提前缓存到`mCachedViews`中，所以滑动10个元素的时候，第11个元素也会被创建，也就多走了一次`bindview`方法。但是滑回去的时候不影响，因为就算提前取了一个缓存数据，只是把`bindview`方法提前了，并不影响总的绑定item数量。

### 局部更新

关于view的局部刷新就是notifyItemChanged(int, Object)方法，下面具体说说：

`notifyItemChange`有两个构造方法：

- notifyItemChanged(int position, @Nullable Object payload)
- notifyItemChanged(int position)

其中`payload`参数可以认为是你要刷新的一个标示，比如我有时候只想刷新`itemView`中的`textview`,有时候只想刷新`imageview`？又或者我只想某一个view的文字颜色进行高亮设置？那么我就可以通过`payload`参数来标示这个特殊的需求了。

payload参数为空，更新整个viewholder

### 性能优化

- bindViewHolder是在UI线程进行的，不能进行耗时操作，同时createview的调用要少于bindview，所以为了减少对象的创建，在createview里去创造监听
- 可以加大cacheview缓存，通过空间换时间
- 如果多个recyclerview的adapter一样，比如嵌套的recyclerview，可以共用一个recyclerviewpool

### 布局和动画原理

#### 布局放置

dispatchlayout()

- dispatchlayoutstep1()——执行预布局，记录viewholder位置信息
- dispatchlayoutstep2()——执行布局，用户最终看到的效果
- dispatchlayoutstep3()——执行动画操作

#### 预布局阶段


1、RecyclerView#dispatchLayoutStep1()

2、RecyclerView#processAdapterUpdatesAndSetAnimationFlags()

3、LinearLayoutManager#onLayoutChildren()

4、LinearLayoutManager#updateAnchorInfoForLayout()



1、处理Adapter变化。

2、决定该执行哪种类型动画。

3、保存当前RecyclerView上的子View的信息。

4、如果需要执行动画，进行预布局。

#### 布局阶段

1、RecyclerView#dispatchLayoutStep2()

2、LinearLayoutManager#layoutChunk()

3、LinearLayoutManager#addDisappearingView()

4、ViewInfoStore#addToDisappearedInLayout()



1、根据数据源中的数据进行布局，真正展示给用户看的最终界面。

2、如果开启动画，将被挤出屏幕的View的保存到消失动画列表中。

#### 开启动画阶段

1、RecyclerView#dispatchLayoutStep3()

2、ViewInfoStore#addToPostLayout()

3、ViewInfoStore#process()

4、ItemAnimator#animateAppearance()



1、清理工作。

2、保存布局后的view的信息。

3、触发动画。

4、动画执行完回收工作。



## Android轻量级存储方案

SharedPreferences 的 Api 使用很友好，数据改变时可以进行监听。但是它在 8.0 之前可能造成ANR（8.0之后优化了），而且不能跨进程。而 DataStore 存在 Preferences DataStore 和 Proto DataStore 这两种方式。



前者适合存储键值对的数据但是效率并不如 SharedPreferences（耗时是两倍左右），后者适合存储一些自定义的数据类型，DataStore 也可以在当数据改变可以进行监听，使用 Flow 以异步一致性方式存储数据，功能强大很多，但还是不能跨进程。



Proto DataStore 感觉在复杂数据的存储上可能会很有优势，当本地需要一些缓存数据对象，如果使用 Proto DataStore 能够快速获取整个对象（比如首页的缓存数据），然后进行数据加载这是很有优势的。但是其速度我也还没和其他方式进行对比，有兴趣的读者可以自己尝试一波。



而 MMKV 虽然不是官方出品的，但是在性能，速率，跨进程上面秒杀官方的两个数据存储方式。如果只是很简单的数据存储而且需要跨进程，MMKV 是首选。

## 图片加载库

### Glide

#### 三级缓存

简单描述：

读取的顺序是：Lru算法缓存、弱引用缓存、磁盘缓存

写入的顺序是：弱引用缓存、Lru算法缓存、磁盘缓存（不准确）

下面叙述一下三级缓存的流程：

当我们的APP中想要加载某张图片时，先去LruCache中寻找图片，如果LruCache中有，则直接取出来使用，如果LruCache中没有，则去WeakReference中寻找，如果WeakReference中有，则从WeakReference中取出图片使用，同时将图片重新放回到LruCache中，如果WeakReference中也没有图片，则去文件系统中寻找，如果有则取出来使用，同时将图片添加到LruCache中，如果没有，则连接网络从网上下载图片。图片下载完成后，将图片保存到文件系统中，然后放到LruCache中。

严格来讲，并没有什么Glide的三级缓存，因为Glide的缓存只有两个模块，一个是内存缓存，一个是磁盘缓存。其中内存缓存又分为Lru算法的缓存和弱引用缓存。

LruCache算法，Least Recently Used，又称为近期最少使用算法。主要算法原理就是把最近所使用的对象的强引用存储在LinkedHashMap上，并且，把最近最少使用的对象在缓存池达到预设值之前从内存中移除。
弱引用缓存：

```
public class Engine implements EngineJobListener,
        MemoryCache.ResourceRemovedListener,
        EngineResource.ResourceListener {
    //弱引用缓存
    private final Map<Key, WeakReference<EngineResource<?>>> activeResources;
    ...
    activeResources = new HashMap<Key, WeakReference<EngineResource<?>>>();
}
```



#### 解决图片加载生命周期

Glide 的使用方式上，一定需要传入一个 context 给它。它为什么需要拿上下文呢？原因就是可以根据不同的上下文进行处理，拿到 context （除了application context）之后，Glide做了一件很巧妙的事情，就是在这个界面上追加一个 fragment，由于 fragment 添加到了 activity 上，是可以捕获到生命周期的，因此可以在 destroy 的时候取消掉当前context下的 glide对象中的加载任务。

为什么标题后面说是 ‘也是bug高发地带’ 呢？ 因为从实现方式上，它是巧妙的利用了fragment的生命周期来实现的‘销毁’动作，那么就类似于另外一个高发bug,延时的匿名内部类(网络请求callback回来)，界面已经销毁，所以当前activity依附的glide也就销毁了的，此时再尝试加载图片的话，就会crash

#### 做到优雅的链式调用

链式调用的好与不好：

1. 编程性强
2. 可读性强
3. 代码简洁
4. 对程序员的业务能力要求高
5. 不太利于代码调试

#### 不支持wrap_content

官方说了的，不支持并且不建议imageview设置wrap_content。因为这样 glide 不知道要加载多大的图片给我们才好，在他的接口（Sizes and dimensions）中也有体现。普通的imageview其实也还好，如果放在列表（RecyclerView）中, 由于我们并不知道目标图片大小是多大的，所以我们选择了wrap_content，那么在上下来回滚动过程中，就会导致图片一会大一会小的bug.

#### support包版本问题

为什么会有这个问题呢？其实刚才已经提到了的 ，由于它用到了 fragmen t，那么自然就有版本冲突问题。support 包大家都懂的，不同的版本，差异可能巨大，有个段子就是说 Google 的 support 包 大概是招了个实习生写的。不同的版本冲突可能会编译不过，可能会有 ‘nosuchmethod’ 等等问题。
 比如我们产线现在的用的是 Glide 版本是 4.3.1，之所以迟迟没有升级到最新版本，就是因为后面的版本 Glide采用了 27编译。。而我们项目才25 。。。 中间这个编译升级的风险。有点不可控。所以一直没有升级上去。
 所以建议，在升级 Glide 版本的时候 看一下对应版本源码中依赖的 support 版本是多少。

### Picasso

双胞胎兄弟之间的对比，使用方式相同，但 Glide 之所以胜出，不仅仅是 Google的推荐，更多应该归功于 GIF 的支持。 在没有 Glide 之前，常用的做法就是写了个自定义 view 然后 用一个 media 去播放。有了 Glide 之后几乎对于 GIF 无感知了的， 内部已经支持了的。可以像普通图片那样去加载并且显示出来动图。

Glide在每个不同大小的imageView中都会重新加载一次图片，并且会自动计算imageView的大小，而Picasso容易造成OOM，因为它每次都会加载一次原图片，从这里相比，Glide不用考虑图片内存浪费。

### fresco

两个都支持 GIF。所以 GIF 这一关pass掉。说到这里不得不提到一个头疼的OOM问题，fresco 之所以很快闯入大家的视线，大概就是因为 Facebook 说他们使用了 native 内存规避掉了 OutOfMemoryError 问题。而且官方还专门写了个demo，把几大流行的开源库都集成进去，为了说明自己的图片加载库加载同样的图片速度更快，内存占用更低。所以 fresco 相比较于 Glide 的（官方）优势就是这两点： 内存以及加载速度。但是我为什么依旧坚持抛弃了 fresco ?

1. “ In Android 4.x and lower, Fresco puts images in a special region of Android memory. This lets your application run faster - and suffer the dreaded OutOfMemoryError much less often.” 官方的原话是这么说的，所以在高版本上面依旧使用的Java 内存，所以不可避免依旧会占用内存。

2. 提到内存，不得不说到另外一个笑话，fresco 最大只支持图片文件大小为 2M 。记得有一次帮其他团队跟踪问题，看到了 fresco 源码中有一个 最大 size 2M 常量 。于是当场找了一个10M的图片作为测试。 Glide 正常显示， fresco显示黑屏。。。

3. 使用方式上，fresco 推荐的是用他提供的 SimpleDraweeView . 这个方式意味着我们的迁移成本会非常的高，要改布局文件，其次还必须给定大小（或者比例）。 当然他也支持代码来加载图片，比如 DraweeHierarchy，但是写起来还是真心很费劲的，很不友好，改动成本居高。

4. fresco 更多是native实现。所以需要对NDK有所了解，但个人对NDK不太了解，相比较于 Glide， 同样遇到问题之后，修改源码的成本，Glide 成本更可控。前者可能就不太好下手了的。

5. Glide 各种 BitmapTransformation，比如圆形，圆角等，更让人喜欢。

6. 这一点就当随意吐槽一下，当然也可以说心疼一下 Facebook。因为在没有 Android studio （gradle构建）的情况下，想必大家都用的是 eclipse 吧。那么就意味着 fresco 得提供 Jar 包. 这一点当时也是把很多人拒之门外了的，可笑的是当 Facebook 费了老大劲的搞出来 jar 包之后，大家早就纷纷转战 gradle 构建工程, 直接 maven 依赖啦。大写的尴尬。

   综上所述，Glide 依旧胜出。

## 小知识

### 内联扩展函数

- #### let：

从源码let函数的结构来看它是只有一个lambda函数块block作为参数的函数,调用T类型对象的let函数，则该对象为函数的参数。在函数块内可以通过 it 指代该对象。返回值为函数块的最后一行或指定return表达式。

适用场景：

**场景一:** 最常用的场景就是使用let函数处理需要针对一个可null的对象统一做判空处理。

**场景二:** 然后就是需要去明确一个变量所处特定的作用域范围内可以使用

```
mVideoPlayer?.let {
	   it.setVideoView(activity.course_video_view)
	   it.setControllerView(activity.course_video_controller_view)
	   it.setCurtainView(activity.course_video_curtain_view)
}
```

- #### with：

将某对象作为函数的参数，在函数块内可以通过 this 指代该对象。返回值为函数块的最后一行或指定return表达式。

适用场景：

适用于调用同一个类的多个方法时，可以省去类名重复，直接调用类的方法即可，经常用于Android中RecyclerView中onBinderViewHolder中，数据model的属性映射到UI上

```
override fun onBindViewHolder(holder: ViewHolder, position: Int){
   val item = getItem(position)?: return
   
   with(item){
   
      holder.tvNewsTitle.text = StringUtils.trimToEmpty(titleEn)
	   holder.tvNewsSummary.text = StringUtils.trimToEmpty(summary)
	   holder.tvExtraInf.text = "难度：$gradeInfo | 单词数：$length | 读后感: $numReviews"
       ...   
   
   }

}
```

- #### run：

run函数实际上可以说是let和with两个函数的结合体，run函数只接收一个lambda函数为参数，以闭包形式返回，返回值为最后一行的值或者指定的return的表达式。

适用场景：

适用于let,with函数任何场景。因为run函数是let,with两个函数结合体，准确来说它弥补了let函数在函数体内必须使用it参数替代对象，在run函数中可以像with函数一样可以省略，直接访问实例的公有属性和方法，另一方面它弥补了with函数传入对象判空问题，在run函数中可以像let函数一样做判空处理

```
override fun onBindViewHolder(holder: ViewHolder, position: Int){
   
  getItem(position)?.run{
      holder.tvNewsTitle.text = StringUtils.trimToEmpty(titleEn)
	   holder.tvNewsSummary.text = StringUtils.trimToEmpty(summary)
	   holder.tvExtraInf = "难度：$gradeInfo | 单词数：$length | 读后感: $numReviews"
       ...   
   
   }

}
```

- #### apply：

从结构上来看apply函数和run函数很像，唯一不同点就是它们各自返回的值不一样，run函数是以闭包形式返回最后一行代码的值，而apply函数的返回的是传入对象的本身。

适用场景：

整体作用功能和run函数很像，唯一不同点就是它返回的值是对象本身，而run函数是一个闭包形式返回，返回的是最后一行的值。正是基于这一点差异它的适用场景稍微与run函数有点不一样。apply一般用于一个对象实例初始化的时候，需要对对象中的属性进行赋值。或者动态inflate出一个XML的View的时候需要给View绑定数据也会用到，这种情景非常常见。特别是在我们开发中会有一些数据model向View model转化实例化的过程中需要用到。

```
mSheetDialogView = View.inflate(activity, R.layout.biz_exam_plan_layout_sheet_inner, null).apply{
   course_comment_tv_label.paint.isFakeBoldText = true
   course_comment_tv_score.paint.isFakeBoldText = true
   course_comment_tv_cancel.paint.isFakeBoldText = true
   course_comment_tv_confirm.paint.isFakeBoldText = true
   course_comment_seek_bar.max = 10
   course_comment_seek_bar.progress = 0

}
```

- #### also：also函数的结构实际上和let很像唯一的区别就是返回值的不一样，let是以闭包的形式返回，返回函数体内最后一行的值，如果最后一行为空就返回一个Unit类型的默认值。而also函数返回的则是传入对象的本身

适用场景：

适用于let函数的任何场景，also函数和let很像，只是唯一的不同点就是let函数最后的返回值是最后一行的返回值而also函数的返回值是返回当前的这个对象。一般可用于多个扩展函数链式调用

```
//kotlin

fun main(args: Array<String>) {
    val result = "testLet".also {
        println(it.length)
        1000
    }
    println(result)
}

//java

public final class AlsoFunctionKt {
   public static final void main(@NotNull String[] args) {
      Intrinsics.checkParameterIsNotNull(args, "args");
      String var2 = "testLet";
      int var4 = var2.length();
      System.out.println(var4);
      System.out.println(var2);
   }
}
```

对比：

![QQ图片20210327223236](/home/alecs/下载/QQ图片20210327223236.png)

主要用apply和run，also，let，with 能做的，apply和run都能做。