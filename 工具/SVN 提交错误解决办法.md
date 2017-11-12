
# svn: Commit failed (details follow): svn: '/***/xxx.c' is scheduled for addition, but is missing 

删除文件夹后点commit提交，但是报错，报错内容如下：
提示 "svn: Commit failed (details follow): svn: '/***/xxx.c' is scheduled for addition, but is missing "

## 原因：
之前用SVN提交过的文件，被标记为"add"状态，等待被加入到仓库。若此时你把这个文件删除了，SVN提交的时候还是会尝试提交这个文件，虽然它的状态已经是 "missing"了。

## 解决：
在命令行下用 "svn revert xxx.c --depth infinity"，在图形界面下，右键--Revert，选中那个文件。这样就告诉SVN把这个文件退回到之前的状态 "unversioned"，也就是不对这个文件做任何修改