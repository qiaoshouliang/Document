<!-- toc -->

# Android Studio Debug的方法

## 进入 debug 模式的两种姿势

![](http://oxshjm7y0.bkt.clouddn.com/20171014193240_V27p58_Screenshot.jpeg)

第一种是“绿色甲虫”(debug app),一开始就进入了调试模式。它的弊端是每次需要从头跑一遍，且由于调试模式下应用程序略卡顿，等你到达调试页面时会觉得老费劲。

第二种方式是`Attach debugger to Android process`，选择应用程序主进程，即可进入调试模式。这种方式的特点是，随时随地自由进入调试模式，不需要重头开始跑应用程序，该方式适合绝大多数调试场景。

## 常见的调试操作

调试界面:

![](http://oxshjm7y0.bkt.clouddn.com/20171014194928_Q659Q2_Screenshot.jpeg)

在 Android Studio 的 debug 标签（假如一开始没有，等触发断点后自然会出现）中有两个面板 debugger 和 console。debugger 又分为 Frames、Threads 和 Variables 三块，分别是堆栈内容、线程、变量区。

在 debugger 标签右边有一些操作按钮，是我们常用的调试操作，下面会一一介绍。（可以用鼠标悬停在上面看每个按钮的具体名称）

### 设置断点

断点有多种类型，我们这里先只谈普通断点。在每行的最前端单击一下即可**添加断点**，在断点上单击一下是**取消断点**。普通模式下断点只是一个普通的红点，但假如是在调试模式下，则红点上会有一个“√”或“✘”表示该行是否会被运行，例如，注释行前的断点会是“✘”。

![](http://oxshjm7y0.bkt.clouddn.com/20171014195729_51zSq1_Screenshot.jpeg)

- 跳到下一个断点（F9）
- 单步调试（F8）
- 进入方法内部（F7）
- 退出当前方法（上档键+F8）



## 几种高效断点

### 条件成立时才触发的条件断点

普通断点在每次运行到时都会被触发，这在多次调用、有“循环”的场景会比较麻烦，比如循环 100 次只希望停留在第 98 次。那么此刻就可以用上条件断点了。

添加条件断点：先在需要的行前左键单击添加普通断点，右键点击该断点出现对话框，在“Condition”处填入条件即可，条件语法同 Java，如 `i == 98`。点击 Done，完成添加。这样当条件未满足时，不会阻塞程序运行；当条件满足时断点被触发。

![img](https://mmbiz.qpic.cn/mmbiz_png/z0MSSiceGv5JA7NVPsrqgrSmteKUWqibbld7AdfEmNl9IFVicNctwE9HT3ejEHNUYVhN5wZ7pptbh42N9Ma2wZyjw/640?wx_fmt=png&wxfrom=5&wx_lazy=1)

### 不会阻塞应用程序的日志断点

![](http://oxshjm7y0.bkt.clouddn.com/20171014212151_iq70Eg_Screenshot.jpeg)

### 被异常触发的异常断点



### 追踪关键点的字段断点和方法断点

![](http://oxshjm7y0.bkt.clouddn.com/20171014213408_9QrgeU_Screenshot.jpeg)

### 组合使用

比如日志断点是“不阻塞”和“输出日志”两个操作的集合，那么我们当然可以加上“设置条件”操作变成“条件日志断点”。

## 调试中的变量

在设置了合适的断点后，我们就可以进行下一步操作 —— 观察变量，准确的说是观察变量的值。

### 变量观测面板

![](http://oxshjm7y0.bkt.clouddn.com/20171014215155_kTGIhw_Screenshot.jpeg)

如上图所示，变量观察面板会列出所有当前能访问到的成员变量和局部变量。

点击变量前的箭头，可以将该实例展开，列出所有字段。

### Add New Watch

![](http://oxshjm7y0.bkt.clouddn.com/20171014215516_vGqT8Z_Screenshot.jpeg)

### 设置变量的值

![](http://oxshjm7y0.bkt.clouddn.com/20171014215609_1BuZsx_Screenshot.jpeg)



### Log

在多线程环境下，光靠 debug 是不行的。有时 debug 本身会带来一些问题混淆了现场，比如因为 debug 时的卡顿造成环境不一致等等，这时应该学会使用打日志的形式帮忙调试。

# Android Studio Lint

## 使用方法

![](http://oxshjm7y0.bkt.clouddn.com/20171014220510_Re707b_Screenshot.jpeg)



检查完毕后会弹出 Inspection 的控制台，并在其中列出详细的检查结果：



![](http://oxshjm7y0.bkt.clouddn.com/20171014221128_eeY3Uk_Screenshot.jpeg)

如上图所展示的，Android Lint 对检查的结果进行了分类，同一个规则（issue）下的问题会聚合，其中针对 Android 的规则类别会在分类前说明是 Android 相关的，主要是六类：

- **Accessibility 无障碍**，例如 `ImageView` 缺少 `contentDescription` 描述，String 编码字符串等问题。

  ![](http://oxshjm7y0.bkt.clouddn.com/20171014223300_rMIrwt_Screenshot.jpeg)

- **Correctness 正确性**，例如 xml 中使用了不正确的属性值，Java 代码中直接使用了超过最低 SDK 要求的 API 等。

  ![](http://oxshjm7y0.bkt.clouddn.com/20171014221359_fZfwYl_Screenshot.jpeg)

- **Internationalization 国际化**，如字符缺少翻译等问题。

  ![](http://oxshjm7y0.bkt.clouddn.com/20171014221255_yiQ6CR_Screenshot.jpeg)

- **Performance 性能**，例如在 `onMeasure`、`onDraw` 中执行 new，内存泄露，产生了冗余的资源，xml 结构冗余等。

  ![](http://oxshjm7y0.bkt.clouddn.com/20171014221212_k3sOx5_Screenshot.jpeg)

- **Security 安全性**，例如没有使用 HTTPS 连接 Gradle，AndroidManifest 中的权限问题等。

- **Usability 易用性**，例如缺少某些倍数的切图，重复图标等。



## 配置

```java
android {
    lintOptions {
        // 设置为 true，则当 Lint 发现错误时停止 Gradle 构建
        abortOnError false
        // 设置为 true，则当有错误时会显示文件的全路径或绝对路径 (默认情况下为true)
        absolutePaths true
        // 仅检查指定的问题（根据 id 指定）
        check 'NewApi', 'InlinedApi'
        // 设置为 true 则检查所有的问题，包括默认不检查问题
        checkAllWarnings true
        // 设置为 true 后，release 构建都会以 Fatal 的设置来运行 Lint。
        // 如果构建时发现了致命（Fatal）的问题，会中止构建（具体由 abortOnError 控制）
        checkReleaseBuilds true
        // 不检查指定的问题（根据问题 id 指定）
        disable 'TypographyFractions','TypographyQuotes'
        // 检查指定的问题（根据 id 指定）
        enable 'RtlHardcoded','RtlCompat', 'RtlEnabled'
        // 在报告中是否返回对应的 Lint 说明
        explainIssues true
        // 写入报告的路径，默认为构建目录下的 lint-results.html
        htmlOutput file("lint-report.html")
        // 设置为 true 则会生成一个 HTML 格式的报告
        htmlReport true
        // 设置为 true 则只报告错误
        ignoreWarnings true
        // 重新指定 Lint 规则配置文件
        lintConfig file("default-lint.xml")
        // 设置为 true 则错误报告中不包括源代码的行号
        noLines true
        // 设置为 true 时 Lint 将不报告分析的进度
        quiet true
        // 覆盖 Lint 规则的严重程度，例如：
        severityOverrides ["MissingTranslation": LintOptions.SEVERITY_WARNING]
        // 设置为 true 则显示一个问题所在的所有地方，而不会截短列表
        showAll true
        // 配置写入输出结果的位置，格式可以是文件或 stdout
        textOutput 'stdout'
        // 设置为 true，则生成纯文本报告（默认为 false）
        textReport false
        // 设置为 true，则会把所有警告视为错误处理
        warningsAsErrors true
        // 写入检查报告的文件（不指定默认为 lint-results.xml）
        xmlOutput file("lint-report.xml")
        // 设置为 true 则会生成一个 XML 报告
        xmlReport false
        // 将指定问题（根据 id 指定）的严重级别（severity）设置为 Fatal
        fatal 'NewApi', 'InlineApi'
        // 将指定问题（根据 id 指定）的严重级别（severity）设置为 Error
        error 'Wakelock', 'TextViewEdits'
        // 将指定问题（根据 id 指定）的严重级别（severity）设置为 Warning
        warning 'ResourceAsColor'
        // 将指定问题（根据 id 指定）的严重级别（severity）设置为 ignore
        ignore 'TypographyQuotes'
    }
}
```

## 常见问题

我们在使用 Android Lint 对项目进行检查后，整理了一些问题及解决方法，下面列举较为常见的场景：

### ScrollView size validation

这也是上文提到过的一个情况，在 ScrollView 的第一层子元素中设置了高度为 `match_parent`，这是错误的写法，实际上在 measure 时这里必定会被当作 `wrap_content` 去处理，因此按照 Lint 的建议，直接改为 `wrap_content` 即可。

### Handler reference leaks

Handler 引用的内存泄露问题，例如下面的例子：

```java
protected static final int STOP = 0x10000;
protected static final int NEXT = 0x10001;

@BindView(R.id.rectProgressBar) QMUIProgressBar mRectProgressBar;
@BindView(R.id.circleProgressBar) QMUIProgressBar mCircleProgressBar;

int count;

private Handler myHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what) {
            case STOP:
                break;
            case NEXT:
                if (!Thread.currentThread().isInterrupted()) {
                    mRectProgressBar.setProgress(count);
                    mCircleProgressBar.setProgress(count);
                }
        }
    }
};
```

首先非静态的内部类或者匿名类会隐式的持有其外部类的引用，内部类使用了外部类的方法/成员变量也会导致其持有外部类引用，因此上面这种情况会导致 handler 持有了外部类，外部类同时持有 handler，handler 是异步的，当 handler 的消息发送出去后，外部类因 hanlder 的持有而无法销毁，最终导致内存泄露。

解决办法则是把该内部类改为 static，内部类中使用的外部类方法/成员变量改为弱引用，具体如下：

```java
@BindView(R.id.rectProgressBar) QMUIProgressBar mRectProgressBar;
@BindView(R.id.circleProgressBar) QMUIProgressBar mCircleProgressBar;

int count;

private ProgressHandler myHandler = new ProgressHandler();

@Override
protected View onCreateView() {
    myHandler.setProgressBar(mRectProgressBar, mCircleProgressBar);
}

private static class ProgressHandler extends Handler {
    private WeakReference<QMUIProgressBar> weakRectProgressBar;
    private WeakReference<QMUIProgressBar> weakCircleProgressBar;

    public void setProgressBar(QMUIProgressBar rectProgressBar, QMUIProgressBar circleProgressBar) {
        weakRectProgressBar = new WeakReference<>(rectProgressBar);
        weakCircleProgressBar = new WeakReference<>(circleProgressBar);
    }

    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what) {
            case STOP:
                break;
            case NEXT:
                if (!Thread.currentThread().isInterrupted()) {
                    if (weakRectProgressBar.get() != null && weakCircleProgressBar.get() != null) {
                        weakRectProgressBar.get().setProgress(msg.arg1);
                        weakCircleProgressBar.get().setProgress(msg.arg1);
                    }
                }
        }

    }
}
```

### Memory allocations within drawing code

onMeasure、onDraw 都是被频繁调用的方法，因此 Lint 不建议在其中执行 new 操作，可以在 onCreateView 等非频繁调用的时机进行 new 操作，并用成员变量保存，再在 onMeasure 中使用成员变量。

### ‘private’ method declared ‘final’

```java
private static final void addLinkMovementMethod(TextView t) {
    MovementMethod m = t.getMovementMethod();

    // ...
}
```

如上面的示例代码，会产生 ‘private’ method declared ‘final’ 的警告，因为私有方法是不会被 override 的，因此完全没有必要声明 `final`。

# Android Clean Code

![](http://oxshjm7y0.bkt.clouddn.com/20171016132945_UizeWZ_Screenshot.jpeg)

# Android Gradle 一些基本用法

## 版本号配置

project build.gradle

![](http://oxshjm7y0.bkt.clouddn.com/20171016135825_pDvr4c_Screenshot.jpeg)

app/module  build.gradle

![](http://oxshjm7y0.bkt.clouddn.com/20171016140034_btDPkp_Screenshot.jpeg)

![](http://oxshjm7y0.bkt.clouddn.com/20171016140054_Slo9rg_Screenshot.jpeg)



#  Android Studio 的一些小技巧  

## 代码提示不区别大小写

使用方法：Editor标签下-Code Completion下-Case sensitive completion选择None，当输入代码，不区别大小写也能弹出代码提示
[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu14.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu14.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

## 变量命名提示

变量命名都是用小写m开头，Android Studio提示却是这样的：
[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu7.png?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu7.png?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
如何输入m，提示mRetrofit呢？
使用方法：Code Style-Java-Code Generation-Fields
[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu8.png?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu8.png?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
输入m，达到预期效果：
[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu6.png?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu6.png?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

## 去掉不用的import control + option + O

![](http://oxshjm7y0.bkt.clouddn.com/20171016145354_lvjEG7_Screenshot.jpeg)

## 方法注释添加 

![](http://oxshjm7y0.bkt.clouddn.com/20171016145831_yVP8nq_Screenshot.jpeg)

## 查找使用 

使用方法：将鼠标放在需要查找的类或变量，选择Find Usages
[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu9.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu9.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

## 快速进入类 Comand + O

[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu13.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu13.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)
当不勾上Include non-project class搜的是当前Module，勾上是全局的。

## 上下移动行 shift + option + up/down

[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu1.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu1.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

## 上下复制行 Command + D

[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu2.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu2.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

## 多行编辑 ALT+ 鼠标左键

[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu3.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu3.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

## 折叠 Command + -/+

![](http://oxshjm7y0.bkt.clouddn.com/20171016161750_KIJZYz_Screenshot.jpeg)

## 删除一行 Command + Del

[![img](http://7q5c2h.com1.z0.glb.clouddn.com/asu4.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)](http://7q5c2h.com1.z0.glb.clouddn.com/asu4.gif?watermark/2/text/5ZC05bCP6b6Z5ZCM5a24/font/5qW35L2T/fontsize/500/fill/I0VGRUZFRg==/dissolve/100/gravity/SouthEast/dx/10/dy/10)

## Android Live Template

![](http://oxshjm7y0.bkt.clouddn.com/20171014224515_eHN5DZ_Screenshot.jpeg)

# 三方库

## Crash report - bugly

https://bugly.qq.com/v2/crash-reporting/crashes/eef48e1f98/2?pid=1


