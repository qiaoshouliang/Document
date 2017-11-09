
### 先举个例子，我们在 Android 开发中输入 Toast ，然后会有如下如下的快速操作：
![](http://static.oschina.net/uploads/img/201608/23075711_o0It.gif)

### 现在我们创建一个自定义的Live Template
设置 -> Editor -> Live Templates ，点击右上角的 + 号，选择 Template Group ，因为我习惯自定义的单独分组先，这样好管理，比如新建一个分组叫做 stormzhang ，然后就会看到有一个 stormzhang 的分组显示在了列表里，这时候鼠标选中该分组，然后再点击右上角的 + 号，点击 Live Template ，然后如下图填写缩写与描述，紧接着把如下代码拷贝到下面的输入框里（PS：单例模式的写法有很多种，这里就随意以其中一种为例）

```
private static $CLASS$ instance = null;
 private $CLASS$(){}
 public static $CLASS$ getInstance() {
    synchronized ($CLASS$.class) {
        if (instance == null) {
            instance = new $CLASS$();
        }
    }

    return instance;
     
 }
```
注意这里，如果你这段代码是一些固定的代码，那么至此就结束了，但是这段代码里是动态的，里面有一些变量，因为每个类的类名如果都需要自己手动更改就太麻烦了，所以有个变量 $CLASS$ ，所以需要点击下面的 Define ，先要定义变量所属的语言范围，点开之后可以看到这里支持 HTML、XML、JSON、Java、C++ 等，很明显，我们这里需要支持 Java ，选择选中 Java :

![image](http://static.oschina.net/uploads/img/201608/23075718_XwWV.png)
紧接着，我们需要给变量 $CLASS$ 定义类型，这里的 CLASS 名字随意取的，为了可读性而已，你高兴可以取名 abc ，真正给这个变量定义类型的是点击 Edit variables 按钮，来对该变量进行编辑，我们选择 calssName() 选项，可以看到还有其他选项，但是看名字大家大概就猜到什么含义了，这里就不一一解释了。

![image](http://static.oschina.net/uploads/img/201608/23075719_EyUD.png)

点击 ok 保存，至此我们定义的一个单例的 Live Template 就完成了。你可以随意打开一个类文件，然后输入 singleton 按 tab 或者 enter 键就可以看到神奇的一幕出现了，是不是很帅？