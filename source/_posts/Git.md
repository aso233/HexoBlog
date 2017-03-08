---
title: Git
date: 2016-06-12 18:13:00
tags: [Git]
---
## Git简介 ##
Git是目前最流行的分布式版本控制系统，它的由来却是偶然：  
很多人都知道，Linus在1991年创建了开源的Linux，从此，Linux系统不断发展，已经成为最大的服务器系统软件了。
<!-- More -->
Linus虽然创建了Linux，但Linux的壮大是靠全世界热心的志愿者参与的，这么多人在世界各地为Linux编写代码，那Linux的代码是如何管理的呢？
事实是，在2002年以前，世界各地的志愿者把源代码文件通过diff的方式发给Linus，然后由Linus本人通过手工方式合并代码！
你也许会想，为什么Linus不把Linux代码放到版本控制系统里呢？不是有CVS、SVN这些免费的版本控制系统吗？因为Linus坚定地反对CVS和SVN，这些集中式的版本控制系统不但速度慢，而且必须联网才能使用。有一些商用的版本控制系统，虽然比CVS、SVN好用，但那是付费的，和Linux的开源精神不符。

不过，到了2002年，Linux系统已经发展了十年了，代码库之大让Linus很难继续通过手工方式管理了，社区的弟兄们也对这种方式表达了强烈不满，于是Linus选择了一个商业的版本控制系统BitKeeper，BitKeeper的东家BitMover公司出于人道主义精神，授权Linux社区免费使用这个版本控制系统。

安定团结的大好局面在2005年就被打破了，原因是Linux社区牛人聚集，不免沾染了一些梁山好汉的江湖习气。开发Samba的Andrew试图破解BitKeeper的协议（这么干的其实也不只他一个），被BitMover公司发现了（监控工作做得不错！），于是BitMover公司怒了，要收回Linux社区的免费使用权。

Linus可以向BitMover公司道个歉，保证以后严格管教弟兄们，嗯，这是不可能的。实际情况是这样的：
Linus花了两周时间自己用C写了一个分布式版本控制系统，这就是Git！一个月之内，Linux系统的源码已经由Git管理了！牛是怎么定义的呢？大家可以体会一下。
Git迅速成为最流行的分布式版本控制系统，尤其是2008年，GitHub网站上线了，它为开源项目免费提供Git存储，无数开源项目开始迁移至GitHub，包括jQuery，PHP，Ruby等等。  
  
## Git安装 ##
git下载地址：[http://git-scm.com/download/](http://git-scm.com/download/ "http://git-scm.com/download/"), 目前最新版本为2.7.0  
下载windows版本的安装包，安装完成之后，在右键菜单可以看到git的选项：  
	
	Git Init Here  (在当前目录初始化一个git仓库)  
	Git Gui  (git图形界面)   
	Git Bash  (git命令行窗口)  

点击Git Bash选项打开git命令行窗口，说明安装成功。  
安装完成之后需要配置name和email

	git config --global user.name "Your Name"  
	git config --global user.email "email@example.com"

注意：git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，不加则表示只用于当前仓库。

## Git基本操作 ##
**1. 创建git仓库**

新建一个GitTest目录，在该目录下打开git命令行窗口输入：  

	git init  
	git status  

git init执行之后，当前目录下将自动生成.git文件夹，保存仓库的版本信息和提交记录等。  
git status 显示如下：  
![](http://i.imgur.com/jTgSGfs.png)  
从输出信息可以看出当前仓库处于默认的master分支上，并且nothing to commit表示当前工作区是干净的，没有任何修改。

**2. 添加文件**  
在目录中添加一个Test.txt文件，文件内容随意输入。  

	git status  

![](http://i.imgur.com/O2oqxF1.png)  

红色表示该文件处于工作区，还未加入暂存区，可使用add命令将其加入：  

	git add Test.txt  

![](http://i.imgur.com/9LU71bb.png)  
绿色表示该文件已经加入暂存区，但还未提交到git仓库中。  

**3. 撤销修改**  
使用checkout命令可撤销修改，工作区的内容还原到修改之前：  

	git checkout Test.txt  

reset HEAD可将暂存区中的内容还原至工作区：

	git reset HEAD

revert HEAD 生成一个新的提交来撤销某次提交  

	git revert HEAD

**4. 提交修改**  
使用commit命令将暂存区中的内容提交到版本仓库中：  

	git commit -m "first commit"  // -m 后为提交的log信息

通过git log可查看仓库的提交信息：  
![](http://i.imgur.com/RQFEXwv.png)

**5. 创建分支** 
	
	git branch develop  
	git branch -a   

![](http://i.imgur.com/hnKK2Kw.png)
  
或者使用  

	git branch -b develop  

创建成功之后将自动切换到develop分支上  

**6. 切换分支**  
	
	git checkout develop

![](http://i.imgur.com/twIcpsU.png)  
切换到另一个分支之前最好确认当前分支的工作区和暂存区是否干净，以免造成混淆。
## Git远程仓库操作 ##
**1. 关联远程仓库**
	
	git remote add <主机名> <网址>  
	git remote rm <主机名>  
	git remote rename <原主机名> <新主机名>  

**2. git克隆**  

支持多种协议： 

	git clone <版本库的网址> <本地目录名>  
  
	git clone http[s]://example.com/path/to/repo.git/  
	git clone ssh://example.com/path/to/repo.git/  
	git clone git://example.com/path/to/repo.git/  
	git clone /opt/git/project.git   
	git clone file:///opt/git/project.git  
	git clone ftp[s]://example.com/path/to/repo.git/  
	git clone rsync://example.com/path/to/repo.git/  

**2. 切换远程分支**
	
	git branch -a  
	git checkout -b 本地分支名 远程主机名/远程分支名   

或者  

	git checkout -t 远程主机名/远程分支名 //默认会在本地建立一个和远程分支名字一样的分支

**3. 远程分支拉取**

- fetch：只拉取信息，并未进行merge  

		git fetch 

	将某个远程主机的更新，全部取回本地

		git fetch <远程主机名> <分支名>

	取回origin主机的master分支


- pull：相当于做了fetch 和 merge

		git pull <远程主机名> <远程分支名>:<本地分支名>  

如果远程分支是与当前分支合并，则冒号后面的部分可以省略。

**4. 远程分支推送** 
	
	git push <远程主机名> <本地分支名>:<远程分支名>   

如果省略远程分支名，则表示将本地分支推送与之存在"追踪关系"的远程分支（通常两者同名），如果该远程分支不存在，则会被新建。 

	
	git push --all origin 

将本地的所有分支都推送到远程主机，这时需要使用--all选项。


## git别名配置 ##
为了便于输入，可以为git常用的命令配置别名，下面是一些常用的别名设置：

	git config --global alias.st status  
	git config --global alias.co checkout  
	git config --global alias.ci commit  
	git config --global alias.br branch  
	git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"


## 在本地模拟远程仓库的创建和推送（工作目录为F:/GitTest/） ##

**1. 创建远程仓库**  

创建G_Server目录，在目录下打开Git Bash并执行创建仓库命令，创建仓库的方式有两种：
  
- 裸仓库，只会生成一类文件用于记录版本库历史记录的.git目录下面的文件;而不会包含实际项目源文件的拷贝。

		git --bare init <目录名>

- 包含实际项目源文件的拷贝。       

		git init  
 
如果你需要操作远程仓库的git库的话，应该使用git init，因为裸仓库只保存了git的历史提交和版本信息，并不允许进行git操作，会得到“This operation must be run in a work tree”错误提示。如果远程仓库不希望让人操作，则最好选择裸仓库。  

**2. 创建本地仓库1，提交内容到远程仓库**

- 创建G_Client1目录，初始化仓库，添加测试文件Test.txt，并提交到本地    

		echo "112233">Test.txt  
		git add Test.txt  
		git ci -m “first commit from client1”  

- 关联远程仓库信息并推送到远程仓库:  

远程仓库为裸仓库  

	git remote add origin file:///F:/GitTest/G_Server/目录名  

远程仓库非裸仓库：  

	git remote add origin file:///F:/GitTest/G_Server/.git
	git push origin master  

如果远程仓库并不是以裸仓库的形式创建，且要推送的分支刚好是当前分支，则此推送可能会失败，:  
![](http://i.imgur.com/smHC7JE.png)  
根据提示修改远程仓库 .git/config配置文件，加入：   
 [receive]  
	denyCurrentBranch = ignore  
重新提交即可成功。  
**3. 创建本地仓库2，拉取远程仓库信息**  

创建G_Client2目录，初始化仓库，关联远程仓库

	git init  
	git remote add origin file:///F:/GitTest/G_Server/目录名  
	git fetch origin  
	git pull origin master  

拉取成功之后，目录下会生成远程仓库中的内容Test.txt，并且通过git log可查看提交记录。

**4. 本地仓库2推送修改内容到远程仓库，本地仓库1拉取更新信息**  

- 修改G_Client2目录下的Test.txt文件内容为“336699”，提交并推送

		echo "336699">Test.txt  
		git add .  
		git ci -m "do some modification by client2"  	
		git push origin master  

- 在G_Client1下拉取远程仓库更新信息，并进行更新  

		git fetch  	
		git pull origin master  

![](http://i.imgur.com/n7nk1y0.png)  

此时Test.txt文件已经修改，通过git log可查看提交记录：  
![](http://i.imgur.com/2spW0oj.png)  

通过本地模拟可基本了解远程仓库一些基本操作，也可将远程仓库放到网络服务器或者托管网站，如github中，将代码或者笔记随时提交到远程仓库，这样既安全也方便自己随时随地查看。


# 2016.09.07补充   #
## 图形工具扩展 ##


- 配置git解决冲突图形工具（以beyond Compare4为例）  
	配置Git安装目录下的/etc/gitconfig文件，追加或者修改如下：  
		
		[diff]  
			tool = bc4  
		[difftool]  
			prompt = false  
		[difftool "bc4"]  
			cmd = "\"D:/Program Files/Beyond Compare 4/BCompare.exe\" \"$LOCAL\" \"$REMOTE\""  
		
		[merge]  
			tool = bc4  
		[mergetool]  
			prompt = false  
		[mergetool "bc4"]  
			cmd = "\"D:/Program Files/Beyond Compare 4/BCompare.exe\" \"$PWD/$LOCAL\" \"$PWD/$REMOTE\" \"$PWD/$BASE\" \"$PWD/$MERGED\""
			keepBackup = false
		    trustExitCode = false  
	
	配置工具：  
	
	    git config --global merge.tool bc4  
	    git config --global diff.tool bc4
	
	先执行git merge branch_name 当合并分支出现冲突时，执行
	
		git mergetool [filename]
	即可打开beyondCompare进行代码合并。  
	（四个界面，LOCAL为本地分支修改内容，BASE为修改前的内容，REMOTE为要合并的分支修改的内容，MERGED为修改之后的最终内容）  
	
	同样也可用
	
		git difftool   
	调用beyondCompare查看修改内容  

- sourceTree

# 2016.09.29补充   #
#### 删除远程仓库的某次错误提交 #### 

	git reset –mixed  	此为默认方式，不带任何参数的git reset，就是这种方式，它回退到某个版本，只保留源码，回退commit和stage信息
	git reset –soft 	回退到某个版本， 只回退了commit的信息，不会恢复stage（如果还要提交，直接commit即可)
	git reset –hard  	彻底回退到某个版本， 本地的源码也会变为上一个版本的内容

方法一. 将本地版本reset到指定版本，然后 git push -f(force)到远程分支  
方法二. 将本地reset到指定版本，删除远程分支，git push origin :master(删除前先备份), 重新提交  

# 2016.10.08补充   #
#### git 删除已经 add 的文件 #### 

使用 git rm 命令即可，有两种选择：  
一种是 git rm --cached "文件路径"，不删除物理文件，仅将该文件从缓存中删除；  
一种是 git rm --f "文件路径"，不仅将该文件从缓存中删除，还会将物理文件删除（不会回收到垃圾桶）。
## Git 常见问题记录##
1. git add 时提示 "Filename too long" 解决办法 
> git config --global core.longpaths true

2. 从ftp服务器克隆： git clone ssh://MartinLin@192.168.201.12:3600/home/MartinLin/repository/LocalPack  
3. 从局域网克隆：git clone //BY-01-1146/LocalPack

参考：  
[http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000](http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000 "http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000")  
[http://www.ruanyifeng.com/blog/2014/06/git_remote.html](http://www.ruanyifeng.com/blog/2014/06/git_remote.html "http://www.ruanyifeng.com/blog/2014/06/git_remote.html")  
[http://my.oschina.net/u/1010578/blog/348731](http://my.oschina.net/u/1010578/blog/348731 "http://my.oschina.net/u/1010578/blog/348731")  
beyondCompare下载地址：[http://www.scootersoftware.com/download.php](http://www.scootersoftware.com/download.php "http://www.scootersoftware.com/download.php")