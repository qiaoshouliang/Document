

    java.lang.RuntimeException: Unable to get provider com.hismart.intercom.ELContentProvider: java.lang.ClassNotFoundException: Didn't find class "com.hismart.intercom.ELContentProvider" on path: DexPathList[[zip file "/data/app/com.hismart.intercom-2.apk"],nativeLibraryDirectories=[/data/app-lib/com.hismart.intercom-2, /vendor/lib, /system/lib]]
是因为用了multidex分包，自定义了application.但是MultiDex.install(this)写错在oncreate里了.应该重写attachBaseContext() ,把MultiDex.install(this)写在里面就好了.