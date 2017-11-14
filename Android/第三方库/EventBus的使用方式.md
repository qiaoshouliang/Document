# 添加EventBus

1. 在app/build.gradle下添加
```
defaultConfig {
...
    javaCompileOptions {
            annotationProcessorOptions {
                arguments = [eventBusIndex : 'org.greenrobot.eventbus.perf.EventBusIndex']
            }
        }
}   
dependencies {

 /* EventBus */
    compile 'org.greenrobot:eventbus:3.0.0'
    annotationProcessor 'org.greenrobot:eventbus-annotation-processor:3.0.1'
}
```

2. 随意创建一个subscribe的event
```
@Subscribe(threadMode = ThreadMode.POSTING)
    public void message (MessageA event){

    }
```
注：为什么要创建这个subscribe的event，主要是由于注解预编译首先要在项目中有一个注解@Subscribe，否则无法预编译

3. 重新Rebuild Project
Rebuild Project 之后就会在build目录下生产EventBusIndex这个类

![image](http://note.youdao.com/yws/public/resource/322d87c79af5911fea0f459e301fdb3f/xmlnote/AEBE5F4F58A8417C9A295EC68E429102/2523)
3. 在Application类下加入
```
EventBus.builder().addIndex(new EventBusIndex()).installDefaultEventBus();
```

