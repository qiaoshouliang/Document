# NodeMcu 驱动安装

1、按住电源键重启，立即按住CMD+R进入Recovery模式。
2、在菜单上选择，打开 Terminal 控制台。
3、运行命令： csrutil enable --without kext
4、输入：reboot

5、系统重启后，访问<http://www.wch.cn/download/CH341SER_MAC_ZIP.html>

下载驱动、安装即可。

```shell
esptool.py --port=/dev/tty.wchusbserial14530 write_flash -fm=dio -fs=32m 0x00000 /Users/qiaoshouliang/IoT/firmware/nodemcu-master-14-modules-2018-02-02-07-51-54-float.bin
```





# smartconfig 配网

## 参考：

https://www.espressif.com/zh-hans/products/software/esp-touch/resources

https://www.cnblogs.com/zeroes/p/nodemcu_build_smartconfig.html

http://nodemcu.readthedocs.io/en/master/en/build/

## APP端

在esp-touch(https://www.espressif.com/zh-hans/products/software/esp-touch/resources)网站中下载`ESP-TOUCH for Android Code`

编译成功的项目放到了github上，https://github.com/qiaoshouliang/EsptouchForAndroid.git

## NodeMCU端

通过wifi模块的startsmart()方法实现类似于SmartConfig的配网方式。

```lua
wifi.setmode(wifi.STATION)
wifi.startsmart(0,
    function(ssid, password)
        print(string.format("Success. SSID:%s ; PASSWORD:%s", ssid, password))
    end
)
```

>由于startsmart在默认固件中是未开启的，需要我们在`app/include/user_config.h`文件中设置`WIFI_SMART_ENABLE`。

## 使用Docker打包NodeMCU固件

- 安装docker
- 运行命令
  ```
  docker run --rm -ti -v `pwd`:/opt/nodemcu-firmware 	marcelstoer/nodemcu-build
  ```



当运行这个命令的时候提示：

```
\\fatal: ambiguous argument 'HEAD': unknown revision or path not in the working tree.

Use '--' to separate paths from revisions, like this:

'git <command> [<revision>...] -- [<file>...]'

```

需要在本地库中创建git，并提交一个版本

详细的builde流程可参考

http://nodemcu.readthedocs.io/en/master/en/build/



# MQTT

## MQTT简介

> MQTT官网：<http://mqtt.org/>
>
> MQTT介绍：[http://www.ibm.com](http://www.ibm.com/support/knowledgecenter/zh/SS9D84_1.0.0/com.ibm.mm.tc.helphome.v10.doc/welcome_page.htm)
>
> MQTT Android github：<https://github.com/eclipse/paho.mqtt.android>
>
> MQTT API：<http://www.eclipse.org/paho/files/javadoc/index.html>
>
> MQTT Android API： <http://www.eclipse.org/paho/files/android-javadoc/index.html>

建议时间充裕的同学有顺序的阅读上文五个链接内容，不充裕的同学请看下面简单的介绍（内容大多来自上面五条链接）。

MQTT 协议 
客户机较小并且 MQTT 协议 高效地使用网络带宽，在这个意义上，其为轻量级。MQTT 协议支持可靠的传送和即发即弃的传输。 在此协议中，消息传送与应用程序脱离。 脱离应用程序的程度取决于写入 MQTT 客户机和 MQTT 服务器的方式。脱离式传送能够将应用程序从任何服务器连接和等待消息中解脱出来。 交互模式与电子邮件相似，但在应用程序编程方面进行了优化。

协议具有许多不同的功能：

- 它是一种发布/预订协议。
- 除提供一对多消息分发外，发布/预订也脱离了应用程序。对于具有多个客户机的应用程序来说，这些功能非常有用。
- 它与消息内容没有任何关系。
- 它通过 TCP/IP 运行，TCP/IP 可以提供基本网络连接。
- 它针对消息传送提供三种服务质量：
  - “至多一次” 
    消息根据底层因特网协议网络尽最大努力进行传递。 可能会丢失消息。 
    例如，将此服务质量与通信环境传感器数据一起使用。 对于是否丢失个别读取或是否稍后立即发布新的读取并不重要。
  - “至少一次” 
    保证消息抵达，但可能会出现重复。
  - “刚好一次” 
    确保只收到一次消息。 
    例如，将此服务质量与记帐系统一起使用。 重复或丢失消息可能会导致不便或收取错误费用。
- 它是一种管理网络中消息流的经济方式。 例如，固定长度的标题仅 2 个字节长度，并且协议交换可最大程度地减少网络流量。
- 它具有一种“遗嘱”功能，该功能通知订户客户机从 MQTT 服务器异常断开连接。请参阅“[最后的消息](http://www.ibm.com/support/knowledgecenter/zh/SS9D84_1.0.0/com.ibm.mm.tc.doc/tc60360_.htm)”发布。

## MQTT服务器搭建

1. 点击[这里](http://activemq.apache.org/apollo/download.html)，下载Apollo服务器，解压后安装。
2. 命令行进入安装目录bin目录下（例：E:>cd E:\MQTT\apache-apollo-1.7.1\bin）。
3. 输入apollo create XXX（xxx为创建的服务器实例名称，例：apollo create mybroker），之后会在bin目录下创建名称为XXX的文件夹。XXX文件夹下etc\apollo.xml文件下是配置服务器信息的文件。etc\users.properties文件包含连接MQTT服务器时用到的用户名和密码，默认为admin=password，即账号为admin，密码为password，可自行更改。
4. 进入XXX/bin目录，输入apollo-broker.cmd run开启服务器，看到如下界面代表搭建完成

![success](http://img.blog.csdn.net/20160917015122907)

之后在浏览器输入<http://127.0.0.1:61680/>，查看是否安装成功。

## MQTT Android客户端具体实现

### 基本概念：

- topic：中文意思是“话题”。在MQTT中订阅了(`subscribe`)同一话题（`topic`）的客户端会同时收到消息推送。直接实现了“群聊”功能。
- clientId：客户身份唯一标识。
- qos：服务质量。
- retained：要保留最后的断开连接信息。
- MqttAndroidClient#subscribe()：订阅某个话题。
- MqttAndroidClient#publish()： 向某个话题发送消息，之后服务器会推送给所有订阅了此话题的客户。
- userName：连接到MQTT服务器的用户名。
- passWord ：连接到MQTT服务器的密码。

### 添加依赖

```groovy
repositories {
    maven {
        url "https://repo.eclipse.org/content/repositories/paho-releases/"
    }
}

dependencies {
    compile 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.1.0'
    compile 'org.eclipse.paho:org.eclipse.paho.android.service:1.1.0'
}
```

### 添加限权

```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
```

### 注册Service

```xml
        <!-- Mqtt Service -->
        <service android:name="org.eclipse.paho.android.service.MqttService" />
        <service android:name="com.dongyk.service.MQTTService"/>
```

### Android端具体实现

```java
package com.dongyk.service;

import android.app.Service;
...

/**
 * MQTT长连接服务
 *
 * @author 一口仨馍 联系方式 : yikousamo@gmail.com
 * @version 创建时间：2016/9/16 22:06
 */
public class MQTTService extends Service {

    public static final String TAG = MQTTService.class.getSimpleName();

    private static MqttAndroidClient client;
    private MqttConnectOptions conOpt;

//    private String host = "tcp://10.0.2.2:61613";
    private String host = "tcp://192.168.1.103:61613";
    private String userName = "admin";
    private String passWord = "password";
    private static String myTopic = "topic";
    private String clientId = "test";

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        init();
        return super.onStartCommand(intent, flags, startId);
    }

    public static void publish(String msg){
        String topic = myTopic;
        Integer qos = 0;
        Boolean retained = false;
        try {
            client.publish(topic, msg.getBytes(), qos.intValue(), retained.booleanValue());
        } catch (MqttException e) {
            e.printStackTrace();
        }
    }

    private void init() {
        // 服务器地址（协议+地址+端口号）
        String uri = host;
        client = new MqttAndroidClient(this, uri, clientId);
        // 设置MQTT监听并且接受消息
        client.setCallback(mqttCallback);

        conOpt = new MqttConnectOptions();
        // 清除缓存
        conOpt.setCleanSession(true);
        // 设置超时时间，单位：秒
        conOpt.setConnectionTimeout(10);
        // 心跳包发送间隔，单位：秒
        conOpt.setKeepAliveInterval(20);
        // 用户名
        conOpt.setUserName(userName);
        // 密码
        conOpt.setPassword(passWord.toCharArray());

        // last will message
        boolean doConnect = true;
        String message = "{\"terminal_uid\":\"" + clientId + "\"}";
        String topic = myTopic;
        Integer qos = 0;
        Boolean retained = false;
        if ((!message.equals("")) || (!topic.equals(""))) {
            // 最后的遗嘱
            try {
                conOpt.setWill(topic, message.getBytes(), qos.intValue(), retained.booleanValue());
            } catch (Exception e) {
                Log.i(TAG, "Exception Occured", e);
                doConnect = false;
                iMqttActionListener.onFailure(null, e);
            }
        }

        if (doConnect) {
            doClientConnection();
        }

    }

    @Override
    public void onDestroy() {
        try {
            client.disconnect();
        } catch (MqttException e) {
            e.printStackTrace();
        }
        super.onDestroy();
    }

    /** 连接MQTT服务器 */
    private void doClientConnection() {
        if (!client.isConnected() && isConnectIsNomarl()) {
            try {
                client.connect(conOpt, null, iMqttActionListener);
            } catch (MqttException e) {
                e.printStackTrace();
            }
        }

    }

    // MQTT是否连接成功
    private IMqttActionListener iMqttActionListener = new IMqttActionListener() {

        @Override
        public void onSuccess(IMqttToken arg0) {
            Log.i(TAG, "连接成功 ");
            try {
                // 订阅myTopic话题
                client.subscribe(myTopic,1);
            } catch (MqttException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onFailure(IMqttToken arg0, Throwable arg1) {
            arg1.printStackTrace();
              // 连接失败，重连
        }
    };

    // MQTT监听并且接受消息
    private MqttCallback mqttCallback = new MqttCallback() {

        @Override
        public void messageArrived(String topic, MqttMessage message) throws Exception {

            String str1 = new String(message.getPayload());
            MQTTMessage msg = new MQTTMessage();
            msg.setMessage(str1);
            EventBus.getDefault().post(msg);
            String str2 = topic + ";qos:" + message.getQos() + ";retained:" + message.isRetained();
            Log.i(TAG, "messageArrived:" + str1);
            Log.i(TAG, str2);
        }

        @Override
        public void deliveryComplete(IMqttDeliveryToken arg0) {

        }

        @Override
        public void connectionLost(Throwable arg0) {
            // 失去连接，重连
        }
    };

    /** 判断网络是否连接 */
    private boolean isConnectIsNomarl() {
        ConnectivityManager connectivityManager = (ConnectivityManager) this.getApplicationContext().getSystemService(Context.CONNECTIVITY_SERVICE);
        NetworkInfo info = connectivityManager.getActiveNetworkInfo();
        if (info != null && info.isAvailable()) {
            String name = info.getTypeName();
            Log.i(TAG, "MQTT当前网络名称：" + name);
            return true;
        } else {
            Log.i(TAG, "MQTT 没有可用网络");
            return false;
        }
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

首先初始化各个参数，之后连接服务器。连接成功之后在<http://127.0.0.1:61680/>看到自动创建了名称为”topic”的`topic`。这里我使用了一个真机和一个模拟器运行程序。<http://127.0.0.1:61680/>服务端看到的是这个样子

![serverPic](http://img.blog.csdn.net/20160918105925850)

这里需要注意两个地方 

1. 模拟器运行的时候`host = "tcp://10.0.2.2:61613"`，因为10.0.2.2 是模拟器设置的特定ip，是你电脑的别名。真机运行的时候`host = "tcp://192.168.1.103:61613"`。192.168.1.103是我主机的IPv4地址，查看本机IP的cmd命令为`ipconfig/all`。 
2. 两次运行时的`clientId`不能一样（为了保证客户标识的唯一性）。

这里为了测试，在`MQTTService`中暴露了一个公共方法`publish(String msg)`给`MainActivity`调用。代码如下

```java
public class MainActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        EventBus.getDefault().register(this);
        startService(new Intent(this, MQTTService.class));
        findViewById(R.id.publishBtn).setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                MQTTService.publish("CSDN 一口仨馍");
            }
        });
    }

    @Subscribe(threadMode = ThreadMode.MAIN)
    public void getMqttMessage(MQTTMessage mqttMessage){
        Log.i(MQTTService.TAG,"get message:"+mqttMessage.getMessage());
        Toast.makeText(this,mqttMessage.getMessage(),Toast.LENGTH_SHORT).show();
    }

    @Override
    protected void onDestroy() {
        EventBus.getDefault().unregister(this);
        super.onDestroy();
    }

}
```

这里使用了`EventBus3.0`发送消息，感兴趣的可以看下[EventBus3.0使用及源码解析](http://blog.csdn.net/qq_17250009/article/details/51872731)。当然，你也可以使用接口回调的方式甚至直接在Service中弹出`Toast`。whatever，现在点击一个客户端`MainActivity`中的`Button`，两个客户端已经能同时弹出消息。已经`get`到数据了。接下来，show time~

### Github地址

https://github.com/qiaoshouliang/MQTTDemo



## MQTT 设备端实现

### 参考：

http://nodemcu.readthedocs.io/en/master/en/modules/mqtt/

### 核心代码：

```Lua
-- init mqtt client with logins, keepalive timer 120sec
m = mqtt.Client("test2", 120,"admin","password")

-- setup Last Will and Testament (optional)
-- Broker will publish a message with qos = 0, retain = 0, data = "offline" 
-- to topic "/lwt" if client don't send keepalive packet
m:lwt("/lwt", "offline", 0, 0)

m:on("connect", function(client) print ("connected") end)
m:on("offline", function(client) print ("offline") end)

-- on publish message receive event
m:on("message", function(client, topic, data) 
  print(topic .. ":" ) 
  if data ~= nil then
    print(data)
  end
end)

-- for TLS: m:connect("192.168.11.118", secure-port, 1)
m:connect("192.168.1.111", 61613, 0, function(client)
  print("connected")
  -- Calling subscribe/publish only makes sense once the connection
  -- was successfully established. You can do that either here in the
  -- 'connect' callback or you need to otherwise make sure the
  -- connection was established (e.g. tracking connection status or in
  -- m:on("connect", function)).

  -- subscribe topic with qos = 0
  client:subscribe("topic", 0, function(client) print("subscribe success") end)
  -- publish a message with data = hello, QoS = 0, retain = 0
--  client:publish("topic", "hello", 0, 0, function(client) print("sent") end)
end,
function(client, reason)
  print("failed reason: " .. reason)
end)

m:close();
-- you can call m:connect again
```





#OTA 升级





# 小循环





# 附录：

固件地址：

https://github.com/nodemcu/nodemcu-firmware

文档地址

http://nodemcu.readthedocs.io/en/master/

nodemcu通过MQTT协议进行通讯

http://blog.csdn.net/blinkdr/article/details/63262633

固件编译

https://xiangzi.org/archives/387

Esp-touch 源码

https://www.espressif.com/zh-hans/products/software/esp-touch/resources

Smartconfig配置教程

https://www.cnblogs.com/zeroes/p/nodemcu_build_smartconfig.html

固件编译介绍

http://nodemcu.readthedocs.io/en/master/en/build/