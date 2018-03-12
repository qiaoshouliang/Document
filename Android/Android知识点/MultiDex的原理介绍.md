# 概述

Android开发者应该都遇到了64K最大方法数限制的问题，针对这个问题，google也推出了multidex分包机制，在生成apk的时候，把整个应用拆成n个dex包（classes.dex、classes2.dex、classes3.dex），每个dex不超过64k个方法。使用multidex，在5.0以前的系统，应用安装时只安装main dex（包含了应用启动需要的必要class），在应用启动之后，需在Application的`attachBaseContext`中调用`MultiDex.install(base)`方法，在这时候才加载第二、第三…个dex文件，从而规避了64k问题。 
当然，在`attachBaseContext`方法中直接install启动second dex会有一些问题，比如install方法是一个同步方法，当在主线程中加载的dex太大的时候，耗时会比较长，可能会触发ANR。不过这是另外一个问题了，解决方法可以参考：[Android最大方法数和解决方案](http://blog.csdn.net/shensky711/article/details/52329035) <http://blog.csdn.net/shensky711/article/details/52329035>。

本文主要分析的是`MultiDex.install()`到底做了什么，如何把secondary dexes中的类动态加载进来。

# MultiDex使用到的路径解析

- **ApplicationInfo.sourceDir**：apk的安装路径，如`/data/app/com.hanschen.multidex-1.apk`
- **Context.getFilesDir()**：返回`/data/data/<packagename>/files`目录，一般通过openFileOutput方法输出文件到该目录
- **ApplicationInfo.dataDir**: 返回`/data/data/<packagename>`目录

# 源码分析

## 代码入口

代码入口很简单，简单粗暴，就调用了一个静态方法`MultiDex.install(base);`，传入一个Context对象

```java
    @Override
    protected void attachBaseContext(Context base) {
        super.attachBaseContext(base);
        MultiDex.install(base);
    }
```

## MultiDex.install分析

下面是主要的代码

```Java
    public static void install(Context context) {
        Log.i("MultiDex", "install");
        if (IS_VM_MULTIDEX_CAPABLE) {
            //VM版本大于2.1时，IS_VM_MULTIDEX_CAPABLE为true，这时候MultiDex.install什么也不用做，直接返回。因为大于2.1的VM会在安装应用的时候，就把多个dex合并到一块
        } else if (VERSION.SDK_INT < 4) {
            //Multi dex最小支持的SDK版本为4
            throw new RuntimeException("Multi dex installation failed. SDK " + VERSION.SDK_INT + " is unsupported. Min SDK version is " + 4 + ".");
        } else {
            try {
                ApplicationInfo e = getApplicationInfo(context);
                if (e == null) {
                    return;
                }

                Set var2 = installedApk;
                synchronized (installedApk) {
                    String apkPath = e.sourceDir;
                    //检测应用是否已经执行过install()了，防止重复install
                    if (installedApk.contains(apkPath)) {
                        return;
                    }

                    installedApk.add(apkPath);

                    //获取ClassLoader，后面会用它来加载second dex
                    DexClassLoader classLoader;
                    ClassLoader loader;
                    try {
                        loader = context.getClassLoader();
                    } catch (RuntimeException var9) {
                        return;
                    }

                    if (loader == null) {
                        return;
                    }

                    //清空目录：/data/data/<packagename>/files/secondary-dexes/，其实我没搞明白这个的作用，因为从后面的代码来看，这个目录是没有使用到的
                    try {
                        clearOldDexDir(context);
                    } catch (Throwable var8) {
                    }

                    File dexDir = new File(e.dataDir, "code_cache/secondary-dexes");
                    //把dex文件缓存到/data/data/<packagename>/code_cache/secondary-dexes/目录，[后有详细分析]
                    List files = MultiDexExtractor.load(context, e, dexDir, false);
                    if (checkValidZipFiles(files)) {
                        //进行安装，[后有详细分析]
                        installSecondaryDexes(loader, dexDir, files);
                    } else {
                        //文件无效，从apk文件中再次解压secondary dex文件后进行安装
                        files = MultiDexExtractor.load(context, e, dexDir, true);
                        if (!checkValidZipFiles(files)) {
                            throw new RuntimeException("Zip files were not valid.");
                        }

                        installSecondaryDexes(loader, dexDir, files);
                    }
                }
            } catch (Exception var11) {
                throw new RuntimeException("Multi dex installation failed (" + var11.getMessage() + ").");
            }
        }
    }
```

这段代码的主要逻辑整理如下：

1. VM版本检测，如果大于2.1就什么都不做(系统在安装应用的时候已经帮我们把dex合并了)，如果系统SDK版本小于4就抛出运行时异常
2. 把apk中的secondary dexes解压到缓存目录，并把这些缓存读取出来。应用第二次启动的时候，会尝试从缓存目录中读取，除非读取出的文件校验失败，否则不再从apk中解压dexes
3. 根据当前的SDK版本，执行不同的安装方法

先来看看`MultiDexExtractor.load(context, e, dexDir, false)`

```Java
    /**
     * 解压apk文件中的classes2.dex、classes3.dex等文件解压到dexDir目录中
     *
     * @param dexDir      解压目录
     * @param forceReload 是否需要强制从apk文件中解压，否的话会直接读取旧文件
     * @return 解压后的文件列表
     * @throws IOException
     */
    static List<File> load(Context context,
                           ApplicationInfo applicationInfo,
                           File dexDir,
                           boolean forceReload) throws IOException {
        File sourceApk = new File(applicationInfo.sourceDir);
        long currentCrc = getZipCrc(sourceApk);
        List files;
        if (!forceReload && !isModified(context, sourceApk, currentCrc)) {
            try {
                //从缓存目录中直接查找缓存文件，跳过解压
                files = loadExistingExtractions(context, sourceApk, dexDir);
            } catch (IOException var9) {
                files = performExtractions(sourceApk, dexDir);
                putStoredApkInfo(context, getTimeStamp(sourceApk), currentCrc, files.size() + 1);
            }
        } else {
            //把apk中的secondary dex文件解压到缓存目录，并把解压后的文件返回
            files = performExtractions(sourceApk, dexDir);
            //把解压信息保存到sharedPreferences中
            putStoredApkInfo(context, getTimeStamp(sourceApk), currentCrc, files.size() + 1);
        }

        return files;
    }
```

首先判断以下是否需要强制从apk文件中解压，再进行下CRC校验，如果不需要从apk重新解压，就直接从缓存目录中读取已解压的文件返回，否则解压apk中的classes文件到缓存目录，再把相应的文件返回。这个方法再往下的分析就不贴出来了，不复杂，大家可以自己去看看。读取后会把解压信息保存到sharedPreferences中，里面会保存时间戳、CRC校验和dex数量。

得到dex文件列表后，要做的就是把dex文件关联到应用，这样应用findclass的时候才能成功。这个主要是通过`installSecondaryDexes`方法来完成的

```Java
    /**
     * 安装dex文件
     *
     * @param loader 类加载器
     * @param dexDir 缓存目录，用以存放opt之后的dex文件
     * @param files  需要安装的dex
     * @throws IllegalArgumentException
     * @throws IllegalAccessException
     * @throws NoSuchFieldException
     * @throws InvocationTargetException
     * @throws NoSuchMethodException
     * @throws IOException
     */
    private static void installSecondaryDexes(ClassLoader loader,
                                              File dexDir,
                                              List<File> files) throws IllegalArgumentException, IllegalAccessException, NoSuchFieldException, InvocationTargetException, NoSuchMethodException, IOException {

        if (!files.isEmpty()) {
            //对不同版本的SDK做不同处理
            if (VERSION.SDK_INT >= 19) {
                MultiDex.V19.install(loader, files, dexDir);
            } else if (VERSION.SDK_INT >= 14) {
                MultiDex.V14.install(loader, files, dexDir);
            } else {
                MultiDex.V4.install(loader, files);
            }
        }

    }
```

可以看到，对于不同的SDK版本，分别采用了不同的处理方法，我们主要分析SDK>=19的情况，其他情况大同小异，读者可以自己去分析。

```Java
    private static final class V19 {
        private V19() {
        }

        /**
         * 安装dex文件
         *
         * @param loader                     类加载器
         * @param additionalClassPathEntries 需要安装的dex
         * @param optimizedDirectory         缓存目录，用以存放opt之后的dex文件
         * @throws IllegalArgumentException
         * @throws IllegalAccessException
         * @throws NoSuchFieldException
         * @throws InvocationTargetException
         * @throws NoSuchMethodException
         */
        private static void install(ClassLoader loader,
                                    List<File> additionalClassPathEntries,
                                    File optimizedDirectory) throws IllegalArgumentException, IllegalAccessException, NoSuchFieldException, InvocationTargetException, NoSuchMethodException {

            //通过反射获取ClassLoader对象中的pathList属性，其实是ClassLoader的父类BaseDexClassLoader中的成员
            Field pathListField = MultiDex.findField(loader, "pathList");
            //通过属性获取该属性的值，该属性的类型是DexPathList
            Object dexPathList = pathListField.get(loader);

            ArrayList suppressedExceptions = new ArrayList();
            //通过反射调用dexPathList的makeDexElements返回Element对象数组。方法里面会读取每一个输入文件，生成DexFile对象，并将其封装进Element对象
            Object[] elements = makeDexElements(dexPathList, new ArrayList(additionalClassPathEntries), optimizedDirectory, suppressedExceptions);

            //将elements数组跟dexPathList对象的dexElements数组合并，并把合并后的数组作为dexPathList新的值
            MultiDex.expandFieldArray(dexPathList, "dexElements", elements);

            //处理异常
            if (suppressedExceptions.size() > 0) {
                Iterator suppressedExceptionsField = suppressedExceptions.iterator();

                while (suppressedExceptionsField.hasNext()) {
                    IOException dexElementsSuppressedExceptions = (IOException) suppressedExceptionsField.next();
                    Log.w("MultiDex", "Exception in makeDexElement", dexElementsSuppressedExceptions);
                }

                Field suppressedExceptionsField1 = MultiDex.findField(loader, "dexElementsSuppressedExceptions");
                IOException[] dexElementsSuppressedExceptions1 = (IOException[]) ((IOException[]) suppressedExceptionsField1.get(loader));
                if (dexElementsSuppressedExceptions1 == null) {
                    dexElementsSuppressedExceptions1 = (IOException[]) suppressedExceptions.toArray(new IOException[suppressedExceptions
                            .size()]);
                } else {
                    IOException[] combined = new IOException[suppressedExceptions.size() + dexElementsSuppressedExceptions1.length];
                    suppressedExceptions.toArray(combined);
                    System.arraycopy(dexElementsSuppressedExceptions1, 0, combined, suppressedExceptions.size(), dexElementsSuppressedExceptions1.length);
                    dexElementsSuppressedExceptions1 = combined;
                }

                suppressedExceptionsField1.set(loader, dexElementsSuppressedExceptions1);
            }

        }

        private static Object[] makeDexElements(Object dexPathList,
                                                ArrayList<File> files,
                                                File optimizedDirectory,
                                                ArrayList<IOException> suppressedExceptions) throws IllegalAccessException, InvocationTargetException, NoSuchMethodException {
            Method makeDexElements = MultiDex.findMethod(dexPathList, "makeDexElements", new Class[]{ArrayList.class, File.class, ArrayList.class});
            return (Object[]) ((Object[]) makeDexElements.invoke(dexPathList, new Object[]{files, optimizedDirectory, suppressedExceptions}));
        }
    }
```

在Android中，有两个ClassLoader，分别是`DexClassLoader`和`PathClassLoader`，它们的父类都是`BaseDexClassLoader`，DexClassLoader和PathClassLoader的实现都是在BaseDexClassLoader之中，而BaseDexClassLoader的实现又基本是通过调用DexPathList的方法完成的。DexPathList里面封装了加载dex文件为DexFile对象（调用了native方法，有兴趣的童鞋可以继续跟踪下去）的方法。 
上述代码中的逻辑如下：

1. 通过反射获取pathList对象
2. 通过pathList把输入的dex文件输出为elements数组，elements数组中的元素封装了DexFile对象
3. 把新输出的elements数组合并到原pathList的dexElements数组中
4. 异常处理

当把dex文件加载到pathList的dexElements数组之后，整个multidex.install基本上就完成了。 
但可能还有些童鞋还会有些疑问，仅仅只是把Element数组合并到ClassLoader就可以了吗？还是没有找到加载类的地方啊？那我们再继续看看，当用到一个类的时候，会用ClassLoader去加载一个类，加载类会调用类加载器的findClass方法

```java
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        List<Throwable> suppressedExceptions = new ArrayList<Throwable>();
        //调用pathList的findClass方法
        Class c = pathList.findClass(name, suppressedExceptions);
        if (c == null) {
            ClassNotFoundException cnfe = new ClassNotFoundException("Didn't find class \"" + name + "\" on path: " + pathList);
            for (Throwable t : suppressedExceptions) {
                cnfe.addSuppressed(t);
            }
            throw cnfe;
        }
        return c;
    }
```

于是继续跟踪：

```java
    public Class findClass(String name, List<Throwable> suppressed) {
        //遍历dexElements数组
        for (Element element : dexElements) {

            DexFile dex = element.dexFile;
            if (dex != null) {
                //继续跟踪会发现调用的是一个native方法
                Class clazz = dex.loadClassBinaryName(name, definingContext, suppressed);
                if (clazz != null) {
                    return clazz;
                }
            }
        }
        if (dexElementsSuppressedExceptions != null) {
            suppressed.addAll(Arrays.asList(dexElementsSuppressedExceptions));
        }
        return null;
    }
```

到现在就清晰了，当加载一个类的时候，会遍历dexElements数组，通过native方法从Element元素中加载类名相应的类

# 总结

到最后，总结整个multidex.install流程，其实很简单，就做了一件事情，把apk中的secondary dex文件通过ClassLoader转换成Element数组，并把输出的数组合与ClassLoader的Element数组合并。