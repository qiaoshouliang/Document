#### 在 android 设备的 Linux 内核中把 USB 驱动的 PID VID 修改以后，也许之前的 adb工具就不能识别设备了，会打印出 "device not found" 的提示，特别是 windows ，adb驱动也不能用了。
这个时候该怎么办呢？
# 一，先搞清现在的 PID，VID 为多少，例如分别为 aaaa、bbbb（十六进制）
对于 linux 如 ubuntu, 插入 设备，再查询 /sys/bus/usb/ 下的相应设备，如 "1-1" 下的idVender, idProduct 获得。
对于 windows, 可从相应设备的属性中获得
# 二，对于windows:
- 修改 adb 驱动中的 android_winusb.inf 文件。对照着别的项目加就可以了。
例如：
; My product
%CompositeAdbInterface% = USB_Install,USB\VID_AAAA&PID_BBBB
%SingleBootLoaderInterface% = USB_Install,USB\VID_AAAA&PID_BBBB&MI_01
-  右击我的电脑->属性->高级->环境变量，添加 ANROID_SDK_HOME 环境变量。如果你有android SDK,就设成SDK的路径；如果没有，那也没关系，设为你觉得方便的任何路径。
- 在前面设置的`ANDROID_SDK_HOME`对应的路径下，寻找`.android`目录，如果没有就创建一个；在`.android`目录下新建一个文件，叫adb_usb.ini, 记住，后缀是 "ini" 哦；添加前面获得的VID 到 adb_usb.ini 中，如 0xAAAA。
-  然后安装 adb 驱动
-  进入 DOS cmd, 分别输入 adb kill-server, adbstart-server, adb devices；若能看到 devices 列出，那么就成功了。
注意：要将手机设为MTP模式而不是充电模式

# 三，对于linux
- 在用户的 home 目录下，寻找 .android目录，如果没有就创建。在.android目录下新建一个文件，叫adb_usb.ini,记住，后缀是 "ini" 哦；添加前面获得的 VID 到 adb_usb.ini 中，如 0xAAAA。
- 在 shell 中 分别输入 adb kill-server, adbstart-server, adb devices；若能看到 devices 列出，那么就成功了。
这里 kill-server，是 kill 掉 adb 的 daemon，否则，adb 还是保留着先前的 Vender信息。start-server 就加载了新的 Vender 信息。

注意：有些人可能用adb 需要超级用户权限，这时第一步中的home 目录应该为 /root，不想麻烦，直接两个 Home 目录都加上得了。