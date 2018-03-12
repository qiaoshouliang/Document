

# 获取某个包下的所有Class

## 思路

- 获取所有的**Dex path**
- 根据每一个**Dex path**，获取**DexFile**，并通过**DexFile**去遍历所有的Class文件

## 获取所有的Dex path

主要分为如下几种情况：

- **当系统支持Multidex的时候**

  即当系统版本大于2.1的时候，所有的dex文件都在`/data/app/com.alibaba.android.arouter.demo-2/base.apk`

- **当系统不支持Multidex的时候**

  即系统版本小于2.1的时候，main dex文件在/data/app/com.hanschen.multidex-1.apk中，secondary dex文件在`/data/data/<packagename>/code_cache/secondary-dexes/`目录中，包括pkgname.apk.classes2.zip，pkgname.apk.classes3.zip ...

- **当使用InstantRun的时候**

  当Android studio打开InstantRun模式的时候，需要再加载InstantRun产生的Dex

  ​

### 当系统支持Multidex的时候

有两种方法：

- 通过`ApplicationInfo`这个对象中的`sourceDir`属性就是dex path

  ApplicationInfo的获取方法如下：

  ```java
  ApplicationInfo applicationInfo = 	context.getPackageManager().getApplicationInfo(context.getPackageName(), 0);
  ```
- 通过`getPackageResourcePath`直接获取`dex path`

  获取方法如下：

  ```
  ctx.getPackageResourcePath()
  ```


注意： 获取到的**dex path**需要存到一个列表（sourcePaths）中，方便后期遍历这个列表，获取**DexFile**

### 当系统不支持Multidex的时候

除了加载Main dex，还需要加载Secondary dex，加载方式如下：

1. 获取secondary dex 的存放目录，

   ```java
   private static final String SECONDARY_FOLDER_NAME = "code_cache" + File.separator + "secondary-dexes";
   File dexDir = new File(applicationInfo.dataDir, SECONDARY_FOLDER_NAME);
   //applicationInfo.dataDir：/data/data/<packagename>
   ```

   这个dexDir指的就是Secondary dex的存放目录/data/data/<packagename>/code_cache/secondary-dexes/

2. 遍历这个目录下的所有dex文件，把每一个dex文件的路径添加到**Dex path**的列表（sourcePaths）中。

   ```java
   int totalDexNumber = getMultiDexPreferences(context).getInt(KEY_DEX_NUMBER, 1);
   Log.d(TAG, "isVMMultidexCapable: " + totalDexNumber);
   File dexDir = new File(applicationInfo.dataDir, SECONDARY_FOLDER_NAME);

   for (int secondaryNumber = 2; secondaryNumber <= totalDexNumber; secondaryNumber++) {
       //for each dex file, ie: test.classes2.zip, test.classes3.zip...
       String fileName = extractedFilePrefix + secondaryNumber + EXTRACTED_SUFFIX;
       File extractedFile = new File(dexDir, fileName);
       if (extractedFile.isFile()) {
           sourcePaths.add(extractedFile.getAbsolutePath());
           //we ignore the verify zip part
       } else {
           throw new IOException("Missing extracted secondary dex file '" + extractedFile.getPath() + "'");
       }
   }
   //从SP中获取MultiDex的dex个数
   private static SharedPreferences getMultiDexPreferences(Context context) {
           return context.getSharedPreferences(PREFS_FILE, Build.VERSION.SDK_INT < Build.VERSION_CODES.HONEYCOMB ? Context.MODE_PRIVATE : Context.MODE_PRIVATE | Context.MODE_MULTI_PROCESS);
       }
   ```

### 当使用InstantRun的时候

首先直观介绍一下InstantRun：

当开启InstantRun的时候，Apk会被分解为多个apk，如下图所示。如果未开启InstantRun的时候只有一个`base.apk`

![](http://oxshjm7y0.bkt.clouddn.com/20171207104056_eLHY6l_Screenshot.jpeg)

-  对于**5.0(SDK = 21)**以上的手机会把这些被拆解的apk路径放到`applicationInfo.splitSourceDirs`中。

![](http://oxshjm7y0.bkt.clouddn.com/20171207103936_VKft9G_Screenshot.jpeg)

- 对于**5.0**以下的手机需要通过反射`"com.android.tools.fd.runtime.Paths"`对象中的`getDexFileDirectory`的方法获取被拆解的apk路径。

代码如下所示：

```java
private static List<String> tryLoadInstantRunDexFile(ApplicationInfo applicationInfo) {
        List<String> instantRunSourcePaths = new ArrayList<>();

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP && null != applicationInfo.splitSourceDirs) {
            // add the split apk, normally for InstantRun, and newest version.
            instantRunSourcePaths.addAll(Arrays.asList(applicationInfo.splitSourceDirs));
            Log.d(Consts.TAG, "Found InstantRun supportqqq");
        } else {
            try {
                // This man is reflection from Google instant run sdk, he will tell me where the dex files go.
                Class pathsByInstantRun = Class.forName("com.android.tools.fd.runtime.Paths");
                Method getDexFileDirectory = pathsByInstantRun.getMethod("getDexFileDirectory", String.class);
                String instantRunDexPath = (String) getDexFileDirectory.invoke(null, applicationInfo.packageName);

                File instantRunFilePath = new File(instantRunDexPath);
                if (instantRunFilePath.exists() && instantRunFilePath.isDirectory()) {
                    File[] dexFile = instantRunFilePath.listFiles();
                    for (File file : dexFile) {
                        if (null != file && file.exists() && file.isFile() && file.getName().endsWith(".dex")) {
                            Log.d(TAG, "tryLoadInstantRunDexFile: "+file.getAbsolutePath());
                            instantRunSourcePaths.add(file.getAbsolutePath());
                        }
                    }
                    Log.d(Consts.TAG, "Found InstantRun support");
                }

            } catch (Exception e) {
                Log.e(Consts.TAG, "InstantRun support error, " + e.getMessage());
            }
        }

        return instantRunSourcePaths;
    }
```

## 根据每一个**Dex path**，获取**DexFile**，并通过**DexFile**去遍历所有的Class文件

核心方法：

```java
//根据Dex path获取DexFile
DexFile dexfile = DexFile.loadDex(path, null, 0);
//获取Class的枚举
Enumeration<String> dexEntries = dexfile.entries();
//遍历这个枚举
while (dexEntries.hasMoreElements()) {
  String className = dexEntries.nextElement();
  //判断这个类是否在packageName这个包下
  if (className.startsWith(packageName)) {
    //添加到一个列表中，后续用
    classNames.add(className);
  }
}
```

因为这个过程比较耗时，所以在线程中完成这些操作。

```java
public static Set<String> getFileNameByPackageName(Context context, final String packageName) throws PackageManager.NameNotFoundException, IOException, InterruptedException {
        final Set<String> classNames = new HashSet<>();

        List<String> paths = getSourcePaths(context);
  //创建一个CountDownLatch，用于在同步线程使用
        final CountDownLatch parserCtl = new CountDownLatch(paths.size());

        for (final String path : paths) {
            Log.d(TAG, "getFileNameByPackageName: "+path);
          //对每一个Dex Path在线程池中分配一个线程，执行这个loadDex的操作。
            DefaultPoolExecutor.getInstance().execute(new Runnable() {
                @Override
                public void run() {
                    DexFile dexfile = null;

                    try {
//                        if (path.endsWith(EXTRACTED_SUFFIX)) {
                            //NOT use new DexFile(path), because it will throw "permission error in /data/dalvik-cache"
                            dexfile = DexFile.loadDex(path, null, 0);
//                        } else {
//                            dexfile = new DexFile(path);
//                        }

                        Enumeration<String> dexEntries = dexfile.entries();
                        while (dexEntries.hasMoreElements()) {
                            String className = dexEntries.nextElement();
                            if (className.startsWith(packageName)) {
                                classNames.add(className);
                            }
                        }
                    } catch (Throwable ignore) {
                        Log.e("ARouter", "Scan map file in dex files made error.", ignore);
                    } finally {
                        if (null != dexfile) {
                            try {
                                dexfile.close();
                            } catch (Throwable ignore) {
                            }
                        }

                        parserCtl.countDown();
                    }
                }
            });
        }
		//当这些线程都执行完成之后返回
        parserCtl.await();

        Log.d(Consts.TAG, "Filter " + classNames.size() + " classes by packageName <" + packageName + ">");
        return classNames;
    }
```

