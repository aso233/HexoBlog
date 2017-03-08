---
title: xutils HttpUtils
date: 2016-06-11 18:13:00
tags:
---
# xutils HttpUtils #

近期由于项目需要开始研究学习xuitls框架，xutils是一款相当比较成熟的框架，主要模块涵盖了界面，数据库，网络以及图片缓存处理，已经基本能够满足一个app所需，此次重点学习其网络模块 HttpUtils。  
github项目托管地址: [https://github.com/wyouflf/xUtils.git](https://github.com/wyouflf/xUtils.git)

HttpUtils主要提供了以下几种接口：  
1. configxxxxx： 配置参数信息，包括：返回结果字符编码，Http本地缓存大小、有效期等，超时时间。  
2. sendxxx：发送请求接口，包括常规的send和sendSync  
3. download：下载接口，重载了多种不同参数配置方法  
xUtils网络通信方式使用了HttpClinet，在HttpUtils构造方法中便初始化了HttpClient，并初始化了部分参数。  

  
此次我们重点分析xutils的下载接口，分析之前先要了解几个关键的java类：Callable, FutrueTask。  
   
Callable：类似Runnable接口，都可以被其他线程执行，Runnable声明方法为run(),Callable声明方法为call();Callable有返回值且可抛出异常，callable一般结合FutureTask进行使用。  

Future：提供访问任务执行结果的接口。  
![](http://i.imgur.com/3zTHmHy.png)  
像之前使用Runnable开始执行任务之后，我们就无法获取任务执行的状态，也无法在任务完成后得到返回值，只能通过公共变量或者接口来实现。通过Futrue和Callable可以实现获取其他线程执行状态，任务执行完成后也可得到返回结果。从Java 1.5开始，就提供了Callable和Future，应该也是为了弥补Runnable的不足。


了解了上述两个关键的java类之后，我们来看看xUtils的下载接口实现：
![](http://i.imgur.com/VD2dx4Y.png)  
调用下载接口后，首先进行一些参数过滤之后创建了HttpRequest对象，封装保存请求参数信息，然后创建了HttpHandler对象，重点就是这个HttpHandler类。它继承了一个实现了Futrue接口的类PriorityAsyncTask<Params, Progress, Result>,在创建实例时便将httpClinet，charset等信息通过构造传递进去，然后设置了一些其他参数后调用了handher.executeExecutor()方法。  
![](http://i.imgur.com/Vg43hgO.png)  
在executeOnExecutor方法中，我们看到了onPreExecute(),是不是很眼熟？没错，就是android异步处理任务AyncTask中的方法。实际上，xutils的http多线程框架是沿用了AyncTask的异步处理机制。下面我们跟着代码分析简单分析一下：  
我们看到了mWoker,mFutrue两个变量，他们在构造方法中进行了初始化：  
![](http://i.imgur.com/NlGXtCG.png)    
![](http://i.imgur.com/Z8jl2mI.png)  
mWoker实现了Callable接口，FutureTask实现了Futrue接口，他们结合在一起便是为了实现我们之前说过的获取其他线程执行状态的功能。在mWoker call()方法中，我们看到熟悉的doInBackground(),这里的mParams就是在executeOnExecutor开始时设置的params。  
doInBackground()是抽象方法，我们在PriorityAsyncTask的子类HttpHandler中看到了其具体实现：  
![](http://i.imgur.com/prWdR2C.png)  
我们看到了真正发送请求以及接受响应的地方。也看到另一个熟悉的方法publishProgress()  
![](http://i.imgur.com/9wMvyvK.png)  
![](http://i.imgur.com/uorSaPp.png)   
![](http://i.imgur.com/eEYsO9d.png)  
可以发现，AsyncTask实际也是通过内部的一个Handler进行UI线程消息的处理，到此我们已经见到了AsyncTask大多数的方法了。  

刚才说到了在构造中初始化了mWorker，mFuture两个对象，回到handher.executeExecutor()方法  
![](http://i.imgur.com/Vg43hgO.png)   
我们看到调用了exec.execute()的方法，这个就是使用线程池来执行Futrue实例，exec是download方法传入
handler.executeOnExecutor(EXECUTOR, request, target, autoResume, autoRename)的EXECUTOR  
![](http://i.imgur.com/lNd0Nr6.png)  
![](http://i.imgur.com/l4Drb1x.png)  
可以看出，xUtils内部使用了一个基于优先级的自定义线程池PriorityExecutor，拥有一个优先级缓存队列PriorityObjectBlockingQueue，可通过下载接口参数配置任务的优先级，当前使用的线程池最大容量为256，线程活跃队列为5个,等待时间1s。

通过对xutils下载模块的学习，深入理解了AsyncTask的工作原理，发现对JAVA并发线程许多东西不太了解，如果不理解其中几个关键的JAVA类，理解框架的内容就比较吃力，今后还需多看点这方面的资料。
  
参考：  
[http://www.cnblogs.com/dolphin0520/p/3949310.html](http://www.cnblogs.com/dolphin0520/p/3949310.html)  
 [http://blog.csdn.net/lmj623565791/article/details/38614699](http://blog.csdn.net/lmj623565791/article/details/38614699)