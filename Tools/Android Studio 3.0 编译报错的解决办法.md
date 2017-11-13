Android Studio 3.0 编译报错的解决办法

```java
Execution failed for task ':app:transformClassesWithProfilers-transformForDebug'. > 7
```
网上还查到如下这两个问题也是同样的解决办法

```
Error:Execution failed for task ':lryapp:transformClassesWithProfilers-transformForDebug'. > 4

java.lang.ArrayIndexOutOfBoundsException：4
```

解决办法如下：

在Run/Debug Configurations中去掉`Enable advanced profiling`这个选型

![](http://oxshjm7y0.bkt.clouddn.com/20171113105905_SMIpXF_Screenshot.jpeg)

这个功能是AS 3.0中的**开启高级分析器功能**

以下是关于这个功能的介绍，但是为什么会导致`:app:transformClassesWithProfilers-transformForDebug'. > 7`这个问题，还需要再研究一下。

默认情况下并不是所有的分析数据都可见。如果您看到一条消息，说“Advanced profiling is unavailable for the selected process”，则需要在运行配置中启用高级分析。

为了显示高级分析数据，Android Studio必须将监控逻辑注入到已编译的应用程序中。高级分析提供的功能包括：

- 所有分析器窗口上的事件时间轴
- 内存分析器中已分配对象的数量
- 内存分析器中的垃圾收集事件
- 有关Network Profiler中所有传输文件的详细信息

要启用高级概要分析，请按照下列步骤操作：

- 选择 `Run > Edit Configurations`
- 在左窗格中选择您的应用程序模块。
- 单击`Profiling`选项卡，然后选中`Enable advanced profiling.`。

现在再次构建并运行应用程序就可以访问完整的概要分析功能集。但是，请注意，高级分析会降低您的构建速度，因此只有在您要开始对应用程序进行概要分析时，才应启用它。