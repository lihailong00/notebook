# GIT使用手册

[toc]

## 设置签名

设置全局签名：

~~~git
git config --global user.name '李文'
git config --global user.email 208827435@qq.com
~~~



重置签名：

~~~git
git config --unset --global user.name
git config --global user.name '李耐思'
~~~



展示gitconfig内容

方法一：

~~~git
cd ~/
cat .gitconfig
~~~

方法二：

~~~git
git config --list
~~~

具体信息我也看不懂。





## 本地提交

### 提交到暂存区

提交单独的文件

~~~git
git add filename
~~~



提交所有的文件

~~~git
git add .
~~~

### 提交到本地库

~~~git
git commit
~~~

然后进入vim编辑器，添加注释即可。



或者：

~~~git
git commit -m '修改信息'
~~~





## 比较

### 工作区与暂存区

>  假定创建了main.cpp

比较main.cpp在工作区与暂存区的差别：

~~~git
git diff main.cpp  //比较main.cpp在工作区与暂存区的差别
git diff  //比较所有文件在工作区与暂存区的差别
~~~

【注意】main.cpp必须是**已追踪**。



### 暂存区与本地库

~~~git
git diff --staged main.cpp  //staged表示已暂存
git diff --staged  //比较所有暂存区和本地库的文件的区别
~~~



### 比较分支

~~~git
git diff <b_01> <b_02>  //比较两个分支的差别
git diff <b_01> <b_02> file  //比较两个分支的某个文件
~~~



## 快速操作

### 重命名文件

比如存在文件file_1，将其改名为file_2后。在git中应进行如下操作

~~~git
git rm file_1
git add file_2
~~~

重命名成功！



还有另一种方法

~~~git
git mv doc_1 doc_2
~~~

将doc_1改名为doc_2。依次类推，git中删除、改名文件只需要在常规的linux命令行前添加git即可。





### 删除文件

~~~git
git rm file -r
~~~

【注意】**git rm, git mv操作的都是工作区和暂存区的文件。**

如果git rm后想恢复改文件，应该这样做（根据git提示的指令也能做到）：

~~~git
git restore --staged <filename>
git restore <filename>
~~~





### 恢复文件

对于已追踪的文件，如果只删除了工作区，暂存区和本地库都没删，那么只需要做：（通过git status提醒也可以）

~~~ git
git restore <file>  //相当于将工作区回溯到上一个版本
~~~



如果已经删除了工作区和暂存区的文件，但是本地库没有修改，可以使用如下指令（可通过git status的提醒操作）：

~~~git
git restore --staged <file>		//将暂存区文件恢复
git restore <file>		//将工作区的文件恢复
~~~



如果三个区域都删除了，那么只能回退到以前的版本，具体指令如下：

~~~git
git checkout HEAD^		//回退到上一个版本
git checkout HEAD^^		//回退到上两个版本
~~~





## 回溯

### git revert

案例：总共提交了5次，发现第3次提交有问题。我们需要撤销第三次提交，该怎么做？

~~~git
git revert 34affk	//第3次提交的哈希值是34affk
~~~

【注意】如果第3次提交的文件会印象到后面的文件，那么git会提醒你。

如果git revert某个版本后，想撤销这次revert，使用git reset --hard <地址> 即可。



### git reset

案例：总共提交了5次，发现第3次以后的所以提交都有问题。我们需要撤销到第三次提交，该怎么做？

~~~git
git reset --(type) 34affk	//第3次提交的哈希值是34affk
~~~

其中type可以是hard，mixed和soft。

hard表示将三个区域的文件都回溯到原来的位置。

mixed表示将本地库和代码区的文件回溯到原来的位置。

soft表示将本地库的文件回溯到原来的位置。



## 分支

### 创建及切换

~~~git
git branch <分支名>  //创建分支
~~~



~~~git
git checkout <分支名>  //切换到该分支
~~~



查看两个分支之间的差别：

~~~git
git diff <branch_1> <branch_2>
~~~

~~~git
git diff <branch_1> <branch_2> filename
~~~

【BUT】有点看不懂信息



合并分支：

~~~git
//先跳到master分支
git merge <branch_name>
~~~

结果：如果显示fastforward，则表明被合并的分支【完全领先】主分支。





合并时产生冲突：

放弃合并：

~~~git
git merge --abort
~~~

继续合并：进入产生冲突的文件，根据提示信息修改好以后退出。再git add git commit。

【疑问】为什么合并分支后两个分支任然有区别？



分支重命名

~~~git
git branch -m <old_branch_name> <new_branch_name>
~~~

删除分支

~~~git
git branch -d <branch_name>
~~~



查看分支：

查看所有分支：

~~~git
git branch -a
~~~

查看本地所有分支：

~~~git
git branch
~~~

查看远程分支：

~~~git
git branch -r
~~~







## 暂存

git stash

使用场景：假定有a, b两个文件。当前已经修改了a文件，但还没有提交到暂存区。突然接到通知，需要保持a文件原封不动并且先将b文件add, commit，然后才能提交a文件。

这时需要将a文件stash：

~~~git
git stash save '描述信息'  //描述信息可以不添加
~~~

【注意】本次stash是暂存所有没有add的文件。

如果文件a还没有被追踪，但是仍然需要暂存它，这时使用：

~~~git
git stash save '描述信息' -u  //u表示untracked
~~~





如果想stash部分文件，需要执行如下指令：

~~~git
git stash -p
~~~

然后依次决定是否stash该文件。



提交完毕后需要将缓存的文件（不在暂存区）恢复到工作区：

~~~git
git stash pop stash@<版本>
~~~

如果想恢复最近一个版本的stash：

~~~git
git stash pop
~~~

【注意】pop后stash list中就再也没有改版本的保存信息。用另一种方法也可以恢复。

~~~git
git stash apply stash@<版本号>
//或者恢复上一个版本
git stash apply
~~~

使用apply后stash列表中还保留着信息。





展示当前stash的版本：

~~~git
git stash list
~~~



如何想查看某个版本stash的内容，可以使用：

~~~git
git stash show -p stash@{版本号}
~~~

如果不写末尾的stash@{版本号}，则默认查看上一个版本stash的内容。





如果不想使用stash中的内容，可以将它drop掉：

~~~git
git stash drop stash@{版本号}
~~~



## 查看日志

前缀：git log



每条信息显示为一行：

~~~git
git log --oneline
~~~



显示最近的6条信息：

~~~git
git log -6
~~~



显示示意图：

~~~git
git log --graph
~~~



查看某个文件的修改信息：

~~~git
git log --grep='文件名'  //grep表示正则表达式
~~~



统计一段时间的提交

~~~git
git log --after='2021.9.1 00:00:00' --before='2021.9.1 15:20:00'  
//日期和时间之间用空格连接
//日期也可以用 - 连接
//before、after的值可以是today,1 week, 3days等比较人性化的词
~~~



查看所以提交记录：

~~~git
git reflog
~~~



## 设置别名

修改~/下的.gitconfig或者~/下的bash_profile即可。具体过程百度搜索。



## 不跟踪文件（我也不太懂）

全局范围类忽略的文件（我不太会）



局部区域忽略文件：

先进入局部区域的根目录

~~~git
//添加.gitignore文件：
git vim .gitignore
//在.gitignore文件中输入不跟踪的文件名
*.py	//不跟踪所有python文件
~~~

【注意】git不会忽略已经跟踪后的文件

[git ignore详解](http://github.com/github/gitignore)



## 建立远程连接

建立连接：

~~~git
git remote add origin 网址
~~~



查看远程地址：

~~~git
git remote -v
~~~



推送到远程地址：

~~~git
git push -u origin <分支名>  //分支名一般是master
~~~

【重要】-u的目的是监测远程库的分支是否发生变化。



删除远程连接：

~~~git
git remote rm '分支名'  //例如分支名是‘origin’
~~~



## 克隆

从远程库下载到本地：

~~~git
git clone '远程网址' '文件名'
~~~

将远程文件克隆到当前目录的某个文件里面。（可以不输入文件名）
