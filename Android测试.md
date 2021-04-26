# Android测试

## 单元测试

### 依赖配置

```
    dependencies {
        // Required -- JUnit 4 framework
        testImplementation 'junit:junit:4.12'
        // Optional -- Robolectric environment
        testImplementation 'androidx.test:core:1.0.0'
        // Optional -- Mockito framework
        testImplementation 'org.mockito:mockito-core:1.10.19'
    }
    
```



### Junit

单元测试框架，使用简单，选择测试类和测试方法，通过断言等方式进行测试方法的设计，典型用例：

```
class ExampleUnitTest {
    @Test
    fun addition_isCorrect() {
        assertEquals(4, 2 + 2)
    }
}
```

重要注解

```
    @Test 指明这是一个测试方法 (@Test注解可以接受2个参数，一个是预期错误
    expected，一个是超时时间timeout，
    格式如 @Test(expected = IndexOutOfBoundsException.class), 
    @Test（timeout = 1000)
    @Before 在所有测试方法之前执行
    @After 在所有测试方法之后执行
    @BeforeClass 在该类的所有测试方法和@Before方法之前执
    行 （修饰的方法必须是静态的）@AfterClass 在该类的所有测试方法和@After
    方法之后执行（修饰的方法必须是静态的）
    @Ignore 忽略此单元测试

```

主要测试方法——断言

```
    assertEquals(expected, actual) 判断2个值是否相等，相等则测试通过。
    assertEquals(expected, actual, tolerance) tolerance 偏差值

```

使用Android Studio自带的Gradle脚本自动化单元测试

点击Android Studio中的**Gradle projects**下的**app/Tasks/verification/test**即可同时测试module下所有的测试类（案例），并在**module下的build/reports/tests/下**生成对应的**index.html测试报告**。

### mockito

Mockito都是用来产生模拟对象的，比如代码中是我们从网络获取一段json数据，转化成一个对象传入到我们的测试方法中。那么就可以直接new一个假的对象，并给它设置我们期望的返回值传给要测试的方法就好了，不需要再去请求网络获取数据。这个过程称之为mock。

示例：

```
@RunWith(MockitoJUnitRunner::class)
class ExampleUnitTest {


    @Mock
     lateinit var alex: Alex
    lateinit var alex2:Alex

    @Test
    fun ex1(){

        alex = mock(Alex::class.java)
        //通过Mockito mock一个对象，那么这个对象的name属性是默认为null的，如果我们不想让它为null，默认为空字符串可以使用RETURNS_SMART_NULLS
        alex2 = mock(Alex::class.java, RETURNS_SMART_NULLS)
        val b = alex.geta()
        //通过when return方法表示调用某个方法时指定返回值
        `when`(alex.geta()).thenReturn(1)
        //通过verify方法可以验证某个方法是否执行过，执行的次数
        verify(alex).geta()
        verify(alex, atLeastOnce()).geta()
        verify(alex, never()).geta()
        verify(alex, times(2)).geta()

    }
}
```

Mockito虽然好用但是也有些不足，比如不能mock static、final、private等对象，使用PowerMock就可以实现了

 mock出来的对象都是虚拟的对象，我们可以验证其执行次数，状态等，如果一个对象是真实的,可以使用spy包装一下,spy对象的方法默认调用真实的逻辑，mock对象的方法默认什么都不做，或直接返回默认值。

```
@Test
    public void testSpy(){
        Person person = getPerson();
        Person spy = spy(person);
        when(spy.getName()).thenReturn("Lily");
        System.out.println(spy.getName());
        verify(spy).getName();
    }
    private Person getPerson(){
        return new Person();
    }

```

踩坑：

- Mockito不能mock static、final、private对象，而kotlin中类和方法都是默认final的，如果非必须，代码在编写的时候最好加上open以方便测试
- 测试时很多方法和api根据版本不同有很大的不同，测试的时候注意版本问题，同时，测试时代码的自动补全功能较弱，需要自己输入括号后才会有补全提示

### powermock

使用添加依赖

```
  testImplementation 'org.powermock:powermock-module-junit4:2.0.2'
  testImplementation 'org.powermock:powermock-api-mockito2:2.0.2'

```

不管是方法传递、static、final、private、new一个对象还是其他方法，基本思想都是先mock一个PowerMockito.mock对象，随后通过when-thenReturn设置依赖对象的返回值跟参数等，随后通过assert或者verify进行测试。

类代码

```
public class Person {
    static int a = 1;
    final int b = 2;
    private int c = 3;
    public int d = 4;

    static int getA(){
        return 1;
    }

    final int getB(){
        return 2;
    }

    private int getC(){
        return 3;
    }

    public int getD(){
        return 4;
    }
}
```

测试代码

```
@RunWith(PowerMockRunner.class)
@PrepareForTest(Person.class)
public class PersonTest {

    @Test
    public void testStatic(){
        PowerMockito.mockStatic(Person.class);
        PowerMockito.when(Person.getA()).thenReturn(5);
        assertEquals(Person.getA(),5);
    }

    @Test
    public void testFinal(){
        Person person = PowerMockito.mock(Person.class);
        PowerMockito.when(person.getB()).thenReturn(6);
        assertEquals(person.getB(),6);
    }

    @Test
    public void testPrivate() throws Exception {
     	Person person2 = PowerMockito.mock(Person.class);
        PowerMockito.when(person2,"getC").thenReturn(2);
        Whitebox.setInternalState(person2,"c",2);
        int res = Whitebox.getInternalState(person2,"c");
        assertEquals(res,2);
    }
}
```

踩坑：

- private测试调用的API是最不同的，开始一直拿不到方法。
- 参数的设置通过Whitebox进行设置，Powermock提供了一个Whitebox的class，可以方便的绕开权限限制，可以get/set private属性，实现注入。也可以调用private方法。也可以处理static的属性/方法，根据不同需求选择不同参数的方法即可。



PowerMock简单实现原理
• 当某个测试方法被注解@PrepareForTest标注以后，在运行测试用例时，会创建一个新org.powermock.core.classloader.MockClassLoader实例，然后加载该测试用例使用到的类（系统类除外）。
• PowerMock会根据你的mock要求，去修改写在注解@PrepareForTest里的class文件（当前测试类会自动加入注解中），以满足特殊的mock需求。例如：去除final方法的final标识，在静态方法的最前面加入自己的虚拟实现等。
• 如果需要mock的是系统类的final方法和静态方法，PowerMock不会直接修改系统类的class文件，而是修改调用系统类的class文件，以满足mock需求

### Mockk

解决mockito跟powermock的一些问题

普通使用

```
fun test() {
	val mother = mockk<Mother>()
	every { mother.giveMoney() } returns 30 // when().thenReturn() in Mockito
	assertEquals(30, mother.giveMoney())
}

```

mockObject

```
object Son {
	fun test5(): Int {
		return 5
	}
}

mockkObject(Son)
every { Son.test5() } returns 10
assertEquals(10, Son.test5())


```

mockStatic

```
class Son {
	Static int test5() {
		return 5
	}
}	

@test
fun test() {
	mockkStatic(Son::class)
	every { Son.test5() } returns 10
	assertEquals(10, Son.test5())
}

```

mockPrivate

```
class Son {
	public int publicResult() {
		return privateResult()
	}

	private int privateReuslt() {
		return 5
	}
}	

@test
fun test() {
	val son = mockk<Son>()
	every { son["privateResult"]() } returns 10
	assertEquals(10, son.publicResult())
}

```

Context

在某些Android某些用户自定义的类中，需要Context才能初始化。

- 可以直接mock Context
- 可以mock 该类

### Robolectric

Robolectric通过**一套能运行在JVM上的Android代码**，解决了在Java单元测试中很难进行Android单元测试的痛点。

依赖配置

```
testImplementation 'androidx.test:core:1.2.0'
testImplementation 'org.robolectric:robolectric:4.3.1'

```

#### Shadow类

Shadow是Robolectric的立足之本，如其名，作为影子，一定是变幻莫测，时有时无，且依存于本尊。Robolectric定义了大量模拟Android系统类行为的Shadow类，当这些系统类被创建的时候，Robolectric会查找对应的Shadow类并创建一个Shadow类与原始类关联。每当系统类的方法被调用的时候，Robolectric会保证Shadow对应的方法会调用。这些Shadow对象，丰富了本尊的行为，能更方便的对Android相关的对象进行测试。 比如，我们可以借助ShadowActivity验证页面是否正确跳转了

```
    /**
     * 验证点击事件是否触发了页面跳转，验证目标页面是否预期页面
     *
     * @throws Exception
     */
    @Test
    public void testJump() throws Exception {
        // 默认会调用Activity的生命周期: onCreate->onStart->onResume
        MainActivity activity = Robolectric.setupActivity(MainActivity.class);
        // 触发按钮点击
        activity.findViewById(R.id.activity_main_jump).performClick();

        // 获取对应的Shadow类
        ShadowActivity shadowActivity = Shadows.shadowOf(activity);
        // 借助Shadow类获取启动下一Activity的Intent
        Intent nextIntent = shadowActivity.getNextStartedActivity();
        // 校验Intent的正确性
        assertEquals(nextIntent.getComponent().getClassName(), SecondActivity.class.getName());
    }

```

#### @Config

可以通过`@Config`定制Robolectric的运行时的行为。这个注解可以用来注释类和方法，如果类和方法同时使用了@Config，那么方法的设置会覆盖类的设置。你可以创建一个基类，用@Config配置测试参数，这样，其他测试用例就可以共享这个配置了

配置sdk

```
@Config(sdk = Build.VERSION_CODES.JELLY_BEAN)
public class SandwichTest {

    @Config(sdk = Build.VERSION_CODES.KITKAT)
    public void getSandwich_shouldReturnHamSandwich() {
    }
}

```

配置Application类

```
@Config(application = CustomApplication.class)
public class SandwichTest {

    @Config(application = CustomApplicationOverride.class)
    public void getSandwich_shouldReturnHamSandwich() {
    }
}

```

指定resource

```
@Config(manifest = "some/build/path/AndroidManifest.xml",
        assetDir = "some/build/path/assetDir",
        resourceDir = "some/build/path/resourceDir")
public class SandwichTest {

    @Config(manifest = "other/build/path/AndroidManifest.xml")
    public void getSandwich_shouldReturnHamSandwich() {
    }
}

```

使用第三方Library Resources

```
@RunWith(RobolectricTestRunner.class)
@Config(libraries = {
    "build/unpacked-libraries/library1",
    "build/unpacked-libraries/library2"
})
public class SandwichTest {
}

```

#### 常用用法

```
    //当Robolectric.setupActivity()方法返回的时候，
    //默认会调用Activity的onCreate()、onStart()、onResume()
    mTestActivity = Robolectric.setupActivity(TestActivity.class);
    
    //获取TestActivity对应的影子类，从而能获取其相应的动作或行为
    ShadowActivity shadowActivity = Shadows.shadowOf(mTestActivity);
    Intent intent = shadowActivity.getNextStartedActivity();
    
    //使用ShadowToast类获取展示toast时相应的动作或行为
    Toast latestToast = ShadowToast.getLatestToast();
    Assert.assertNull(latestToast);
    //直接通过ShadowToast简单工厂类获取Toast中的文本
    Assert.assertEquals("hahaha", ShadowToast.getTextOfLatestToast());
    
    //使用ShadowAlertDialog类获取展示AlertDialog时相应的
    //动作或行为（暂时只支持app包下的，不支持v7。。。）
    latestAlertDialog = ShadowAlertDialog.getLatestAlertDialog();
    AlertDialog latestAlertDialog = ShadowAlertDialog.getLatestAlertDialog();
    Assert.assertNull(latestAlertDialog);
        
    //使用RuntimeEnvironment.application可以获取到
    //Application，方便我们使用。比如访问资源文件。
    Application application = RuntimeEnvironment.application;
    String appName = application.getString(R.string.app_name);
    Assert.assertEquals("WanAndroid", appName);
    
    //也可以直接通过ShadowApplication获取application
    ShadowApplication application = ShadowApplication.getInstance();
    Assert.assertNotNull(application.hasReceiverForIntent(intent));

```

#### 自定义shadow

```
    @Implements(Person.class)
    public class ShadowPerson {
    
        @Implementation
        public String getName() {
            return "AndroidUT";
        }
    
    }
    
    @RunWith(RobolectricTestRunner.class)
    @Config(constants = BuildConfig.class,
            sdk = 23,
            shadows = {ShadowPerson.class})
    
        Person person = new Person();
        //实际上调用的是ShadowPerson的方法，输出JsonChao
        Log.d("test", person.getName());
         
        ShadowPerson shadowPerson = Shadow.extract(person);
        //测试通过
        Assert.assertEquals("JsonChao", shadowPerson.getName());
        
    }

```

#### 踩坑：

- Robolectric集成配置时需要特别注意sdk跟jdk的版本，版本冲突非常厉害。
- Robolectric的核心就是shadow类，通过模拟Android系统类的一些行为来进行测试，其实跟mock差不多



## 界面测试

设置对 Espresso 库的依赖项引用

```
    dependencies {
        androidTestImplementation 'androidx.test.espresso:espresso-core:3.1.0'
    }
    
```

在测试设备上关闭动画 - 如果让系统动画在测试设备上保持开启状态，可能会导致意外结果或导致测试失败。通过以下方式关闭动画：在“设置”中打开“开发者选项”，然后关闭以下所有选项：

- **窗口动画缩放**
- **过渡动画缩放**
- Animator 时长缩放

## TDD

### 定义

测试驱动开发（TDD，Test-Driven Development），用一句话说就是**写代码只为了修复失败的测试**。

测试驱动开发让我们把处理问题的方式从被动修复问题转变为主动暴露问题。

测试驱动开发有点像我们玩游戏，大多数游戏每一个关卡的设计都是有点难，但是又不会太难的。



### 好处

不用再长时间调试代码

在不使用测试驱动的情况下，假如你修改了一个电商 App 中处理商品列表的函数，然后你想试试搜索出来时该函数是否正确处理了请求下来的列表，那你需要经历八个步骤：安装—闪屏页—主页—点击搜索框—输入关键字—点击搜索—请求列表—处理列表。

如果是在找出 bug 的地方，打开了 Debugger 走这个流程，而且断点打得多的话，你还要一次又一次地继续到下一个断点，这个时间短则几十秒，长则几分钟。

几分钟又几分钟的积累下来，严重的话可能一天下来有三分之一的时间都是在调试代码，而且可能最后发现是一个小小的错误导致的。

如果使用测试驱动，你可以给这个函数模拟一个商品列表，在这个功能实现之前你就已经知道什么时候算是能做完了。

如果后续需求有变动，需要重构代码，你也不用再一步步点击，测试运行时间不超过 5 秒，而且写一个单元测试的时间一般就是几秒钟，长的话也就几分钟。

如果单个单元测试的时间过长，那就说明这个测试是有问题的，不是测试中测试的点太多，就是测试的函数太长，需要进行重构。

对自己的代码有信心

如果不使用测试驱动开发，当技术老大问你都搞定了吧，你只能心虚地说搞定了，然后交给测试人员去测试，找到问题了再修复。

如果使用测试驱动开发，你把测试都跑一遍，知道大多数的功能都是正常运行的，你交付软件给测试人员和技术老大的时候也就不用心虚了。

优化代码结构

使用测试驱动开发，会倒逼你去优化代码，因为难懂的、职责不明确的类和函数是难以测试的。



### 需要注意的问题

#### 遗留测试

和生产代码一样，测试代码会有遗留代码，当项目被其他接收的时候，如果这些遗留测试的命名没有清晰地说明这些测试的目的，而且也没有注释说明这些测试的意义，那当测试这些失败的时候，新进的开发者就会很迷惑，不知道怎么做，最后的选择可能是放弃测试驱动或删除掉这部分的测试代码。

#### 可维护性

不仅是遗留代码，即便是新写的测试，命名也应该是清晰地表明当前测试的目的，否则可能第二天你就忘了自己当时为什么要写这个测试了。



### 测试驱动开发的过程

不同于传统的软件开发流程是设计—编码—测试，测试驱动开发的流程是**测试—编码—重构**。

#### 测试

在测试阶段，我们要**写刚好失败的测试**。

我们需要测试的代码大多数都是公共（public）函数，这个函数可能是给我们自己或提供给其他开发者使用的。

先写测试能让我们站在用户的角度去看待我们的函数，这个角度能让我们能写出具有高可用性的 API。

之所以测试要“刚好失败”，是因为失败的测试暗示着应用的部分功能缺失，如果你一口气写的测试太多，可能导致写了几个小时都还没有一个测试能运行，弄得自己越写越没劲。

#### 编码

在编码阶段，我们要**写刚好能通过测试的代码**

上面已经说了不能一口气写太多测试，这样我们就不用一口气写太多代码了，我们可以让失败的测试来时刻提醒我们专注于实现当前缺失的功能。

每次通过测试，我们就能知道工作取得进展了，一般为一个功能写一个测试到实现功能代码的过程也就几分钟。如果超过这个时间，一般都是因为我们写的函数没有做到单一职责，而职责过多的函数是难以维护的。

之所以这个阶段写的代码不需要太完善，只需要“刚好能通过测试”，是因为我们会在下一步来对代码进行重构。

#### 重构

在重构阶段，我们要**找出现有代码的问题，优化代码质量**。

重构是 TDD 的最后一步，重构能让我们进行 TDD 的步伐更稳健。

使用 TDD 而不进行重构会带来大量的烂代码，不论我们的测试覆盖率有多高，烂代码还是烂代码。

良好的代码质量能提供我们后续的开发效率，是 TDD 中必不可少的一步。

### Mock

Mock 也就是模拟单元测试中需要用到的对象和方法，这样能避免创建对象带来的麻烦。

#### 使用Mock

- 设备式测试（Instrumented tests）

通过把单元测试换成设备式测试，我们可以获取到 Activity 的真实实例。但设备式测试的问题就在于运行时间太长，当你的电脑性能比较差，或者 APK 包很大时，运行速度更是慢得吓人。

- Robolectric

通过 Robolectric 模拟点击事件并检查视图上的文本，我们可以实现同时检验视图以及 Presenter 的逻辑，但是这么做的问题就在于这个测试方法的职责不是单一的。

如果我们真的想检验视图的展示是否正确，正确的做法应该是通过 Mock 提供数据给 Activity。

而且 Robolectric 的本质是建立了一个沙盒让我们能够在沙盒中进行测试，需要相对比较多的资源来完成一次测试，这样就导致了用了 Robolectric 的单元测试运行速度也很慢，快的话几十秒，慢的话甚至要几分钟。

- Mock

在 Presenter 的方法中会调用 View 接口提供的各种方法实现与 View 的一个通信，比如显示和隐藏 Loading 动画，所以 Presenter 的 getView() 方法的返回值不能为空。

而 MVP 的实现方式的其中一种是通过 Presenter 的 attachView() 方法绑定 Presenter 和 View，这种情况下我们就可以 mock 一个 View 接口，并将 View 传入 attachView() 方法实现绑定，这样 Presenter 中的 getView() 就不为空了。

- Mockk



Presenter示例

```
@RunWith(MockitoJUnitRunner.class)
public class GoodsPresenterTest {

    private GoodsPresenter presenter;

    @Mock
    GoodsContract.View view;

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
        presenter = new GoodsPresenter();
        presenter.attachView(view);
    }

    @Test
    public void testGetGoods() {
        Goods goods = presenter.getGoods(1);
        assert goods.name.equals("纸巾");
    }

}
```

