
## 加入分包的配置，防止method的个数查过65k

```
/*加入分包配置*/
defaultConfig {
      ...

        multiDexEnabled true

       ...
    }
    
dependencies {
    /* multiDex */
    compile 'com.android.support:multidex:1.0.1'
    }
    
```
如果你的工程中已经含有Application类,那么让它继承android.support.multidex.MultiDexApplication类
如果你的Application已经继承了其他类并且不想做改动，那么还有另外一种使用方式,覆写attachBaseContext()方法:

```
public class MyApplication extends FooApplication {  
    @Override  
    protected void attachBaseContext(Context base) {  
        super.attachBaseContext(base);  
        MultiDex.install(this);  
    }  
}  
```

## OutOfMemoryError: Java heap space
当运行时如果看到如下错误:
```
UNEXPECTED TOP-LEVEL ERROR:  
java.lang.OutOfMemoryError: Java heap space 
```
在dexOptions中有一个字段用来增加java堆内存大小:
```
android {  
    // ...  
    dexOptions {  
        javaMaxHeapSize "2g"  
    }  
}
```


## 错误Conflict with dependency 'com.google.code.findbugs:jsr305' 解决方法
```
Android{

    configurations.all {
      resolutionStrategy.force 'com.google.code.findbugs:jsr305:1.3.9'
    }
}
```