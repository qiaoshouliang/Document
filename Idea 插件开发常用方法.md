
## 创建文件时用到的方法

### 获取常用工具类

```
myFactory = JavaPsiFacade.getElementFactory(mProject);
```
### 获取鼠标选中的目录
通过AnActionEvent获取到Ideview,然后调用`getOrChooseDiretory()` 获取鼠标右击选中的目录
```
IdeView ideView = anActionEvent.getRequiredData(LangDataKeys.IDE_VIEW);
PsiDirectory directory = ideView.getOrChooseDirectory();
```
### 创建Java类
通过DirectoryService创建Java类

```
myDirectoryService = JavaDirectoryService.getInstance();
PsiClass psiClass = myDirectoryService.createClass(directory, "Text", JavaTemplateUtil.INTERNAL_CLASS_TEMPLATE_NAME);
```
### 设置包名
```
PsiJavaFile javaFile = (PsiJavaFile) psiClass.getContainingFile();
PsiPackage psiPackage = myDirectoryService.getPackage(directory);
javaFile.setPackageName(psiPackage.getQualifiedName());
```
### 设置类的权限

```
psiClass.getModifierList().setModifierProperty(PsiModifier.PUBLIC,true);
```

### psiClass类中添加接口

```
PsiClass view = myFactory.createInterface("View");
psiClass.add(view);
```

### 根据名字全局查找PsiClass
```
private PsiClass getPsiClassByName(String name) {

        PsiClass[] psiClasses = myShortNamesCache.getClassesByName(name, myProjectScope);//NotNull
        PsiClass psiClass = null;
        if (psiClasses.length != 0) {//if the class already exist.
            psiClass = psiClasses[0];
        }//and
        return psiClass;
    }
```

### 根据PsiFile查找PsiClass
```
  if ((psiFile1 instanceof PsiJavaFile) && ((PsiJavaFile) psiFile1).getClasses().length > 0) {
                psiClass = ((PsiJavaFile) psiFile1).getClasses()[0];
            }
```

### 手动设置Action的名字和图标
```
Presentation presentation = getTemplatePresentation();
presentation.setText(fileType);
presentation.setIcon(IconLoader.getIcon("/icons/icon_tf.png"));
```
其中icons要放到Resource目录下：

![](https://ws3.sinaimg.cn/large/006tNc79ly1fhx7vo19o8j30bi079jrp.jpg)

注意图片的命名规则

### 在ActionGroup中手动添加Acton

```
public class AddMVPFile extends DefaultActionGroup implements DumbAware {

    public AddMVPFile() {
        setPopup(true);
        Presentation presentation = getTemplatePresentation();
        presentation.setText("MVPFile");
        presentation.setIcon(IconLoader.getIcon("/icons/icon_tf.png"));


        List<String> fileTypes = new ArrayList<>();
        fileTypes.add("Contract");
        fileTypes.add("PresenterImpl");
        fileTypes.add("ModelImpl");

        for (String fileType:fileTypes){
            add(new AddFile(fileType));
        }
    }
}
```
其中AddFile是Acton,要实现如下所示的效果，需要加入 `setPopup(true);`否则Action是平铺开来的，没办法放到MVPFile下

![](https://ws3.sinaimg.cn/large/006tNc79ly1fhx81xtwdgj30lh0dcmzc.jpg)

### 通过复写Action 的 update 来控制Action是否可见
```
@Override
    public void update(AnActionEvent e) {
        super.update(e);
        IdeView ideView = e.getRequiredData(LangDataKeys.IDE_VIEW);
        PsiDirectory directory = ideView.getOrChooseDirectory();
        if (directory.getName().equals("contract"))
            e.getPresentation().setEnabledAndVisible(true);
        else
            e.getPresentation().setEnabledAndVisible(false);

    }
```
`e.getPresentation().setEnabled(true);`用来设置该Action是否可用,
`e.getPresentation().setEnabledAndVisible(true);`用来设置该Action可用并且可见,可以灵活选用

### 显示错误信息
```
Messages.showErrorDialog("Generation failed, " +
                            "your class name MUST END WITH 'Contract' or 'Presenter'.",
                    "Class Name Error");
```

### Dialog 设置
```
//设置Dialog的标题
setTitle("New Mvp File");

//设置Dialog的最小大小
setMinimumSize(new Dimension(260, 120));

//设置Dialog在屏幕中间，public void setLocationRelativeTo(Component c)设置窗口相对于指定组件的位置。 
//如果组件当前未显示，或者 c 为 null，则此窗口将置于屏幕的中央。
setLocationRelativeTo(null);
```
`setLocationRelativeTo（null）`可以使其屏幕居中，但如果IDE不全屏，显示的效果就不好看了，想使Dialog在IDE窗口居中显示，可以这样设置：

`setLocationRelativeTo(WindowManager.getInstance().getFrame(actionEvent.getProject())`


注意setMinimumSize和setLocationRelativeTo的先后位置，如果setLocationRelativeTo在前，则创建出来的窗口的左上角居中，因为这时窗口还没有大小。


### 导入需要的类

要想使用如下的方法导入import，前提条件是需要导入的类必须包含包名，例如`Log.e`需要写成`android.util.Log.e(TAG,field.toString());`：

```
JavaCodeStyleManager styleManager = JavaCodeStyleManager.getInstance(project);
styleManager.optimizeImports(file);
styleManager.shortenClassReferences(targetClass);
```

### 获取PsiElment下所有的PsiStatement

```
for (PsiStatement psiStatement : psiMethod.getBody().getStatements()) {
    EventLogger.log(psiStatement.getText());
   // 查找setContentView
    if (psiStatement.getFirstChild() instanceof PsiMethodCallExpression) {
        PsiReferenceExpression methodExpression = ((PsiMethodCallExpression) psiStatement.getFirstChild()).getMethodExpression();
        if (methodExpression.getText().equals("setContentView")) {
           setContentViewStatement = psiStatement;
        } else if (methodExpression.getText().equals("initView")) {
           hasInitViewStatement = true;               
           
        }            
        
    }
}
```
