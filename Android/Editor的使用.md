
# 首先介绍几个对象

## CaretModel对象
CaretModel对象用于描述插入光标，通过CaretModel对象，可以实现如下功能：
```
moveToOffset(int offset)：将光标移动到指定位置（offset）
getOffset()：获取当前光标位置偏移量
getCaretCount：获取光标数量（可能有多个位置有光标）
void addCaretListener(CaretListener listener) ，void removeCaretListener(CaretListener listener)：添加或移除光标监听器（CareListener）
Caret addCaret(VisualPosition visualPosition)：加入新的光标
……
```

## SelectionModel对象
SelectionModel对象用于描述光标选中的文本段，通过SelectionModel对象可以实现如下功能：


```
String getSelectedText() ：获取选中部分字符串。
int getSelectionEnd()：获取选中文本段末尾偏移量
int getSelectionStart()：获取选中文本段起始位置偏移量
void setSelection(int start, int end)：设置选中，将staert到end部分设置为选中
void removeSelection()：将选中文本段删除
void addSelectionListener(SelectionListener listener)：添加监听器，用于监听光标选中变化。
void selectLineAtCaret()：将光标所在的行设置为选中。
void selectWordAtCaret(boolean honorCamelWordsSettings):将光标所在的单词设置为选中。honorCamelWordsSettings表示是否驼峰命名分隔，如果为true，则大写字母为单词的边界
……
```




## Document对象
与Editor中的其他对象一样，通过Editor对象的一个getter函数即可得到Document对象：

```
Document document = editor.getDocument();
```
Document对象用于描述文档文件，通过Document对象可以很方便的对Editor中的文件进行操作。可以做如下这些事情：
```
String getText()、String getText( TextRange range)：获取Document对象对应的文件字符串。
int getTextLength()：获取文件长度。
int getLineCount()：获取文件的行数
int getLineNumber(int offset)：获取指定偏移量位置对应的行号offset取值为[0,getTextLength()-1]。
int getLineStartOffset(int line)：获取指定行的第一个字符在全文中的偏移量，行号的取值范围为：[0,getLineCount()-1]
int getLineEndOffset(int line)：获取指定行的最后一个字符在全文中的偏移量，行号的取值范围为：[0,getLineCount()-1]
void insertString(int offset, CharSequence s)：在指定偏移位置插入字符串
void deleteString(int startOffset, int endOffset)：删除[startOffset,endOffset]位置的字符串，如果文件为只读，则会抛异常。
void replaceString(int startOffset, int endOffset, CharSequence s)：替换[startOffset,endOffset]位置的字符串为s
void addDocumentListener( DocumentListener listener)：添加Document监听器，在Document内容发生变化之前和变化之后都会回调相应函数。
……
```

# 应用实例--在指定位置下添加一个PsiElement

![](https://ws3.sinaimg.cn/large/006tNc79ly1fhzepew87bg30lf0a2181.gif)

思路如下：

- 获取选中的文本
- 获取选中文本所在行末尾的PsiElement
- 创建一个`Log.e(TAG, "MainActivity < onCreate > " + log2.toString());`的PsiStatement
- 将这个PsiStatement添加到PsiElement的后边

## 获取选中的文本

```
//获取SelectionModel对象
SelectionModel selectionModel = editor.getSelectionModel();

//拿到选中部分字符串
String selectedText = selectionModel.getSelectedText();
```
## 获取选中文本所在行末尾的PsiElement
```
//获取当前所在的行号
int line = editor.getCaretModel().getLogicalPosition().line;
EventLogger.log(line+"");

Document document = editor.getDocument();
int offset1 = document.getLineEndOffset(line);
```