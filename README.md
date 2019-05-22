# Android.Activity-notes
Android四大组件之Activity的一些理解和整理。结合了入门的‘’第一行代码‘’和进阶的‘’Android开发艺术探索‘’两本书和一个实例Demo~

## 开发艺术探索之Activity
> 该部分主要是个人对开发艺术探索此部分的一些整理和笔记主要包括：
  * Activity的生命周期全面分析
  * Activity的启动模式
  * IntentFilter的匹配规则
---
### 1. Activity的生命周期全面分析
> 生命周期主要包括两种情况：1.用户正常使用情况下的生命周期  2.由于Activity被系统回收或者设备配置改变导致Activity被销毁重建情况下的生命周期。

**1.1 用户正常使用情况下的生命周期**
Activity的生命周期和启动模式：
![图片](http://images2015.cnblogs.com/blog/776204/201511/776204-20151130161336030-1114688010.png)
* Activity第一次启动：必然走完onCreate->onStart->onResume。
* Activity切换到后台（ 用户打开新的Activity或者切换到桌面）：根据被启动的新的是否透明来判断生命周期的回调，onPause->onStop(如果新Activity采用了透明主题，则当前Activity不会回调onstop)。
* Activity从后台到前台：重新可见，onRestart->onStart->onResume。
* 彻底的退出Activity：onPause->onStop->onDestory。
* 可见性：onStart则开始可见、onStop之后则不可见 所以onStart开始到onStop之前，Activity可见。
* 交互、具有焦点：onResume后开始获得焦点、onPause后么得焦点了 所以onResume到onPause之前，Activity可以接受用户交互。
* ps：在新Activity启动之前，栈顶的Activity需要先onPause后，新Activity才能启动。所以不能在onPause执行耗时操作。然而onstop中也不可以太耗时，资源回收和释放可以放在onDestroy中。

**1.2 异常情况下的生命周期分析**
* 系统配置变化导致Activity销毁重建
> 系统配置的变化一般指的是比如旋转屏幕了原来的竖屏变成了横屏这时就会销毁原Activity重新创建，对于这样的情况一般就是通过在变化前通过onStop之前调用onSaveInstanceState来保存状态。然后在创建的时候可以通过Bundle的传入在onStart之后调用onRestoreInstanceState来恢复之前保存的数据。具体的流程：通过onSaveIntanceState保存Bundle数据-->然后作为中介的Window，Activity委托Window、Window委托它上面的顶级容器一个ViewGroup-->最后通过顶层的这个容器来保存数据。此处特别类似事件的分发机制。
* 资源内存不足导致低优先级的Activity被回收
> 首先Activity的优先级可以分为前台最高、其次可见非前台、后台。除了上述的方法来保存数据，也可以直接通过Manifest来设置android:configChanges=”orientation”这样在系统配置变化后不重新创建Activity，也不会执行 onSaveInstanceState 和onRestoreInstanceState 方法，而是调用 onConfigurationChnaged 方法。
>> configChanges 一般常用三个选项：1.locale 系统语言变化2.keyborardHidden 键盘的可访问性发生了变化，比如用户调出了键盘3.orientation 屏幕方向变化

### 2. Activity的启动模式
**2.1 Activity的LaunchMode启动模式**
> Android中使用活动栈来管理Activity。
* standrd：就是标准模式，不管怎么样只要启动一次活动就会创建一个实例在活动栈中，并且无论这个活动是否曾经存在过。具体该Activity属于哪个活动栈是看是谁启动了该Activity。但是其实也都是Activity启动Activity因为如果用Application，他的context是没有任务栈的。一个待启动的Activity可以通过设置标志位为FLAG_ACTIVITY_NEW_TASH这样就会为这个Activity单独创建一个活动栈了
* singleTop：栈顶复用模式，如其名一样就是如果待启动的Activity是在该活动栈的栈顶 则不去创建直接复用 并且会回调onNewIntent方法。这样待创建的Activity的onCreate和onStart方法也不会被执行。应用：比如浏览器的书签模式 ps：可以一个活动栈重复创建的 允许多个
* singleTask：栈内复用模式，依旧如其名只要栈内有就不会再次创建了 是一种单实例的模式 如果有就撸光上面的把这个打头并且调用onNewIntent方法 只要不存在的时候才会创建一个实例。应用：比如浏览器的主页模式
* singleInstance：么得什么好说的 就是此模式的Activity一个独自位于一个活动栈中
* 问题：假设两个任务栈，前台任务栈为12，后台任务栈为XY。Y的启动模式是singleTask。现在请求Y，整个后台任务栈会被切换到前台。
* 设置启动模式：1.manifest中 设置下的 android:launchMode 属性 2.启动Activity的 intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK); 3.如果同时存在一律以第二个为准。第一种无法裸用FLAG_ACTIVITY_CLEAR_TOP 第二种无法singleInstance 4.可以通过命令行 adb shell dumpsys activity 命令查看栈中的Activity信息

**2.2 Activity的Flags**
> 在Manifest中可以通过这些FLAG可以设定启动模式、可以影响Activity的运行状态。
* FLAG_ACTIVITY_NEW_TASK：为Activity指定“singleTask”启动模式
* FLAG_ACTIVITY_SINGLE_TOP：为Activity指定“singleTop”启动模式
* FLAG_ACTIVITY_CLEAR_TOP：其实是半标志位，要配合使用的 就是用来撸光他上面所有的Activity的FLAG_ACTIVITY_NEW_TASK配合实现singleTask
* FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS：就是该Activity不会在最近启动的Activity的列表中

### 3. IntentFilter的匹配规则

* **Activity调用方式**
> 1.显示调用：直接包括本.this和被启动的.class 包括包名和类名  
  2.隐式调用：不通过包名啥的，通过IntentFilter的匹配规则来找到满足这些过滤信息的组件 必须全部满足才行
  
* **匹配规则**
> 1.首先IntentFilter包含的过滤信息有action、category、data  
  2.要启动一个activity必须同时匹配以上三者才行，缺一不可   
  3.但是一个Activity可以有多个IntentFilter 也就是说可以有多种打开方式
  
* **action**
> 这里action是字符串 匹配就是完全一样的意思 但是IntentFilter里可以有多个action 也就是同理咯 有一个action匹配了就代表了该部分匹配了
* **category**
> 同样也是字符串  与上面不同的是 这里的category可以没有的可以没有的，但是只要Intent里有 有多少都得与IntentFilter中完全完全一样才行   
  另外 在一般的startActivity中都会默认的android.intent.category.DEFAULT 这个category 所以啊为了匹配category一般至少有个android.intent.category.DEFAULT
* **data**
> 1.首先讲下 data的构成==mimeType + URI mimeType是媒体类型比如是image/jpeg、audio/mpeg4-generic和video/等，可以表示图片、文本、视频等不同的  
> 2.而URI的结构：  
  ` <scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>] --> 
    http://www.baidu.com:80/search/info`   
>  * scheme：URI的模式，比如http、file、content等，默认值是 file 
>  * host：URI的主机名:www.baidu.com
>  * port：URI的端口号:80
>  * path、pathPattern和pathPrefix：这三个参数描述路径信息:search/info  
>3.data的写法：
  ```
  <data android:scheme="string"
    android:host="string"
    android:port="string"
    android:path="string"
    android:pathPattern="string"
    android:pathPrefix="string"
    android:mimeType="string"/>
  ```     
> 4.具体的匹配规则：
>  * 这里是反的哦 就是IntentFilter定义了data则Intent必须要有可匹配的data
>  * 细节和特例上 如果IntentFilter没有定义data 但是URI都是有默认的content和file的 所以为了匹配 Intent里的URI的scheme也就是URI的模式必须设置为> content或者file 另外在Intent设置data 的时候必须使用setDataAndType的方法 因为只设置setData 和 setType会清除另一方的值
 * **隐式调用需注意**
 > * 首先是异常的问题：必须有Activity能匹配我们的隐式Intent 否则会抛出异常android.content.ActivityNotFoundException 
 > * 如果作为入口Activity：以下缺一不可
 ```
 <action android:name="android.intent.action.MAIN" />
 <category android:name="android.intent.category.LAUNCHER" />
 ```
