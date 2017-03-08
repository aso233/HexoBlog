---
title: AndfixTool
date: 2016-06-12 18:13:00
tags:
---
# apktool #
近期研究阿里的[Andfix](https://github.com/alibaba/AndFix)，这是一个可实现代码热修复功能的开源项目，使用其提供的apkpatch.jar生成新旧apk的差分包，然后将差分包放置指定位置进行加载，旧的包加载完成后可实现方法的替换，可用于紧急修复线上的bug。生成差分包的jar包代码并没有开源，所以我们的目的是弄清jar包的工作原理并自己实现编译出一个可用的jar包。  
 
使用jd-gui打开apkpatch-1.0.3.jar，jar包结构如下:  
![](http://i.imgur.com/x44V76U.png)  
jar包包含的代码比较多，主要包括几个重点：  
1.  com.euler.patch ： 此包名下的代码是阿里自定义添加的代码，也是andfix的生成差分包的主要逻辑代码。  
2.  org.jf.* ：此包名下其实是smali/baksmali的源代码，baksmali反编译apk，smali重新打包apk,dexlib2是核心，baksmali和smali都需要引用到dexlib2中的定义的方法和接口，util存放一些工具类。smali/baksmali源码地址：[https://github.com/JesusFreke/smali](https://github.com/JesusFreke/smali)  
3.  brut.* ： 此包名下是我们常用的反编译工具apktool的源码，其实也是基于smali/baksmali的基础之上，封装了一些功能逻辑和参数校验等。apktool源码地址：[https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)   


经过一番挣扎，弄清楚jar包中各个包的来源之后，重点就是对apktool源码进行分析并在其基础上加入andfix的逻辑。  

下载apktool源码，导入android Studio：  
![](http://i.imgur.com/yNMTuim.png)

## 编译Apktool源码 ##
apktool源码编译方法如下（[官方说明](http://ibotpeaches.github.io/Apktool/build/)）：

> gradlew.bat clean  
> gradlew.bat applyPatches  
> gradlew.bat build fatJar  

其中，applyPatches其实是gradle插件的task：[net.minecrell.gitpatcher](https://github.com/Minecrell/gitpatcher)，工程的主build.bradle中引用了此插件:  
![](http://i.imgur.com/TUfZYMH.png)  
此插件的作用是当执行applyPatches时：  
1. updateSubmodules：检查submodule的git库状态是否处于最新状态，若不是则失败退出。目前发现执行之前必须将smali切换到master分支并且checkout到 *398630d才可update成功，应该是与apktool源码有关（源码依赖于398630d这个分支），还未做深入研究，后续可通过研究gitpatcher插件得到结论。  
2. applyPatches： 将patches目录下的patch文件一一打入submodule中并将最后生成的结果输出到target目录。我们看一下patches文件目录：  
![](http://i.imgur.com/1GNXJ3H.png)    
这些patch就是apktool对于smali源码的修改内容，从patch的文件名可以大概猜出此patch做了什么修改，可以看出大部分的patch只是修改了smali目录下的各个目录的build.gradle，用于构造自己的项目结构。我们后面也可将andfix对于smali源码的修改以生成patch的形式放到此目录中。  
需要注意的是git提供了两种简单的patch方案，一是用git diff生成的标准patch，二是git format-patch生成的Git专用Patch。专用patch包含比较详细的信息（可自行百度）：  
![](http://i.imgur.com/eIZ5fql.png)
经过实践发现在apktool中使用的是专用补丁，而标准补丁并不能用。专用补丁主要指令如下：  
> git ci -a -m "baksmali-andfix"  
> git format-patch -M master  
> git am D:/xxx.patch  

在执行gradlew.bat build fatjar中经常遇到执行test的时候出错导致编译失败，目前做法是之间将报错的测试代码注释。


## Apktool源码结构 ##
完成源码编译之后，我们可以看到目录下多了brut.apktool.smali目录，目录下就是经过patch之后的smali源码(如果感兴趣可用对比工具比较此目录和smali目录文件的区别便可看出patch修改的内容)。每次执行gradlew.bat clean之后，此目录就会被清除，下一次执行编译之前必须先执行gradlew.bat applyPatches：   
![](http://i.imgur.com/B5Uv75H.png)  

理解了applyPatches的作用之后再来看apktool源码的结构就很清晰了：  

1. smali  
![](http://i.imgur.com/KMmwwtn.png)  
此Module完全引用了smali的源码，从其git的远程仓库中便可看出：  
![](http://i.imgur.com/ZjU4RvV.png)  
2. brut.j.*  
![](http://i.imgur.com/VD1Hi6B.png)  
此目录下主要是一些common、util类，服务于brut.apktool

3. brut.apktool  
![](http://i.imgur.com/R83LfBt.png)   
查看apktool-lib的build.gradle可以看出其引用了其他的所有工程：  
![](http://i.imgur.com/xTd4o4X.png)  
而apktool-cli只引用了apktool-lib,apktool-cli是最终的调用者，其只包含一个Main.java文件，实现的功能其实很简单，接收并过滤命令行的参数，调用引用库中方法。

## 加入andfix逻辑 ##
了解了apktool的代码结构，现在我们要将andfix的代码加入，只需让其取代apktool-ci的位置。  
修改settings.gradle:  
![](http://i.imgur.com/8tikWER.png)  
加入andfix源码：  
![](http://i.imgur.com/rfbPkLm.png)  
参照apktool-ci的build.gradle修改相应配置，主class类等：  
![](http://i.imgur.com/W7yTNEY.png)
刚加入代码时遇到一个问题：andfix引用了smali/dexlib2中的类，但是andfix又修改dexlib2的源码而且引用了andfix中的类，因此形成了一个circle dependences的问题，后来通过将dexlib2中引用到的andfix中的类移至dexlib2中解决了此问题，不知道阿里那边是如何编译的，暂时没想到更好的解决办法。  
![](http://i.imgur.com/g9fngSY.png)  
andfix基本逻辑思路如下：  

1 实现关键类的equals方法  

- com.boyaa.patch.Main为入口类，接收命令输入的参数信息，过滤成功之后调用com.boyaa.patch.ApkPatch的doPatch方法，andfix的主要逻辑便是在此方法中。DexDiff.diff对比出两个apk中修改过的方法，buildCode将这些方法转换成smali文件，并重新打包成dex文件，build写入签名信息，release写入md5等校验信息。    
![](http://i.imgur.com/kzkg1Ht.png)  
- org.jf.dexlib2.diff.DexDiffer类load两个apk的dex文件，严格按照[dex文件的格式](https://source.android.com/devices/tech/dalvik/dex-format.html "dex文件格式")将dex文件各个数据段的偏移地址读取到内存中，最终返回一个DexBackedDexFile对象，通过DexBackedDexFile.getClasses()获取到dex文件中所有方法，然后一一进行对比，将修改过或者新添加的方法加入DiffInfo中：  
![](http://i.imgur.com/3VCiXX2.png)  

	![](http://i.imgur.com/LYK2he3.png)
- 可以看到对比两个方法的不同时是通过getImplementation()进行对比，getImplementaion得到一个DexBackedMethodImplementation对象，而andfix做的一个重要处理就是给DexBackedMethodImplementation添加equals的实现 ： 
![](http://i.imgur.com/h7fc811.png)
getInstrctions()得到一个DexBackedInstruction对象，一样也为其实现equals方法：  
![](http://i.imgur.com/9e8ezFI.png)  
而DexBackedInstruction又有许多子类，要为其所有的子类加上equals方法：  
![](http://i.imgur.com/jQKyZU3.png)

2 为DiffInfo中的方法添加注解   
从1中得到了修改过的方法集合之后将这些class转换成smali文件，andfix在写入smali文件之前为这些修改过的方法添加了注解，解析时判断如果有此注解便认为此方法需要更新。写入smali文件的方法在org.jf.baksmali.Adaptors.ClassDefinition的writeTo()中：  

![](http://i.imgur.com/JJ26eQq.png)  

andfix在writeDirectMethods和writeVirtualMethods中都做了添加注解处理，MethodReplaceAnnotation便是自定义的注解类：

![](http://i.imgur.com/QJyZqj1.png)  


而在org.jf.dexlib2.dexbacked.DexBackedMethod中需要添加这个额外的字段和方法：  

![](http://i.imgur.com/uFxJmdO.png)
![](http://i.imgur.com/PQ7JOAV.png)  

并且在get方法中将额外的annotion一并加入：  
![](http://i.imgur.com/0yz2mvO.png)  


3 添加_CF后缀  
将andfix通过jar包生成的dex文件转换成jar包后查看可发现类名多了“_CF”后缀，主要通过TypeGenUtil实现。：  
![](http://i.imgur.com/Rj25yPg.png)  
在jg-gui中全局搜索TypeGenUtil:  
![](http://i.imgur.com/nFGKcsL.png)  
在每个地方都使用TypeGenUtil进行字符串替换即可。


目前将修改后的代码生成patch文件，放在gradle/smali-patches中:  
![](http://i.imgur.com/TbZpAC0.png)

0001-baksmali-method-access.patch : 修改了baksmali中的disassembleClass方法为

0002-dexlib2-addFiles.patch： 添加了andfix源码

0003-baksmali-dexlib2-inject-annotion.patch ： 添加注解

0004-dexlib2-add-equals-implements.patch：添加了equals方法实现

0005-baksmali-add-subffix-_CF.patch：添加CF后缀

0006-baksmali-cancel-some-test.patch： 注释部分报错测试代码

0007-dexlib2-add-instuction-equals.patch： 为DexBackedInstruction子类添加equals方法实现

参考：[http://sunzeduo.blog.51cto.com/2758509/1540085](http://sunzeduo.blog.51cto.com/2758509/1540085)
