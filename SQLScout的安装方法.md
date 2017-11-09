官网：http://www.idescout.com/download/
# 1. 安装SQLScout插件
- 打开Android Studio
- Settings(on Windows and Linux) or Preferences(Mac)
- Plugins
- Browse Repositories...
- 选择SQLScout并安装
# 2. 激活SQLScout
在试用期过后，需要购买一个商业证书来激活SQLScout。
通过这里 [购买商业证书] (https://www.idescout.com/secure/buy)，然后点击Activate按钮。
# 3. 破解
注意：以下破解只供学习讨论，请勿传播
##  3.1 破解SQLScout
通过前面的方法安装SQLScout插件之后，进入Intellij IDEA插件安装目录：

```
~/Library/Application Support/AndroidStudiox.x/SQLScout/lib/

```
反编译SQLScout.jar
进入com/idescout/sqlite/license/，使用javassist修改License.class如下：

```
ClassPool pool = ClassPool.getDefault();
CtClass c = pool.get("com.idescout.sqlite.license.License");
CtMethod m1 = c.getDeclaredMethod("isValidLicense");
m1.setBody("{ return true; }");

CtMethod m2 = c.getDeclaredMethod("isValidLicense", new CtClass[]{pool.makeClass("com.intellij.openapi.project.Project")});
m2.setBody("{ return true; }");

c.writeFile();
```

编译后复制License.class文件，替换原来的License.class。
然后jar cvf SQLScout.jar ./*打包jar。
最后替换~/Library/Application Support/AndroidStudio/SQLScout/lib/下的SQLScout.jar文件，重启Android Studio。
3.3 破解文件下载
使用方式，下载下面的SQLScout.jar和SQLScout_console_part.jar，替换~/Library/Application Support/AndroidStudio../SQLScout/lib/SQLScout.jar文件，重启AndroidStudio即可。

```
SQLScout 2.0.8:
支持Android Studio 2.3
下载：
https://github.com/wangjiegulu/wangjiegulu.github.com/tree/master/file/SQLScout/2.0.8
SQLScout 2.0.7:
支持Android Studio 2.2
下载：
https://github.com/wangjiegulu/wangjiegulu.github.com/tree/master/file/SQLScout/2.0.7
```

