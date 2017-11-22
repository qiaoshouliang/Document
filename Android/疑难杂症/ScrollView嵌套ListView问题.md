# ScrollView嵌套ListView问题

## 进入页面不从顶部开始显示的问题解决 

解决方案——取消ListView的焦点

```
listView.setFocusable(false);
```

实测在代码中通过setFocusable(false)可以解决这个问题 
但是在xml里设置`Android:focusable=”false”`并不起作用

> **同样的方法适用于GridView**