# 解决方案

在 Android 开发中,应用启动速度是一个非常重要的点，应用启动优化也是一个非常重要的过程。对于应用启动优化，其实核心思想就是在启动过程中少做事情，具体实践的时候无非就是下面几种：

- 异步加载
- 延时加载
- 懒加载

不用一一去解释,做过启动优化的估计都使用过，本篇文章将详细讲解一下一种延时加载的实现以及其原理.
其实这种加载的实现是非常简单的,但是其中的原理可能比较复杂，还涉及到`Looper/Handler/MessageQueue/VSYNC`等。以及其中碰到的一些问题，还会有一些我自己额外的思考。

## 优化后的DelayLoad的实现

一提到`DelayLoad`,大家可能第一时间想到的就是在 `onCreate `里面调用` Handler.postDelayed`方法, 将需要 Delay 加载的东西放到这里面去初始化, 这个也是一个比较方便的方法. Delay一段时间再去执行,这时候应用已经加载完成,界面已经显示出来了, 不过这个方法有一个致命的问题: 延迟多久?
大家都知道,在 Android 的高端机型上,应用的启动是非常快的 , 这时候只需要 Delay 很短的时间就可以了, 但是在低端机型上,应用的启动就没有那么快了,而且现在应用为了兼容旧的机型,往往需要 Delay 较长的时间,这样带来体验上的差异是很明显的.

这里先说优化方案:

1. 首先 , 创建 Handler 和 Runnable 对象, 其中 Runnable 对象的 run方法里面去更新 UI 线程.

   ```java
   private Handler myHandler = new Handler();
   private Runnable mLoadingRunnable = new Runnable() {

     @Override
     public void run() {
       updateText(); //更新UI线程
     }
   };
   ```

2. 在主 Activity 的 onCreate 中加入下面的代码

   ```
   getWindow().getDecorView().post(new Runnable() {

     @Override
     public void run() {
       myHandler.post(mLoadingRunnable);
     }
   });
   ```

其实实现的话非常简单,我们来对比一下三种方案的效果.

## 2. 三种写法的差异对比

为了验证我们优化的 DelayLoad的效果,我们写了一个简单的app , 这个 App 中包含三张不同大小的图片,每张图片下面都会有一个 TextView , 来标记图片的显示高度和宽度. MainActivity的代码如下:

```java
public class MainActivity extends AppCompatActivity {
  private static final int DEALY_TIME = 300 ;

  private ImageView imageView1;
  private ImageView imageView2;
  private ImageView imageView3;
  private TextView textView1;
  private TextView textView2;
  private TextView textView3;

  private Handler myHandler = new Handler();
  private Runnable mLoadingRunnable = new Runnable() {

    @Override
    public void run() {
      updateText();
    }
  };

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);

    imageView1 = (ImageView) findViewById(R.id.image1);
    imageView2 = (ImageView) findViewById(R.id.image2);
    imageView3 = (ImageView) findViewById(R.id.image3);

    textView1 = (TextView) findViewById(R.id.text1);
    textView2 = (TextView) findViewById(R.id.text2);
    textView3 = (TextView) findViewById(R.id.text3);

//  第一种写法:直接Post
    myHandler.post(mLoadingRunnable);

//  第二种写法:直接PostDelay 300ms.
//  myHandler.postDelayed(mLoadingRunnable, DEALY_TIME);

//  第三种写法:优化的DelayLoad
//  getWindow().getDecorView().post(new Runnable() {
//    @Override
//    public void run() {
//      myHandler.post(mLoadingRunnable);
//    }
//  });

    // Dump当前的MessageQueue信息.
    getMainLooper().dump(new Printer() {

      @Override
      public void println(String x) {
        Log.i("Gracker",x);
      }
    },"onCreate");
}

  private void updateText() {
    TraceCompat.beginSection("updateText");
    textView1.setText("image1 : w=" + imageView1.getWidth() +
      " h =" + imageView1.getHeight());
    textView2.setText("image2 : w=" + imageView2.getWidth() +
      " h =" + imageView2.getHeight());
    textView3.setText("image3 : w=" + imageView3.getWidth() +
      " h =" + imageView3.getHeight());
    TraceCompat.endSection();
}
```

我们需要关注两个点:

- updateText 这个函数是什么时候被执行的?
- App 启动后,三个图片的长宽是否可以被正确地显示出来?
- 是否有 Delay Load 的效果?

### 2.1 第一种写法

1. updateText执行的时机?
   下面是第一种写法的Trace图:
   [![第一种写法](http://androidperformance.com/images/app-lunch/1.png)](http://androidperformance.com/images/app-lunch/1.png)第一种写法
   可以看到 updateText 是在 Activity 的 w/onStart/onResume三个回调执行完成后才去执行的.

2. 图片的宽高是否正确显示?
   [![第一种写法](http://androidperformance.com/images/app-lunch/2.png)](http://androidperformance.com/images/app-lunch/2.png)第一种写法

   从图片看一看到,宽高并没有显示. 这是为什么呢? 这个问题就要从Activity 的 `onCreate/onStart/onResume`三个回调说起了. 其实Activity 的 `onCreate/onStart/onResume`三个回调中,并没有执行Measure和Layout操作, 这个是在后面的performTraversals中才执行的. 所以在这之前宽高都是0.

3. 是否有 Delay Load 的效果?
   并没有. 因为我们知道, 应用启动的时候,要等两次 `performTraversals` 都执行完成之后才会显示第一帧, 而 `updateText `这个方法在第一个 `performTraversals` 执行之前就执行了. 所以` updateText `方法的执行时间是算在应用启动的时间里面的.

### 2.2 第二种写法

第二种写法我们Delay了300ms .我们来看一下表现.

1. updateText执行的时机?
   [![第二种写法](http://androidperformance.com/images/app-lunch/3.png)](http://androidperformance.com/images/app-lunch/3.png)第二种写法

   可以看到,这种写法的话,updateText是在两个performTraversals 执行完成之后(这时候 APP 的第一帧才显示出来)才去执行的, 执行完成之后又调用了一次 performTraversals 将 TextView 的内容进行更新.

2. 图片的宽高是否正确显示?
   [![第二种写法](http://androidperformance.com/images/app-lunch/4.png)](http://androidperformance.com/images/app-lunch/4.png)第二种写法

   从上图可以看到,图片的宽高是正确显示了出来. 原因上面已经说了,measure/layout执行完成后,宽高的数据就可以获取了.

3. 是否有 Delay Load 的效果?
   不一定,取决于 Delay的时长.
   从前面的 Trace 图上我们可以看到 , updateText 方法由于 Delay 了300ms, 所以在应用第一帧显示出来170ms之后, 图片的文字信息才进行了更新. 这个是有 Delay Load 的效果的.
   但是这里只是一个简单的TextView的更新, 如果是较大模块的加载 , 用户视觉上会有很明显的 “ 空白->内容填充” 这个过程, 或者会附加”闪一下”特效…这显然是我们不想看到的.

   有人会说:可以把Delay的时间减小一点嘛,这样就不会闪了. 话是这么说,但是由于 Android 机器的多元性(其实就是有很多高端机器,也有很多低端机器) , 在这个机子上300ms的延迟算是快,在另外一个机子上300ms算是很慢.

   我们将Delay时间调整为50ms, 其Trace图如下:

   [![第二种写法:Delay 50ms](http://androidperformance.com/images/app-lunch/5.png)](http://androidperformance.com/images/app-lunch/5.png)第二种写法:Delay 50ms

   可以看到,updateText 方法在第一个 performTraversals 之后就执行了,所以也没有 Delay Load 的效果(虽然宽高是正确显示了,因为在第一个 performTraversals 方法中就执行了layout和measure).

### 2.3 第三种写法

经过前两个方法 , 我们就会想, 如果能不使用Delay方法, updateText 方法能在 第二个performTraversals 方法执行完成后(即APP第一帧在屏幕上显示),马上就去执行,那么即起到了 Delay Load的作用,又可以正确显示图片的宽高.
第三种写法就是这个效果:

1. updateText执行的时机?

   [![第三种写法](http://androidperformance.com/images/app-lunch/6.png)](http://androidperformance.com/images/app-lunch/6.png)第三种写法

   可以看到这种写法. updateText 在第二个 performTraversals 方法执行完成后马上就执行了, 然后下一个 VSYNC 信号来了之后, TextView就更新了.

2. 图片的宽高是否正确显示?
   当然是正确显示的.如图:
   [![第三种写法](http://androidperformance.com/images/app-lunch/7.png)](http://androidperformance.com/images/app-lunch/7.png)第三种写法

3. 是否有 Delay Load 的效果?
   从 Trace 图上看, 是有 Delay Load的效果的, 而且可以在应用第一帧显示后马上进行数据 Load , 不用考虑 Delay时间的长短.

## 3. 一些思考

关于优化的 Delay Load 的实现,从代码层面来看其实是非常简单的.其带来的效果也是很赞的.
但是实现之后我们还需要思考一下,为何这么做就可以实现这种功能呢?很显然要回答这个问题,我们需要知道更底层的一些东西.这个还涉及到 `Handler/Message/MessageQueue/Looper/VSYNC/ViewRootImpl`等知识. 往大里说应该还涉及到AMS/WMS等.由于涉及到的东西比较多,我就不在这一篇里面阐述了, 下一篇文章将会从从原理上讲解一下为何优化的 Delay Load 会起作用.

# 原理

其中会涉及到一些 Android 中的比较重要的类，以及 Activity 生命周期中比较重要的几个函数。
其实这个其中的原理比较简单，不过要弄清楚其实现的过程，还是一件蛮好玩的事情，其中会用到一些工具，自己加调试代码等，一步一步下来，自己对 Activity 的启动的理解又深了一层，希望大家读完之后也会对大家有一定的帮助。

上一篇中我们最终使用的 DelayLoad 的核心方法是在 Activity 的 onCreate 函数中加入下面的方法 ：

```java
getWindow().getDecorView().post(new Runnable() {
    @Override
    public void run() {
        myHandler.post(mLoadingRunnable);
    }
});
```

我们一一来看涉及到的类和方法

## 1. Activity.getWindow 及 PhoneWindow 的初始化时机

Activity 的 getWindow 方法获取到的是一个 PhoneWindow 对象：

```Java
public Window getWindow() {
    return mWindow;
}
```

这个 mWindow 就是一个 PhoneWindow 对象，其初始化的时机为这个 Activity attach 的时候：

```java
  final void attach(Context context, ActivityThread aThread,
          Instrumentation instr, IBinder token, int ident,
          Application application, Intent intent, ActivityInfo info,
          CharSequence title, Activity parent, String id,
          NonConfigurationInstances lastNonConfigurationInstances,
          Configuration config, String referrer, IVoiceInteractor voiceInteractor) {
      attachBaseContext(context);

      mFragments.attachActivity(this, mContainer, null);

      mWindow = PolicyManager.makeNewWindow(this);
      mWindow.setCallback(this);
      mWindow.setOnWindowDismissedCallback(this);
      mWindow.getLayoutInflater().setPrivateFactory(this);
      ........

  // PolicyManager.makeNewWindow(this) 最终会调用 Policy 的 makeNewWindow 方法
  public Window makeNewWindow(Context context) {
      return new PhoneWindow(context);
  }

}
```

这里需要注意 Activity 的 attach 方法很早就会调用的，是要早于 Activity 的 onCreate 方法的。 

### 总结：

- PhoneWindow 与 Activity 是一对一的关系，通过上面的初始化过程你应该更加清楚这个概念
- Android 中对 PhoneWindow 的注释是 ：Android-specific Window ，可见其重要性
- PhoneWindow 中有很多大家比较熟悉的方法，比如 setContentView / addContentView 等 ； 也有几个重要的内部类，比如：DecorView ;

## 2. PhoneWindow.getDecorView 及 DecorView 的初始化时机

上面我们说到 DecorView是 PhoneWindow 的一个内部类，其定义如下：

```java
private final class DecorView extends FrameLayout implements RootViewSurfaceTaker
```

那么 DecorView 是什么时候初始化的呢？DecorView 是在 Activity 的父类的 onCreate 方法中被初始化的，比如我例子中的 MainActivity 是继承自 android.support.v7.app.AppCompatActivity ，当我们调用 MainActivity 的 super.onCreate(savedInstanceState); 的时候，就会调用下面的

```java
protected void onCreate(@Nullable Bundle savedInstanceState) {
    getDelegate().installViewFactory();
    getDelegate().onCreate(savedInstanceState);
    super.onCreate(savedInstanceState);
}
```

由于我们导入的是 support.v7 包里面的AppCompatActivity， getDelegate() 得到的就是AppCompatDelegateImplV7 ，其 onCreate 方法如下：

```java
public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    mWindowDecor = (ViewGroup) mWindow.getDecorView();
    ......
}
```

就是这里的 mWindow.getDecorView() ，对 DecorView 进行了实例化：

```java
public final View getDecorView() {
    if (mDecor == null) {
        installDecor();
    }
    return mDecor;
}
```

第一次调用 getDecorView 的时候，会进入 installDecor 方法，这个方法对 DecorView 进行了一系列的初始化 ，其中比较重要的几个方法有：generateDecor / generateLayout 等，generateLayout 会从当前的 Activity 的 Theme 提取相关的属性，设置给 Window，同时还会初始化一个 startingView，添加到 DecorView上，也就是我们所说的 startingWindow。

### 总结

- Decor 有装饰的意思，DecorView 官方注释为 “This is the top-level view of the window, containing the window decor” , 我们可以理解为 DecorView 是我们当前 Activity 的最下面的布局。所以我们打开 DDMS 查看 Tree Overview 的时候，可以发现最根部的那个 View 就是 DecorView：
  [![DelayLoad](http://androidperformance.com/images/applunch2/1.png)](http://androidperformance.com/images/applunch2/1.png)DelayLoad
- 应用从桌面启动的时候，在主 Activity 还没有显示的时候，如果主题没有设置窗口的背景，那么我们就会看到白色（这个和手机的Rom也有关系），如果应用启动很慢，那么用户得看好一会白色。如果要避免这个，则可以在 Application 或者 Activity 的 Theme 中设置 WindowBackground , 这样就可以避免白色（当然现在各种大厂都是SplashActivity+广告我也是可以理解的）

## 3. Post

当我们调用 DecorView 的 Post 的时候，其实最终会调用 View 的 Post ，因为 DecorView 最终是继承 View 的：

```java
public boolean post(Runnable action) {
    final AttachInfo attachInfo = mAttachInfo;
    if (attachInfo != null) {
        return attachInfo.mHandler.post(action);
    }
    // Assume that post will succeed later
    ViewRootImpl.getRunQueue().post(action);
    return true;
}
```

注意这里的 mAttachInfo ，我们调用 post 是在 Activity 的 onCreate 中调用的，那么此时 mAttachInfo 是否为空呢？答案是 mAttachInfo 此时为空。

这里有一个点就是 Activity 的各个回调函数都是干嘛的？是不是平时自己写应用的时候，貌似在 onCreate 里面搞定一切就OK了， onResume ？ onStart？没怎么涉及到嘛，其实不然。
onCreate 顾名思义就是 Create ，我们在前面看到，Activity 的 onCreate 函数做了很多初始化的操作，包括 PhoneWindow/DecorView/StartingView/setContentView等，但是 onCreate 只是初始化了这些对象.
真正要设置为显示则在 Resume 的时候，不过这些对开发者是透明了，具体可以看 ActivityThread 的 handleResumeActivity 函数，handleResumeActivity 中除了调用 Activity 的 onResume 回调之外，还初始化了几个比较重要的类：ViewRootImpl / ThreadedRenderer。

ActivityThread.handleResumeActivity:

```java
if (r.window == null && !a.mFinished && willBeVisible) {
    r.window = r.activity.getWindow();
    View decor = r.window.getDecorView();
    decor.setVisibility(View.INVISIBLE);
    ViewManager wm = a.getWindowManager();
    WindowManager.LayoutParams l = r.window.getAttributes();
    a.mDecor = decor;
    l.type = WindowManager.LayoutParams.TYPE_BASE_APPLICATION;
    l.softInputMode |= forwardBit;
    if (a.mVisibleFromClient) {
        a.mWindowAdded = true;
        wm.addView(decor, l);
    }
```

主要是 wm.addView(decor, l); 这句，将 decorView 与 WindowManagerImpl联系起来，这句最终会调用到 WindowManagerGlobal 的 addView 函数，

```java
public void addView(View view, ViewGroup.LayoutParams params,
        Display display, Window parentWindow) {
    ......
    ViewRootImpl root;
    View panelParentView = null;
    ......
        root = new ViewRootImpl(view.getContext(), display);
        view.setLayoutParams(wparams);

        mViews.add(view);
        mRoots.add(root);
        mParams.add(wparams);
    }

    // do this last because it fires off messages to start doing things
    try {
        root.setView(view, wparams, panelParentView);
    } catch (RuntimeException e) {
      ......
    }
}
```

我们知道 ViewRootImpl 是 View 系统的一个核心类，其定义如下：

```java
public final class ViewRootImpl implements ViewParent,
        View.AttachInfo.Callbacks, HardwareRenderer.HardwareDrawCallbacks
```

ViewRootImpl 初始化的时候会对 AttachInfo 进行初始化，这就是为什么之前的在 onCreate 的时候 attachInfo 为空。ViewRootImpl 里面有很多我们比较熟悉也非常重要的方法，比如 performTraversals / performLayout / performMeasure / performDraw / draw 等。
我们继续 addView 中的root.setView(view, wparams, panelParentView); 传入的 view 为 decorView，root 为 ViewRootImpl ，这个函数中将 ViewRootImpl 的mView 变量 设置为传入的view，也就是 decorView。
这样来看，ViewRootImpl 与 DecorView 的关系我们也清楚了。

扯了一圈，我们再回到大标题的 Post 函数上，前面有说这个 Post 走的是 View 的Post 函数，由于 在 onCreate 的时候 attachInfo 为空，所以会走下面的分支：ViewRootImpl.getRunQueue().post(action);
注意这里的 getRunQueue 得到的并不是 Looper 里面的那个 MessageQueue，而是由 ViewRootImpl 维持的一个 RunQueue 对象，其核心为一个 ArrayList ：

```java
private final ArrayList<HandlerAction> mActions = new ArrayList<HandlerAction>();

        void post(Runnable action) {
            postDelayed(action, 0);
        }

        void postDelayed(Runnable action, long delayMillis) {
            HandlerAction handlerAction = new HandlerAction();
            handlerAction.action = action;
            handlerAction.delay = delayMillis;

            synchronized (mActions) {
                mActions.add(handlerAction);
            }
        }

        void executeActions(Handler handler) {
            synchronized (mActions) {
                final ArrayList<HandlerAction> actions = mActions;
                final int count = actions.size();

                for (int i = 0; i < count; i++) {
                    final HandlerAction handlerAction = actions.get(i);
                    handler.postDelayed(handlerAction.action, handlerAction.delay);
                }

                actions.clear();
            }
        }
```

当我们执行了 Post 之后 ，其实只是把 Runnable 封装成一个 HandlerAction 对象存入到 ArrayList 中，当执行到 executeActions 方法的时候，将存在这里的 HandlerAction 再通过 executeActions 方法传入的 Handler 对象重新进行 Post。
那么 executeActions 方法是什么时候执行的呢？传入的 Handler 又是哪个 Handler 呢？

## 4. PerformTraversals

我们之前讲过，ViewRootImpl 的 performTraversals 方法是一个很核心的方法，每一帧绘制都会走一遍，调用各种 measure / layout / draw 等 ，最终将要显示的数据交给 hwui 去进行绘制。
我们上一节讲到的 executeActions ，就是在 performTraversals 中执行的：

```
// Execute enqueued actions on every traversal in case a detached view enqueued an action
getRunQueue().executeActions(mAttachInfo.mHandler);
```

可以看到这里传入的 Handler 是 mAttachInfo.mHandler ，上一节讲到 mAttachInfo 是在 ViewRootImpl 初始化的时候一起初始化的：

```
mAttachInfo = new View.AttachInfo(mWindowSession, mWindow, display, this, mHandler, this);
```

这里的 mHandler 是一个 ViewRootHandler 对象：

```
final class ViewRootHandler extends Handler{
    ......
}
......
final ViewRootHandler mHandler = new ViewRootHandler();
```

我们注意到 ViewRootHandler 在创建的时候并没有传入一个 Looper 对象，这意味着此 ViewRootHandler 的 Looper 就是 mainLooper。

**这下我们就清楚了，我们在 onCreate 中 Post 的 runnable 对象，最终还是在第一个 performTraversals 方法执行的时候，加入到了 MainLooper 的 MessageQueue 里面了。**

绕了一圈终于我们终于把文章最前面的那句话解释清楚了，当然中间还有很多的废话，不过我估计能耐着性子看到这里的人会很少，所以如果你看到了这里，可以在底下的评论里面将 index ++ ；这里 index = 0 ；就是看看几个人是真正认真看了这篇文章的。

## 5. UpdateText

接着 performTraversals 我们继续说，话说在[第一篇文章](http://www.androidperformance.com/2015/11/18/Android-app-lunch-optimize-delay-load.html) 我们有讲到，Activity 在启动时，会在第二次执行 performTraversals 才会去真正的绘制，原因在于第一次执行 performTraversals 的时候，会走到 Egl 初始化的逻辑，然后会重新执行一次 performTraversals 。
所以前一篇文章的评论区有人问为何在 run 方法里面还要 post 一次，如果在 run 方法里面直接执行 updateText 方法 ，那么 updateText 就会在第一个 performTraversals 之后就执行，而不是在第一帧绘制完成后才去执行，所以我们又 Post 了一次 。所以大概的处理步骤如下：

> 第一步：Activity.onCreate –> Activity.onStart –> Activity.onResume
>
> 第二步：ViewRootImpl.performTraversals –>Runnable
>
> 第三步：Runnable –> ViewRootImpl.performTraversals
>
> 第四步：ViewRootImpl.performTraversals –> UpdateText
>
> 第五步：UpdateText

## 6. 总结

其实一路跟下来发现其实原理很简单，其实 DelayLoad 其实只是一个很小的点，关键是教大家如何去跟踪一个自己不认识的知识点或者优化，这里面主要用到了两个工具：Systrace 和 Method Trace， 以及源码编译和调试。
关于 Systrace 和 Method Trace 的使用，之后会有详细的文章去介绍，这两个工具非常有助于理解源码和一些技术的实现。

### Systrace

[![Systrace](http://androidperformance.com/images/applunch2/2.png)](http://androidperformance.com/images/applunch2/2.png)Systrace

### Method Trace

[![Method Trace](http://androidperformance.com/images/applunch2/3.png)](http://androidperformance.com/images/applunch2/3.png)Method Trace

### 源码编译与调试

[![源码编译与调试](http://androidperformance.com/images/applunch2/4.png)](http://androidperformance.com/images/applunch2/4.png)源码编译与调试



### 代码

本文章所所涉及到的代码我放到了Github上：
https://github.com/Gracker/DelayLoadSample