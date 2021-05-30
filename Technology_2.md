# Technology_2

## Android UI

### 概述

显示界面以及与用户交互，一般体现为activity

### 工作原理

ViewRoot对应于ViewRootImpl类，它是连接WindowManager和DecorView的纽带，在ActivityThread中，当Acitivity对象被创建完毕后，会将DecorView添加到Window中，同时创建ViewRootImpl对象，并将ViewRootImpl对象跟DecorView建立关联。

View的绘制流程是从ViewRoot的performTraversals方法开始的，经过measure、layout、draw三个过程将一个View绘制出来，其中measure用来测量View的宽和高，layout用来确定View在父容器中的放置位置，而draw则负责把View绘制在屏幕上。

#### onMeasure

MeasureSpec：代表一个32位int值，高两位代表SpecMode，低30位代表SpecSize，SpecMode是测量模式，SpecSize是指在某种测量模式下的规格大小。

SpecMode有三类

- UNSPECIFIED——父容器不对View有任何限制，要多大给多大，这种情况用于系统内部，表示一种测量的状态。
- EXACTLY——父容器已经检测出View所需要的精确大小，这个时候View的最终大小就是SpecSize所指定的值。它对应于LayoutParams中的match_parent和具体的数值这两种模式。
- AT_MOST——父容器指定 了一个可用大小即SpecSize，View的大小不能大于这个值，具体是什么值要看不同View的具体实习那。它对应于LayoutParams中的wrap_content。

对于顶级View（DecorView），其MeasureSpec由窗口的尺寸和其自身的LayoutParams来共同确定;对于普通View，其MeasureSpec由父容器的MeasureSpec和自身的LayoutParams来共同确定，MeasureSpec一旦确定后，onMeasure中就可以确定View的测量宽高。

如果是一个原始的View，那么通过measure方法就完成了测量过程，如果是一个ViewGroup，除了完成自己的测量过程外，还会遍历去调用所有子元素的measure方法，各个子元素再递归去执行这个流程。

- View的measure过程

直接调用View的OnMeasure

注意：直接继承View的自定义控件需要重写onMeasure方法并设置wrap_content时的自身大小，否则在布局中使用wrap_content就相当于使用match_content

- ViewGroup的measure过程

ViewGroup是一个抽象类，因此没有重写View的onMeasure方法，提供了一个叫measureChildren的方法

measureChilde的思想就是取出子元素的LayoutParams，然后再通过getChildMeasureSpec来创建子元素的MeasureSpec，接着将MeasureSpec直接传递给View的measure方法来进行测量。

View的measure过程是三大流程里最复杂的一个，measure完成以后，通过getMeasured_Width/Height方法就可以正确的获得View的测量宽/高。

注意：在某些极端情况下，系统可能需要多次measure才能确定最终的测量宽/高，在这种情况下，在onMeasure方法中拿到的测量宽/高很可能是不准确的。一个比较好的习惯是在onLayout方法中去获取View的测量宽高或者最终宽高。

#### onLayout

layout方法确定View本身的位置，onLayout则会确定所有子元素的位置。

layout方法大致流程：

首先通过setFrame方法来设定View的四个顶点的位置，即初始化mLeft、mRight、mTop、mBotttom四个值，四个顶点已确定，那么View在父容器中的位置也就确定了。

onLayout跟onMeasure类似，不同ViewGroup实现不同，不过思想类似。

#### onDraw

Draw过程，View的绘制过程遵循以下几步：

1. 绘制背景background.draw(canvas)
2. 绘制自己(onDraw)
3. 绘制children(dispatchDraw)
4. 绘制装饰(onDrawScrollBars)

## 动画绘制流程

动画的基本用法

```
ScaleAnimation animation = new ScaleAnimation(0,1,0,1);
animation.setDuration(300);
animation.setFillAfter(true);
view.startAnimation(animation);
```

### 流程

从startAnimation进入，有四个方法

- setStartTime（）——对变量赋值，没有实际逻辑
- setAnimation（）——将动画跟View绑定起来
- invalidateParentCached（）——给mPrivateFlags添加了一个标志位
- invalidate（）——不断循环调用到ViewRootImpl的invalidateChildInParent（）中，随后又调用到scheduleTraversals中，这个方法是将performTraversals封装到一个Runnable中，然后扔到Choreographer的待执行队列中，这些待执行的 Runnable 将会在最近的一个 16.6 ms 屏幕刷新信号到来的时候被执行。而 `performTraversals()` 是 View 的三大操作：测量、布局、绘制的发起者。

View 树里面不管哪个 View 发起了布局请求、绘制请求，统统最终都会走到 ViewRootImpl 里的 scheduleTraversals()，然后在最近的一个屏幕刷新信号到了的时候再通过 ViewRootImpl 的 performTraversals() 从根布局 DecorView 开始依次遍历 View 树去执行测量、布局、绘制三大操作。这也是为什么一直要求页面布局层次不能太深，因为每一次的页面刷新都会先走到 ViewRootImpl 里，然后再层层遍历到具体发生改变的 View 里去执行相应的布局或绘制操作。

当调用了 View.startAniamtion() 之后，动画并没有马上就被执行，这个方法只是做了一些变量初始化操作，接着将 View 和 Animation 绑定起来，然后调用重绘请求操作，内部层层寻找 mParent，最终走到 ViewRootImpl 的 scheduleTraversals 里发起一个遍历 View 树的请求，这个请求会在最近的一个屏幕刷新信号到来的时候被执行，调用 performTraversals 从根布局 DecorView 开始遍历 View 树。

总结

1. **首先，当调用了 View.startAnimation() 时动画并没有马上就执行，而是通过 invalidate() 层层通知到 ViewRootImpl 发起一次遍历 View 树的请求，而这次请求会等到接收到最近一帧到了的信号时才去发起遍历 View 树绘制操作。**
2. **从 DecorView 开始遍历，绘制流程在遍历时会调用到 View 的 draw() 方法，当该方法被调用时，如果 View 有绑定动画，那么会去调用applyLegacyAnimation()，这个方法是专门用来处理动画相关逻辑的。**
3. **在 applyLegacyAnimation() 这个方法里，如果动画还没有执行过初始化，先调用动画的初始化方法 initialized()，同时调用 onAnimationStart() 通知动画开始了，然后调用 getTransformation() 来根据当前时间计算动画进度，紧接着调用 applyTransformation() 并传入动画进度来应用动画。**
4. **getTransformation() 这个方法有返回值，如果动画还没结束会返回 true，动画已经结束或者被取消了返回 false。所以 applyLegacyAnimation() 会根据 getTransformation() 的返回值来决定是否通知 ViewRootImpl 再发起一次遍历请求，返回值是 true 表示动画没结束，那么就去通知 ViewRootImpl 再次发起一次遍历请求。然后当下一帧到来时，再从 DecorView 开始遍历 View 树绘制，重复上面的步骤，这样直到动画结束。**
5. 有一点需要注意，动画是在每一帧的绘制流程里被执行，所以动画并不是单独执行的，也就是说，如果这一帧里有一些 View 需要重绘，那么这些工作同样是在这一帧里的这次遍历 View 树的过程中完成的。每一帧只会发起一次 perfromTraversals() 操作。

### drawable

Drawable是一种可以在Canvas上进行绘制的抽象的概念，它的种类很多，最常见的颜色和图片都可以是一个Drawable，在实际开发中，Drawable常被用来作View的背景使用。

Drawable一般都是通过XML来定义的，当然我们也可以通过代码来创建具体的Drawable对象，只是用代码创建会稍显复杂。在Android设计中，Drawable是一个抽象类，它是所有Drawable对象的基类，每个具体的Drawable都是它的子类。

Drawable内部宽高这个参数比较重要，通过getIntrinsicWidth/Height可以获取到它们。但不是所有的Drawable都有内部宽/高，比如一张图片形成的Drawable，它的内部宽/高就是图片的宽/高，但是一个颜色形成的Drawable，它就没有内部宽/高的概念。另外，Drawable的内部宽/高不等同于它的大小，一般来说是没有大小概念的，当用作View的背景时，Drawable会被拉伸到View的同等大小。

分类

- BitmapDrawable——图片

sample

```
<?xml version="1.0" encoding="utf-8"?>
<bitmap
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@drawable/ic_launcher"
    android:antialias="true"
    android:dither="true"
    android:filter="true"
    android:gravity="top"
    android:tileMode="disabled"
    />
```

android:src
  类型：Drawable resource。必需。 引用一个drawable resource.
android:antialias
  类型：Boolean。是否开启抗锯齿。开启后图片会变得更平滑些，因此一般建议开启，设置为true即可。
android:dither
  类型：Boolean。是否允许抖动，如果位图与屏幕的像素配置不同时，开启这个选项可以让高质量的图片在低质量的屏幕上保持较好的显示效果（例如：一个位图的像素设置是 ARGB 8888，但屏幕的设置是RGB 565，开启这个选项可以是图片不过于失真）一般建议开启，为true即可。
android:filter
  类型：Boolean。是否允许对位图进行滤波。当图片被压缩或者拉伸时，使用滤波可以获得平滑的外观效果。一般建议开启，为true即可
android:gravity
  当图片小于容器尺寸时，设置此选项可以对图片经典定位，这个属性比较多，不同选项可以使用‘|’来组合使用。
android:mipMap
  纹理映射处理技术，不太懂，不过一般也不用，默认为false
android:tileMode
  平铺模式。共有以下几个值
  disabled ：默认值，表示不使用平铺
  clamp ：复制边缘色彩
  repeat ：X、Y 轴进行重复图片显示，也就是我们说要说的平铺
  mirror ：在水平和垂直方向上使用交替镜像的方式重复图片的绘制
代码使用方式

```
val mBitmap = BitmapFactory.decodeResource(resources,R.drawable.ic_launcher)
        val mBitmapDrawable = BitmapDrawable(resources,mBitmap)
        mBitmapDrawable.setTileModeXY(Shader.TileMode.MIRROR, Shader.TileMode.MIRROR);//平铺方式
        mBitmapDrawable.setAntiAlias(true);//抗锯齿
        mBitmapDrawable.setDither(true);//防抖动
//设置到imageView上即可
        binding.imageview.setImageDrawable(mBitmapDrawable);
```



- NinePatchDrawable——类似bitmap

sample

```
<?xml version="1.0" encoding="utf-8"?>
<nine-patch xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@drawable/ic_launcher"
    android:dither="true"/>
```

属性和BitmapDrawable中属性的含义相同，这里不过多描述。一般情况下不建议代码创建.9图，因为Android虽然可以使用Java代码创建NinePatchDrawable，但是极少情况会那么做，这是因为由于Android SDK会在编译工程时对点九图片进行编译，形成特殊格式的图片。使用代码创建NinePatchDrawable时只能针对编译过的点九图片资源，对于没有编译过的点九图片资源都当做BitmapDrawable对待。还有点需要特别注意的是，点九图只能适用于拉伸的情况，对于压缩的情况并不适用，如果需要适配很多分辨率的屏幕时需要把点九图做的小一点。


- ShapeDrawable

ShapeDrawable对于Xml的shape标签，在实际开发中我们经常将其作为背景图片使用，因为ShapeDrawable可以帮助我们通过颜色来构造图片，也可以构造渐变效果的图片



sample

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:innerRadius="20dp"
    android:shape="ring"
    android:thickness="8dp"
    android:useLevel="false" >

    <gradient android:angle="0"
        android:startColor="@color/teal_700"
        android:centerColor="#5027844F"
        android:endColor="#fff"
        android:useLevel="false"
        android:type="sweep"
        />
</shape>

```

**android:shape**
  这个属性表示图像的形状，可以是rectangle（矩形）、oval（椭圆）、line（横线）、ring（圆环）。默认为rectangle。

**<corners>**
  指定边角的半径，数值越大角越圆，数值越小越趋近于直角，参数为：

```
<corners
android:radius="integer"
android:topLeftRadius="integer"
android:topRightRadius="integer"
android:bottomLeftRadius="integer"
android:bottomRightRadius="integer" />
```

**<gradient>**
  设置颜色渐变与<solid>为互斥标签，因为solid表示纯色填充，而gradient表示渐变填充。

自定义一个旋转动画

```
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/shape_drawable"
    android:pivotX="50%"
    android:pivotY="50%"
    android:fromDegrees="0"
    android:toDegrees="360"
    >
</rotate>
```

```
<style name="CustomProgressStyle" >
    <item name="android:indeterminateDrawable">@drawable/progress_rotate</item>
    <item name="android:minWidth">72dp</item>
    <item name="android:maxWidth">72dp</item>
    <item name="android:minHeight">72dp</item>
    <item name="android:maxHeight">72dp</item>
</style>
```

应用到progressbar

```
<ProgressBar
     android:layout_width="100dp"
     android:layout_height="100dp"
     android:layout_centerInParent="true"
     style="@style/CustomProgressStyle"
     android:indeterminateDuration="700"
     />
```



- LayerDrawable

一个LayerDrawable是一个可以管理一组drawable对象的drawable。在LayerDrawable的drawable资源按照列表的顺序绘制，列表的最后一个drawable绘制在最上层。



sample

```
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
        <shape android:shape="rectangle">
            <solid android:color="@color/teal_700" />
        </shape>
    </item>

    <item android:bottom="6dp">
        <shape android:shape="rectangle">
            <solid android:color="#ffffff"/>
        </shape>
    </item>

    <item android:bottom="2dp"
        android:left="2dp"
        android:right="2dp">
        <shape android:shape="rectangle">
            <solid android:color="#ffffff" />
        </shape>
    </item>
</layer-list>
```

android:id
  资源ID，一个为这个item定义的唯一的资源ID。 使用:”@+id/name”.这样的方式。可以检索或修改这个drawable通过下面的方式：View.findViewById() or Activity.findViewById().
android:top
  Integer，Drawable相对于View的顶部的偏移量，单位像素
android:right
  Integer，Drawable相对于View的右边的偏移量，单位像素
android:bottom
  Integer，Drawable相对于View的底部的偏移量，单位像素
android:left
  Integer，Drawable相对于View的左边的偏移量，单位像素
android:drawable
  Drawable资源，可以引用已有的drawable资源，也可在item中自定义Drawable。默认情况下，layer-list中的Drawable都会被缩放至View的大小，因此在必要的情况下，我们可以使用android:gravity属性来控制图片的展示效果，防止图片变形或者被过度拉伸。

- StateListDrawable

StateListDrawable对于xml的<selector>标签，这个标签可以说是我们最常用的标签了，在开发中，有时候我们需要一个View在点击前显示某种状态，而在点击后又切换到另外一种状态，这时我们就需要利用<selector>标签来实现啦。

sample

```
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!--获取焦点状态-->
    <item
    android:state_focused="true"
    android:drawable="@color/color_state"
    />

    <!--按下状态-->
    <item android:state_pressed="true"
    android:drawable="@color/color_state" />

    <!--默认状态下-->
    <item android:drawable="@color/normal" />
</selector>
```

应用到button上

```
<Button
       android:padding="8dp"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:layout_centerInParent="true"
       android:text="Selector_State"
       android:textColor="#fff"
       android:background="@drawable/selector_drawable"
       />
```

android:constantSize
  StateListDrawable的固有大小是否随着其状态改变而改变，因为在状态改变后，StateListDrawable会切换到别的Drawable，而不同的Drawable其大小可能不一样。true表示大小不变，这时其固有大小是内容所有Drawable的固有大小的最大值。false则会随着状态改变而改变，默认值为false
android:variablePadding
  表示 StateListDrawable的padding是否随状态的改变而改变，默认值为false，一般建议设置为false就行。
android:dither
  是否开启抖动效果，开启后可使高质量的图片在低质量的屏幕上仍然有较好的显示效果，一般建议开启，设置为true。

- LevelListDrawable

LevelListDrawable对应于<level-list>标签，也表示一个Drawable的集合，但集合中的每个Drawable都一个等级。根据不同等级，LevelListDrawable会切换到相应的Drawable。

sample

```
<?xml version="1.0" encoding="utf-8"?>
<level-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/image4"
        android:maxLevel="0"
        />

    <item android:drawable="@drawable/image1"
          android:maxLevel="1"
        />

    <item android:drawable="@drawable/image2"
        android:maxLevel="2"
        />

    <item android:drawable="@drawable/image3"
        android:maxLevel="3"
        />
</level-list>
```

- TransitionDrawable

很多时候我们在实现渐变的动画效果时，都会使用到animation，但实际上我们有既简单又完美的解决方法，没错，它就是TransitionDrawable啦，TransitionDrawable用于实现两个Drawable之间的淡入淡出的效果，它对应的是<transition&gt标签

sample

```
<?xml version="1.0" encoding="utf-8"?>
<transition
xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:id="@[+][package:]id/resource_name"
        android:top="dimension"
        android:right="dimension"
        android:bottom="dimension"
        android:left="dimension" />
</transition>
```

语法中的属性比较简单，其中 android:top，android:bottom，android:left，android:right分别表示Drawable四周的偏移量。
android:id
  资源ID,drawable资源的唯一标识。使用”@+id/name”方式来给这个item定义一个新的资源ID。可以使用View.findViewById()或者 Activity.findViewById()等方式检索和修改这个item。

- InsertDrawable

有时候我们可能需要为一个全屏的LinearLayout布局指定背景图，但我们不想让背景图充满屏幕，这时我们就需要使用到InsetDrawable了，InsetDrawable对应<inset>标签，它可以将其他Drawable内嵌到自己当中，并可以在四周预留出一定的间距。当我们希望View的背景比实际区域小时，就可以采用InsetDrawable来实现，这个效果并没有什么特殊之处，因为layerDrawable也是可以达到相同的预期效果的。

sample

```
<?xml version="1.0" encoding="utf-8"?>
<inset
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/drawable_resource"
    android:insetTop="dimension"
    android:insetRight="dimension"
    android:insetBottom="dimension"
    android:insetLeft="dimension" />
```



- ScaleDrawable

ScaleDrawable对应<scale>标签，主要基于当前的level，对指定的Drawable进行缩放操作。有点需要特别注意的是我们如果定义好了ScaleDrawable，要将其显示出来的话，必须给ScaleDrawable设置一个大于0小于10000的等级（级别越大Drawable显示得越大，等级为10000时就没有缩放效果了），否则将无法正常显示。
sample

```
<?xml version="1.0" encoding="utf-8"?>
<scale
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/drawable_resource"
    android:scaleGravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                          "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                          "center" | "fill" | "clip_vertical" | "clip_horizontal"]
    android:scaleHeight="percentage"
    android:scaleWidth="percentage" />
```



- ClipDrawable

ClipDrawable是通过设置一个Drawable的当前显示比例来裁剪出另一张Drawable，我们可以通过调节这个比例来控制裁剪的宽高，以及裁剪内容占整个View的权重，通过ClipDrawable的setLevel()方法控制显示比例，ClipDrawable的level值范围在[0,10000]，level的值越大裁剪的内容越少，当level为10000时则完全显示，而0表示完全裁剪，不可见。需要注意的是在给clip元素中android:drawable属性设置背景图片时，图片不能是9图，因为这涉及到裁剪这张图片，如果设置为九图，裁剪的实际情况会与想要的效果不一样。
sample

```
 <?xml version="1.0" encoding="utf-8"?>
 <clip
 xmlns:android="http://schemas.android.com/apk/res/android"
 android:drawable="@drawable/drawable_resource"
 android:clipOrientation=["horizontal" | "vertical"]
 android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                  "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                  "center" | "fill" | "clip_vertical" | "clip_horizontal"] />
```



- ColorDrawable

ColorDrawable 是最简单的Drawable，它实际上是代表了单色可绘制区域，它包装了一种固定的颜色，当ColorDrawable被绘制到画布的时候会使用颜色填充Paint，在画布上绘制一块单色的区域。 在xml文件中对应<color>标签，它只有一个android:color属性，通过它来决定ColorDrawable的颜色。
sample

```
<?xmlversion="1.0" encoding="utf-8"?>
<color xmlns:android="http://schemas.android.com/apk/res/android"
  android:color="@color/normal"
/>
```

- GradientDrawable

GradientDrawable 表示一个渐变区域，可以实现线性渐变、发散渐变和平铺渐变效果

sample

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle"
    >

    <gradient android:angle="90"
        android:startColor="@color/colorPrimary"
        android:centerColor="#fff"
        android:endColor="@color/color_state"
        android:type="linear"
        />
</shape>
```



### canvas

绘制类

- 绘制基本图形——颜色、点、直线、矩形等
- 画布变换

图层

在Canvas中每次的save()都存将先前的状态保存下来，产生一个新的绘图层，
 我们可以随心所欲地地画而不会影响其他已画好的图，最后用restore()将这个图层合并到原图层
 这像是栈的概念，每次save()，新图层入栈(注意可以save多次)，只有栈顶的层可以进行操作，restore()弹栈

旋转

```
    private void stateTest(Canvas canvas) {
        canvas.drawLine(mCoo.x + 500, mCoo.y + 200, mCoo.x + 900, mCoo.y + 400, mRedPaint);
        canvas.drawRect(mCoo.x + 100, mCoo.x + 100, mCoo.y + 300, mCoo.y + 200, mRedPaint);
        canvas.save();//保存canvas状态
        //(角度,中心点x,中心点y)
        canvas.rotate(45, mCoo.x + 100, mCoo.y + 100);
        mRedPaint.setColor(Color.parseColor("#880FB5FD"));
        canvas.drawRect(mCoo.x + 100, mCoo.x + 100, mCoo.y + 300, mCoo.y + 200, mRedPaint);
        canvas.restore();//图层向下合并
    }

```

平移

```
    private void stateTest(Canvas canvas) {
        canvas.save();
        canvas.translate(mCoo.x, mCoo.y);//将原点平移到坐标系原点
        canvas.drawLine(500, 200, 900, 400, mRedPaint);
        canvas.drawRect(100, 100, 300, 200, mRedPaint);
        canvas.save();//保存canvas状态
        //(角度,中心点x,中心点y)
        canvas.rotate(45, 100, 100);
        mRedPaint.setColor(Color.parseColor("#880FB5FD"));
        canvas.drawRect(100, 100, 300, 200, mRedPaint);
        canvas.restore();//图层向下合并
        canvas.restore();
    }

```

缩放

```
    private void stateTest(Canvas canvas) {
        canvas.save();
        canvas.translate(mCoo.x, mCoo.y);//将原点平移到坐标系原点
        canvas.drawLine(500, 200, 900, 400, mRedPaint);
        canvas.drawRect(100, 100, 300, 200, mRedPaint);
        canvas.save();//保存canvas状态
        //(角度,中心点x,中心点y)
        canvas.scale(2, 2, 100, 100);
        mRedPaint.setColor(Color.parseColor("#880FB5FD"));
        canvas.drawRect(100, 100, 300, 200, mRedPaint);
        canvas.restore();//图层向下合并
        canvas.restore();
    }

```

斜切

```
    private void stateTest(Canvas canvas) {

        canvas.save();
        canvas.translate(mCoo.x, mCoo.y);//将原点平移到坐标系原点
        canvas.drawLine(500, 200, 900, 400, mRedPaint);
        canvas.drawRect(100, 100, 300, 200, mRedPaint);
        canvas.save();//保存canvas状态
        canvas.skew(1f,0f);
        mRedPaint.setColor(Color.parseColor("#880FB5FD"));
        canvas.drawRect(100, 100, 300, 200, mRedPaint);
        canvas.restore();//图层向下合并
        canvas.restore();
    }

```

Canvas的裁剪

##### 内剪裁：(区域内的之后绘制的内容保存)

##### 外剪裁：(区域外的之后绘制的内容保存)--注意API26及以上可用



## View的事件分发流程

对于一个跟ViewGroup，点击事件产生后，首先会传递它，这时它的dispatchTouchEvent就会被调用，如果这个ViewGroup的onInterceptTouchEvent方法返回true就表示拦截当前事件，接着事件就会交给这个ViewGroup处理，即调用onTouchEvent方法;如果这个VIiewGroup的onInterceptTouchEvent方法返回false就表示不拦截当前事件，这时当前事件就会传递给它的子元素，接着子元素的dispatchTouchEvent方法就会被调用，如此反复直到事件被最终处理

## activity生命周期

略

## fragment生命周期

略

