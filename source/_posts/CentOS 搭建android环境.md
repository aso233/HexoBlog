---
title: CentOS 搭建android环境
date: 2016-06-12 18:13:00
tags:
---
# CentOS 搭建android环境 #

1. 安装JAVA环境
	- 下载jdk或者jre ：
  	[http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html "http://www.oracle.com/technetwork/java/javase/downloads/index.html")    
	- 配置环境变量  
	在/etc/profile中加入java路径，如：  
	`export JAVA_HOME=/usr/local/soft/jdk1.6.0_14 `  
	`export PATH=$JAVA_HOME/bin:$PATH `  
	`export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar `  


2. 安装android SDK
	- 下载android SDK： [https://developer.android.com/studio/index.html](https://developer.android.com/studio/index.html "https://developer.android.com/studio/index.html") 并解压  
	- 配置环境变量  
	在/etc/profile中加入android SDK解压路径，如：  
	`export ANDROID_HOME=/usr/local/soft/android-sdk-linux `  
	`export PATH=$ANDROID_HOME/tools:$PATH`
	- 更新SDK  
	`android list sdk -a` 列出当前所有SDK和工具版本列表  
	`android update sdk -a -u -t 1,2,3` 选择更新列表中的某几项(list sdk -a列表的序号)，不加-t参数则表示全部更新

3. 安装ANT    
	- 下载ant包：[http://ant.apache.org/bindownload.cgi](http://ant.apache.org/bindownload.cgi "http://ant.apache.org/bindownload.cgi")，并解压  
	- 配置环境变量  
	在/etc/profile中加入ant解压路径，如：   
	`ANT_HOME=/usr/local/soft/apache-ant-1.9.7`  
	`PATH=$PATH:$ANT_HOME/bin `   

4. ANT打包错误
	- “Cannot run program "/usr/local/android-sdk-linux/build-tools/19.0.3/aapt": error=2, No such file or director”  
Android SDK里面提供的一些工具，比如adb、aapt、zipalign等， 都会依赖32-bit的库，如果系统是64-bit的话，也必须安装这些32-bit的库，如下：  
    `sudo yum install -y glibc.i686 zlib.i686 libstdc++.i686`  

	- “/lib64/libc.so.6: version 'GLIBC_2.14' not found (required by /usr/local/soft/android-sdk-linux/build-tools/24.0.1/aapt)”  
	使用android最新build tool版本（目前为24.0.1）则要求系统glibc库为2.14。两种方案：一是指定android build tool的版本为较低的版本（如19.1）， 二是更新libc库，记录更新方法如下：  
		
		执行`strings /lib64/libc.so.6 | grep GLIBC`	查看系统libc版本：  
GLIBC_2.2.5  
GLIBC_2.2.6  
GLIBC_2.3    
GLIBC_2.3.2
GLIBC_2.3.3  
GLIBC_2.3.4  
GLIBC_2.4  
GLIBC_2.5  
GLIBC_2.6  
GLIBC_2.7  
GLIBC_2.8  
GLIBC_2.9  
GLIBC_2.10  
GLIBC_2.11  
GLIBC_2.12  
GLIBC_PRIVATE  
系统最高版本是2.12，但是android编译需要2.14，于是自己编译安装：  
    `wget http://ftp.gnu.org/gnu/glibc/glibc-2.14.tar.gz`  
	`tar -zxvf glibc-2.14.tar.gz && cd glibc-2.14 && mkdir build && cd build`  
	`../configure --prefix=/opt/glibc-2.14`  
	`make -j4`  
	`make install`  
	`cp -r /etc/ld.so.c* /opt/glibc-2.14/etc/`  
	`ln -sf /opt/glibc-2.14/lib/libc-2.14.so /lib64/libc.so.6`    
	`export LD_LIBRARY_PATH=/opt/glibc-2.14/lib:$LD_LIBRARY_PATH`  

		再次执行 `strings /lib64/libc.so.6 | grep GLIBC`   
GLIBC_2.2.5  
GLIBC_2.2.6  
GLIBC_2.3    
GLIBC_2.3.2
GLIBC_2.3.3  
GLIBC_2.3.4  
GLIBC_2.4  
GLIBC_2.5  
GLIBC_2.6  
GLIBC_2.7  
GLIBC_2.8  
GLIBC_2.9  
GLIBC_2.10  
GLIBC_2.11  
GLIBC_2.12  
GLIBC_2.13  
GLIBC_2.14  
GLIBC_PRIVATE   
出现2.14说明更新成功。

	- Error:  Multilib version problems found. This often means that the root
       cause is something else and multilib version checking is just
       pointing out that there is a problem. Eg.:
       
         1. You have an upgrade for libstdc++ which is missing some
            dependency that another package requires. Yum is trying to
            solve this by installing an older version of libstdc++ of the
            different architecture. If you exclude the bad architecture
            yum will tell you what the root cause is (which package
            requires what). You can try redoing the upgrade with
            --exclude libstdc++.otherarch ... this should give you an error
            message showing the root cause of the problem.
       
         2. You have multiple architectures of libstdc++ installed, but
            yum can only see an upgrade for one of those arcitectures.
            If you don't want/need both architectures anymore then you
            can remove the one with the missing update and everything
            will work.
       
         3. You have duplicate versions of libstdc++ installed already.
            You can use "yum check" to get yum show these errors.
       
       		...you can also use --setopt=protected_multilib=false to remove
       this checking, however this is almost never the correct thing to
       do as something else is very likely to go wrong (often causing
       much more problems).
       
      	 	Protected multilib versions: libstdc++-4.4.7-17.el6.i686 != libstdc++-4.4.7-16.el6.x86_64   
		
		应该是库的版本不对吧，先更新一下64位的库，然后再安装：  

		`yum  update  libstdc++-4.4.7-16.el6.x86_64`  
		`yum install libstdc++.i686 `


参考：  
[https://linux.cn/article-5966-1.html](https://linux.cn/article-5966-1.html "https://linux.cn/article-5966-1.html")  
[http://fpliu-blog.chinacloudsites.cn/it/os/android/sdk/installation/on-centos](http://fpliu-blog.chinacloudsites.cn/it/os/android/sdk/installation/on-centos "http://fpliu-blog.chinacloudsites.cn/it/os/android/sdk/installation/on-centos")  
[http://www.cnblogs.com/qingchen1984/p/5757181.html](http://www.cnblogs.com/qingchen1984/p/5757181.html "http://www.cnblogs.com/qingchen1984/p/5757181.html")