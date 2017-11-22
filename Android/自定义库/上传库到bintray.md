
 # 1.首先创建Library
![image](https://ws1.sinaimg.cn/large/006tNc79ly1fgv9i8lo4wj30u00jognd.jpg)

# 在Library目录下创建bintray.gradle文件
![image](https://ws3.sinaimg.cn/large/006tNc79ly1fgv9jzutroj30b70dzjs7.jpg)
## 文件内容如下
```java
apply plugin: 'com.github.dcendents.android-maven'
apply plugin: 'com.jfrog.bintray'

version = "0.1.2" // 修改为你的版本号

def siteUrl = 'https://github.com/h3clikejava/ExtendImageView' // 修改为你的项目的主页
def gitUrl = 'https://github.com/h3clikejava/ExtendImageView.git' // 修改为你的Git仓库的url

group = "com.qiaoshouliang" // Maven Group ID for the artifact，一般填你唯一的包名

install {
    repositories.mavenInstaller {
        // This generates POM.xml with proper parameters
        pom {
            project {
                packaging 'aar'
                // Add your description here
                name 'test' //项目描述
                url siteUrl
                // Set your license
                licenses {
                    license {
                        name 'The Apache Software License, Version 2.0'
                        url 'http://www.apache.org/licenses/LICENSE-2.0.txt'
                    }
                }
                developers {
                    developer {
                        id 'qiaoshouliang' //填写的一些基本信息
                        name 'qiaoshouliang'
                        email 'qiaoshouliang@126.com'
                    }
                }
                scm {
                    connection gitUrl
                    developerConnection gitUrl
                    url siteUrl
                }
            }
        }
    }
}
task sourcesJar(type: Jar) {
    from android.sourceSets.main.java.srcDirs
    classifier = 'sources'
}
task javadoc(type: Javadoc) {
    source = android.sourceSets.main.java.srcDirs
    classpath += project.files(android.getBootClasspath().join(File.pathSeparator))
}
task javadocJar(type: Jar, dependsOn: javadoc) {
    classifier = 'javadoc'
    from javadoc.destinationDir
}
artifacts {
    archives javadocJar
    archives sourcesJar
}
Properties properties = new Properties()
properties.load(project.rootProject.file('local.properties').newDataInputStream())
bintray {
    user = properties.getProperty("bintray.user")
    key = properties.getProperty("bintray.apikey")
    configurations = ['archives']
    pkg {
        repo = "maven"
        name = "widget" //发布到JCenter上的项目名字
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
}
```

 # 2. 在项目根目录build.gradle下加入


```java
    dependencies {
    
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath 'com.jfrog.bintray.gradle:gradle-bintray-plugin:1.4'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.4.1'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }

```
如图 
![image](https://ws3.sinaimg.cn/large/006tNc79ly1fgvageflmlj30t60dnmys.jpg)

# 3. 在local.properties中加入如下两行，用来配置bintray的用户名的APIKey
```java
bintray.user=josenqiao
bintray.apikey=f1ccb3d650a6b39aeab4d4a0e1ad97f867e52f39
```
登录Bintray来查看自己的APIKey，在Edit Profile中进行查看

![](https://ws3.sinaimg.cn/large/006tKfTcly1fgygajthfoj30cr0dct9m.jpg)
![](https://ws2.sinaimg.cn/large/006tKfTcly1fgygczz0skj30tb0ax75f.jpg)

---

# 4.在Bintray中创建maven库
![](https://ws2.sinaimg.cn/large/006tKfTcly1fgygwg2llij30go0jc75j.jpg)
Name需要和Pkg中的repo相同

```java
pkg {
        repo = "maven"
        name = "widget" //发布到JCenter上的项目名字
        websiteUrl = siteUrl
        vcsUrl = gitUrl
        licenses = ["Apache-2.0"]
        publish = true
    }
```
# 5.配置成功之后在项目的根目录下使用命令
./gradlew clean build bintrayUpload

# 6. 需要注意的问题
## 6.1 注册Bintray的时候需要选择For an Open Source Account Sign Up Here

![](https://ws1.sinaimg.cn/large/006tKfTcly1fgygf4wcuzj30q40en421.jpg)
==否则上传的时候无法找到你对应的 repo = 'maven'==
![](https://ws4.sinaimg.cn/large/006tKfTcly1fgygot19hjj30rn0auwfy.jpg)
如果是从Start YOUR FREE TRIAL注册的用户，Edit Porfile中没有Repositories这个配置项。

![](https://ws2.sinaimg.cn/large/006tKfTcly1fgygi0hpe2j30pd0dzjsj.jpg)












