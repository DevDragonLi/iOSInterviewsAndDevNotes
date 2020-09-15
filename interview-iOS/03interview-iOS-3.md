
# interview-iOS PartThree (profound understanding) 

-   [SEL和Method和IMP？](#sel和method和imp)
-   [Autorelease的原理 ?](#autorelease的原理)
-   [ARC的工作原理](#arc的工作原理)
-   [weak弱引用的代码逻辑实现?](#weak弱引用的代码逻辑实现)
-   [大文件离线下载怎么处理?会遇到哪些问题?又如何解决](#大文件离线下载怎么处理会遇到哪些问题又如何解决)
-   [Socket建立网络连接的步骤](#socket建立网络连接的步骤)

## SEL和Method和IMP？
> 谈下对IMP的理解?

<details>
<summary> 参考内容 </summary>

- SEL是“selector”的一个类型，表示一个方法的名字 
- Method（我们常说的方法）表示一种类型，这种类型与selector和实现(implementation)相关
- IMP定义为 id (*IMP) (id, SEL, …)。这样说来,IMP是一个指向函数的指针，这个被指向的函数包括id(“self”指针)，调用的SEL（方法名），再加上一些其他参数.说白了IMP就是实现方法。
- 知名框架AFN源码涉及IMP的代码

```objc
 NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration ephemeralSessionConfiguration];
        NSURLSession * session = [NSURLSession sessionWithConfiguration:configuration];
        NSURLSessionDataTask *localDataTask = [session dataTaskWithURL:nil];
        IMP originalAFResumeIMP = method_getImplementation(class_getInstanceMethod([self class], @selector(af_resume)));
        Class currentClass = [localDataTask class];
        
        while (class_getInstanceMethod(currentClass, @selector(resume))) {
            Class superClass = [currentClass superclass];
            IMP classResumeIMP = method_getImplementation(class_getInstanceMethod(currentClass, @selector(resume)));
            IMP superclassResumeIMP = method_getImplementation(class_getInstanceMethod(superClass, @selector(resume)));
            if (classResumeIMP != superclassResumeIMP &&
                originalAFResumeIMP != classResumeIMP) {
                [self swizzleResumeAndSuspendMethodForClass:currentClass];
            }
            currentClass = [currentClass superclass];
        }
        
        [localDataTask cancel];
        [session finishTasksAndInvalidate];
```

</details>

## Autorelease的原理 ?
<details>
<summary> 参考内容 </summary>

- ARC下面,我们使用`@autoreleasepool{}`来使用一个Autoreleasepool,实际上UIKit 通过RunLoopObserver 在RunLoop二次Sleep间Autoreleasepool进行Pop和Push,将这次Loop产生的autorelease对象释放 对编译器会编译大致如下:

	```objc
	
	void *DragonLiContext = objc_ AutoreleasepoolPush();
	// {} 的 code 
	objc_ AutoreleasepoolPop(DragonLiContext);
	
	```

- 释放时机: 当前RunLoop迭代结束时候释放.

</details>


## ARC的工作原理
<details>
<summary> 参考内容 </summary>

- Automatic Reference Counting，自动引用计数，即ARC,ARC会自动帮你插入retain和release语句,ARC编译器有两部分，分别是前端编译器和优化器
- 前端编译器:前端编译器会为“拥有的”每一个对象插入相应的release语句。如果对象的所有权修饰符是__strong，那么它就是被拥有的。如果在某个方法内创建了一个对象，前端编译器会在方法末尾自动插入release语句以销毁它。而类拥有的对象（实例变量/属性）会在dealloc方法内被释放。事实上，你并不需要写dealloc方法或调用父类的dealloc方法，ARC会自动帮你完成一切。此外，由编译器生成的代码甚至会比你自己写的release语句的性能还要好，因为编辑器可以作出一些假设。在ARC中，没有类可以覆盖release方法，也没有调用它的必要。ARC会通过直接使用objc_release来优化调用过程。而对于retain也是同样的方法。ARC会调用objc_retain来取代保留消息

- ARC优化器: 虽然前端编译器听起来很厉害的样子，但代码中有时仍会出现几个对retain和release的重复调用。ARC优化器负责移除多余的retain和release语句，确保生成的代码运行速度高于手动引用计数的代码


</details>


## weak弱引用的代码逻辑实现?
<details>
<summary> 参考内容 </summary>

	```objc
		objc_storeWeak() 实现 
	// HaveOld:	 true - 变量有值
	// 			false - 需要被及时清理，当前值可能为 nil
	// HaveNew:	 true - 需要被分配的新值，当前值可能为 nil
	// 			false - 不需要分配新值
	// CrashIfDeallocating: true - 说明 newObj 已经释放或者 newObj 不支持弱引用，该过程需要暂停
	// 			false - 用 nil 替代存储
	template bool HaveOld, bool HaveNew, bool CrashIfDeallocating>
	static id storeWeak(id *location, objc_object *newObj) {
		// 该过程用来更新弱引用指针的指向
		// 初始化 previouslyInitializedClass 指针
	    Class previouslyInitializedClass = nil;
	    id oldObj;
	    // 声明两个 SideTable
	    // ① 新旧散列创建
	    SideTable *oldTable;
	    SideTable *newTable;
		// 获得新值和旧值的锁存位置（用地址作为唯一标示）
		// 通过地址来建立索引标志，防止桶重复
		// 下面指向的操作会改变旧值
	  retry:
	    if (HaveOld) {
	    	// 更改指针，获得以 oldObj 为索引所存储的值地址
	        oldObj = *location;
	        oldTable = &SideTables()[oldObj];
	    } else {
	        oldTable = nil;
	    }
	    if (HaveNew) {
	    	// 更改新值指针，获得以 newObj 为索引所存储的值地址
	        newTable = &SideTables()[newObj];
	    } else {
	        newTable = nil;
	    }
		// 加锁操作，防止多线程中竞争冲突
	    SideTable::lockTwoHaveOld, HaveNew>(oldTable, newTable);
		// 避免线程冲突重处理
		// location 应该与 oldObj 保持一致，如果不同，说明当前的 location 已经处理过 oldObj 可是又被其他线程所修改
	    if (HaveOld  &&  *location != oldObj) {
	        SideTable::unlockTwoHaveOld, HaveNew>(oldTable, newTable);
	        goto retry;
	    }
	    // 防止弱引用间死锁
	    // 并且通过 +initialize 初始化构造器保证所有弱引用的 isa 非空指向
	    if (HaveNew  &&  newObj) {
	    	// 获得新对象的 isa 指针
	        Class cls = newObj->getIsa();
	        // 判断 isa 非空且已经初始化
	        if (cls != previouslyInitializedClass  &&  
	            !((objc_class *)cls)->isInitialized()) {
	        	// 解锁
	            SideTable::unlockTwoHaveOld, HaveNew>(oldTable, newTable);
	            // 对其 isa 指针进行初始化
	            _class_initialize(_class_getNonMetaClass(cls, (id)newObj));
	            // 如果该类已经完成执行 +initialize 方法是最理想情况
	            // 如果该类 +initialize 在线程中 
	            // 例如 +initialize 正在调用 storeWeak 方法
	            // 需要手动对其增加保护策略，并设置 previouslyInitializedClass 指针进行标记
	            previouslyInitializedClass = cls;
				// 重新尝试
	            goto retry;
	        }
	    }
	    // ② 清除旧值
	    if (HaveOld) {
	        weak_unregister_no_lock(&oldTable->weak_table, oldObj, location);
	    }
	    // ③ 分配新值
	    if (HaveNew) {
	        newObj = (objc_object *)weak_register_no_lock(&newTable->weak_table, 
	                                                      (id)newObj, location, 
	                                                      CrashIfDeallocating);
	        // 如果弱引用被释放 weak_register_no_lock 方法返回 nil 
	        // 在引用计数表中设置若引用标记位
	        if (newObj  &&  !newObj->isTaggedPointer()) {
	        	// 弱引用位初始化操作
				// 引用计数那张散列表的weak引用对象的引用计数中标识为weak引用
	            newObj->setWeaklyReferenced_nolock();
	        }
	        // 之前不要设置 location 对象，这里需要更改指针指向
	        *location = (id)newObj;
	    }
	    else {
	        // 没有新值，则无需更改
	    }
	    SideTable::unlockTwoHaveOld, HaveNew>(oldTable, newTable);
	    return (id)newObj;
	}
	
	```


</details>


## 大文件离线下载怎么处理?会遇到哪些问题?又如何解决
<details>
<summary> 参考内容 </summary>

- NSURLSessionDataTask 大文件离线断点下载 (AFN等框架,旧的connection类已经废弃)

- 内存飙升问题:(apple 默认实现机制导致),在下载文件的过程中，系统会先把文件保存在内存中，等到文件下载完毕之后再写入到磁盘! 在下载文件时，`一边下载一边写入到磁盘`，减小内存使用  

- 具体实现方法: 
	- 1.`NSFileHandle` 文件句柄 
	- 2.`NSOutputStream` 输出流
	
	```objc
	// code copy from jianshu 
	///: 1. NSFileHandle
	-(void)URLSession:(NSURLSession *)session dataTask:(nonnull NSURLSessionDataTask *)dataTask 
	didReceiveResponse:(nonnull NSURLResponse *)response 
	completionHandler:(nonnull void (^)(NSURLSessionResponseDisposition))completionHandler {
	      //接受到响应的时候 告诉系统如何处理服务器返回的数据
	      completionHandler(NSURLSessionResponseAllow);
	      //得到请求文件的数据大小
	      self.totalLength = response.expectedContentLength;
	      //拼接文件的全路径
	      NSString *fileName = response.suggestedFilename;
	      NSString *cachePath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
	      NSString *fullPath = [cachePath stringByAppendingPathComponent:fileName];
	
	      //【1】在沙盒中创建一个空的文件
	      [[NSFileManager defaultManager] createFileAtPath:fullPath contents:nil attributes:nil];
	      //【2】创建一个文件句柄指针指向该文件（默认指向文件开头）
	      self.handle = [NSFileHandle fileHandleForWritingAtPath:fullPath];
	}
	-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {
	      //【3】使用文件句柄指针来写数据（边写边移动）
	      [self.handle writeData:data];
	      //累加已经下载的文件数据大小
	      self.currentLength += data.length;
	      //计算文件的下载进度 = 已经下载的 / 文件的总大小
	      self.progressView.progress = 1.0 * self.currentLength / self.totalLength;
	}
	-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
	      //【4】关闭文件句柄
	      [self.handle closeFile];
	}
	///:NSOutputStream
	-(void)URLSession:(NSURLSession *)session dataTask:(nonnull NSURLSessionDataTask *)dataTask 
	didReceiveResponse:(nonnull NSURLResponse *)response 
	completionHandler:(nonnull void (^)(NSURLSessionResponseDisposition))completionHandler {
	      //接受到响应的时候 告诉系统如何处理服务器返回的数据
	      completionHandler(NSURLSessionResponseAllow);
	      //得到请求文件的数据大小
	      self.totalLength = response.expectedContentLength;
	      //拼接文件的全路径
	      NSString *fileName = response.suggestedFilename;
	      NSString *cachePath = [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) lastObject];
	      NSString *fullPath = [cachePath stringByAppendingPathComponent:fileName];
	
	      //（1）创建输出流，并打开
	      self.outStream = [[NSOutputStream alloc] initToFileAtPath:fullPath append:YES];
	      [self.outStream open];
	}
	-(void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {
	      //（2）使用输出流写数据
	      [self.outStream write:data.bytes maxLength:data.length];
	      //累加已经下载的文件数据大小
	      self.currentLength += data.length;
	      //计算文件的下载进度 = 已经下载的 / 文件的总大小
	      self.progressView.progress = 1.0 * self.currentLength / self.totalLength;
	}
	-(void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {
	      //（3）关闭输出流
	      [self.outStream close];
	}
	
	```

- 开始(resume) | 暂停(suspend) | 取消( | 恢复等

	```
	[self.dataTask cancel];
	//默认情况下取消下载不能进行恢复，若要取消之后还可以恢复，可以清空下载任务，再新建
	self.dataTask = nil;
	
	```

- **下载进度值发生跳跃错乱** :已经下载的数据 / 文件的总数据，在第一个代理方法中，我们得到的文件大小并不是真正的文件大小，而是剩余未下载的大小，所以在第一次开始下载时，可以得到正确的数据，但是在下载过程中执行其他操作，就会使得到的数据大小发生变化，从而导致下载进度值出现问题 `解决方案`：`文件总大小 = 已经下载的数据 + 剩余未下载的数据` 


- 优化性能(句柄和流一样) 只有第一次接收到响应的时候才需要创建空的文件(lazy load )

</details>


## Socket建立网络连接的步骤

<details>
<summary> 参考内容 </summary>


> 建立Socket连接至少需要一对套接字，其中一个运行于客户端，称为ClientSocket ，另一个运行于服务器端，称为ServerSocket 。套接字之间的连接过程分为三个步骤：服务器监听，客户端请求，连接确认。(知名的框架 AsyncSocket)

- 服务器监听：服务器端套接字并不定位具体的客户端套接字，而是处于等待连接的状态，实时监控网络状态，等待客户端的连接请求

- 客户端请求：指客户端的套接字提出连接请求，要连接的目标是服务器端的套接字。为此，客户端的套接字必须首先描述它要连接的服务器的套接字，指出服务器端套接字的地址和端口号，然后就向服务器端套接字提出连接请求

- 连接确认：当服务器端套接字监听到或者说接收到客户端套接字的连接请求时，就响应客户端套接字的请求，建立一个新的线程，把服务器端套接字的描述发给客户端，一旦客户端确认了此描述，双方就正式建立连接。而服务器端套接字继续处于监听状态，继续接收其他客户端套接字的连接请求

- AsyncSocket 相关代码

```

// socket连接
-(void)socketConnectHost{}
    self.socket    = [[AsyncSocket alloc] initWithDelegate:self];
    NSError *error = nil;
    [self.socket connectToHost:self.socketHost onPort:self.socketPort withTimeout:-1 error:&error];
}

心跳通过计时器来实现 // NStimer
-(void)onSocket:(AsyncSocket *)sock didConnectToHost:(NSString     *)host port:(UInt16)port
{

    LFLog(@"socket连接成功");
    // 每隔1s像服务器发送心跳包
    self.connectTimer = [NSTimer scheduledTimerWithTimeInterval:1 target:self selector:@selector(longConnectToSocket) userInfo:nil repeats:YES];
    // 在longConnectToSocket方法中进行长连接需要向服务器发送的讯息
    [self.connectTimer fire];
}

// socket发送数据是以栈的形式存放，所有数据放在一个栈中，存取时会出现粘包的现象，所以很多时候服务器在收发数据时是以先发送内容字节长度，再发送内容的形式，得到数据时也是先得到一个长度，再根据这个长度在栈中读取这个长度的字节流，如果是这种情况，发送数据时只需在发送内容前发送一个长度，发送方法与发送内容一样
NSData   *dataStream  = [@8 dataUsingEncoding:NSUTF8StringEncoding];

[self.socket writeData:dataStream withTimeout:1 tag:1];
// 接收数据

-(void)onSocket:(AsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag
{
    // 对得到的data值进行解析与转换即可
    [self.socket readDataWithTimeout:30 tag:0];

}

```

</details>



## 链接

- [面试题系列目录](../README.md)
- **上一份**: [interview-iOS-2](02interview-iOS-2.md)
- **下一份**: [interview-iOS-4](04interview-iOS-4.md)

## 赞赏一下旺仔(收集整理不易，且赞且珍惜)

</p>
<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18ff90e4c8344f86aa69c34065bb379a~tplv-k3u1fbpfcp-zoom-1.image" width="300" height="300">
<img src="../images/wechat.JPG" width="300" height="300">
</p>

