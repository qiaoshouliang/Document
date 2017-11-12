
# VFS简介

虚拟文件系统（VFS）是一个Intellij Platform组件，它封装了大部分对活动文件的处理操作，为了达成以下目的：
- 提供一个处理文件的通用API，而不关心文件的具体位置（无论文件位于磁盘上、归档文件中还是HTTP服务器上）
- 追踪文件变化，并且在检测到文件内容发生更改时能提供新旧两个版本的文件
- 建立文件在VFS和持久化存储之间的关联

为了达到后两个目的，VFS为用户磁盘上的内容管理了一个持久化的快照，快照只存储那些通过VFS API访问过的和被异步更新后发生变化了的文件。快照是应用程序级别而不是工程级别的，也就是说，如果一个文件被多个项目引用，它的内容只会在快照中存在一个副本。所有的VFS访问操作都要经过快照。

如果通过VFS API访问文件时该文件不存在于快照中，VFS将从磁盘中加载该文件并存入快照，然后从快照返回数据。快照中只存储那些可被直接访问的信息，例如一个文本文件的内容会被全部存入快照，而另一些不可被直接访问的内容例如jar文件等，VFS只会存储它的元数据，例如文件名、文件尺寸和时间戳等。快照这一特性意味着IDE中显示的文件系统和文件内容可能并不总是能匹配磁盘上的实际内容。在IDE中，有一个观察者进程会从文件系统中接收到文件更改的通知并通知IDE进行刷新操作，刷新操作基于文件的时间戳，如果文件内容被更改而时间戳维持不变的话IDE将不会更新该文件的内容到快照。

# 获取VirtualFile

## 从IOFile获取

假设我们已经有一个位于磁盘上的io.java文件，要将其添加到VFS，可通过如下操作进行：
- 创建一个File实例File ioFile = new File("./io.java")
- 将文件从磁盘载入VFSVritualFile virtualFile = LocalFileSystem.getInstance().refreshAndFindFileByIoFile(ioFile)
- 刷新VFSvirtualFile.refresh(false, true)

其中refresh方法的两个参数分别为是否进行异步刷新和是否进行递归刷新。通过以上代码逻辑即可获得IOFile在VFS中的实例。

## 使用FileChooser获取

SDK中为我们提供了文件选择组件，通过简单的步骤可以直接获得一个VirtualFile实例：
- 创建一个FileChooserDescriptor实例，FileChooserDescriptor封装了一组对需要选择的目标文件的特征描述
- 通过`FileChooser.chooseFile()`获取VirtualFile实例

## 从URL获取

VirtualFileManager类提供了findFileByUrl()和refreshAndFindFileByUrl()方法让我们可以从URL获取VirtualFile实例

## 从VFS获取

IDE所有的文件访问都要通过VFS，上述几种方式最终都是将文件添加到VFS的快照之后再提供访问。当文件已经被快照管理时，可以通过FilenameIndex.getFilesByName()获取到VirtualFile实例。

# 写入内容到VirtualFile

正如Android不允许在UI线程中进行耗时操作一样，Intellij Platform也不允许在主线程中进行实时的文件写入，而需要通过一个异步任务来进行。

```
ApplicationManager.getApplication().invokeLater(new Runnable() { 
    @Override
    public void run() { 
        new WriteCommandAction(project) {
            @Override
            protected void run(@NotNull Result result) throws Throwable {
            //writing to file
            } 
        }.execute();
    }
});
```

上面是一个写入文件的示例，需要说明的是，WriteCommandAction也并不是实时写入的，如果有多个WriteCommandAction操作，可能会被合并到某一时间同时执行写操作，所以如果在一组WriteCommandAction中有对不同文件的写操作，而该文件在这组操作中是动态获取的，那么最好在写操作执行前先将WriteCommandAction与文件建立一个映射关系。

VFS就介绍到这里，下一篇文章将介绍如何操作工程中的目录、代码和其他资源文件

 

---

# PSI简介

PSI（Program Structure Interface）是Intellij Platform中一个非常重要的概念，在IDE所管理的Project中，每个目录，Package，源代码和资源文件都会被抽象成相应的PSI对象。本文将以PsiDirectory、PsiJavaFile和XmlFile为例介绍插件对文件目录、Java类和DOM对象的操作。


## 通用方法

```
FilenameIndex.getFilesByName()//通过给定名称（不包含具体路径）搜索对应文件
ReferencesSearch.search()//类似于IDE中的Find Usages操作
RefactoringFactory.createRename()//重命名
FileContentUtil.reparseFiles()//通过VirtualFile重建PSI
```
## Java专用方法
```
ClassInheritorsSearch.search()//搜索一个类的所有子类
JavaPsiFacade.findClass()//通过类名查找类
PsiShortNamesCache.getInstance().getClassesByName()//通过一个短名称（例如LogUtil）查找类
PsiClass.getSuperClass()//查找一个类的直接父类
JavaPsiFacade.getInstance().findPackage()//获取Java类所在的Package
OverridingMethodsSearch.search()//查找被特定方法重写的方法
```

## 使用PSI对象创建一个目录，并在目录中创建一个Java类。

创建目录

```
//获取Project根目录
PsiDirectory baseDir = PsiDirectoryFactory.getInstance(project).createDirectory(project.getBaseDir());
//递归查找要创建的目录
PsiDirectory subDir = baseDir.findSubdirectory(moduleName);
......
//判断要创建的目录是否已存在
boolean isExist = subDir == null;
//如不存在，创建新目录
if(!isExist) {
subDir = moduleDir.createSubdirectory(moduleName); 
}
//创建Java类

//创建PsiClass
PsiClass clazz = JavaDirectoryService.getInstance().createClass(subDir, className)
//添加package字段
((PsiJavaFile) clazz.getContainingFile()).setPackageName(pkgName);
//package字段可通过文件路径解析或使用JavaPsiFacade获得。此处添加字段的操作必须在WriteCommandAction中异步进行。
```

## 操作DOM对象

### 操作从外部读入的DOM

获取XmlFile实例
此处通过FileChooser引导用户从文件系统中选取一个xml文件

```
FileChooserDescriptor descriptor = new FileChooserDescriptor(true, false, false, false, false, false);
VirtualFile virtualFile = FileChooser.chooseFile(descriptor, project, null);
if (virtualFile != null) {
if (!virtualFile.isDirectory() && virtualFile.getName().endsWith("xml")) {
    xmlFile = (XmlFile) PsiManager.getInstance(project).findFile(virtualFile);
    }
}
//遍历所有深度为1的Tag
XmlDocument document = xml.getDocument();
if (document != null) {
    XmlTag rootTag = document.getRootTag();
    if (rootTag != null) {
        XmlTag[] subTags = rootTag.getSubTags();
        for (XmlTag tag : subTags) {
        //do something you want
        }
    }
}
```

### 生成XmlFile并写入文件


```
//创建XmlFile
XmlFile xmlFile = (XmlFile) PsiFileFactory.getInstance(project).createFileFromText(xmlName, StdFileTypes.XML);
//向XmlFile写入属性
XmlDocument document = xmlFile.getDocument();
if (document != null && document.getRootTag() != null) {
    XmlTag rootTag = document.getRootTag();
    rootTag.getAttribute(attrName).setValue(attrValue);//set value for exists attr.      
    rootTag.setAttribute(name,value);//add a new attr and setting value
}
//写入文件
//写入操作同样需要在WriteCommandAction中异步进行。先判断目标文件是否已存在，如果存在则执行删除
PsiFile psiFile = directory.findFile(xmlName);
if (psiFile != null) {
    psiFile.delete();
}
directory.add(file);
```
PSI对象操作就介绍到这里，下篇将介绍PSI的进阶用法

---

# PSI对象操作的进阶用法

## 使用模板

模板可以大大节省我们编写代码的工作量，在IDEA中，可以通过Preferences -> Editor -> File and Code Templates来查看和编辑已有的模板，或是添加新的模板。在模板中可以使用IDE提供的一些宏变量，常用的有NAME、PACKAGE_NAME、USER、DATE等，在新建模板时也可以使用自定义宏变量，自定义的宏变量将在创建文件时弹出一个输入框让用户对其赋值。

同样的，在插件开发中也可以使用文件模板，将公共部分代码进行抽离，减少创建PSI对象的复杂程度，下面将介绍模板的使用方法。

### 创建模板

在插件中模板文件必须存放于源文件目录的fileTemplates/internal目录内,模板文件必须以${模板唯一名称}.${对应代码文件的扩展名}.ft命名，例如我们要创建一个java类的模板，命名可能为myTemplate.java.ft，模板文件内也可以使用宏变量，下面我们将创建一个java类通用的模板，包含package字段和类注释。


```
#if (${PACKAGE_NAME} && ${PACKAGE_NAME} != "")package ${PACKAGE_NAME};#end
/**
* Author: ${USER}
* Created on ${DATE}
*/
public class ${NAME} {        
}
```

### 注册模板

在使用模板前必须先向插件系统注册改模板，编辑plugin.xml文件，在extensions节点加入模板注册信息。

```
<extensions defaultExtensionNs="com.intellij">
    <!-- Add your extensions here -->
    <internalFileTemplate name="myTemplates"/>
</extensions>
```

此处的模板名称必须是在IDE内唯一的名称，否则调用模板时会发生冲突。

### 使用模板

在创建JavaClass时可以调用该模板

```
PsiClass clazz = JavaDirectoryService.getInstance().createClass(directory, className, templateName);
```
以上就是模板使用的相关方法，so easy~


## 为PsiClass添加类导入、接口实现、成员变量和方法

生成PsiClass后我们可以为其添加一系列Java类的内容，下面介绍各种内容的添加方式，注意，所有的添加操作都必须异步进行。

### 添加类导入

要为PsiClass导入一个类，那我们必须搜索到这个类在哪，怎么搜索呢，分为两部曲:
- 创建SearchScope
SearchScope是个什么玩意呢，望文生义，它就是用来描述搜索范围的，在这里我们把搜索范围定义为整个Project，IDE会搜索Project内所有的类（包括引用的类库）来寻找目标。

```
GlobalSearchScope searchScope = GlobalSearchScope.allScope(project);
```

- 创建好SearchScope，使用它进行搜索

```
PsiClass[] psiClasses = PsiShortNamesCache.getInstance(project).getClassesByName(className, searchScope);
```

搜索完毕之后会返回一个PsiClass数组，包含所有类名匹配的PsiClass，接下来可以过滤出目标类并生成import，同样是两步:
- 生成PsiImportStatement
```
PsiImportStatement importStatement = psiElementFactory.createImportStatement(psiClass);
```
- 添加到PsiClass
```
((PsiJavaFile) clazz.getContainingFile()).getImportList().add(importStatement)
```
齐活儿~

### 添加接口实现

添加接口实现同样需要先搜索接口类，搜索的套路还是和上面类导入的套路一样一样的，我们直接从生成接口字段PsiJavaCodeReferenceElement开始
- 生成PsiJavaCodeReferenceElement
```
PsiJavaCodeReferenceElement ref = psiElementFactory.createClassReferenceElement(psiClasses);
```
- 添加到PsiClass

```
clazz.getImplementsList().add(ref);
```
需要一提的是添加接口实现声明之后接口类会被自动导入，是不是很Nice~

### 添加成员变量

有两个方式为PsiClass添加成员变量
- 从字符串生成

```
PsiElementFactory factory = JavaPsiFacade.getInstance(project).getElementFactory();
PsiField field = factory.createFieldFromText("public int a = 0;", psiClass);
```
使用此种方式生成成员必须在字符串中显示指定成员的类型和修饰符，并且需要自行保证无语法错误。
- 通过Factory生成


```
PsiElementFactory factory = JavaPsiFacade.getInstance(project).getElementFactory();
PsiField field = factory.createField("a",PsiType.INT);
field.getModifierList().setModifierProperty(PsiModifier.PUBLIC,true);
```

明显通过此种方式生成的Field更能保证代码质量，也更符合我们对代码进行面向对象操作的理念。

### 添加方法

Method同样有通过Text和通过Factory生成两种方式。
- 通过字符串生成
```
PsiElementFactory factory = JavaPsiFacade.getInstance(project).getElementFactory();
PsiMethod method = factory.createMethodFromText("public String toString(){}",psiClass);
```
- 通过Factory生成
```
PsiElementFactory factory = JavaPsiFacade.getInstance(project).getElementFactory();
PsiMethod method = factory.createField("getCount",PsiType.INT);
```
- 为Method添加注解
```
method.getModifierList().addAnnotation("Override")
```
同样的，可以为PsiMethod添加修饰符和方法体，添加方式和上文添加成员变量相同。

格式化代码
```
CodeStyleManager.getInstance(project).reformat(psiClass);
```
在编辑器中打开
```
FileEditorManager.getInstance(project).openTextEditor(new OpenFileDescriptor(project, virtualFile), true);
```
注册自定义类型文件

在编写处理自定义语言或IDE未支持的文件类型时，需要自行注册文件类型到IDE，此处通过一个简单的例子来阐述。
SVG是基于XML，用于描述二维矢量图形的一种图形格式，在IDE中我们可以使用解析标准DOM的方式对其进行解析，通过两个简单步骤即可实现：
1. 添加自定义FileTypeFactory
此处需要新建一个继承了FileTypeFactory的类

```
public class SVGFileTypeFactory extends FileTypeFactory {
@Override
public void createFileTypes(FileTypeConsumer fileTypeConsumer) {
    fileTypeConsumer.consume(XmlFileType.INSTANCE,"svg");
    }
}
```

2.注册FileTypeFactory
编辑plugin.xml文件，在extensions节点加入

```
<extensions defaultExtensionNs="com.intellij">
<fileTypeFactory implementation="com.guide.plugin.SVGFileTypeFactory"/>
</extensions>
```

完成以上两个步骤之后就能在IDE中使用XmlFile来描述SVG文件。

Intellij IDEA 插件开发系列到这里就结束了，虽然这样属于黑科技的文章想必也不会有啥人看。

# 语言结构化支持的角度探究插件开发的相关技巧
这一个系列的前面四篇文章对IntelliJIDEA插件开发的一些基础和通用的方法做了介绍，本篇将会深入一步，从语言结构化支持的角度探究插件开发的相关技巧。
依据Wiki的介绍，编程语言是用来定义计算机程序的形式语言。它是一种被标准化的交流技巧，用来向计算机发出指令。编程语言的描述一般可以分为语法及语义。语法是说明编程语言中，哪些符号或文字的组合方式是正确的，语义则是对于编程的解释。

对语言中语法和语义的分析，就是编译原理里词法分析和语法分析的概念。本篇文章仅会简略介绍针对于完全未知的语言类型的支持，而将更多的重点放在对已有语言类型的继承和扩展支持上。
针对于插件开发者，推荐一个十分有用的插件PsiViewer，可以方便地在IDE内查看编辑器内的内容所对应的PsiElement类型，省去了很多查找文档的操作。

支持未知类型的语言

定义基础语言描述

IDE之所以能仅通过扩展名就能识别出源文件所使用的语言，就是因为IDE内定义的Language和FileType为其提供了支持。

定义Language

简单继承Language类，在构造函数中传入语言ID和其他参数即可。

定义文件类型

继承LanguageFileType类，在实现的方法中分别返回源文件的名称、描述、默认扩展名和Icon即可。

定义FlieTypeFactory

继承FileTypeFactory类，在createFileTypes方法中注册语言，并在plugin.xml的extensions节点中通过fileTypeFactory标签注册Factory。

完成以上几步后IDE便能通过文件扩展名识别出源文件并为其应用图标。

定于词法和语法规范

定义Token和Element

此处定义的Token和Element分别对应于后续词法分析中的Token和IDE内部表示的PsiElement，继承IElementType，完成定义。

定义语法

IDEA支持通过BNF范式来定义语法，按照规则编写一个bnf文件，当语法被定义完成后，可以通过IDE生成PsiElement定义和PSI Parser。

定义Lexer

词法分析器定义了一个文件的内容是如何被分解为Token的，创建Lexer的一个简单的方式是使用JFlex。定义一个lexer规则文件然后通过IDE即可生成Lexer。
实现ParserDefinition接口，创建一个Parser定义，并在plugin.xml中的extensions节点中使用lang.parserDefinition标签注册该parser。

以上便是为语言做最基础支持的步骤，完成文件类型和语法、词法分析器注册后，IDE便能为自定义语言提供文件识别、关键字检查和语法检查等支持。下面结合对weex-language-support插件的开发过程，介绍对扩展了html语法的weex脚本所做的语法高亮、自动提示、Lint等其它一系列的支持的实现。

weex语言支持

识别weex脚本

由于weex脚本文件的结构和html类似，均包括<script>、<style>标签和模板部分，所以我们的Language选择直接继承HTMLLanguage


```
public class WeexFileType extends LanguageFileType {
    public static final WeexFileType INSTANCE = new WeexFileType();
    private WeexFileType() {
        super(HTMLLanguage.INSTANCE);
    }
//…………
}
定义完FileType之后，还需要实现一个对应的FileTypeFactory：

public class WeexFileTypeFactory extends FileTypeFactory {
    @Override
    public void createFileTypes(@NotNull FileTypeConsumer fileTypeConsumer) {
        fileTypeConsumer.consume(WeexFileType.INSTANCE);
    }
}
```

最后在plugin.xml内注册


```
<extensions defaultExtensionNs="com.intellij">
<fileTypeFactory implementation="com.taobao.weex.WeexFileTypeFactory"/>
</extensions>
```

完成这一步之后，IDE便能正确将以we作为扩展名的文件识别为weex文件，因为继承了HTMLLanguage，html语言原有的js和css的自动补全和lint规则也将被一并继承过来。

支持Weex DSL

weex定义了一系列的DSL规则，包括自定义标签、自定义标签属性和数据绑定支持等，接下来我们一步步对这些特性进行支持

自定义标签支持

要让IDE能正确识别自定义的标签（如<list>、<slider>等）并对其应用正确的补全和lint规则，我们需要实现自定义的XmlTagNameProvider和XmlElementDescriptorProvider，其中XmlTagNameProvider定义了所能提供的标签的列表，XmlElementDescriptorProvider定义了每个标签的详情。
定义一个WeexTagNameProvider，实现XmlTagNameProvider和XmlElementDescriptorProvider接口：

```
XmlTagNameProvider

    @Override
public void addTagNameVariants(List<LookupElement> list, @NotNull XmlTag xmlTag, String prefix) {
if (!(xmlTag instanceof HtmlTag)) {
return;
}
Set<String> tags = getWeexTagNames();
for (String s : tags) {
LookupElement element = LookupElementBuilder
.create(s)
.withInsertHandler(XmlTagInsertHandler.INSTANCE)
.withBoldness(true)
.withIcon(WeexIcons.TYPE)
.withTypeText("weex component");
list.add(element);
}
```

如此在输入标签的时候IDE就能将weex标签加入自动补全列表中。

```
XmlElementDescriptorProvider

    @Nullable
@Override
public XmlElementDescriptor getDescriptor(XmlTag xmlTag) {
PsiFile declare = getDeclare(xmlTag);
return new WeexTagDescriptor(xmlTag.getName(), declare);
}
```

此处的关键是如何将XmlTag与它的声明文件即PsiFile关联起来，此处的桥梁就是XmlElementDescriptor。创建一个实现了XmlElementDescriptor的类，在getDeclaration()方法中返回PsiFile对象，如此便能在执行Open Declaeation的时候跳转到该tag的声明文件内。
在某些时候，我们的声明文件可能并不是位于某个确定位置的目录内，而是处于归档文件中，此处演示如何从一个jar归档中获取PsiFile的引用：

```
public void findPsiFileFromJar(String jarPath, String targetName) {
VirtualFile vf = JarFileSystem.getInstance().findLocalVirtualFileByPath(jarPath);
VirtualFile target = vf.findChild(targetName);
PsiFile declare = PsiManager.getInstance(ProjectUtil.guessProjectForFile(target)).findFile(target);
}
```

最后，为了应用上述实现的功能，需要在plugin.xml中进行注册：


```
<extensions defaultExtensionNs="com.intellij">
<xml.tagNameProvider implementation="com.taobao.weex.tags.WeexTagNameProvider"/>
<xml.elementDescriptorProvider implementation="com.taobao.weex.tags.WeexTagNameProvider"/>
</extensions>
```

自定义标签属性支持

按照上面的套路，大家肯定能想到标签属性也是有相应的Provider支持的，它就是XmlAttributeDescriptorsProvider，创建相应的实现类，实现以下方法：

getAttributeDescriptors(XmlTag xmlTag)

该方法需要返回XmlTag对应的所有属性的Descriptor列表

getAttributeDescriptor(String s, XmlTag xmlTag)

该方法返回名为s的XmlTag所对应的单个Descriptor

此处的Descriptor为XmlAttributeDescriptor实例，描述了每个标签属性的信息，创建实现类，实现以下关键方法：

getName()

返回要匹配的属性名称

isEnumerated()

判断属性值是否是枚举属性

getEnumeratedValues()

如果属性值是可枚举的，此处返回所有可能的属性值

同样地，最后需要在plugin.xml中进行注册：


```
<extensions defaultExtensionNs="com.intellij">
<xml.attributeDescriptorsProvider implementation="com.taobao.weex.attributes.WeexAttrDescriptorProvider"/>
</extensions>
```

自动补全支持

事实上上文所述的标签支持和标签属性支持已经为IDE提供了部分补全所需的支持，例如在输入标签名称和标签属性名时，IDE已经能根据我们提供的Provider和Descriptor提供一些上下文无关的补全提示，接下来我们来提供与上下文相关的补全支持。
Weex使用双大括号的方式来绑定script中声明的变量，所以我们需要先对script进行分析，得到其中声明的所有变量的函数。此处暂且不对分析JavaScript语言结构进行深入解读，后续有时间单独作文。
需要补充的一点是，为了让插件支持html语法，需要在plugin.xml中声明对语言模块的依赖：

```
<depends>com.intellij.modules.xml</depends>
<depends>com.intellij.modules.lang</depends>
<depends>JavaScript</depends>
```

并不是所有IDE都包含JavaScript模块，本插件开发过程中选择IDEA 15作为SDK，并手动将/Applications/IntelliJ IDEA 15.app/Contents/plugins/JavaScriptLanguage目录下的jar文件添加到Project Structure的Classpath中。
言归正传，下面开始介绍如何对标签属性值进行自动补全支持。事实上很简单：一个关键的类CompletionContributor
创建一个继承了CompletionContributor的类，在构造函数中调用extend()方法添加补全的相关逻辑：
extend(@Nullable CompletionType type, @NotNull ElementPattern<? extends PsiElement> place, CompletionProvider provider)

CompleteType
定义补全类型，此处传入CompletionType.BASIC即可。
ElementPattern
定义补全操作需要匹配的目标Element类型，简单来说就是指定在何处该出发自动补全，此处可传入PlatformPatterns.psiElement(XmlAttributeValue.class)，表示在键入标签属性值时触发补全逻辑。
CompletionProvider
补全逻辑真正的Provider。

创建我们自己的CompletionProvider，实现addCompletions方法：


```
protected void addCompletions(@NotNull CompletionParameters completionParameters, ProcessingContext processingContext, @NotNull CompletionResultSet resultSet) {
if (completionParameters.getPosition().getContext() instanceof XmlAttributeValue) {
final XmlAttributeValue value = (XmlAttributeValue) completionParameters.getPosition().getContext();
for (String s : getCandidateValues(value)) {
resultSet.addElement(LookupElementBuilder.create("{{" + s + "}}")
.withLookupString(s) //匹配查找的字符串
.withIcon(WeexIcons.TYPE) //候选列表中显示的icon
.withInsertHandler(new InsertHandler<LookupElement>() {
@Override
public void handleInsert(InsertionContext insertionContext, LookupElement lookupElement) {
performInsert(value, insertionContext, lookupElement);
}
}) // 插入补全内容时的Handler
.withBoldness(true) //加粗
}
}
}
```

如此就可以在输入标签属性值的时候触发补全逻辑，此处的难点在于如何从js中分析出函数和变量名，并分析出变量的类型和标签属性值的类型是否匹配，处于篇幅限制并未详细解释，如果想了解可以查看weex-language-support的源码或向我咨询。
最后，在plugin.xml中进行注册：

```
<extensions defaultExtensionNs="com.intellij">
<completion.contributor language="HTML" implementationClass="com.taobao.weex.complection.WeexCompletionContributor"/>
</extensions>
```

CompletionContributor不能支持情况下的补全

在编写weex插件的过程中，发现CompletionContributor并不能在键入标签内容时触发补全（即<text>{{value}}</text>的情况），为了支持该种情况下的补全，我们想到了一个tricky的办法：Intention
IDEA可以帮助你处理一些代码中可能导致错误的情况，当分析出可能存在问题时，IDEA为提供一个Intention列表，包含一些建议实施的修复或改进的行为。在OSX上，默认可以通过Command + Enter触发Intention。
创建一个PsiElementBaseIntentionAction，实现以下关键方法：
isAvailable(@NotNull Project project, Editor editor, @NotNull PsiElement element)
判断当前活动的Element（光标所在Element）处是否可以激活Intention，其中project、editor参数为当前的Project和当前活动的编辑器对象。
invoke(@NotNull final Project project, final Editor editor, @NotNull PsiElement element)
用户触发Intention后需要需要执行的逻辑

在此处我们依旧获取了js中导出的所有变量，并创建了一个多级下拉框来供用户选择需要插入的变量值。顺带稍微介绍一下IDEA内置的一系列魔改Swing控件，这些控件大部分以JB开头，例如JBTable，JBColor等，编写插件时使用这些控件能保持与IDE一致的外观风格，让插件没有违和感。此处我们使用ListPopup控件来实现选择列表，通过JBPopupFactory.getInstance().createListPopup可以获得相应的ListPopup实例，并且可以调用ListPopop#showInBestPositionFor方法让ListPopup显示在屏幕上最恰当的位置（光标附近）。
还是一样的步骤，往plugin.xml中注册Intention：


```
<extensions defaultExtensionNs="com.intellij">
<intentionAction>
<className>com.taobao.weex.intention.TagTextIntention</className>
<category>Weex</category>
<descriptionDirectoryName>WeexTagTextCompletion</descriptionDirectoryName>
</intentionAction>
</extensions>
```

Lint

Lint是IDE所必须具备的另一项能力，在代码编写阶段，通过分析上下文，找出可能存在的错误或可改进的写法，对用户进行提示，在IDE中，我们熟悉的红色错误下划线，黄色警告下划线都是由Annotator来实现的。创建Annotator的实例，实现annotate方法

```
annotate(@NotNull PsiElement psiElement, @NotNull AnnotationHolder annotationHolder)
```
其中psiElement是等待进行检查的元素

创建Annotation

Annotation分为Error、Warning、WeakWarning、Info几个级别，对应编辑器中呈现出的各种不同的下划线颜色。使用AnnotationHolder#createXXXAnnotation方法可以创建一个Annotation实例。

附加QuickFix

如上文所述，IDE可以提供一个Intention列表来提供一些建议执行的修复或优化行为，在Annotation中可以通过registerFix()方法来附加一个修复动作。

Find Usages

终于快要写完了……
查找某个变量在工程中的引用是对这个变量执行重构的前提，插件中通过PsiReferenceContributor和PsiReference为引用分析提供支持。
创建一个继承了PsiReferenceContributor的实例，在registerReferenceProviders方法中处理引用查找。

```
@Override
public void registerReferenceProviders(@NotNull PsiReferenceRegistrar psiReferenceRegistrar) {
psiReferenceRegistrar.registerReferenceProvider(
XmlPatterns.xmlTag().withLanguage(HTMLLanguage.INSTANCE)
.andNot(XmlPatterns.xmlTag().withLocalName("script")) //仅在<template>标签内查找
.andNot(XmlPatterns.xmlTag().withLocalName("style")),
new PsiReferenceProvider() {
@NotNull
@Override
public PsiReference[] getReferencesByElement(@NotNull PsiElement psiElement, @NotNull ProcessingContext processingContext) {
if (psiElement instanceof XmlTag) {
if (((XmlTag) psiElement).getSubTags().length == 0) {
String text = ((XmlTag) psiElement).getValue().getText();
if (WeexFileUtil.containsMustacheValue(text)) {
List<PsiReference> references = new ArrayList<PsiReference>();
Map<String, TextRange> vars = WeexFileUtil.getVars(text);
for (Map.Entry<String, TextRange> entry : vars.entrySet()) {
if (WeexFileUtil.getAllVarNames(psiElement).keySet().contains(entry.getKey())) {
references.add(new DataReference((XmlTag) psiElement, entry.getValue(), entry.getKey()));
}
}
return references.toArray(new PsiReference[references.size()]);
}
}
}
return new PsiReference[0];
}
}
);
}
```

最后在plugin.xml中注册：


```
<extensions defaultExtensionNs="com.intellij">
<psi.referenceContributor implementation="com.taobao.weex.refrences.WeexReferenceContributor"/>
</extensions>
```
