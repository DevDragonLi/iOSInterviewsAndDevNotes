
# 一份"有点难"的iOS面试题：MrPeak 

> Facebook · Software engineer 

>[题目出处](https://zhuanlan.zhihu.com/p/22834934)

> Question list 

- 谈下iOS开发中知道的哪些锁? 哪个性能最差?SD和AFN使用的哪个? 一般开发中你最常用哪个? 哪个锁apple存在问题又是什么问题?
- iOS下如何实现指定线程数目的线程池?
- 如何用HTTP实现长连接？
- http的post和get啥区别
- 使用atomic一定是线程安全的吗？
- 数据库建表的时候索引有什么用？
- 介绍下iOS设备获取唯一设备号的历史变迁
- 如何使用runtime hook一个class的某个方法，又如何hook某个instance的方法？
- 聊下HTTP的POST的body体使用form-urlencoded和multipart/form-data的区别。
- 通过[UIImage imageNamed:]生成的对象什么时候被释放？
- applicationWillEnterForeground和applicationDidBecomeActive都会在哪些场景下被调用？举例越多越好。
- 如何终止正在运行的工作线程？
- iOS下所有的本地持久化方案。


## 谈下iOS开发中知道的哪些锁? 哪个性能最差?SD和AFN使用的哪个? 一般开发中你最常用哪个? 哪个锁apple存在问题又是什么问题?
 
- 我们在使用多线程的时候多个线程可能会访问同一块资源，这样就很容易引发数据错乱和数据安全等问题，这时候就需要我们保证每次只有一个线程访问这一块资源，锁 应运而生

- OSSpinLock 自旋锁 ,存在的问题是, 优先级反转问题,破坏了spinlock

- `@synchronized` 性能最差,但是SD和AFN等其他框架很多使用这个.

- dispatch_semaphore 信号量 : 保持线程同步为线程加锁

```objc
dispatch_semaphore_t signal = dispatch_semaphore_create(1); 

dispatch_time_t overTime = dispatch_time(DISPATCH_TIME_NOW, 1.0f * NSEC_PER_SEC);
//Thread1
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"Thread1 waiting");
    dispatch_semaphore_wait(signal, overTime); //signal 值 -1
    NSLog(@"Thread1");
    dispatch_semaphore_signal(signal); //signal 值 +1
    NSLog(@"Thread1 send signal");
});

//Thread2
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
    NSLog(@"Thread2 waiting");
    dispatch_semaphore_wait(signal, overTime);
    NSLog(@"Thread2");
    dispatch_semaphore_signal(signal);
    NSLog(@"Thread2 send signal");
});
dispatch_semaphore_create(1)： 若传入为0(over time 失效) 则阻塞线程并等待timeout,时间到后会执行其后的语句,

dispatch_semaphore_wait(signal, overTime)：可以理解为 lock,会使得 signal 值 -1

dispatch_semaphore_signal(signal)：可以理解为 unlock,会使得 signal 值 +1

```
- tips:一定要说具体项目使用场景.(YY较多框架使用)

## iOS下如何实现指定线程数目的线程池?

- 循环通过pthread_create创建线程，创建s_tfthread对象做为线程句，加入线程数组,s_tftask_content->methord初始化为空函数

- 创建任务执行函数，执行完通过task初始化函数后，在执行函数中通过pthread_cond_wait信号将当前创建的线程挂起

- 创建完之后，程序中将会有n个挂起状态的线程，当需要执行新的task的时候查找，我们就可以根据不同的task标志在k_threads中查询出空闲线程，并创建新的s_tftask_content加入s_tfthread的任务列表，通过pthread_cond_signal重新唤醒该线程继续执行任务

## 如何用HTTP实现长连接？
- HTTP是无状态的，要维持一个长连接可以用心跳包方式
- 丢包，沾包 ,实际上http连接进行轮询.(嘀嘀打车早期版本采用的方式)
- 定时轮询会存在延迟 用户体验就不好

## http的post和get啥区别

- ~~无非post带参数,某种层面比get不受限制~~ 

- 从语义角度分析

	- 安全性:不会引发 server 端的改变 
	- 幂等:同一个方法请求多次结果相同
	- 可缓存

## 使用atomic一定是线程安全的吗？
- BOOL 类型 修饰符不影响

## 数据库建表的时候索引有什么用？
- 创建索引可以大大提高系统的性能，加快数据的检索速度，加速表和表之间的连接，保证数据库表中每一行数据的唯一性
- 但是有些列不应该创建索引，需要综合考虑.

## 介绍下iOS设备获取唯一设备号的历史变迁
#### iOS中获取设备唯一标示符的方法一直随版本的更新而变化
- iOS 2.0版本以后UIDevice提供一个获取设备唯一标识符的方法uniqueIdentifier
- iOS6是用WiFi的mac地址
- iOS7-iOS10.2通过KeyChain来保存获取到的UDID,因为APP删了再装回来，也可以从KeyChain中读取回来 (广告收益计算等)
- [10.3后用户删掉一个 App，之后再重装就只能手动登录一次](http://mt.sohu.com/20170310/n482880968.shtml) 


## 如何使用runtime hook一个class的某个方法，又如何hook某个instance的方法？

> 这个问题,首先要考虑怎么回答才能不被套路 

- 考虑 hook是否有公开头文件的类，有的话写一个Utility函数，再使用category，
- 没有的话就建一个类作为新函数载体，然后先为被hook的类增加函数，再替换。
- 如何hook某个instance的方法，应该可以定义一个函数指针变量(IMP要谈及吧)，hook时将要调用的地址赋给这个变量，调用时把这个变量当作函数来用 (RAC框架 hook 谈及)

## 聊下HTTP的POST的body体使用form-urlencoded和multipart/form-data的区别。

- multipart/form-data是当上传文件或者二进制数据和非ASCII数据使用 ,AFN请求如何设置? 

```objc
[self.requestSerializer setValue:@"multipart/form-data" forHTTPHeaderField:@"content-type"];
```
- form-urlencoded是默认的mime内容编码类型，是通用的，但是它在传输比较大的二进制或者文本数据时效率极低
- 交互:GET,POST,PUT,PATCH,DELETE等,AFN的PATCH貌似数组存在问题.

## 通过[UIImage imageNamed:]生成的对象什么时候被释放？
- 此类加载,针对小图标,使用场景较多图片.
- `@autoreleasepool` 如果没有使用局部释放池，并且在主线程，则是当前主线程Runloop一次循环结束前释放。
- imageWithContentsOfFile 加载适用于大图片,不常用的图片,一般无引用时候,会释放


## applicationWillEnterForeground和applicationDidBecomeActive都会在哪些场景下被调用？举例越多越好。
- 推送、做支付
- 跳app
- 后台杀进程的时候、IM、第三方授权分享登录回调情况下等

## 如何终止正在运行的工作线程？

- 线程中调用exit、pthread_exit、pthread_kill、pthread_cancel
- NSOperation ,接口设计的cancle 实际上只能取消还未运行的,已经运行的无法取消.

## iOS下所有的本地持久化方案?
- 沙盒 
 - plist文件（属性列表）
 - preference（偏好设置）
 
- NSKeyedArchiver（归档）

- SQLite 3

- CoreData
- Realm (Swift OC 不可以互操作)

## 链接

- [面试题系列目录](../README.md)
- **下一份**: [interview-iOS-2](interview-iOS-2.md)
