# Git命令

我每天使用 Git ，但是很多命令记不住。

一般来说，日常使用只要记住下图6个命令，就可以了。但是熟练使用，恐怕要记住60～100个命令。

![img](http://www.ruanyifeng.com/blogimg/asset/2015/bg2015120901.png)

下面是我整理的常用 Git 命令清单。几个专用名词的译名如下。

 - Workspace：工作区
 - Index / Stage：暂存区
 - Repository：仓库区（或本地仓库）
 - Remote：远程仓库

## 一、新建代码库 - git init

 ```shell
 # 在当前目录新建一个Git代码库
 $ git init

 # 新建一个目录，将其初始化为Git代码库
 $ git init [project-name]

 # 下载一个项目和它的整个代码历史
 $ git clone [url]

 ```

## 二、配置 git config

Git的设置文件为`.gitconfig`，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）。

 ```shell
 # 显示当前的Git配置
 $ git config --list

 # 编辑Git配置文件
 $ git config -e [--global]

 # 设置提交代码时的用户信息
 $ git config [--global] user.name "[name]"
 $ git config [--global] user.email "[email address]"

 ```

## 三、增加/删除文件 git add/rm

 ```shell
 # 添加指定文件到暂存区
 $ git add [file1] [file2] ...

 # 添加指定目录到暂存区，包括子目录
 $ git add [dir]

 # 添加当前目录的所有文件到暂存区
 $ git add .

 # 添加每个变化前，都会要求确认
 # 对于同一个文件的多处变化，可以实现分次提交
 $ git add -p

 # 删除工作区文件，并且将这次删除放入暂存区
 $ git rm [file1] [file2] ...

 # 停止追踪指定文件，但该文件会保留在工作区
 $ git rm --cached [file]

 # 改名文件，并且将这个改名放入暂存区
 $ git mv [file-original] [file-renamed]

 ```

## 四、代码提交 git commit

 ```shell
 # 提交暂存区到仓库区
 $ git commit -m [message]

 # 提交暂存区的指定文件到仓库区
 $ git commit [file1] [file2] ... -m [message]

 # 提交工作区自上次commit之后的变化，直接到仓库区
 $ git commit -a

 # 提交时显示所有diff信息
 $ git commit -v

 # 使用一次新的commit，替代上一次提交
 # 如果代码没有任何新变化，则用来改写上一次commit的提交信息
 $ git commit --amend -m [message]

 # 重做上一次commit，并包括指定文件的新变化
 $ git commit --amend [file1] [file2] ...

 ```

## 五、分支 git branch

 ```shell
 # 列出所有本地分支
 $ git branch

 # 列出所有远程分支
 $ git branch -r

 # 列出所有本地分支和远程分支
 $ git branch -a

 # 新建一个分支，但依然停留在当前分支
 $ git branch [branch-name]

 # 新建一个分支，并切换到该分支
 $ git checkout -b [branch]

 # 新建一个分支，指向指定commit
 $ git branch [branch] [commit]

 # 新建一个分支，与指定的远程分支建立追踪关系
 $ git branch --track [branch] [remote-branch]

 # 切换到指定分支，并更新工作区
 $ git checkout [branch-name]

 # 切换到上一个分支
 $ git checkout -

 # 建立追踪关系，在现有分支与指定的远程分支之间
 $ git branch --set-upstream [branch] [remote-branch]

 # 合并指定分支到当前分支
 $ git merge [branch]

 # 选择一个commit，合并进当前分支
 $ git cherry-pick [commit]

 # 删除分支
 $ git branch -d [branch-name]

 # 删除远程分支
 $ git push origin --delete [branch-name]
 $ git branch -dr [remote/branch]

 ```

## 六、标签 git tag

 ```shell
 # 列出所有tag
 $ git tag

 # 新建一个tag在当前commit
 $ git tag [tag]

 # 新建一个tag在指定commit
 $ git tag [tag] [commit]

 # 删除本地tag
 $ git tag -d [tag]

 # 删除远程tag
 $ git push origin :refs/tags/[tagName]

 # 查看tag信息
 $ git show [tag]

 # 提交指定tag
 $ git push [remote] [tag]

 # 提交所有tag
 $ git push [remote] --tags

 # 新建一个分支，指向某个tag
 $ git checkout -b [branch] [tag]

 ```

## 七、查看信息 git status

 ```shell
 # 显示有变更的文件
 $ git status

 # 显示当前分支的版本历史
 $ git log

 # 显示commit历史，以及每次commit发生变更的文件
 $ git log --stat

 # 搜索提交历史，根据关键词
 $ git log -S [keyword]

 # 显示某个commit之后的所有变动，每个commit占据一行
 $ git log [tag] HEAD --pretty=format:%s

 # 显示某个commit之后的所有变动，其"提交说明"必须符合搜索条件
 $ git log [tag] HEAD --grep feature

 # 显示某个文件的版本历史，包括文件改名
 $ git log --follow [file]
 $ git whatchanged [file]

 # 显示指定文件相关的每一次diff
 $ git log -p [file]

 # 显示过去5次提交
 $ git log -5 --pretty --oneline

 # 显示所有提交过的用户，按提交次数排序
 $ git shortlog -sn

 # 显示指定文件是什么人在什么时间修改过
 $ git blame [file]

 # 显示暂存区和工作区的差异
 $ git diff

 # 显示暂存区和上一个commit的差异
 $ git diff --cached [file]

 # 显示工作区与当前分支最新commit之间的差异
 $ git diff HEAD

 # 显示两次提交之间的差异
 $ git diff [first-branch]...[second-branch]

 # 显示今天你写了多少行代码
 $ git diff --shortstat "@{0 day ago}"

 # 显示某次提交的元数据和内容变化
 $ git show [commit]

 # 显示某次提交发生变化的文件
 $ git show --name-only [commit]

 # 显示某次提交时，某个文件的内容
 $ git show [commit]:[filename]

 # 显示当前分支的最近几次提交
 $ git reflog

 ```

## 八、远程同步

 ```shell
 # 下载远程仓库的所有变动
 $ git fetch [remote]

 # 显示所有远程仓库
 $ git remote -v

 # 显示某个远程仓库的信息
 $ git remote show [remote]

 # 增加一个新的远程仓库，并命名
 $ git remote add [shortname] [url]

 # 取回远程仓库的变化，并与本地分支合并
 $ git pull [remote] [branch]

 # 上传本地指定分支到远程仓库
 $ git push [remote] [branch]

 # 强行推送当前分支到远程仓库，即使有冲突
 $ git push [remote] --force

 # 推送所有分支到远程仓库
 $ git push [remote] --all

 ```

## 九、撤销

 ```shell
 # 恢复暂存区的指定文件到工作区
 $ git checkout [file]

 # 恢复某个commit的指定文件到暂存区和工作区
 $ git checkout [commit] [file]

 # 恢复暂存区的所有文件到工作区
 $ git checkout .

 # 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
 $ git reset [file]

 # 重置暂存区与工作区，与上一次commit保持一致
 $ git reset --hard

 # 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
 $ git reset [commit]

 # 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
 $ git reset --hard [commit]

 # 重置当前HEAD为指定commit，但保持暂存区和工作区不变
 $ git reset --keep [commit]

 # 新建一个commit，用来撤销指定commit
 # 后者的所有变化都将被前者抵消，并且应用到当前分支
 $ git revert [commit]

 # 暂时将未提交的变化移除，稍后再移入
 $ git stash
 $ git stash pop

 ```



## 十、克隆 git clone

远程操作的第一步，通常是从远程主机克隆一个版本库，这时就要用到`git clone`命令。

 ```shell
$ git clone <版本库的网址
 ```

比如，克隆jQuery的版本库。

 ```shell
 $ git clone https://github.com/jquery/jquery.git

 ```

该命令会在本地主机生成一个目录，与远程主机的版本库同名。如果要指定不同的目录名，可以将目录名作为`git clone`命令的第二个参数。

 ```shell
 $ git clone <版本库的网址 <本地目录名

 ```

`git clone`支持多种协议，除了HTTP(s)以外，还支持SSH、Git、本地文件协议等，下面是一些例子。

 ```shell
 $ git clone http[s]://example.com/path/to/repo.git/
 $ git clone ssh://example.com/path/to/repo.git/
 $ git clone git://example.com/path/to/repo.git/
 $ git clone /opt/git/project.git 
 $ git clone file:///opt/git/project.git
 $ git clone ftp[s]://example.com/path/to/repo.git/
 $ git clone rsync://example.com/path/to/repo.git/

 ```

SSH协议还有另一种写法。

 ```shell
 $ git clone [user@]example.com:path/to/repo.git/

 ```

通常来说，Git协议下载速度最快，SSH协议用于需要用户认证的场合。各种协议优劣的详细讨论请参考[官方文档](http://git-scm.com/book/en/Git-on-the-Server-The-Protocols)。

## 十一、远程 git remote

为了便于管理，Git要求每个远程主机都必须指定一个主机名。`git remote`命令就用于管理主机名。

不带选项的时候，`git remote`命令列出所有远程主机。

 ```shell
 $ git remote
 origin

 ```

使用`-v`选项，可以参看远程主机的网址。

 ```shell
 $ git remote -v
 origin  git@github.com:jquery/jquery.git (fetch)
 origin  git@github.com:jquery/jquery.git (push)

 ```

上面命令表示，当前只有一台远程主机，叫做origin，以及它的网址。

克隆版本库的时候，所使用的远程主机自动被Git命名为`origin`。如果想用其他的主机名，需要用`git clone`命令的`-o`选项指定。

 ```shell
 $ git clone -o jQuery https://github.com/jquery/jquery.git
 $ git remote jQuery
 ```

上面命令表示，克隆的时候，指定远程主机叫做jQuery。

`git remote show`命令加上主机名，可以查看该主机的详细信息。

 ```shell
 $ git remote show <主机名
 ```

`git remote add`命令用于添加远程主机。

 ```shell
 $ git remote add <主机名 <网址
 ```

`git remote rm`命令用于删除远程主机。

 ```shell
 $ git remote rm <主机名
 ```

`git remote rename`命令用于远程主机的改名。

 ```shell
 $ git remote rename <原主机名 <新主机名
 ```

## 十二、git fetch

一旦远程主机的版本库有了更新（Git术语叫做commit），需要将这些更新取回本地，这时就要用到`git fetch`命令。

 ```shell
 $ git fetch <远程主机名

 ```

上面命令将某个远程主机的更新，全部取回本地。

`git fetch`命令通常用来查看其他人的进程，因为它取回的代码对你本地的开发代码没有影响。

默认情况下，`git fetch`取回所有分支（branch）的更新。如果只想取回特定分支的更新，可以指定分支名。

 ```shell
 $ git fetch <远程主机名 <分支名

 ```

比如，取回`origin`主机的`master`分支。

 ```shell
 $ git fetch origin master

 ```

所取回的更新，在本地主机上要用"远程主机名/分支名"的形式读取。比如`origin`主机的`master`，就要用`origin/master`读取。

`git branch`命令的`-r`选项，可以用来查看远程分支，`-a`选项查看所有分支。

 ```shell
 $ git branch -r
 origin/master

 $ git branch -a
 * master
   remotes/origin/master

 ```

上面命令表示，本地主机的当前分支是`master`，远程分支是`origin/master`。

取回远程主机的更新以后，可以在它的基础上，使用`git checkout`命令创建一个新的分支。

 ```shell
 $ git checkout -b newBrach origin/master

 ```

上面命令表示，在`origin/master`的基础上，创建一个新分支。

此外，也可以使用`git merge`命令或者`git rebase`命令，在本地分支上合并远程分支。

 ```shell
 $ git merge origin/master
 # 或者
 $ git rebase origin/master

 ```

上面命令表示在当前分支上，合并`origin/master`。

## 十三、git pull

`git pull`命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并。它的完整格式稍稍有点复杂。

 ```shell
 $ git pull <远程主机名 <远程分支名:<本地分支名

 ```

比如，取回`origin`主机的`next`分支，与本地的`master`分支合并，需要写成下面这样。

 ```shell
 $ git pull origin next:master

 ```

如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

 ```shell
 $ git pull origin next

 ```

上面命令表示，取回`origin/next`分支，再与当前分支合并。实质上，这等同于先做`git fetch`，再做`git merge`。

 ```shell
 $ git fetch origin
 $ git merge origin/next

 ```

在某些场合，Git会自动在本地分支与远程分支之间，建立一种追踪关系（tracking）。比如，在`git clone`的时候，所有本地分支默认与远程主机的同名分支，建立追踪关系，也就是说，本地的`master`分支自动"追踪"`origin/master`分支。

Git也允许手动建立追踪关系。

 ```shell
 git branch --set-upstream master origin/next

 ```

上面命令指定`master`分支追踪`origin/next`分支。

如果当前分支与远程分支存在追踪关系，`git pull`就可以省略远程分支名。

 ```shell
 $ git pull origin

 ```

上面命令表示，本地的当前分支自动与对应的`origin`主机"追踪分支"（remote-tracking branch）进行合并。

如果当前分支只有一个追踪分支，连远程主机名都可以省略。

 ```shell
 $ git pull

 ```

上面命令表示，当前分支自动与唯一一个追踪分支进行合并。

如果合并需要采用rebase模式，可以使用`--rebase`选项。

 ```shell
 $ git pull --rebase <远程主机名 <远程分支名:<本地分支名

 ```

如果远程主机删除了某个分支，默认情况下，`git pull` 不会在拉取远程分支的时候，删除对应的本地分支。这是为了防止，由于其他人操作了远程主机，导致`git pull`不知不觉删除了本地分支。

但是，你可以改变这个行为，加上参数 `-p` 就会在本地删除远程已经删除的分支。

 ```shell
 $ git pull -p
 # 等同于下面的命令
 $ git fetch --prune origin 
 $ git fetch -p

 ```

## 十四、git push

`git push`命令用于将本地分支的更新，推送到远程主机。它的格式与`git pull`命令相仿。

 ```shell
 $ git push <远程主机名> <本地分支名>:<远程分支名>
 ```

注意，分支推送顺序的写法是<来源地:<目的地，所以`git pull`是<远程分支:<本地分支，而`git push`是<本地分支:<远程分支。

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。

 ```shell
 $ git push origin master
 ```

上面命令表示，将本地的`master`分支推送到`origin`主机的`master`分支。如果后者不存在，则会被新建。

如果省略本地分支名，则表示删除指定的远程分支，因为这等同于推送一个空的本地分支到远程分支。

 ```shell
 $ git push origin :master
 # 等同于
 $ git push origin --delete master
 ```

上面命令表示删除`origin`主机的`master`分支。

如果当前分支与远程分支之间存在追踪关系，则本地分支和远程分支都可以省略。

 ```shell
 $ git push origin
 ```

上面命令表示，将当前分支推送到`origin`主机的对应分支。

如果当前分支只有一个追踪分支，那么主机名都可以省略。

 ```shell
 $ git push
 ```

如果当前分支与多个主机存在追踪关系，则可以使用`-u`选项指定一个默认主机，这样后面就可以不加任何参数使用`git push`。

 ```shell
 $ git push -u origin master
 ```

上面命令将本地的`master`分支推送到`origin`主机，同时指定`origin`为默认主机，后面就可以不加任何参数使用`git push`了。

不带任何参数的`git push`，默认只推送当前分支，这叫做simple方式。此外，还有一种matching方式，会推送所有有对应的远程分支的本地分支。Git 2.0版本之前，默认采用matching方法，现在改为默认采用simple方式。如果要修改这个设置，可以采用`git config`命令。

 ```shell
 $ git config --global push.default matching
 # 或者
 $ git config --global push.default simple
 ```

还有一种情况，就是不管是否存在对应的远程分支，将本地的所有分支都推送到远程主机，这时需要使用`--all`选项。

 ```shell
 $ git push --all origin
 ```

上面命令表示，将所有本地分支都推送到`origin`主机。

如果远程主机的版本比本地版本更新，推送时Git会报错，要求先在本地做`git pull`合并差异，然后再推送到远程主机。这时，如果你一定要推送，可以使用`--force`选项。

 ```shell
 $ git push --force origin 
 ```

上面命令使用`--force`选项，结果导致远程主机上更新的版本被覆盖。除非你很确定要这样做，否则应该尽量避免使用`--force`选项。

最后，`git push`不会推送标签（tag），除非使用`--tags`选项。

 ```shell
 $ git push origin --tags
 ```

## 十五、子模块 git submodule

### 1. 添加子模块

进入 `MainProject` 使用 **git submodule add** 进行添加,操作命令：

```Shell
git clone https://github.com/imtianx/MainProject.git
cd MainProject/
git submodule add https://github.com/imtianx/liba.git
git submodule add https://github.com/imtianx/libb.git
```

添加 `liba` `libb` 子模块。 `.gitmodules` 中存放的为子模块的信息，使用 `cat` 或 `vim` 查看内容为：

```
[submodule "liba"]
	path = liba
	url = https://github.com/imtianx/liba.git
[submodule "libb"]
	path = libb
	url = https://github.com/imtianx/libb.git

```

> **.gitmodules文件**：保存项目 URL 与已经拉取的本地目录之间的映射，有多个子模块则含有多条记录，会随着版本控制一起被拉去和推送的。

此时文件目录树如下：

```
.
├── README.md
├── liba
│   ├── README.md
│   ├── a.txt
│   └── a2.txt
├── libb
│   ├── README.md
│   ├── b.txt
│   └── b2.txt
└── test.text


```

最后，**提交添加的子模块到主目录**

```
$ git commit -m "add liba and libb submodules"
[master 6b15e30] add liba and libb submodules
 3 files changed, 8 insertions(+)
 create mode 100644 .gitmodules
 create mode 160000 liba
 create mode 160000 libb

```

###  2. 更新子模块

往往子模块是单独开发的，这里以更新 `liba` 为例（为了测试，这里先在liba仓库添加了一个文件）：

```
cd liba/
git fetch
git merge origin/master

```

此外，还可以在主目录下 直接用下面的命令更新 `libb`子模块：

```
git submodule update --remote liba
```

**使用下面的方式，更新所有子模块：**

```
git submodule update --remote
```



### 3. 删除子模块

在日常开发中，有添加，当然也会有删除 子模块的需求。 这里主项目包含两个子模块：`liba`、`libb`，以删除 `liba` 为例说明：

- 使用 `git rm --cached liba` 将liba 从版本控制中删除（本地仍保留有），若不需要可不带 `--cached`进行完全删除。
- 使用 `vim .gitmodules` 可打开vim编辑,删除对应的内容

```Shell
 [submodule "liba"]
          path = liba
          url = https://github.com/imtianx/liba.git
          branch = dev
```

- 使用 `vim .git/config` 可打开vim编辑,删除对应的内容

```Shell
[submodule "liba"]
         url = https://github.com/imtianx/liba.git
         active = true
```

- 使用 `rm -rf .git/modules/liba`, 删除.git下的缓存模块，最后提交项目。

经过上面的删除后还可以进行添加子模块。

### 4. 克隆含子模块的仓库

若需要克隆含有子模块的仓库，直接 进行克隆是无法拉取之模块的代码，可加上 **--recursive**参数，如下：

```Shell
git clone --recursive https://github.com/imtianx/MainProject.git
```

或者使用下面的三部操作：

```Shell
git clone  https://github.com/imtianx/MainProject.git
git submodule init
git submodule update
```



## 十五、其他

```shell
 # 生成一个可供发布的压缩包
 $ git archive
```



# git 错误解决方法

错误：

```verilog
error: RPC failed; curl 18 transfer closed with outstanding read data remaining

fatal: The remote end hung up unexpectedly

fatal: early EOF

fatal: index-pack failed

```

解决办法：

修改Git的传输字节限制即可。

\# git config --global http.postBuffer  524288000
