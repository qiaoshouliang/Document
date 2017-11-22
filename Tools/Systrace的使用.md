# Systrace启动

你可以通过命令行或者Device Monitor两种方式收集Systrace信息，以下以命令行为例介绍收集方式(因为我Device Monitor的方式报错)。 
首先进入sdk下的platform-tools/systrace目录下: 
![这里写图片描述](http://img.blog.csdn.net/20151005185945133) 
然后在命令下执行以下命令来收集数据: 
`python systrace.py --time=10 -o mynewtrace.html sched gfx view wm`

为了方便起见我们会使用alias的方式

```java
alias st-start='/Users/qiaoshouliang/Library/Android/sdk/platform-tools/systrace/systrace.py'
alias systrace='st-start -t 8 gfx input view sched freq wm am hwui workq res dalvik sync disk load perf hal rs idle mmc'
```

以后直接就可以用systrace命令生产trace文件

上面的参数–time为间隔时间,-o为文件名，更详细的参数信息如下:

| 参数名                                      | 意义                          |
| ---------------------------------------- | --------------------------- |
| `-h,--help`                              | 帮助信息                        |
| `-o <FILE>`                              | 保存的文件名                      |
| `-t N,--time=N`                          | 多少秒内的数据，默认为5秒，以当前时间点往后倒N个时间 |
| `-b N,--buf-size=N`                      | 单位为千字节,限制数据大小               |
| `-k <KFUNCS> --ktrace=<KFUNCS>`          | 追踪特殊的方法                     |
| `-l,--list-categories`                   | 设置追踪的标签                     |
| `-a <APP_NAME>,--app=<APP_NAME>`         | 包名                          |
| `--from-file=<FROM_FILE>`                | 创建报告的来源trace文件              |
| `-e <DEVICE_SERIAL>,--serial=<DEVICE_SERIAL>` | 设备号                         |

其中标签可选项如下:

| 标签名     | 意义                      |
| ------- | ----------------------- |
| gfx     | Graphics                |
| input   | Input                   |
| view    | View                    |
| webview | Webview                 |
| vm      | Window Manager          |
| am      | Activity Manager        |
| audio   | Audio                   |
| video   | Video                   |
| camera  | Camera                  |
| hal     | Hardware Modules        |
| res     | Resource Loading        |
| dalvik  | Dalvik VM               |
| rs      | RenderScript            |
| sched   | Cpu Scheduling          |
| freq    | Cpu Frequency           |
| membus  | Memory Bus Utilization  |
| idle    | Cpu Idle                |
| disk    | Disk input and output   |
| load    | Cpu Load                |
| sync    | Synchronization Manager |
| workq   | Kernel Workqueues       |

以上标签并不支持所有机型,还有要想在输出中看到任务的名称，需要加上sched.

上面的命令执行完后，会生成一个html文件: 
![这里写图片描述](http://img.blog.csdn.net/20151005190209942) 
打开该文件后，我们会看到如下页面: 
![这里写图片描述](http://img.blog.csdn.net/20151005205803709)

# systrace快捷键

| 快捷键  | 作用             |
| ---- | -------------- |
| w    | 放大             |
| s    | 缩小             |
| a    | 左移             |
| d    | 右移             |
| f    | 返回选中区域，切放大选中区域 |

![这里写图片描述](http://img.blog.csdn.net/20151006135736416)

# Alerts

Alerts一栏标记了以下性能有问题的点，你可以点击该点查看详细信息,右边侧边栏还有一个Alerts框，点击可以查看每个类型的Alerts的数量:

![这里写图片描述](http://img.blog.csdn.net/20151008145232031)

# Frame

在每个包下都有Frame一栏，该栏中都有一个一个的`F`代表每一个`Frame`，用颜色来代表性能的好坏，依次为`绿-黄-红`(性能越来越差),点击某一个`F`,会显示该Frame绘制过程中的一些Alerts信息: 
![这里写图片描述](http://img.blog.csdn.net/20151008145904811)

如果你想查看Frame的耗时，可以点击某个F标志，然后按`m`键: 

![这里写图片描述](http://img.blog.csdn.net/20151008152510281)

