# Technology_4

## 内存优化工具

### Memory Profiler

#### 概述

Memory Profiler 是 [Android Profiler](https://links.jianshu.com/go?to=https%3A%2F%2Fdeveloper.android.google.cn%2Fstudio%2Fpreview%2Ffeatures%2Fandroid-profiler.html) 中的一个组件，可帮助您识别导致应用卡顿、冻结甚至崩溃的内存泄漏和流失。 它显示一个应用内存使用量的实时图表，可以捕获堆转储、强制执行垃圾回收以及跟踪内存分配。

作用

- 1）、**实时图表展示应用内存使用量**。
- 2）、**用于识别内存泄漏、抖动等**。
- 3）、**提供捕获堆转储、强制GC以及根据内存分配的能力**。

优点

- 1）、**方便直观**
- 2）、线下使用

#### 为什么应分析应用内存

Android 提供一个托管内存环境—当它确定您的应用不再使用某些对象时，垃圾回收器会将未使用的内存释放回堆中。 虽然 Android 查找未使用内存的方式在不断改进，但对于所有 Android 版本，系统都必须在某个时间点短暂地暂停您的代码。 大多数情况下，这些暂停难以察觉。 不过，如果您的应用分配内存的速度比系统回收内存的速度快，则当收集器释放足够的内存以满足您的分配需要时，您的应用可能会延迟。 此延迟可能会导致您的应用跳帧，并使系统明显变慢。

尽管您的应用不会表现出变慢，但如果存在内存泄漏，则即使应用在后台运行也会保留该内存。 此行为会强制执行不必要的垃圾回收 Event，因而拖慢系统的内存性能。 最后，系统被迫终止您的应用进程以回收内存。 然后，当用户返回您的应用时，它必须完全重启。

为帮助防止这些问题，您应使用 Memory Profiler 执行以下操作：

- 在时间线中查找可能会导致性能问题的不理想的内存分配模式。
- 转储 Java 堆以查看在任何给定时间哪些对象耗尽了使用内存。 长时间进行多个堆转储可帮助识别内存泄漏。
- 记录正常用户交互和极端用户交互期间的内存分配以准确识别您的代码在何处短时间分配了过多对象，或分配了泄漏的对象。

####  内存分析器概述

当您第一次打开 Memory Profiler 时，您将看到应用程序内存使用的详细时间线和访问工具以强制垃圾收集、捕获堆转储和记录内存分配。

![img](https://developer.android.google.cn/studio/images/profile/memory-profiler-callouts_2x.png)

**图 1.**内存分析器



如图 1 所示，Memory Profiler 的默认视图包括以下内容：

1. 强制垃圾收集事件的按钮。

2. 用于[捕获堆转储的](https://developer.android.google.cn/studio/profile/memory-profiler?hl=en#capture-heap-dump)按钮。

   **注意：**仅当连接到运行 Android 7.1（API 级别 25）或更低版本的设备时，堆转储按钮右侧才会显示一个用于[记录内存分配的](https://developer.android.google.cn/studio/profile/memory-profiler?hl=en#record-allocations)按钮。

3. 一个下拉菜单，用于指定探查器捕获内存分配的频率。选择适当的选项可以帮助您 [在分析时提高应用程序性能](https://developer.android.google.cn/studio/profile/memory-profiler?hl=en#performance)。

4. 用于放大/缩小时间线的按钮。

5. 跳转到实时内存数据的按钮。

6. 事件时间线，显示活动状态、用户输入事件和屏幕旋转事件。

7. 内存使用时间线，包括以下内容：

   - 每个内存类别使用了多少内存的堆叠图，如左侧的 y 轴和顶部的颜色键所示。
   - 虚线表示已分配对象的数量，如右侧的 y 轴所示。
   - 每个垃圾收集事件的图标。

但是，如果您使用的是运行 Android 7.1 或更低版本的设备，则默认情况下并非所有分析数据都可见。如果您看到一条消息“所选进程无法使用高级分析”，则您需要 [启用高级分析](https://developer.android.google.cn/studio/preview/features/android-profiler#advanced-profiling) 才能看到以下内容：

- 事件时间线
- 分配的对象数
- 垃圾收集事件

在 Android 8.0 及更高版本上，始终为可调试应用启用高级分析。

内存是如何计算的

根据 Android 系统，您在 Memory Profiler 顶部看到的数字（图 2）基于您的应用提交的所有私有内存页面。此计数不包括与系统或其他应用程序共享的页面。

![img](https://developer.android.google.cn/studio/images/profile/memory-profiler-counts_2x.png)

**图 2.** Memory Profiler 顶部的内存计数图例



内存计数中的类别如下：

- **Java**：来自 Java 或 Kotlin 代码分配的对象的内存。

- **Native**：来自从 C 或 C++ 代码分配的对象的内存。

  即使您没有在您的应用程序中使用 C++，您也可能会看到这里使用了一些本机内存，因为 Android 框架使用本机内存代表您处理各种任务，例如处理图像资产和其他图形时 - 即使您的代码've 是用 Java 或 Kotlin 编写的。

- **Graphics**：用于图形缓冲区队列的内存，用于向屏幕显示像素，包括 GL 表面、GL 纹理等。（请注意，这是与 CPU 共享的内存，而不是专用的 GPU 内存。）

- **堆栈**：应用程序中的本机堆栈和 Java 堆栈使用的内存。这通常与您的应用程序正在运行的线程数有关。

- **代码**：您的应用程序用于代码和资源的内存，例如 dex 字节码、优化或编译的 dex 代码、.so 库和字体。

- **其他**：您的应用程序使用的内存，系统不确定如何分类。

- **Allocated**：您的应用程序分配的 Java/Kotlin 对象的数量。这不包括在 C 或 C++ 中分配的对象。

  当连接到运行 Android 7.1 及更低版本的设备时，此分配计数仅在 Memory Profiler 连接到您正在运行的应用程序时开始。因此，在开始分析之前分配的任何对象都不会被考虑在内。但是，Android 8.0 及更高版本包含一个设备上的分析工具，用于跟踪所有分配，因此该数字始终代表 Android 8.0 及更高版本上应用中未完成的 Java 对象总数。

与之前的 Android Monitor 工具的内存计数相比，新的 Memory Profiler 以不同方式记录您的内存，因此您现在的内存使用量似乎更高。Memory Profiler 会监视一些增加总数的额外类别，但如果您只关心 Java 堆内存，那么“Java”数字应该与上一个工具中的值相似。尽管 Java 编号可能与您在 Android Monitor 中看到的不完全匹配，但新编号涵盖了自应用从 Zygote 分叉以来已分配给应用 Java 堆的所有物理内存页。因此，这可以准确表示您的应用实际使用了多少物理内存。

注意：当使用运行 Android 8.0（API 级别 26）及更高版本的设备时，内存分析器还会在您的应用中显示一些误报的本机内存使用情况，这些使用情况实际上属于分析工具。为 ~100k Java 对象添加了多达 10MB 的内存。在 IDE 的未来版本中，这些数字将从您的数据中过滤掉。

#### 查看内存分配

内存分配向您*展示了*内存中的每个 Java 对象和 JNI 引用是如何分配的。具体来说，Memory Profiler 可以显示以下有关对象分配的信息：

- 分配了哪些类型的对象以及它们使用了多少空间。
- 每个分配的堆栈跟踪，包括在哪个线程中。
- 当对象被*释放时*（仅当使用 Android 8.0 或更高版本的设备时）。

如果您的设备运行的是 Android 8.0 或更高版本，您可以随时查看您的对象分配，如下所示： 在时间线中拖动以选择您要查看其分配的区域（如视频 1 所示）。无需开始记录会话，因为 Android 8.0 及更高版本包含一个设备上的分析工具，可不断跟踪您的应用程序的分配。

<video controls="" poster="https://developer.android.google.cn/studio/images/profile/memory-profiler-allocations-jvmti_2x.png" data-vscid="342j13m5g" style="box-sizing: inherit; border: 0px; height: auto; max-width: 100%; color: rgb(32, 33, 36); font-family: Roboto, &quot;Noto Sans&quot;, &quot;Noto Sans JP&quot;, &quot;Noto Sans KR&quot;, &quot;Noto Naskh Arabic&quot;, &quot;Noto Sans Thai&quot;, &quot;Noto Sans Hebrew&quot;, &quot;Noto Sans Bengali&quot;, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"></video>



**视频 1.**使用 Android 8.0 及更高版本，选择现有时间线区域以查看对象分配

如果您的设备运行的是 Android 7.1 或更低版本，请单击Memory Profiler 工具栏中的**记录内存分配** ![img](https://developer.android.google.cn/studio/images/buttons/profiler-record.png)。记录时，内存分析器会跟踪应用中发生的所有分配。完成后，单击**停止录制** ![img](https://developer.android.google.cn/studio/images/buttons/profiler-record-stop.png) （相同按钮；参见视频 2）以查看分配。

<video controls="" poster="https://developer.android.google.cn/studio/images/profile/memory-profiler-allocations-record_2x.png" data-vscid="wty8cm1ov" style="box-sizing: inherit; border: 0px; height: auto; max-width: 100%; color: rgb(32, 33, 36); font-family: Roboto, &quot;Noto Sans&quot;, &quot;Noto Sans JP&quot;, &quot;Noto Sans KR&quot;, &quot;Noto Naskh Arabic&quot;, &quot;Noto Sans Thai&quot;, &quot;Noto Sans Hebrew&quot;, &quot;Noto Sans Bengali&quot;, sans-serif; font-size: 16px; font-style: normal; font-variant-ligatures: normal; font-variant-caps: normal; font-weight: 400; letter-spacing: normal; orphans: 2; text-align: start; text-indent: 0px; text-transform: none; white-space: normal; widows: 2; word-spacing: 0px; -webkit-text-stroke-width: 0px; background-color: rgb(255, 255, 255); text-decoration-thickness: initial; text-decoration-style: initial; text-decoration-color: initial;"></video>



**视频 2.**对于 Android 7.1 及更低版本，您必须明确记录内存分配

在您选择时间线的一个区域后（或当您使用运行 Android 7.1 或更低版本的设备完成录制会话时），分配的对象列表将显示在时间线下方，按类名分组并按堆计数排序。

**注意：**在 Android 7.1 及更低版本上，您最多可以记录 65535 次分配。如果您的记录会话超过此限制，则只有最近的 65535 次分配会保存在记录中。（Android 8.0 及更高版本没有实际限制。）

要检查分配记录，请执行以下步骤：

1. 浏览列表以查找堆计数异常大且可能泄漏的对象。要帮助查找已知类，请单击“

   类名称”

    列标题以按字母顺序排序。然后单击一个类名。该 

   实例的查看

   窗格显示在右侧，显示出类的每个实例，如图3。

   - 或者，您可以通过单击**过滤器** ![img](https://developer.android.google.cn/studio/images/buttons/profiler_filter.png)或按 Control+F（Mac 上为 Command+F）并在搜索字段中输入类或包名称来快速定位对象。如果您从下拉菜单中选择**按调用堆栈排列，**您还可以按方法名称进行搜索 。如果要使用正则表达式，请选中**Regex**旁边的框。如果您的搜索查询区分大小写，请选中“**区分大小写**”旁边的框 。

2. 在**实例视图**窗格中，单击一个实例。该**调用堆栈**选项卡下出现，表示在该实例被分配在哪个线程。

3. 在**Call Stack**选项卡中，右键单击任意行并选择 **Jump to Source**以在编辑器中打开该代码。

![img](https://developer.android.google.cn/studio/images/profile/memory-profiler-allocations-detail_2x.png)

**图 3.**每个分配对象的详细信息显示在右侧的**实例视图中**



您可以使用已分配对象列表上方的两个菜单来选择要检查的堆以及如何组织数据。

从左侧的菜单中，选择要检查的堆：

- **默认堆**：当系统没有指定**堆**时。
- **映像堆**：系统启动映像，包含在启动期间预加载的类。这里的分配保证永远不会移动或消失。
- **zygote heap**：在 Android 系统中分叉出应用进程的写时复制堆。
- **app heap**：您的应用程序在其上分配内存的主堆。
- **JNI 堆**：显示分配和释放 Java 本机接口 (JNI) 引用的位置的堆。

从右侧的菜单中，选择如何安排分配：

- **按类排列**：根据类名称对所有分配进行分组。这是默认设置。
- **按包排列**：根据包名称对所有分配进行分组。
- **按调用堆栈排列**：将所有分配分组到相应的调用堆栈中。

在分析时提高应用程序性能

为了在分析时提高应用程序性能，默认情况下内存分析器会定期对内存分配进行采样。在运行 API 级别 26 或更高级别的设备上进行测试时，您可以使用**分配跟踪**下拉菜单更改此行为。可用选项如下：

- **Full**：捕获内存中的所有对象分配。这是 Android Studio 3.2 及更早版本中的默认行为。如果您的应用程序分配了大量对象，您可能会在分析时观察到应用程序的明显减速。
- **采样**：定期对内存中的对象分配进行**采样**。这是默认选项，在分析时对应用程序性能的影响较小。在短时间内分配大量对象的应用程序可能仍会表现出明显的减速。
- **Off**：停止跟踪应用程序的内存分配。

注意：默认情况下，Android Studio 在执行 CPU 记录时停止跟踪实时分配，并在 CPU 记录完成后重新开启。您可以在[CPU 记录配置对话框中](https://developer.android.google.cn/studio/profile/cpu-profiler#configurations)更改此行为 。

查看全局JNI引用

*Java Native Interface* (JNI) 是一种允许Java 代码和本机代码相互调用的框架。

JNI 引用由本机代码手动管理，因此本机代码使用的 Java 对象可能会保持活动太长时间。如果 JNI 引用未先明确删除就被丢弃，则 Java 堆上的某些对象可能无法访问。此外，可以用尽全局 JNI 引用限制。

要解决此类问题，请使用Memory Profiler 中的**JNI 堆**视图浏览所有全局 JNI 引用并按 Java 类型和本机调用堆栈对其进行过滤。通过这些信息，您可以找到创建和删除全局 JNI 引用的时间和位置。

在您的应用程序运行时，选择要检查的时间线部分，然后从类列表上方的下拉菜单中选择**JNI 堆**。然后，您可以像往常一样检查堆中的对象，并双击**Allocation Call Stack**选项卡中的对象，以查看 JNI 引用在代码中的分配和释放位置，如图 4 所示。

![img](https://developer.android.google.cn/studio/images/memory-profiler-jni-heap_2x.png)

**图 4.**查看全局 JNI 引用



要检查应用 JNI 代码的内存分配，您必须将应用部署到运行 Android 8.0 或更高版本的设备。

本机内存你分析器

Android Studio Memory Profiler 包括一个 Native Memory Profiler，用于部署到运行 Android 10 的物理设备的应用；[Android Studio 4.2 预览版](https://developer.android.google.cn/studio/preview)目前支持 Android 11 设备。

本机内存分析器在特定时间段内跟踪本机代码中对象的分配/解除分配，并提供以下信息：

- **分配：** 在选定时间段内通过`malloc()`或`new`操作员分配的对象计数。
- 解除分配**：**在选定时间段内通过`free()`或 `delete`操作员解除分配的对象计数。
- **分配大小：**选定时间段内所有分配的聚合大小（以字节为单位）。
- **释放大小：**选定时间段内所有已释放内存的聚合大小（以字节为单位）。
- **总记录数：**在价值**分配**列在零下值**解除分配**列。
- **剩余大小：**在价值**分配大小**列在零下值**解除分配大小**列。

![本机内存分析器](https://developer.android.google.cn/studio/images/profile/native_memory_profiler.png)

要启动记录，请单击Memory Profiler 窗口顶部的**Record native allocations**：

![记录本机分配按钮](https://developer.android.google.cn/studio/images/profile/record_native_allocations.png)

当您准备好完成录制时，单击**停止录制**。

默认情况下，Native Memory Profiler 使用 32 字节的样本大小：每次分配 32 字节的内存时，都会拍摄内存快照。较小的样本量会产生更频繁的快照，从而产生更准确的内存使用数据。较大的样本量会产生不太准确的数据，但它会消耗更少的系统资源并提高记录时的性能。

要更改 Native Memory Profiler 的样本大小：

1. 选择**运行 > 编辑配置**。
2. 在左侧窗格中选择您的应用模块。
3. 单击**Profiling**选项卡，然后在标记为**Native memory sampling interval (bytes)**的字段中输入样本大小 。
4. 再次构建并运行您的应用程序。

注意：Native Memory Profiler 提供的内存数据不同于[内存分析器为 Java heap](https://developer.android.google.cn/studio/profile/memory-profiler#record-allocations)提供的数据。与 Java 堆上的对象不同，本机内存分析器仅跟踪通过 C/C++ 分配器进行的分配，包括本机 JNI 对象。

Native Memory Profiler 建立`heapprofd`在 Perfetto 性能分析工具堆栈之上。有关 Native Memory Profiler 内部结构的更多信息，请参阅 [`heapprofd`文档](https://perfetto.dev/docs/data-sources/native-heap-profiler)。

#### 捕获堆转储

堆转储显示在您捕获堆转储时应用程序中的哪些对象正在使用内存。尤其是在扩展用户会话之后，堆转储可以通过显示仍在内存中您认为不应再存在的对象来帮助识别内存泄漏。

捕获堆转储后，您可以查看以下内容：

- 您的应用程序分配了哪些类型的对象，以及每个对象的数量。
- 每个对象使用了多少内存。
- 在您的代码中保存对每个对象的引用。
- 分配对象的调用堆栈。（当您在记录分配时捕获堆转储时，调用堆栈当前仅适用于 Android 7.1 及更低版本的堆转储。）

要捕获堆转储，请单击 Memory Profiler 工具栏中的**Dump Java heap** ![img](https://developer.android.google.cn/studio/images/buttons/profiler-heap-dump.png)。转储堆时，Java 内存量可能会暂时增加。这是正常的，因为堆转储发生在与您的应用程序相同的进程中，并且需要一些内存来收集数据。

堆转储出现在内存时间线下方，显示堆中的所有类类型，如图 5 所示。

![img](https://developer.android.google.cn/studio/images/profile/memory-profiler-dump_2x.png)

**图 5.**查看堆转储



如果您需要更精确地了解何时创建转储，您可以通过调用在应用程序代码的关键点创建堆转储 `dumpHprofData()`。

在类列表中，您可以看到以下信息：

- **分配**：堆中的分配数量。

- **本机大小**：此对象类型使用的本机内存总量（以字节为单位）。此列仅适用于 Android 7.0 及更高版本。

  您将在此处看到一些在 Java 中分配的对象的内存，因为 Android 将本机内存用于某些框架类，例如 `Bitmap`.

- **Shallow Size**：此对象类型使用的 Java 内存总量（以字节为单位）。

- **保留大小**：由于此类的所有实例而保留的内存总大小（以字节为单位）。

您可以使用已分配对象列表上方的两个菜单来选择要检查的堆转储以及如何组织数据。

从左侧的菜单中，选择要检查的堆：

- **默认堆**：当系统没有指定**堆**时。
- **app heap**：您的应用程序在其上分配内存的主堆。
- **映像堆**：系统启动映像，包含在启动期间预加载的类。这里的分配保证永远不会移动或消失。
- **zygote heap**：在 Android 系统中分叉出应用进程的写时复制堆。

从右侧的菜单中，选择如何安排分配：

- **按班级排列**：根据班级名称对所有分配进行分组。这是默认设置。
- **按包排列**：根据包名称对所有分配进行分组。
- **按调用堆栈排列**：将所有分配分组到相应的调用堆栈中。仅当您在记录分配时捕获堆转储时，此选项才有效。即便如此，堆中可能有对象在您开始记录之前已分配，因此这些分配首先出现，仅按类名列出。

默认情况下，列表按**保留大小**列排序。要按不同列中的值排序，请单击该列的标题。

单击类名以打开右侧的**Instance View**窗口（如图 6 所示）。每个列出的实例包括以下内容：

- **深度**：从任何 GC 根到所选实例的最短跳数。
- **本机大小**：本机内存中此实例的大小。此列仅适用于 Android 7.0 及更高版本。
- **Shallow Size**：此实例在 Java 内存中的大小。
- **保留大小**：此实例支配的内存大小（根据[支配树](https://en.wikipedia.org/wiki/Dominator_(graph_theory))）。

注意：默认情况下，堆转储并*没有*显示每个分配的对象堆栈跟踪。要获取堆栈跟踪，您必须在单击**Dump Java heap**之前 开始 [记录内存分配](https://developer.android.google.cn/studio/profile/memory-profiler?hl=en#record-allocations)。然后，您可以在**Instance View 中**选择一个实例，并在**References**选项卡旁边 看到**Call Stack**选项卡，如图 6 所示。但是，很可能在您开始记录分配之前已经分配了一些对象，因此调用堆栈不可用对于那些对象。包含调用堆栈的实例在图标上用“堆栈”标记表示 ![img](https://developer.android.google.cn/studio/images/profile/memory-profiler-icon-stack.png). （遗憾的是，由于堆栈跟踪要求您执行分配记录，因此您目前无法在 Android 8.0 上看到堆转储的堆栈跟踪。）

要检查您的堆，请按照下列步骤操作：

1. 浏览列表以查找堆计数异常大且可能泄漏的对象。要帮助查找已知类，请单击“

   类名称”

    列标题以按字母顺序排序。然后单击一个类名。该 

   实例的查看

   窗格显示在右侧，显示出类的每个实例，如图6。

   - 或者，您可以通过单击**过滤器** ![img](https://developer.android.google.cn/studio/images/buttons/profiler_filter.png)或按 Control+F（Mac 上为 Command+F）并在搜索字段中输入类或包名称来快速定位对象。如果您从下拉菜单中选择**按调用堆栈排列，**您还可以按方法名称进行搜索 。如果要使用正则表达式，请选中**Regex**旁边的框。如果您的搜索查询区分大小写，请选中“**区分大小写**”旁边的框 。

2. 在实例视图

   窗格中，单击一个实例。所述参考标签下出现，显示每个参考到该对象。

   或者，单击实例名称旁边的箭头以查看其所有字段，然后单击字段名称以查看其所有引用。如果要查看某个字段的实例详细信息，请右键单击该字段并选择**Go to Instance**。

3. 在**References**选项卡中，如果您确定一个可能泄漏内存的引用，请右键单击它并选择**Go to Instance**。这将从堆转储中选择相应的实例，向您显示其自己的实例数据。

在堆转储中，查找由以下任一原因引起的内存泄漏：

- 长期引用`Activity`，`Context`， `View`，`Drawable`，并可能保持在参考其他物体`Activity`或`Context`容器。
- `Runnable`可以保存`Activity`实例的非静态内部类，例如 a 。
- 保存对象超过必要时间的缓存。

将堆转储另存为HPRDF文件

捕获堆转储后，只有在分析器运行时才能在内存分析器中查看数据。当您退出分析会话时，您将丢失堆转储。因此，如果您想保存以供稍后查看，请将堆转储导出到 HPROF 文件。在 Android Studio 3.1 及更低版本中，**Export capture to file** ![img](https://developer.android.google.cn/studio/images/buttons/profiler-export-hprof.png)按钮位于时间轴下方工具栏的左侧；在 Android Studio 3.2 及更高版本中，**Sessions**窗格中每个**Heap Dump**条目的右侧都有一个**Export Heap Dump**按钮。在出现的“**导出为”** 对话框中，使用文件扩展名保存文件。`.hprof`

要使用不同的 HPROF 分析器，例如 [jhat](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jhat.html)，您需要将 HPROF 文件从 Android 格式转换为 Java SE HPROF 格式。您可以使用目录中`hprof-conv`提供的工具 执行此操作`android_sdk/platform-tools/`。运行`hprof-conv` 带有两个参数的命令：原始 HPROF 文件和写入转换后的 HPROF 文件的位置。例如：

```
hprof-conv heap-original.hprof heap-converted.hprof
```

导入堆转储文件

要导入 HPROF ( `.hprof`) 文件，请单击**会话**窗格中的 **启动新的分析会话** ，选择**从文件加载**，然后从文件浏览器中选择文件。![img](https://developer.android.google.cn/studio/images/buttons/ic_plus.png)

您还可以通过将 HPROF 文件从文件浏览器拖到编辑器窗口中来导入它。

#### 泄漏检测

在 Memory Profiler 中分析堆转储时，您可以过滤 Android Studio 认为可能表明您的应用程序`Activity`和 `Fragment`实例存在内存泄漏的分析数据。

过滤器显示的数据类型包括：

- `Activity` 已被销毁但仍在被引用的实例。
- `Fragment`没有有效`FragmentManager`但仍在被引用的实例。

在某些情况下，例如以下情况，过滤器可能会产生误报：

- A`Fragment`已创建但尚未使用。
- A`Fragment`正在缓存，但不是作为`FragmentTransaction`.

要使用此功能，请先[捕获堆转储](https://developer.android.google.cn/studio/profile/memory-profiler?hl=en#capture-heap-dump) 或[将堆转储文件](https://developer.android.google.cn/studio/profile/memory-profiler?hl=en#import-hprof) 导入 Android Studio。要显示可能泄漏内存的片段和活动，请选中Memory Profiler 的堆转储窗格中的**Activity/Fragment Leaks**复选框，如图 7 所示。

![Profiler：内存泄漏检测](https://developer.android.google.cn/studio/images/profile/profiler-memory-leak-detection.png)

**图 7.**过滤堆转储以查找内存泄漏。



### 分析你的记忆的技巧

在使用 Memory Profiler 时，您应该强调您的应用程序代码并尝试强制内存泄漏。在您的应用程序中引发内存泄漏的一种方法是在检查堆之前让它运行一段时间。泄漏可能会渗透到堆中分配的顶部。但是，泄漏越小，您需要运行应用程序才能看到它的时间越长。

您还可以通过以下方式之一触发内存泄漏：

- 在不同的活动状态下将设备从纵向旋转到横向并再次旋转多次。旋转设备往往会导致应用程序泄漏的 `Activity`， `Context`或 `View`因为系统重新创建对象`Activity`，如果您的应用程序保存到其中的一个参考对象在其他地方，系统无法垃圾收集。
- 处于不同活动状态时在您的应用程序和另一个应用程序之间切换（导航到主屏幕，然后返回到您的应用程序）。

### Memory Analyzer

MAT使用步骤：

```
1.在Android Studio中打开Memory profiler，dump一段时间内的内存，并保存内存快照（Hprof）
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7f705113897ef?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



接下来将Android Studio的内存快照转换为MAT支持的格式

```
2.打开终端，切换到Android SDK的platform-tools目录下
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7f7f4accb8085?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



```
3.执行转换命令：hprof-conv 内存快照的地址 转换后写入内存快照地址
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7f8d139effdb2?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



```
4.打开MAT，点击workbench进入MAT工作界面
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7f93af8fc632a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



```
5.点击File->open file,打开转换后的hprof文件。
复制代码
```



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7f93ed7a019b3?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)





![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7f98d29724a09?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



怎么打开MAT就介绍完了，MAT工具十分的强大，操作也相对比较复杂，本文无法全面的介绍MAT全部功能，我们根据下图的标识，来着重介绍一些MAT中常用的功能，请仔细对号阅读。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c80097e1b94549?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**1.Histogram**

查看当前内存中每一个class具体产生了多少实例，以及这些实例的Shallow Heap和Retained Heap。例如：一个activity，在内存中产生一个以上的实例，那么这个activity就非常有可能发生内存泄漏。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7fb7ad7448c94?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



在选中对象上右击选择List objetcts。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7fb9fb59aca17?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



with outgoing references：当前类引用了哪些类。

with incoming references：当前类被哪些类引用。

这两个属性在内存泄漏的调试中经常使用，我们可以根据当前类的引用链一直追溯到真正导致内存泄漏的类，从而排除内存泄漏。

**2.Dominator Tree**

以百分比的形式展示出在当前内存中占据内存最多的**对象实例**。它也是我们在减少APP内存占用时需要重点要观察的地方之一。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7fcb46f5dd482?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**3.Top Consumers**

通过图形的形式列出来比较占用内存的对象。它的下面还有一个Biggest Objects，从名字上就能看出，它里面包含了在内存中占据内存最多的几个对象的信息。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7ff6b4a8441c1?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



Top Consumers和上面介绍的Dominator Tree都是我们在考虑减少APP内存占用时需要重点观察的地方。找到最占内存的对象实例，并尽可能的减小它占据的内存，如果是内存泄漏则应该直接回收它。

**4.Leak Suspects**

在Leak Suspects中会直接给出MAT对于内存中存在问题的分析，点击Details就能查看导致当前内存问题类的的引用链，MAT会自动化的帮助我们找到内存泄漏的具体原因，这也是MAT中查找内存泄漏最快的方法。

不过有时候，手机系统的内存问题也会在这里面给出反馈。对于手机系统的bug，可以不必理会。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c8007be518c2fc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**5.OQL**

OQL一种数据查询语言，使用它我们就可以一种类似SQL语句形式，查询出我们需要的类的信息。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7fe5f7b17d175?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**6.thread_overview**

产看当前内存中存在的线程信息。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c7feaff93fd1a4?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**7.unreachable Objects Histogram**

内存中可被回收的对象，但是现在未被回收的对象，这些未被回收对象可以作为参考，并不一定是导致APP内存泄漏的原因。

### LeakCanary

Leak Canary是大名鼎鼎的Square公司专门为检测Android内存泄漏而开发一个第三方框架。需要注意的是，LeakCanary只能用来监控内存泄漏，它并不支持监控其他的内存问题。

github地址：[github.com/square/leak…](https://github.com/square/leakcanary)

英文帮助文档：[square.github.io/leakcanary/](https://square.github.io/leakcanary/)

**LeakCanary的使用**

最新版的LeakCanary在使用时，不需要做任何初始化操作，只需要在项目的build.gradle中添加以下依赖即可。

```
dependencies  { 
  // debugImplementation，因为LeakCanary应该只在调试版本中运行。
  debugImplementation'com.squareup.leakcanary:leakcanary-android:2.0-beta-2' 
}
复制代码
```

运行APP后会在手机生成一个Leaks的APP，当在我们在调试集成了LeakCanary的APP时（仅在debug模式下使用），如果检测到内存泄漏时，LeakCanary将自动在手机上显示通知，并将内存泄漏的信息保存在LeaksAPP中。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c80205553c200d?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



**排查内存泄漏**

当产生内存泄漏后，LeakCanary会给出如下图所示的内存泄漏的引用链。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c802affea6102a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



泄漏跟踪中的每个节点都是Java对象，可以是类，对象数组或实例。 每个节点都有一个对下一个节点的引用。在UI中，该引用为紫色。

在LeakCanary给出报告中，每一个节点都标识了是否正在发生泄漏，在它后面的括号中还给出相应的解释。

- Leaking：YES 正在发生泄漏，
- Leaking：NO 没有发生泄漏，
- Leaking：UNKNOWN 未知。

大致观察LeakCanary的报告后，我们就需要开始缩小观察范围来确定内存泄漏的原因。

在LeakCanary有这样一条规则，如果一个节点没有泄漏，那么指向它的任何先前引用都不是泄漏源，也不会泄漏。同样，如果一个节点泄漏，那么泄漏跟踪下的任何节点也会泄漏。由此，我们可以推断出**内存泄漏原因出现在最后一次Leaking：NO和第一次Leaking：YES之间类中**。

在本例中就对应下图的这四个部分，泄漏的原因往往就出这里面。在报告中用**红色下波浪线标出来的部分，是LeakCanary认为导致内存泄漏的原因，也是我们接下来要重点排查的地方**。



![img](https://user-gold-cdn.xitu.io/2019/8/11/16c807d9559b5e54?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

我们直接查看Application中的leakedViews，会发现正是在Application中保存View的代码导致了内存泄漏的发生，接下来就是尝试如何修复这段代码了。



```
val leakedViews = mutableListOf <View>（）
复制代码
```

示例只是给出，如何根据LeakCanary的报告，缩小排查范围，并一步步找到内存泄漏的原因，请不要去关注示例中的这段代码是如何会导致内存泄漏的。

实际开发中，通过引入LeakCanary基本就可以找到绝大多数的内存泄漏。通过定制LeakCanary，我们甚至有能力在APP发布后依然能够保证获取开发时遗漏的内存泄漏点，关于定制请参考LeakCanary的官方文档：[square.github.io/leakcanary/…](https://square.github.io/leakcanary/recipes/)

### 实际应用场景

**小团队**

这类型团队是当前国内占比较多的一部分，Android开发组长期只有一两个人，APP的日活跃用户也比较少，却有着大量的需求亟待完成，甚至于需求本身可能都十分模糊。对于这样的团队关注的重心要集中在业务和功能上，如何保证APP不出bug才是重点。

**建议**：在APP中集成LeakCanary，整理、收集内存泄漏的报告，在空闲时尝试调试内存问题。调试之后一定要做充分测试，防止出现其他bug。

**中等规模团队**

这类型的团队基本长期都有三个人以上，APP的日活跃用户数比较多。这种团队的leader要适时关注一下APP的使用流畅度，着手解决APP中的内存泄漏、卡顿等问题，并定期发布相关的团队报告，让团队中其他人引以为戒。

**建议**：在APP中集成LeakCanary，每个开发人员在完成自己开发任务的同时，也要保证自己开发的功能不会出现被LeakCanary捕获的内存泄漏。如果不能根据LeakCanary定位内存泄漏的点，需要进一步使用MAT来排查。

团队的leader在版本发布前，要使用Memory Profiler监测每个新功能的内存时间线，图像的时间线相对平滑则是合格的。如果出现了剧烈波动的锯齿图像，表明出现了内存抖动，要着手修复，保持这样的节奏基本可以避免绝大多数内存方面的性能问题。监测的任务也可以交给团队内的测试人员。



## 性能检测工具

### cpu性能分析器

优化应用的 CPU 使用率能带来诸多好处，如提供更快、更顺畅的用户体验，以及延长设备电池续航时间。

您可以使用 CPU 性能剖析器在与应用交互时实时检查应用的 CPU 使用率和线程活动，也可以检查记录的方法跟踪数据、函数跟踪数据和系统跟踪数据的详情。

CPU 性能剖析器记录和显示的具体信息类型取决于您选择的记录配置：

- **System Trace**：捕获精细的详细信息，以便您检查应用与系统资源的交互情况。

- **Method and function traces**：对于应用进程中的每个线程，您可以了解一段时间内执行了哪些方法 (Java) 或函数 (C/C++)，以及每个方法或函数在其执行期间消耗的 CPU 资源。您还可以使用方法和函数跟踪数据来识别调用方和被调用方。调用方是指调用其他方法或函数的方法或函数，而被调用方是指被其他方法或函数调用的方法或函数。您可以使用这些信息来确定哪些方法或函数过于频繁地调用通常会消耗大量资源的特定任务，并优化应用的代码以避免不必要的工作。

  记录方法跟踪数据时，您可以选择“sampled”或“instrumented”记录方式。记录函数跟踪数据时，只能使用“sampled”记录方式。

#### cpu分析器概览

如需打开 CPU 性能剖析器，请按以下步骤操作：

1. 依次选择 **View > Tool Windows > Profiler** 或点击工具栏中的 **Profile** 图标 ![img](https://developer.android.com/studio/images/buttons/toolbar-android-profiler_dark.png?hl=zh-cn)。

   如果 **Select Deployment Target** 对话框显示提示，请选择需将您的应用部署到哪个设备上以进行性能剖析。如果您已通过 USB 连接设备但系统未列出该设备，请确保您已[启用 USB 调试](https://developer.android.com/studio/debug/dev-options?hl=zh-cn#enable)。

2. 点击 **CPU** 时间轴上的任意位置以打开 CPU 性能剖析器。

当您打开 CPU 性能剖析器时，它会立即开始显示应用的 CPU 使用率和线程活动。系统会显示类似于图 1 的界面。

![img](https://developer.android.com/studio/images/profile/cpu_profiler_L2-2X.png?hl=zh-cn)

**图 1.** CPU 性能剖析器中的时间轴。



如图 1 所示，CPU 性能剖析器的默认视图包括以下时间轴：

1. **事件时间轴**：显示应用中的 Activity 在其生命周期内不断转换经历各种不同状态的过程，并指示用户与设备的交互，包括屏幕旋转事件。如需了解如何在搭载 Android 7.1（API 级别 25）及更低版本的设备上启用事件时间轴，请参阅[启用高级性能剖析](https://developer.android.com/studio/profile/android-profiler?hl=zh-cn#advanced-profiling)。

2. **CPU 时间轴**：显示应用的实时 CPU 使用率（以占总可用 CPU 时间的百分比表示）以及应用当前使用的线程总数。此时间轴还会显示其他进程（如系统进程或其他应用）的 CPU 使用率，以便您可以将其与您应用的 CPU 使用率进行对比。您可以通过沿时间轴的横轴方向移动鼠标来检查历史 CPU 使用率数据。

3. 线程活动时间轴

   ：列出属于应用进程的每个线程，并使用下面列出的颜色在时间轴上指示它们的活动。记录跟踪数据后，您可以从此时间轴上选择一个线程，以在跟踪数据窗格中检查其数据。

   - **绿色**：表示线程处于活动状态或准备使用 CPU。也就是说，线程处于正在运行或可运行状态。
   - **黄色**：表示线程处于活动状态，但它正在等待一项 I/O 操作（如磁盘或网络 I/O），然后才能完成它的工作。
   - **灰色**：表示线程正在休眠且没有消耗任何 CPU 时间。 当线程需要访问尚不可用的资源时，就会出现这种情况。在这种情况下，要么线程主动进入休眠状态，要么内核将线程置于休眠状态，直到所需的资源可用。

   CPU 性能剖析器还会报告 Android Studio 和 Android 平台添加到应用进程的线程的 CPU 使用率，这些线程包括 `JDWP`、`Profile Saver`、`Studio:VMStats`、`Studio:Perfa` 和 `Studio:Heartbeat` 等（不过，它们在线程活动时间轴上显示的确切名称可能有所不同）。Android Studio 报告此数据是为了方便您确定线程活动和 CPU 使用率什么时候是由您的应用的代码实际引发的。

#### 记录跟踪数据

如需开始记录跟踪数据，请从 CPU 性能剖析器上方或下方的下拉菜单中[选择记录配置](https://developer.android.com/studio/profile/cpu-profiler?hl=zh-cn#configurations)，然后点击 **Record**。

![img](https://developer.android.com/studio/images/profile/cpu-profiler.png?hl=zh-cn)

**图 2.** CPU 性能剖析器显示正在进行的记录的状态、持续时间和类型。



与您的应用交互，然后在完成时点击 **Stop**。性能剖析器会自动在跟踪数据窗格中显示其跟踪信息，如图 3 所示：

![img](https://developer.android.com/studio/images/profile/sample-java-methods.png?hl=zh-cn)

**图 3.** 记录方法跟踪数据后的 CPU 性能剖析器。



1. **选定范围**：确定需在跟踪数据窗格中检查所记录时间的哪一部分。当您首次记录跟踪数据时，CPU 性能剖析器会自动在 CPU 时间轴上选择记录的完整长度。如需仅检查已记录的时间范围中的一部分的跟踪数据，请拖动突出显示区域的边缘。

2. **“Interaction”部分**：沿着时间轴显示用户互动和应用生命周期事件。

3. “Threads”部分

   ：沿时间轴针对每一个线程显示线程状态活动（例如运行、休眠等）和

   调用图表

   （在 System Trace 中则为跟踪事件图表）。

   - 使用[鼠标和键盘快捷键](https://developer.android.com/studio/profile/cpu-profiler?hl=zh-cn#ui-shortcuts)在时间轴上导航。
   - 双击线程名称，或在选中线程时按 Enter 键以展开或折叠线程。
   - 选择某个线程即可在“Analysis”窗格中查看更多信息。 按住 Shift 或 Ctrl（Mac 上为 Command）可选择多个线程。
   - 选择方法调用（或 System Trace 中的跟踪事件），以在“Analysis”窗格中查看更多信息。

4. **“Analysis”窗格**：显示您选择的时间范围和线程/方法调用的跟踪数据。在此窗格中，您可以选择如何查看每个堆栈轨迹（使用“Analysis”标签页），以及如何测量执行时间（使用“Time reference”下拉菜单）。

5. **“Analysis”窗格标签页**：选择如何显示跟踪数据详细信息。如需详细了解各个选项，请参阅[检查跟踪数据](https://developer.android.com/studio/profile/cpu-profiler?hl=zh-cn#inspect-traces)。

6. "Time reference"菜单

   ：选择以下选项之一，以确定如何测量每次调用的时间信息（仅在

   示例/跟踪 Java 方法

   中受支持）：

   - **Wall clock time**：该时间信息表示实际经过的时间。
   - **Thread time**：该时间信息表示实际经过的时间减去线程没有占用 CPU 资源的那部分时间。对于任何给定的调用，其线程时间始终小于或等于其挂钟时间。使用线程时间可以让您更好地了解线程的实际 CPU 使用率中有多少是给定方法或函数占用的。

7. Filter

   ：按函数、方法、类或软件包名称过滤跟踪数据。例如，如果您需快速识别与特定调用相关的跟踪记录数据，请在搜索字段中输入名称。在

    

   Flame chart

    

   标签页中，会突出显示包含符合搜索查询条件的调用、软件包或类的调用堆栈。在

    

   Top down

    

   和

    

   Bottom up

    

   标签页中，这些调用堆栈优先于其他跟踪结果。您还可以通过勾选搜索字段旁边的相应方框来启用以下选项：

   - **Regex**：如需在您的搜索中包含正则表达式，请使用此选项。
   - **Match case**：如果您的搜索区分大小写，请使用此选项。



选择记录配置

在开始记录跟踪信息之前，请为需捕获的分析信息选择适当的记录配置：

- 对 Java 方法采样

  ：在应用的 Java 代码执行期间，频繁捕获应用的调用堆栈。分析器会比较捕获的数据集，以推导与应用的 Java 代码执行有关的时间和资源使用信息。

  基于采样的跟踪存在一个固有的问题，那就是如果应用在捕获调用堆栈后进入一个方法并在下次捕获前退出该方法，分析器将不会记录该方法调用。如果您想要跟踪生命周期如此短的方法，应使用插桩跟踪。

- 跟踪 Java 方法

  ：在运行时检测应用，从而在每个方法调用开始和结束时记录一个时间戳。系统会收集并比较这些时间戳，以生成方法跟踪数据，包括时间信息和 CPU 使用率。

  请注意，与检测每个方法相关的开销会影响运行时性能，并且可能会影响分析数据；对于生命周期相对较短的方法，这一点更为明显。此外，如果应用在短时间内执行大量方法，则分析器可能很快就会超出其文件大小限制，因而不能再记录更多跟踪数据。

- 对 C/C++ 函数采样

  ：捕获应用的原生线程的采样跟踪数据。如需使用此配置，您必须将应用部署到搭载 Android 8.0（API 级别 26）或更高版本的设备上。

  在内部，此配置使用 `simpleperf` 跟踪应用的原生代码。如果需为 `simpleperf` 指定其他选项，如对特定设备 CPU 采样或指定高精度采样持续时间，您可以[从命令行使用 `simpleperf`](https://developer.android.com/ndk/guides/simpleperf-commands?hl=zh-cn)。

- 跟踪系统调用

  ：捕获非常翔实的细节，以便您检查应用与系统资源的交互情况。您可以检查线程状态的确切时间和持续时间、直观地查看所有内核的 CPU 瓶颈在何处，并添加需分析的自定义跟踪事件。当您排查性能问题时，此类信息至关重要。如需使用此配置，您必须将应用部署到搭载 Android 7.0（API 级别 24）或更高版本的设备上。

  使用此跟踪配置时，您可以通过检测代码，直观地标记性能剖析器时间轴上的重要代码例程。如需检测 C/C++ 代码，请使用由 `trace.h` 提供的[原生跟踪 API](https://developer.android.com/topic/performance/tracing/custom-events-native?hl=zh-cn)。如需检测 Java 代码，请使用 [`Trace`](https://developer.android.com/reference/android/os/Trace?hl=zh-cn) 类。如需了解详情，请参阅[检测您的应用代码](https://developer.android.com/topic/performance/tracing/custom-events?hl=zh-cn)。

  此跟踪配置建立在 `systrace` 的基础之上。您可以[使用 `systrace` 命令行实用程序](https://developer.android.com/studio/command-line/systrace?hl=zh-cn)指定除 CPU 性能剖析器提供的选项之外的其他选项。`systrace` 提供的其他系统级数据可帮助您检查原生系统进程并排查丢帧或帧延迟问题。

  在搭载 Android 9（API 级别 28）或更高版本的设备上，您可以使用一个名为“System Tracing”的系统应用来[记录设备上的系统跟踪数据](https://developer.android.com/topic/performance/tracing/on-device?hl=zh-cn)。

创建、修改或查看记录配置

您可以在 **CPU Recording Configurations** 对话框中创建、修改和查看记录配置，从 CPU 性能剖析器顶部的记录配置下拉菜单中选择 **Edit configurations** 即可打开该对话框。

如需查看某个现有记录配置的设置，请在 **CPU Recording Configurations** 对话框的左侧窗格中选择该配置。

如需创建一个新的记录配置，请执行以下操作：

1. 点击对话框左上角的 **Add** 图标 ![img](https://developer.android.com/studio/images/buttons/ic_plus.png?hl=zh-cn)。这样会创建一个包含一些默认设置的新配置。

2. 为您的配置命名。

3. 选择一种 **Trace Technology**。

4. 对于采样记录配置，以微秒 (μs) 为单位指定 **Sampling interval**。此值表示应用的每个调用堆栈样本的时间间隔。指定的时间间隔越短，达到记录数据的文件大小限制就越快。

5. 对于写入连接设备的记录数据，以兆字节 (MB) 为单位指定

    

   File size limit

   。当您停止记录时，Android Studio 会解析此数据并将其显示在分析器窗口中。因此，如果您提高此限制并记录大量的数据，Android Studio 解析文件所需的时间会大大增加，并且可能会变得无响应。

   **注意**：如果您使用的连接设备搭载的是 Android 8.0（API 级别 26）或更高版本，那么对跟踪数据的文件大小没有限制，系统会忽略此值。不过，您仍需留意每次记录后设备收集了多少数据，Android Studio 可能会无法解析大型跟踪文件。例如，如果您记录的是采样时间间隔很短的采样跟踪数据，或是在应用于短时间内调用许多方法的情况下记录检测跟踪数据，那么很快就会生成大型跟踪文件。

6. 如需接受所做的更改并继续对其他配置进行更改，请点击 **Apply**。如需接受进行的所有更改并关闭对话框，请点击 **OK**。

使用 Debug API 记录 CPU 活动

您可以使用 [`Debug` API](https://developer.android.com/reference/android/os/Debug?hl=zh-cn)，让应用能够在 CPU 性能剖析器中开始和停止记录 CPU 活动。

当您的应用调用 [`startMethodTracing(String tracePath)`](https://developer.android.com/reference/android/os/Debug?hl=zh-cn#startMethodTracing(java.lang.String)) 时，CPU 性能剖析器将开始记录；当您的应用调用 [`stopMethodTracing()`](https://developer.android.com/reference/android/os/Debug?hl=zh-cn#stopMethodTracing()) 时，CPU 性能剖析器将停止记录。在记录使用此 API 触发的 CPU 活动时，CPU 性能剖析器会将 **Debug API** 显示为正在发挥作用的 CPU 记录配置。

如需使用 `Debug` API 控制 CPU 活动的记录，请将检测的应用部署到搭载 Android 8.0（API 级别 26）或更高版本的设备上。

**重要提示**：`Debug` API 应该与用于开始和停止 CPU 活动记录的其他方法（如 CPU 性能剖析器图形界面中的按钮，以及记录配置中用于在应用启动时自动记录的设置）分开使用。

由于存在 8MB 的缓冲区空间限制，因此 `Debug` API 中的 `startMethodTracing(String tracePath)` 方法专为时间间隔较短或难以手动启动/停止记录的场景而设计。如需设置持续时间更长的记录，请使用 Android Studio 中的性能剖析器界面。

如需了解详情，请参阅[通过检测您的应用生成跟踪日志](https://developer.android.com/studio/profile/generate-trace-logs?hl=zh-cn)。

在应用启动过程中记录 CPU 活动

如需在应用启动过程中自动开始记录 CPU 活动，请执行以下操作：

1. 依次选择 **Run > Edit Configurations**。
2. 在 **Profiling** 标签中，勾选 **Start recording a method trace on startup** 旁边的复选框。
3. 从菜单中选择 CPU 记录配置。
4. 点击 **Apply**。
5. 依次选择 **Run > Profile**，将应用部署到搭载 Android 8.0（API 级别 26）或更高版本的设备。



#### 导出跟踪数据

在您使用 CPU 性能剖析器记录 CPU 活动后，您可以将相应数据导出为 `.trace` 文件，以便与他人共享或日后进行检查。

如需从 CPU 时间轴导出跟踪文件，请执行以下操作：

1. 在 CPU 时间轴上，右键点击需导出的记录的方法跟踪数据或系统跟踪数据。
2. 从菜单中选择 **Export trace**。
3. 浏览到需保存文件的目标位置，指定文件名，然后点击 **OK**。

如需从 **Sessions** 窗格导出跟踪文件，请执行以下操作：

1. 在 **Sessions** 窗格中，右键点击需导出的记录的跟踪数据。
2. 点击会话条目右侧的 **Export method trace** 或 **Export system trace** 按钮。
3. 浏览到需保存文件的目标位置，指定文件名，然后点击 **OK**。

#### 导入跟踪数据

您可以导入使用 [`Debug` API](https://developer.android.com/reference/android/os/Debug?hl=zh-cn) 或 CPU 性能剖析器创建的 `.trace` 文件。

如需导入跟踪文件，请在性能剖析器的 **Sessions** 窗格中点击 **Start new profiler session** 图标 ![img](https://developer.android.com/studio/images/buttons/ic_plus.png?hl=zh-cn)，然后选择 **Load from file**。

您可以检查导入到 CPU 性能剖析器中的跟踪数据，就像检查直接在 CPU 性能剖析器中捕获的跟踪数据一样，但有下面几点不同：

- CPU Activity 未显示在 CPU 时间轴上（在 System Trace 中除外）。
- **Threads** 部分中的时间轴不会显示线程状态，如正在运行、等待或休眠（在 System Trace 中除外）。

#### 检查跟踪记录

CPU 性能分析器中的轨迹视图提供了多种方法查看来自所记录的轨迹的信息。

对于方法轨迹和函数轨迹，您可以直接在 **Threads** 时间轴中查看 **Call Chart**，并从 **Analysis** 窗格中查看 **Flame Chart**、**Top Down**、**Bottom Up** 和 **Events** 标签页。对于系统轨迹，您可以直接在 **Threads** 时间轴中查看 **Trace Events**，并从 **Analysis** 窗格中查看 **Flame Chart**、**Top Down**、**Bottom Up** 和 **Events** 标签页。

使用[鼠标和键盘快捷键](https://developer.android.com/studio/profile/cpu-profiler?hl=zh-cn#ui-shortcuts)可以更轻松地导航 **Call Charts** 或 **Trace Events**。

### 使用 Call Chart 检查跟踪数据

**Call Chart** 以图形方式来呈现方法跟踪数据或函数跟踪数据，其中调用的时间段和时间在横轴上表示，而其被调用方则在纵轴上显示。对系统 API 的调用显示为橙色，对应用自有方法的调用显示为绿色，对第三方 API（包括 Java 语言 API）的调用显示为蓝色。图 4 显示了一个调用图表示例，说明了给定方法或函数的 Self 时间、Children 时间和 Total 时间的概念。如需详细了解这些概念，请参阅有关如何[使用“Top Down”和“Bottom Up”标签页检查跟踪数据](https://developer.android.com/studio/profile/cpu-profiler?hl=zh-cn#top_down_bottom_up)的部分。

![img](https://developer.android.com/studio/images/profile/call_chart_1-2X.png?hl=zh-cn)

**图 4.** 一个调用图表示例，说明了方法 D 的 Self 时间、Children 时间和 Total 时间。



**提示**：如需跳转到某个方法或函数的源代码，请右键点击该方法或函数，然后选择 **Jump to Source**。在“分析”窗格的任意标签页中都可以执行此操作。

### 使用“Flame Chart”标签检查跟踪数据

**Flame Chart** 标签页提供一个倒置的调用图表，用来汇总完全相同的调用堆栈。也就是说，将具有相同调用方顺序的完全相同的方法或函数收集起来，并在火焰图中将它们表示为一个较长的横条（而不是将它们显示为多个较短的横条，如调用图表中所示）。这样更方便您查看哪些方法或函数消耗的时间最多。不过，这也意味着，横轴不代表时间轴，而是表示执行每个方法或函数所需的相对时间。

为帮助说明此概念，不妨考虑图 5 中的调用图表。请注意，方法 D 多次调用 B（B1、B2 和 B3），其中一些对 B 的调用也调用了 C（C1 和 C3）。

![img](https://developer.android.com/studio/images/profile/call_chart_2-2X.png?hl=zh-cn)

**图 5.** 一个调用图表，其中的多个方法调用具有相同的调用方顺序。



由于 B1、B2 和 B3 具有相同的调用方顺序 (A → D → B)，因此系统将它们汇总在一起，如图 6 所示。同样，也将 C1 和 C3 汇总在一起，因为它们也具有相同的调用方顺序 (A → D → B → C)。请注意，C2 不包括在内，因为它具有不同的调用方顺序 (A → D → C)。

![img](https://developer.android.com/studio/images/profile/flame_chart_aggregation-2X.png?hl=zh-cn)

**图 6.** 汇总具有相同调用堆栈的完全相同的方法。



汇总的调用用于创建火焰图，如图 7 所示。请注意，对于火焰图中的任何给定调用，占用最多 CPU 时间的被调用方最先显示。

![img](https://developer.android.com/studio/images/profile/flame_chart-2X.png?hl=zh-cn)

**图 7.** 图 5 中所示调用图表的火焰图表示形式。



### 使用“Top Down”和“Bottom Up”检查跟踪数据

**Top Down** 标签显示一个调用列表，在该列表中展开方法或函数节点会显示它的被调用方。图 8 显示了图 4 中调用图表的自上而下图。图中的每个箭头都从调用方指向被调用方。

如图 8 所示，在 **Top Down** 标签页中展开方法 A 的节点会显示它的被调用方，即方法 B 和 D。在此之后，展开方法 D 的节点会显示它的被调用方，即方法 B 和 C，依此类推。与 **Flame chart** 标签页类似，“Top Down”树也汇总了具有相同调用堆栈的完全相同的方法的跟踪信息。也就是说，**Flame chart** 标签页提供了 **Top down** 标签页的图形表示方式。

**Top Down** 标签提供以下信息来帮助说明在每个调用上所花的 CPU 时间（时间也可表示为在选定范围内占线程总时间的百分比）：

- **Self**：方法或函数调用在执行自己的代码（而非被调用方的代码）上所花的时间，如图 4 中的方法 D 所示。
- **Children**：方法或函数调用在执行它的被调用方（而非自己的代码）上所花的时间，如图 4 中的方法 D 所示。
- **Total**：方法的 **Self** 时间和 **Children** 时间的总和。这表示应用在执行调用时所用的总时间，如图 4 中的方法 D 所示。

![img](https://developer.android.com/studio/images/profile/top_down_tree-2X.png?hl=zh-cn)

**图 8.** 一个“Top Down”树。



![img](https://developer.android.com/studio/images/profile/bottom_up_tree-2X.png?hl=zh-cn)

**图 9.** 图 8 中方法 C 的“Bottom Up”树。



**Bottom Up** 标签页显示一个调用列表，在该列表中展开函数或方法的节点会显示它的调用方。沿用图 8 中所示的跟踪数据示例，图 9 提供了方法 C 的“Bottom Up”树。在该“Bottom Up”树中打开方法 C 的节点会显示它独有的各个调用方，即方法 B 和 D。请注意，尽管 B 调用 C 两次，但在“Bottom Up”树中展开方法 C 的节点时，B 仅显示一次。在此之后，展开 B 的节点会显示它的调用方，即方法 A 和 D。

**Bottom Up** 标签页用于按照占用的 CPU 时间由多到少（或由少到多）的顺序对方法或函数排序。您可以检查每个节点以确定哪些调用方在调用这些方法或函数上所花的 CPU 时间最多。与“Top Down”树相比，“Bottom Up”树中每个方法或函数的时间信息参照的是每个树顶部的方法（顶部节点）。CPU 时间也可表示为在该记录期间占线程总时间的百分比。下表说明了如何解读顶部节点及其调用方（子节点）的时间信息。

|                                           | Self                                                         | Children                                                     | Total                             |
| :---------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :-------------------------------- |
| “Bottom Up”树顶部的方法或函数（顶部节点） | 表示方法或函数在执行自己的代码（而非被调用方的代码）上所花的总时间。与“Top Down”树相比，此时间信息表示在记录的持续时间内对此方法或函数的所有调用时间的总和。 | 表示方法或函数在执行它的被调用方（而非自己的代码）上所花的总时间。与“Top Down”树相比，此时间信息表示在记录的持续时间内对此方法或函数的被调用方的所有调用时间的总和。 | Self 时间和 Children 时间的总和。 |
| 调用方（子节点）                          | 表示被调用方在由调用方调用时的总 Self 时间。以图 9 中的“Bottom Up”树为例，方法 B 的 Self 时间将等于每次执行由方法 B 调用的方法 C 所用的 Self 时间的总和。 | 表示被调用方在由调用方调用时的总 Children 时间。以图 9 中的“Bottom Up”树为例，方法 B 的 Children 时间将等于每次执行由方法 B 调用的方法 C 所用的 Children 时间的总和。 | Self 时间和 Children 时间的总和。 |

**注意**：对于给定的记录，当分析器达到文件大小限制时，Android Studio 会停止收集新数据（不过，不会停止记录）。在执行检测跟踪时，这种情况通常发生得更快，因为与采样跟踪相比，此类跟踪会在更短的时间内收集更多的数据。如果您将检查时间范围延长至达到限制后的记录时段，则跟踪数据窗格中的时间数据不会发生变化（因为没有新数据可用）。此外，当您仅选择没有可用数据的那部分记录时，对于时间信息，轨迹窗格将显示 **NaN**。

### 使用“Events”表格检查轨迹

“Events”表格列出了当前所选线程中的所有调用。您可以点击列标题对它们进行排序。通过选择表格中的某一行，您可以在时间轴上导航到所选调用的开始时间和结束时间。这样，您就可以在时间轴上准确定位事件。

![img](https://developer.android.com/studio/images/profile/system-trace-events-table.png?hl=zh-cn)

**图 10.** 查看“Analysis”窗格中的“Events”标签页。

### 检查系统轨迹

检查系统跟踪数据时，您可以在 **Threads** 时间轴中检查 **Trace Events**，以查看每个线程上所发生事件的详细信息。将鼠标指针悬停在某个事件上，可查看该事件的名称以及在每种状态下所花费的时间。点击事件可在 **Analysis** 窗格中查看详情。

### 检查系统轨迹：CPU 核心

除了 CPU 调度数据外，系统轨迹还包括按核心记录的 CPU 频率。它可以显示每个核心上的活动数量，让您可以了解哪些是新型移动处理器中的[“大”核心或“小”核心](https://en.wikipedia.org/wiki/ARM_big.LITTLE)。

![img](https://developer.android.com/studio/images/profile/system-trace-cpu-cores.png?hl=zh-cn)

**图 11.** 查看渲染线程的 CPU 活动和轨迹事件。

**CPU Cores** 窗格（如图 11 所示）显示每个核心上安排的线程活动。将鼠标指针悬停在某个线程活动上，可查看该核心在该特定时间在哪个线程上运行。

如需详细了解如何检查系统轨迹信息，请参阅 `systrace` 文档的[调查界面性能问题](https://developer.android.com/topic/performance/tracing/navigate-report?hl=zh-cn#analysis)部分。

### 检查系统轨迹：帧渲染时间轴

您可以检查应用在主线程和 `RenderThread` 上渲染每个帧所用的时间，以调查造成界面卡顿和帧速率低的瓶颈。

如需查看帧渲染数据，请使用使您可以**跟踪系统调用**的配置来[记录轨迹](https://developer.android.com/studio/profile/cpu-profiler?hl=zh-cn#method_traces)。记录轨迹后，在 **Display** 部分的 **Frames** 时间轴下查找有关每个帧的信息，如图 12 所示。

![img](https://developer.android.com/studio/images/profile/system-trace-render-thread.png?hl=zh-cn)

**图 12.** 每个用时超过 16 毫秒的帧都以红色显示。



![img](https://developer.android.com/studio/images/profile/system-trace-buffer-queue.png?hl=zh-cn)

**图 13.** “Display”部分的详细视图。

**Display** 部分中显示的跟踪记录如下：

- **Frames**：在绘制帧时。长帧（大于 16 毫秒）显示为红色。
- **SurfaceFlinger**：在 [SurfaceFlinger](https://source.android.com/devices/graphics/surfaceflinger-windowmanager?hl=zh-cn#surfaceflinger) 处理帧缓冲区时。SurfaceFlinger 是负责发送要显示的缓冲区的系统进程。
- **VSYNC**：同步显示流水线的[信号](https://source.android.com/devices/graphics/implement-vsync?hl=zh-cn)。错过 VSYNC 的帧将产生额外的输入延迟。这在高刷新率显示器上尤为重要。
- **BufferQueue**：有多少帧缓冲区排队等待 SurfaceFlinger 使用。对于部署到搭载 Android 9 或更高版本的设备的应用，这一跟踪记录会显示应用 Surface [BufferQueue](https://source.android.com/devices/graphics?hl=zh-cn#bufferqueue) 的缓冲区计数（0、1 或 2）。它可帮助您了解图像缓冲区在 Android 图形组件之间切换时的状态。例如，值 2 表示该应用当前处于三重缓冲状态，这可能会导致额外的输入延迟。

### 检查系统轨迹：进程内存 (RSS)

对于部署到搭载 Android 9 或更高版本的设备的应用，**Process Memory (RSS)** 部分会显示该应用当前使用的物理内存量。

![img](https://developer.android.com/studio/images/profile/system-trace-process-memory.png?hl=zh-cn)

**图 14.** 在性能分析器中查看物理内存。

**Total**

这是您的进程当前使用的物理内存总量。在基于 Unix 的系统上，这被称为“驻留集大小”，是匿名分配、文件映射和共享内存分配所使用的所有内存的组合。

对于 Windows 开发者，驻留集大小类似于工作集大小。

**Allocated**

此计数器跟踪进程的正常内存分配目前占用了多少物理内存。这些分配均匿名（不由特定文件支持）且不公开（不共享）。在大多数应用中，这由堆分配量（使用 `malloc` 或 `new`）和堆栈内存组成。从物理内存中换出时，这些分配会写入系统交换文件。

**File Mappings**

此计数器会跟踪进程用于文件映射的物理内存量，也就是说，通过内存管理器从文件映射至内存区域的内存。

**Shared**

此计数器跟踪在此进程和系统中其他进程之间共享的内存所用的物理内存量。



### 网络性能分析器

网络性能剖析器会在时间轴上显示实时网络活动，包括发送和接收的数据以及当前的连接数。这便于您检查应用传输数据的方式和时间，并适当优化底层代码。

如需打开网络性能剖析器，请按以下步骤操作：

1. 依次点击 **View > Tool Windows > Profiler**（您也可以点击工具栏中的 **Profile** 图标 ![img](https://developer.android.com/studio/images/buttons/toolbar-android-profiler.png?hl=zh-cn)）。
2. 从 Android Profiler 工具栏中选择要分析的设备和应用进程。如果您已通过 USB 连接设备但系统未列出该设备，请确保您已[启用 USB 调试](https://developer.android.com/studio/debug/dev-options?hl=zh-cn#enable)。
3. 点击 **NETWORK** 时间轴上的任意位置以打开网络性能剖析器。

#### 为什么应分析应用的网络Activity

当您的应用向网络发出请求时，设备必须使用高功耗的移动或 WLAN 无线装置来收发数据包。无线装置不仅要消耗电力来传输数据，而且还要消耗额外的电力来开启并且不锁定屏幕。

使用网络性能剖析器，您可以查找频繁出现的短时网络活动峰值，这些峰值意味着，您的应用要求经常开启无线装置，或要求无线装置长时间不锁定屏幕以处理集中出现的大量短时请求。这种模式说明您可以通过批量处理网络请求，减少必须开启无线装置来发送或接收数据的次数，从而优化应用，改善电池性能。这种方式还能让无线装置切换到低功耗模式，延长批量处理请求之间的间隔时间，节省电量。

如需详细了解优化应用网络 Activity 的相关技巧，请阅读[减少网络耗电量](https://developer.android.com/topic/performance/power/network?hl=zh-cn)。

#### 网络性能分析器概览

窗口顶部显示事件时间轴。在时间轴 (1) 上，您可以点击并拖动以选择时间轴的一部分来检查网络流量。

![img](https://developer.android.com/studio/images/profile/networkprofiler_2x.png?hl=zh-cn)

**图 1.** 网络性能剖析器窗口



在时间轴下方的窗格 (2) 中，您可以选择以下某个标签页，以详细了解时间轴上选定时段内的网络活动：

- **Connection View**：列出了在时间轴上选定时段内从您应用的所有 CPU 线程发送或接收的文件。对于每个请求，您可以检查大小、类型、状态和传输时长。 您可以通过点击任意列标题来对此列表排序。您还会看到时间轴上选定时段的明细数据，从而了解每个文件的发送或接收时间。

- Thread View

  ：显示您应用的每个 CPU 线程的网络活动。 如图 2 所示，您可以在此视图中检查各网络请求由应用的哪些线程负责。

   

  

  **图 2.** 在 **Thread View** 中检查应用线程的网络请求

从 **Connection View** 或 **Thread View** 中点击请求名称，可检查有关已发送或已接收数据的详细信息 (3)。点击各个标签页可查看响应标头和正文、请求标头和正文或调用堆栈。

在 **Response** 和 **Request** 标签页中，点击 **View Parsed** 链接可显示格式化文本，点击 **View Source** 链接可显示原始文本。

![img](https://developer.android.com/studio/images/profile/network-profiler-text_2X.png?hl=zh-cn)

**图 3.** 通过点击相应链接在原始文本（左侧）和格式化文本（右侧）之间切换



**注意**：如果您使用的是 [`HttpURLConnection`](https://developer.android.com/reference/java/net/HttpURLConnection?hl=zh-cn) API，则不会在 **Request** 标签页中看到标头，除非您使用 [`setRequestProperty`](https://developer.android.com/reference/java/net/URLConnection?hl=zh-cn#setRequestProperty(java.lang.String, java.lang.String)) 方法将其添加到您的代码中，如以下示例所示。

```
URL url = new URL(MY_URL_EN); HttpURLConnection urlConnection = (HttpURLConnection) url.openConnection(); ... // Sets acceptable encodings in the request header. urlConnection.setRequestProperty("Accept-Encoding", "identity");
```

#### 排查网络连接问题

如果网络性能剖析器检测到流量值，但无法识别任何受支持的网络请求，您会收到以下错误消息：

```
**Network Profiling Data Unavailable:** There is no information for the network traffic you've selected. 
```

目前，网络性能剖析器仅支持 [`HttpURLConnection`](https://developer.android.com/reference/java/net/HttpURLConnection?hl=zh-cn) 和 [`OkHttp`](http://square.github.io/okhttp/) 网络连接库。如果您的应用使用的是其他网络连接库，您可能无法在网络性能剖析器中查看网络活动。如果您收到了这条错误消息，但您的应用使用的确实是 `HttpURLConnection` 或 `OkHttp`，请[报告错误](https://developer.android.com/studio/report-bugs?hl=zh-cn)或[搜索问题跟踪器](https://issuetracker.google.com/issues?q=componentid%3A317727%2B&hl=zh-cn)，在与您的问题有关的现有报告中加入您的反馈。您还可以使用这些资源来请求对其他库的支持。

### 能耗性能分析器

能耗性能剖析器可帮助您了解应用在哪里耗用了不必要的电量。

能耗性能剖析器会监控 CPU、网络无线装置和 GPS 传感器的使用情况，并直观地显示其中每个组件消耗的电量。能耗性能剖析器还会显示可能会影响耗电量的系统事件（唤醒锁定、闹钟、作业和位置信息请求）的发生次数。

能耗性能剖析器并不会直接测量耗电量，而是使用一种模型来估算设备上每项资源的耗电量。

#### 能耗性能剖析器概览

当您在搭载 Android 8.0 (API 26) 或更高版本的关联设备或 Android 模拟器中运行您的应用时，能耗性能剖析器便会显示为 **Profiler** 窗口中的一行。

如需打开能耗性能剖析器，请按以下步骤操作：

1. 依次选择 **View > Tool Windows > Profiler** 或点击工具栏中的 **Profile** 图标 ![img](https://developer.android.com/studio/images/buttons/toolbar-android-profiler.png?hl=zh-cn)。

   如果 **Select Deployment Target** 对话框显示提示，请选择需将您的应用部署到哪个设备上以进行性能剖析。如果您已通过 USB 连接设备但系统未列出该设备，请确保您已[启用 USB 调试](https://developer.android.com/studio/debug/dev-options?hl=zh-cn#enable)。

2. 点击 **Energy** 时间轴中的任意位置以打开能耗性能剖析器。

当您打开能耗性能剖析器时，它会立即开始显示应用的估算耗电量。系统会显示类似于图 1 的界面。

![img](https://developer.android.com/studio/images/profile/energy-profiler-L1_2x.png?hl=zh-cn)

**图 1.** 能耗性能剖析器中的时间轴。



如图 1 所示，能耗性能剖析器的默认视图包括以下时间轴：

1. **“Event”时间轴**：显示应用中的 Activity 在其生命周期内不断转换而经历各种不同状态的过程。此时间轴还会指示用户与设备的交互，包括屏幕旋转事件。
2. **“Energy”时间轴**：显示应用的估算耗电量。
3. **“System”时间轴**：显示可能会影响耗电量的系统事件。

如需查看 CPU、网络和位置信息 (GPS) 资源，以及相关系统事件的具体耗电量情况，请将鼠标指针放在 **Energy** 时间轴中的条形上方。

检查系统事件：唤醒锁定、作业和闹钟

您可以使用能耗性能剖析器查找可能会影响耗电量的系统事件，包括唤醒锁定、作业和闹钟：

- [唤醒锁定](https://developer.android.com/training/scheduling/wakelock?hl=zh-cn)是一种机制，可在设备进入休眠模式时使 CPU 或屏幕保持开启状态。例如，播放视频的应用可以使用唤醒锁定，以便在用户未与设备交互时使屏幕保持开启状态。请求唤醒锁定不是一项耗电量很高的操作，但未撤消唤醒锁定会导致屏幕或 CPU 保持开启状态的时间超过必要时间，从而加快电池耗电速度。如需了解详情，请参阅有关[使用唤醒锁定](https://developer.android.com/topic/performance/vitals/wakelock?hl=zh-cn)的指南。
- 您可以使用[闹钟](https://developer.android.com/training/scheduling/alarms?hl=zh-cn)定期在应用上下文之外运行后台任务。当闹钟触发时，它可能会唤醒设备并运行耗电量很高的代码。如需了解详情，请参阅有关[使用闹钟](https://developer.android.com/topic/performance/vitals/wakeup?hl=zh-cn)的指南。
- 您可以使用[作业](https://developer.android.com/reference/android/app/job/JobScheduler?hl=zh-cn)在指定条件下（例如恢复网络连接时）执行相关操作。您可以使用 `JobBuilder` 创建作业，并使用 `JobScheduler` 对这些作业进行调度。在许多情况下，建议您使用 `JobScheduler` 对作业进行调度，而不是使用闹钟或唤醒锁定。
- 位置信息请求使用 GPS 传感器，这会消耗大量电量。如需了解如何在发出位置信息请求时节省电量，请参阅[优化电池的位置](https://developer.android.com/guide/topics/location/battery?hl=zh-cn)。

借助能耗性能剖析器，您可以轻松找到应用使用各项功能的位置，以便您就如何使用各项功能做出明智的决策。

能耗性能剖析器会在 **Energy** 时间轴下的 **System** 时间轴中显示一个彩色编码的条形，以表示系统事件处于活动状态的时间范围。唤醒锁定用红色条形表示，作业和闹钟用黄色条形表示，位置信息事件用浅紫色条形表示。

图 2 显示了能耗性能剖析器，并在代码编辑器中定位到了未释放唤醒锁定对应的源代码。

![img](https://developer.android.com/studio/images/profile/energy-profiler-L2_2x.png?hl=zh-cn)

**图 2.** 使用能耗性能剖析器查找唤醒锁定。



1. 如需打开 **System Event** 窗格并显示唤醒锁定等事件的详细信息，请在 **Energy** 时间轴中选择一个时间范围。
2. 如需打开 **Wake Lock Details** 窗格并显示特定唤醒锁定的详细信息，请在 **System Event** 窗格中选择该唤醒锁定。
3. 如需打开代码编辑器并跳转到唤醒锁定的源代码，请在 **Wake Lock Details** 窗格中双击调用堆栈顶部的调用方法条目。
4. 用于获取唤醒锁定的调用会在源代码编辑器中突出显示。

有关显示其他系统事件详情的说明与唤醒锁定基本相同，只不过对应的详细信息窗格中提供了特定于各种事件的信息。例如，**Job Details** 窗格会显示调度作业以及完成作业的代码部分的调用堆栈。

