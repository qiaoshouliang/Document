#原理 

java.util.ServiceLoader这个类主要是从META-INF/services这个目录下的配置文件加载给定接口或者基类的实现，ServiceLoader会根据给定的类的full name来在META-INF/services下面找对应的文件，在这个文件中定义了所有这个类的子类或者接口的实现类，返回一个实例。

# 例子

下面以一个具体的例子来说明一下ServiceLoader的具体使用，类似Hadoop FileSystem中的实现。

首先定义一个接口，具体如下：

```java
public interface IService {  
    public String sayHello();  
      
    public String getScheme();  
}  

```

该接口有两个子类，分别为HDFSService和LocalService：

```java
 public class HDFSService implements IService {  
   
     @Override  
     public String sayHello() {  
         return "Hello HDFS!!";  
     }  
   
     @Override  
     public String getScheme() {  
         return "hdfs";  
     }  
 }  

```

```java
public class LocalService implements IService {  
  
    @Override  
    public String sayHello() {  
        return "Hello Local!!";  
    }  
  
    @Override  
    public String getScheme() {  
         return "local";  
     }  
   
 }  

```

需要在META-INF/services下以IService这个类的全名来新建立一个文件，文件中的内容为两个实现类的全名，如下：

```java
org.hadoop.java.HDFSService  
org.hadoop.java.LocalService  
```

所有的实现和配置都已经完成，下面写一个测试类来看一下结果:

```Java
 public class ServiceLoaderTest {  
   
     /** 
      * @param args 
      */  
     public static void main(String[] args) {  
         //need to define related class full name in /META-INF/services/....  
         ServiceLoader<IService> serviceLoader = ServiceLoader  
                 .load(IService.class);  
         for (IService service : serviceLoader) {  
             System.out.println(service.getScheme()+"="+service.sayHello());  
         }  
     }  
   
 }  

```

具体的输出来如下：

```java
hdfs=Hello HDFS!!  
local=Hello Local!!  
```

# Android中SPI的使用

在Android中怎么注册META-INF/services中呢？

- 首先，在java的同级目录中new一个包目录resources，然后在resources新建一个目录META-INF/services，再新建一个file，file的名称就是接口的全限定名。
- 然后，在这个file中添加该接口的实现类。

怎样通过代码将实现类动态写入到META-INF/services中呢？

该项功能在google的auto service（[auto](https://github.com/google/auto)）中使用过，那我么来分析一下是怎么实现的。

- 通过Filer获取META-INF/services下的文件	
  ```java
  String resourceFile = "META-INF/services/" + providerInterface;
  FileObject existingFile = filer.getResource(StandardLocation.CLASS_OUTPUT, "",
        resourceFile);
  ```

- 将文件中的内容读到**oldServices**中,

  ```java
  Set<String> oldServices = ServicesFiles.readServiceFile(existingFile.openInputStream());
  --------------------------------------------------------------------------------------
  //ServicesFiles.java
  static Set<String> readServiceFile(InputStream input) throws IOException {
      HashSet<String> serviceClasses = new HashSet<String>();
      Closer closer = Closer.create();
      try {
        // TODO(gak): use CharStreams
        BufferedReader r = closer.register(new BufferedReader(new InputStreamReader(input, UTF_8)));
        String line;
        while ((line = r.readLine()) != null) {
          int commentStart = line.indexOf('#');
          if (commentStart >= 0) {
            line = line.substring(0, commentStart);
          }
          line = line.trim();
          if (!line.isEmpty()) {
            serviceClasses.add(line);
          }
        }
        return serviceClasses;
      } catch (Throwable t) {
        throw closer.rethrow(t);
      } finally {
        closer.close();
      }
    }
  ```

- 将要写入的service和读出的service放到一个列表中，一起写入到`META-INF/services/**`

  ```java
  SortedSet<String> allServices = Sets.newTreeSet();
  allServices.addAll(oldServices);//从META-INF/services/**读出的列表
  Set<String> newServices = new HashSet<String>(providers.get(providerInterface));
  allServices.addAll(newServices);//需要写入的新的列表
  FileObject fileObject = filer.createResource(StandardLocation.CLASS_OUTPUT, "",
                          resourceFile);//重新创建从META-INF/services/**，覆盖原来的
  OutputStream out = fileObject.openOutputStream();
  ServicesFiles.writeServiceFile(allServices, out);//写入到META-INF/services/**
  out.close();
  --------------------------------------------------------------------------------------
  //ServicesFiles.java
  static void writeServiceFile(Collection<String> services, OutputStream output)
        throws IOException {
      BufferedWriter writer = new BufferedWriter(new OutputStreamWriter(output, UTF_8));
      for (String service : services) {
        writer.write(service);
        writer.newLine();
      }
      writer.flush();
    }
  ```

  ​

# 总结

可以看到ServiceLoader可以根据IService把定义的两个实现类找出来，返回一个ServiceLoader的实现，而ServiceLoader实现了Iterable接口，所以可以通过ServiceLoader来遍历所有在配置文件中定义的类的实例。