#Android性能优化----工具篇
<br/><br/>

##概览
<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Android 提供了多种工具帮助开发者调试android程序，保证应用的性能和稳定，如果你知道某一个工具，那么百度或者google会查到很多详细的介绍与帮助文档，但通常的情况是 初学的开发者们并不清楚都有哪些工具可以帮助他们开发。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本文的目的便是如此，下面按照工具所处环境 手机端工具、编译器端工具进行分类概览。之后我们将按照 UI、内存、代码审查等方面对下列工具进行功能描述，具体使用帮助，还请移步百度/google：

<br/>

####【手机端】
- **[Show GPU Overdraw](#show)**（调试GPU过度绘制） 检测过度绘制					
- **[Profile GPU Rendering](#GPU呈现模式分析)**（GPU呈现模式分析） 每帧画面所需要渲染的时间。			
- **[Show GPU view updates](#显示GPU视图更新)**（显示GPU视图更新）当GPU正在绘图时，闪烁显示	
- **[Strict Mode](#严格模式)** （严格模式） 当程序在主线程上长时间执行操作闪烁屏幕

<br/>

####【PC/编译环境】

- **[Memory Monitor](#内存监控)**：查看整个app所占用的内存，以及发生GC的时刻
- **[Allocation Tracker](#内存分配)**：使用此工具来追踪内存的分配。
- **[Heap Tool](#内存快照)**：查看当前内存快照，便于对比分析哪些对象有可能是泄漏了的。
- **Battery History Tool**  电量审查工具（Android 5.0，不属于编译器的工具） 
- **[traceview](#计算方法时间)** 工具（计算每个方法占用CPU时间）
- **[Networking Traffic Tool](#网络监控)** （android studio）网络请求发生的时间，每次请求的数据量等信息
- **[Hierarchy viewer](#层级显示工具)** 层级显示工具
- **[Lint](#代码审查)** 代码审查工具 给出代码优化建议



<br/><br/><br/>

##功能与启动方式描述

<br/>

         
###【UI】Show GPU Overdraw 调试过渡绘制[](id:show)     

<br/>
&nbsp;&nbsp;**[描述]**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Overdraw(过度绘制)描述的是屏幕上的某个像素在同一帧的时间内被绘制了多次。在多层次的UI结构里面，如果不可见的UI也在做绘制的操作，这就会导致某些像素区域被绘制了多次。这就浪费大量的CPU以及GPU资源。

<br/>

**【启动方式】**	

打开Android 开发者模式，找到”调试GPU过度绘制选项”，选择“显示过度绘制区域”。


![过度绘制调试](http://img.blog.csdn.net/20151015001029215)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开我们的应用，显示效果如下：

 <img src="http://img.blog.csdn.net/20151015001103186" width = "200" height = "" alt="调试过渡绘制"  align= center  />


<br/>


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;屏幕上我们可以看到各种颜色：
    
- 无/白色：绘制1次    
- 蓝色：绘制2次（理想状态）		
- 绿色：绘制3次			
- 浅红：绘制4次（要优化了）		
- 深红：绘制5次或5次以上。（必须要优化了）

<br/>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**过度绘制对性能的影响是不能忽视的，尤其在复杂布局上显示动画效果。而常规的编码很难避免过度绘制，我们需要采取一些特殊的手段。具体关于如何优化过度绘制，我们在下一篇文章中详解.**

<br/><br/><br/>
[](id:GPU呈现模式分析)
###【UI】Profile GPU Rendering  GPU呈现模式分析

<br/>

**[描述]**	
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了保证给用户流畅的界面显示，我们应该做到界面的渲染时间保持在16ms一下，Android提供了工具给我测试我们的界面每一帧的渲染时间。由此我们可以观察我们界面找到耗时较长的部分进行优化。		

<br/>

**【启动方式】**	
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开android开发者模式，找到“GPU呈现模式分析”，我们可以选择显示为条形或线型。


![GPU呈现模式分析](http://img.blog.csdn.net/20151015001436885)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之后打开我们的应用

 <img src="http://img.blog.csdn.net/20151015001504897" width = "200" height = "" alt="调试GPU呈现模式分析"  align= center  />

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;随着界面的刷新，界面上会滚动显示垂直的柱状图来表示每帧画面所需要渲染的时间，柱状图越高表示花费的渲染时间越长。	

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;中间有一根绿色的横线，代表16ms，我们需要确保每一帧花费的总时间都低于这条横线，这样才能够避免出现卡顿的问题。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每一条柱状线都包含三部分，蓝色代表测量绘制Display List的时间，红色代表OpenGL渲染Display List所需要的时间，黄色代表CPU等待GPU处理的时间。

<br/>

**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;值得注意的是，从工具的检查效果上来看，除了常规的View绘制性能以外，手机性能、数据加载都会被分摊到每帧画面的绘制时间上，从而超过16ms线**

<br/><br/>
[](id:Hierarchy viewer)
###【UI】Hierarchy viewer层级显示工具
<br/>

**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;影响界面渲染效率的因素除了过渡绘制以外，过于复杂和过深的层级嵌套也是元凶之一。Android sdk 提供了Hierarchy viewer这个工具来帮助我们查看界面的层级嵌套关系。同时我们也可以通过这个工具了解每个一个View的绘制时间。

<br/>

**【启动方式】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先连接手机到电脑.
然后打开 sdk 目录下的tools文件夹，找到“Hierarchy viewer.bat”文件，双击启动.

 <img src="http://img.blog.csdn.net/20151015001610598" width = "400" height = "300" alt="调试过渡绘制"  align= center  />


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选择我们要查看的Activity，然后点击 Load View Hierarchy .

 <img src="http://img.blog.csdn.net/20151015001748544" width = "400" height = "300" alt="view 层级显示"  align= center  />


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这时我们可以看到我们activity布局嵌套关系，并且点击view后可以看到这个View和它的子View们绘制时间的总和。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;另外我们还可以看到View的下方有三个小点，这三个小点和他们的颜色含义是：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从左到右：测量、布局、画图时间

- 红色：该View所用时间超过大部分View很多（改找原因优化了）
- 黄色：该View所用时间超过大部分View
- 绿色：该View所用时间低于大部分View

**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;值得注意的是，每一次载入View的绘制时间都是不同的，受手机影响很大，单次的测量时间并不能作为依据。只能在同一台手机优化前后做对比，并且还应该多次测量。		
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果在优化后想了解确切的优化数据，使用这个工具是很不错的选择。因为Hierarchy viewer所测得时间是纯View加载，不会受到数据加载影响的。**


**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果在使用过程中，你突然你的发现你的Hierarchy viewer工具没有时间显示了，请移步[Hierarchyviewer时间不显示](http://blog.csdn.net/u010255127/article/details/47761507)**
<br/><br/>
[](id:严格模式)
###【UI】Strict Mode 严格模式

**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;android应用的主线程是UI线程，我们不应该在UI线程中操作复杂费时的事情，负责容易导致UI界面卡死。出现ANR。Android系统为了开发提供了工具 ： 严格模式，我们可以通过这个工具来查看我们的界面是否在主线程中做了耗时操作。
当我们的界面在做耗时操作时，屏幕会以闪烁来提醒。


**【启动方式】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开android开发者选项，找到 “启动严格模式” 开启。

![这里写图片描述](http://img.blog.csdn.net/20151015001841156)




**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通常来说，我们的应用不会产生较大问题。一般的场景为：主线程操作数据库，大量数据的存取导致线程阻塞，所以如果你的应用对数据的操作频繁且量大，建议放入子线程中。**



<br/><br/>

[](id:显示GPU视图更新)
####【UI】how GPU view updates显示GPU视图更新

<br/>

**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Android系统还提供了一个工具，当GPU在更新视图时，更新趋于会闪烁提醒。此工具暂时还未发现作用。

<br/>

**【启动方式】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开android开发者选项，找到 “显示GPU视图更新” 开启。

![图片](http://img.blog.csdn.net/20151015001917994)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当使用GPU进行绘图是，闪烁显示窗口中的视图。

**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个功能看起来似乎有点鸡肋。这里介绍一个技术技巧：硬件加速。从Android3.0 (API level 11)开始，Android的2D显示管道被被设计得更加支持硬加速了．硬加速使用GPU承担了所有在View的canvas上执行的绘制操作。**

		androidmanifest.xml中application添加 
  		android:hardwareAccelerated="true"。
  		需要注意的是：android 3.0以上才可以使用。


**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但带来方便的同时，似乎会引起一些奇怪的问题，详情移步：**[Android硬件加速的一些问题和错误](http://http://blog.csdn.net/icyfox_bupt/article/details/18732001)
<br/><br/>

[](id:代码审查)
###【代码审查】Lint 静态代码审查工具

<br/>


**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Lint是Android提供的一个静态扫描应用源码并找出其中的潜在问题的一个强大的工具。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;例如，如果我们在onDraw方法里面执行了new对象的操作，Lint就会提示我们这里有性能问题，并提出对应的建议方案。Lint已经集成到Android Studio中了，我们可以手动去触发这个工具，点击工具栏的Analysis -> Inspect Code，触发之后，Lint会开始工作，并把结果输出到底部的工具栏，我们可以逐个查看原因并根据指示做相应的优化修改。

<br/>

**【启动方式】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;运行Lint工具，在Android Studio的“Analyze”菜单中选择“Inspect Code…”。当Android Studio完成了对项目的检测之后，它会在窗口底部显示出分析结果。

 <img src="http://img.blog.csdn.net/20151015002159522" width = "500" height = "250" alt="Lint工具"  align= center  />

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**通过实际使用发现，Lint检查出了所有我们项目中所有不合理的地方，有的问题优先级非常低不需要修改，好在Lint提供了筛选功能。具体用法还带考量。**

<br/><br/>

[](id:内存监控)
###【内存】Memory Monitor 内存监视器

<br/>

**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Memory Monitor Android studio提供的可以动态的监视内存使用工具。可以很好的帮助我们查看程序的内存使用情况。


![这里写图片描述](http://img.blog.csdn.net/20151015002312806)



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果你在Memory Monitor里面查看到短时间发生了多次内存的涨跌，这意味着很有可能发生了内存抖动。发生了频繁的GC操作。这里我们就应该优化了。

**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;引起内容抖动最常见的原因是在循环逻辑中创建了对象，常见的循环逻辑有：for，while循环控制语句，onDraw()方法，ListView的getView方法等等，我们应避免在这些逻辑中创建对象。**

<br/><br/>
[](id:内存分配)
###【内存】Allocation Tracker 跟踪内存分配

<br/>

**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当你用Memory Monitor监测到内存抖动后，可以使用Allocation Tracker来进行内存分配定位。通过allocation tracker，不仅知道分配了哪类对象，还可以知道在哪个线程、哪个类、哪个文件的哪一行。

<br/>

**【启动方式】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;运行DDMS，只需简单的选择应用进程并单击Allocation tracker标签，就会打开一个新的窗口，单击“Start Tracing”按钮；
然后，让应用运行你想分析的代码。运行完毕后，单击“Get Allocations”按钮，一个已分配对象的列表就会出现第一个表格中。
单击第一个表格中的任何一项，在表格二中就会出现导致该内存分配的栈跟踪信息。


![Allocation Tracker](http://img.blog.csdn.net/20151015002422353)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;尽管在性能关键的代码路径上移除所有的内存分配操作不是必须的，甚至有时候是不可能的，但allocation tracker可以帮你识别代码中的一些重要问题。举例来说，许多应用中发现的一个普遍错误：每次进行绘制都创建一个新的Paint对象。将Paint的创建移到一个实例区域里，是一个能极大提高程序性能的简单举措。

<br/><br/>
[](id:内存快照)
###【内存】Heap Tool 内存快照

<br/>

**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Heap Tool可以查看当前的内存快照. Heap Tool提供的是一个内存的总体情况，图表显示的内容比较简单，如果要具体分析的话最好生成.hprof文件，使用MAT工具进行分析。

![Heap Tool](http://img.blog.csdn.net/20151015002453931)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;额……说实话这个内存快照看不懂……

<br/><br/>
[](id:计算方法时间)
###【内存】Traceview 计算方法占用CPU时间

<br/>

**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过traceview 我们可以得到我们每个执行的方法所占用CPU的时间，也就是说如果我们发现某个界面发生卡顿，那么我们可以通过这个工具定位到这个方法。
具体的使用参考：http://www.oschina.net/news/56500/traceview-android

<br/>

**【启动方式】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;打开DDMS，选择一个进程，然后按上面的“Start Method Profiling”按钮，等红色小点变成黑色以后就表示TraceView已经开始工作了。然后我就可以滑动一下列表（现在手机上的操作肯定会很卡，因为Android系统在检测Dalvik虚拟机中每个Java方法的调用，这是我猜测的）。操作最好不要超过5s，因为最好是进行小范围的性能测试。然后再按一下刚才按的按钮，等一会就会出现上面这幅图，然后就可以开始分析了。

![这里写图片描述](http://img.blog.csdn.net/20151015002521844)

<br/><br/>
[](id:网络监控)
###【网络】Networking Traffic Tool

<br/>

**【描述】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看网络请求发生的时间，每次请求的数据量等信息

<br/>

**【启动方式】**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在DDMS中 选择要进程，点击Network Statistics标签 即可。

![这里写图片描述](http://img.blog.csdn.net/20151015002559943)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;应该是有更多选项和显示的……可是不知道在哪里修改……

<br/>

**&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过这个工具我们可以详细查看我们的应用流量情况，包括网络包得大小，传输速度，传输平率等等，最简单的功能：当你对你的网络传输数据进行了压缩，但有不知道具体效果如何，可以使用这个工具。另外，当你发现你的应用有大量短小且平凡的请求是，可以考虑将请求合并，从而节省手机电量与流量。**




