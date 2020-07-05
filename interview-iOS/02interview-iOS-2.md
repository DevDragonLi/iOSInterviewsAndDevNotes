## interview-iOS -2 


## weak修饰的释放则自动被置为nil的实现原理

<details>
<summary> 参考内容 </summary>

- Runtime维护着一个Weak表，用于存储指向某个对象的所有Weak指针
- Weak表是Hash表，Key是所指对象的地址，Value是Weak指针地址的数组
- 在对象被回收的时候，经过层层调用，会最终触发下面的方法将所有Weak指针的值设为nil。
- runtime源码，objc-weak.m 的 arr_clear_deallocating 函数
- weak指针的使用涉及到Hash表的增删改查，有一定的性能开销.

</details>

## HTTPS的加密原理

<details>
<summary> 参考内容 </summary>

- 服务器端用非对称加密(RSA)生成公钥和私钥
- 然后把公钥发给客户端, 服务器则保存私钥
- 客户端拿到公钥后, 会生成一个密钥, 这个密钥就是将来客户端和服务器用来通信的钥匙
- 然后客户端用公钥对密钥进行加密, 再发给服务器
- 服务器拿到客户端发来的加密后的密钥后, 再使用私钥解密密钥, 到此双方都获得通信的钥匙

</details>

## 网络通讯中加密方式有哪些，各自的原理?
<details>
<summary> 参考内容 </summary>

- md5(哈希算法)：把任意长度的字符串加密成一个128bit的大整数，并且是不可逆的 (已不再安全)
- RSA(非对称算法加密)：产生一对非对称的公钥和私钥，公钥加密，私钥解密。私钥加密，公钥解密
- AES(对称加密)：加密和解密的密钥是同一个
- base64(现代密码学的基础)：原本8 bit一组的数据改为6bit一组，不足的地方补0，每两个0用一个 = 表示 

</details>

## 开发中iOS缓存的理解

<details>
<summary> 参考内容 </summary>

> 一般`缓存` 针对展示类的UI层数据比较多.比如可举例个人资料界面. 

- 网络优先：开始总是从网络获取，如果获取失败，从本地获取(旧数据)。
- 本地优先：在一段时间内从本地获取，当超过这个时间，然后重新请求网络数据(展示同时覆盖旧数据)。
- 混合(智能)：打开程序先从本地获取展示，然后请求数据，请求完成后刷新界面(一般而言)。

</details>

## 你认为开发中那些导致crash?

<details>
<summary> 参考内容 </summary>

> 当iOS设备上的App应用闪退时，操作系统会生成一个crash日志，保存在设备上。crash日志上有很多有用的信息，比如每个正在执行线程的完整堆栈跟踪信息和内存映像，这样就能够通过解析这些信息进而定位crash发生时的代码逻辑，从而找到App闪退的原因。

> 通常来说，crash产生来源于两种问题：违反iOS系统规则导致的crash和App代码逻辑BUG导致的crash

### 1.应用逻辑的Bug

- SEGV：（Segmentation Violation，段违例），无效内存地址，比如空指针，未初始化指针，栈溢出等；
- SIGABRT：收到Abort信号，可能自身调用abort()或者收到外部发送过来的信号；
- SIGBUS：总线错误。与SIGSEGV不同的是，SIGSEGV访问的是无效地址（比如虚存映射不到物理内存），而SIGBUS访问的是有效地址，但总线访问异常（比如地址对齐问题）；
- SIGILL：尝试执行非法的指令，可能不被识别或者没有权限；
- SIGFPE：Floating Point Error，数学计算相关问题（可能不限于浮点计算），比如除零操作；
- SIGPIPE：管道另一端没有进程接手数据；
常见的崩溃原因基本都是代码逻辑问题或资源问题，比如数组越界，访问野指针或者资源不存在，或资源大小写错误等

### 2.违反iOS系统规则产生crash的三种类型
- 内存报警闪退
	- 当iOS检测到内存过低时，它的VM系统会发出低内存警告通知，尝试回收一些内存；如果情况没有得到足够的改善，iOS会终止后台应用以回收更多内存；最后，如果内存还是不足，那么正在运行的应用可能会被终止掉。在Debug模式下，可以主动将客户端执行的动作逻辑写入一个log文件中，这样程序童鞋可以将内存预警的逻辑写入该log文件，当发生如下截图中的内存报警时，就是提醒当前客户端性能内存吃紧，可以通过Instruments工具中的Allocations 和 Leaks模块库来发现内存分配问题和内存泄漏问题。

- 响应超时
	- 当应用程序对一些特定的事件（比如启动、挂起、恢复、结束）响应不及时，苹果的Watchdog机制会把应用程序干掉，并生成一份相应的crash日志。

- 用户强制退出
  - 一看到“用户强制退出”，首先可能想到的双击Home键，然后关闭应用程序。不过这种场景一般是不会产生crash日志的，因为双击Home键后，所有的应用程序都处于后台状态，而iOS随时都有可能关闭后台进程，当应用阻塞界面并停止响应时这种场景才会产生crash日志。这里指的“用户强制退出”场景，是稍微比较复杂点的操作：先按住电源键，直到出现“滑动关机”的界面时，再按住Home键，这时候当前应用程序会被终止掉，并且产生一份相应事件的crash日志。


</details>  

## SDWebImage 


<details>
<summary> 参考内容 </summary>

### 1.SDWebImage 加载图片的流程


1.入口 setImageWithURL:placeholderImage:options: 会先把 placeholderImage 显示，然后 SDWebImageManager 根据 URL 开始处理图片。

2.进入 SDWebImageManager-downloadWithURL:delegate:options:userInfo:，交给 SDImageCache 从缓存查找图片是否已经下载 queryDiskCacheForKey:delegate:userInfo:.

3.先从内存图片缓存查找是否有图片，如果内存中已经有图片缓存，SDImageCacheDelegate 回调 imageCache:didFindImage:forKey:userInfo: 到 SDWebImageManager。

4.SDWebImageManagerDelegate 回调 webImageManager:didFinishWithImage: 到 UIImageView+WebCache 等前端展示图片。

5.如果内存缓存中没有，生成 NSInvocationOperation 添加到队列开始从硬盘查找图片是否已经缓存。

6.根据 URLKey 在硬盘缓存目录下尝试读取图片文件。这一步是在 NSOperation 进行的操作，所以回主线程进行结果回调 notifyDelegate:。

7.如果上一操作从硬盘读取到了图片，将图片添加到内存缓存中（如果空闲内存过小，会先清空内存缓存）。SDImageCacheDelegate 回调 imageCache:didFindImage:forKey:userInfo:。进而回调展示图片。

8.如果从硬盘缓存目录读取不到图片，说明所有缓存都不存在该图片，需要下载图片，回调 imageCache:didNotFindImageForKey:userInfo:。

9.共享或重新生成一个下载器 SDWebImageDownloader 开始下载图片。

10.图片下载由 NSURLConnection 来做，实现相关 delegate 来判断图片下载中、下载完成和下载失败。

11.connection:didReceiveData: 中利用 ImageIO 做了按图片下载进度加载效果。

12.connectionDidFinishLoading: 数据下载完成后交给 SDWebImageDecoder 做图片解码处理。

13.图片解码处理在一个 NSOperationQueue 完成，不会拖慢主线程 UI。如果有需要对下载的图片进行二次处理，最好也在这里完成，效率会好很多。

14.在主线程 notifyDelegateOnMainThreadWithInfo: 宣告解码完成，imageDecoder:didFinishDecodingImage:userInfo: 回调给 SDWebImageDownloader。

15.imageDownloader:didFinishWithImage: 回调给 SDWebImageManager 告知图片下载完成。

16.通知所有的 downloadDelegates 下载完成，回调给需要的地方展示图片。

17.将图片保存到 SDImageCache 中，内存缓存和硬盘缓存同时保存。写文件到硬盘也在以单独 NSInvocationOperation 完成，避免拖慢主线程。

18.SDImageCache 在初始化的时候会注册一些消息通知，在内存警告或退到后台的时候清理内存图片缓存，应用结束的时候清理过期图片。

19.SDWI 也提供了 UIButton+WebCache 和 MKAnnotationView+WebCache，方便使用。

20.SDWebImagePrefetcher 可以预先下载图片，方便后续使用。


### 2. SDImageCache是怎么做数据管理的？

  SDImageCache分两个部分，一个是内存层面的，一个是硬盘层面的。内存层面的相当是个缓存器，以Key-Value的形式存储图片。当内存不够的时候会清除所有缓存图片。用搜索文件系统的方式做管理，文件替换方式是以时间为单位，剔除时间大于一周的图片文件。当SDWebImageManager向SDImageCache要资源时，先搜索内存层面的数据，如果有直接返回，没有的话去访问磁盘，将图片从磁盘读取出来，然后做Decoder，将图片对象放到内存层面做备份，再返回调用层。


### 3.内部做Decoder的原因 (典型的空间换时间)
  由于UIImage的imageWithData函数是每次画图的时候才将Data解压成ARGB的图像，所以在每次画图的时候，会有一个解压操作，这样效率很低，但是只有瞬时的内存需求。为了提高效率通过SDWebImageDecoder将包装在Data下的资源解压，然后画在另外一张图片上，这样这张新图片就不再需要重复解压了   
  
 
</details>
  
## crash的收集和定位bug的方式
<details>
<summary> 参考内容 </summary>

- iTunes Connect（Manage Your Applications - View Details - Crash Reports),但是前提用户设置->隐私->诊断与用量->诊断与用量数据开启.一般不推荐

- 自己实现应用内崩溃收集，并上传服务器.(收集异常，存储到本地，下次用户打开程序时上传给我们)
  - 在程序启动时加上一个异常捕获监听，用来处理程序崩溃时的回调动作UncaughtExceptionHandler是一个函数指针，该函数需要我们实现，可以取自己想要的名字。当程序发生异常崩溃时，该函数会得到调用，这跟C，C++中的回调函数的概念是一样的

  ```
	  NSSetUncaughtExceptionHandler (&UncaughtExceptionHandler)。 程序启动代理方法
	  //:collection crash info by DragonLi
	void UncaughtExceptionHandler(NSException *exception) {
	    NSArray *callStack = [exception callStackSymbols];
	    NSString *reason = [exception reason];
	    NSString *name = [exception name];
	    
	    NSDateFormatter *formatter = [[NSDateFormatter alloc] init];
	    [formatter setDateFormat:@"YYYY-MM-dd HH:mm:ss"];
	    NSString * dateStr = [formatter stringFromDate:[NSDate date]];
	    
	    NSString * iOS_Version = [[UIDevice currentDevice] systemVersion];
	    NSString * PhoneSize    =   NSStringFromCGSize([[UIScreen mainScreen] bounds].size);
	    NSString * App_Version = [[[NSBundle mainBundle] infoDictionary] objectForKey:@"CFBundleShortVersionString"];
	    NSString * iPhoneType = @"当前设备名字";
	    NSString *uploadString = @"所有拼接信息";
	    // 存储到本地沙盒.下次启动找寻
		}
  
  ```

- 第三方收集crash (比如说集成友盟,使用dSYM分析定位代码)

- 上报的方式，时机，策略（优缺点）等


</details>


## 链接

- [面试题系列目录](../README.md)
- **上一份**: [01一份"有点难"的iOS面试题MrPeak2016年](01一份"有点难"的iOS面试题MrPeak2016年.md)
- **下一份**: [interview-iOS-3](03interview-iOS-3.md)
