# interview-iOS - 4 

 -   [用户需要上传和下载一个重要的资料文件，应该如何判断用户本次是否上传成功和下载成功了?](#用户需要上传和下载一个重要的资料文件应该如何判断用户本次是否上传成功和下载成功了)
-   [ReactiveCocoa(RAC)如何防止UIButton短时间内多次重复点击，大概思路?](#reactivecocoarac如何防止uibutton短时间内多次重复点击大概思路)
-   [倒计时如何实现 ？](#倒计时如何实现)
-   [熟悉CocoaPods么？能大概讲一下工作原理么？](#熟悉cocoapods么能大概讲一下工作原理么)
-   [使用SDWebImage内存爆涨的问题](#使用sdwebimage内存爆涨的问题)
-   [isa指针的作用](#isa指针的作用)
-   [测试都有哪些方式?优缺点呢](#测试都有哪些方式优缺点呢)
-   [Xcode8开始后自动配置开发证书过程?](#xcode8开始后自动配置开发证书过程)
-   [项目中的图片上传功能如何实现，为什么使用队列上传，为什么不用异步上传](#项目中的图片上传功能如何实现为什么使用队列上传为什么不用异步上传)
-   [项目中你是怎么处理网络速度慢、中断抖动等网络请求中的问题?](#项目中你是怎么处理网络速度慢中断抖动等网络请求中的问题)

## 用户需要上传和下载一个重要的资料文件，应该如何判断用户本次是否上传成功和下载成功了?
<details>
<summary> 参考内容 </summary>

- 用MD5验证文件的完整性！(仅仅通过代码来判断当前次的请求发送结束或者收到数据结束不可以的)
- 当客户端上传一个文件的时候，在请求body里面添加该文件的MD5值来告诉服务器，服务器接受文件完毕以后通过校验收到的文件的MD5值与请求body里面的MD5值来最终确定本次上传是否成功
- 当客户端下载一个文件的时候，在响应头里面收到了服务器附带的该文件的MD5值，文件下载结束以后，通过获取下载后文件的MD5值与本次请求服务器返回的响应头中的MD5值做一个比较，来最终判断本次下载是否成功
- MD5，是一个将任意长度的数据字符串转化成短的固定长度的值的单向操作。任意两个字符串不应有相同的散列值
- MD5校验可以应用在多个领域，比如说机密资料的检验，下载文件的检验，明文密码的加密等。MD5校验原理举例：如客户往我们数据中心同步一个文件，该文件使用MD5校验，那么客户在发送文件的同时会再发一个存有校验码的文件，我们拿到该文件后做MD5运算，得到的计算结果与客户发送的校验码相比较，如果一致则认为客户发送的文件没有出错，否则认为文件出错需要重新发送。


</details>

## ReactiveCocoa(RAC)如何防止UIButton短时间内多次重复点击，大概思路? 
> 需要有RAC使用经验才可问此题

<details>
<summary> 参考内容 </summary>

- 建立一个`flag`或者使用 `filter`

- 事件完成 flag重置，否则一直skip或者filter (某个RAC群友抛砖引玉)

</details>

## 倒计时如何实现 ？
<details>
<summary> 参考内容 </summary>

- NSTimer ,精度不一定准确
- GCD 
- DisplayLink
- RAC 

	
	```objc
	-(void)startTime{
	    __block int timeout=30; //倒计时时间
	    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	    dispatch_source_t _timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0,queue);
	    dispatch_source_set_timer(_timer,dispatch_walltime(NULL, 0),1.0*NSEC_PER_SEC, 0); //每秒执行
	    dispatch_source_set_event_handler(_timer, ^{
	        if(timeout<=0){ //倒计时结束，关闭
	            dispatch_source_cancel(_timer);
	            dispatch_async(dispatch_get_main_queue(), ^{
	                //设置界面的按钮显示 根据自己需求设置
	                [l_timeButton setTitle:@"发送验证码" forState:UIControlStateNormal];
	                l_timeButton.userInteractionEnabled = YES;
	            });
	        }else{
	            //            int minutes = timeout / 60;
	            int seconds = timeout % 60;
	            NSString *strTime = [NSString stringWithFormat:@"%.2d", seconds];
	            dispatch_async(dispatch_get_main_queue(), ^{
	                //设置界面的按钮显示 根据自己需求设置
	                NSLog(@"____%@",strTime);
	                [l_timeButton setTitle:[NSString stringWithFormat:@"%@秒后重新发送",strTime] forState:UIControlStateNormal];
	                l_timeButton.userInteractionEnabled = NO;
	                
	            });
	            timeout--;
	            
	        }
	    });
	    dispatch_resume(_timer);
	    
	}
	
	 [[RACSignal interval:1 onScheduler:[RACScheduler mainThreadScheduler]]subscribeNext:^(NSDate * date) {
	        static NSInter = 60; 
	        60 --;
	       //  处理显示逻辑
	    }];
	    
	```

</details>

## 熟悉CocoaPods么？能大概讲一下工作原理么？

<details>
<summary> 参考内容 </summary>

> **Podfile.lock**：在pod install以后会生成一个Podfile.lock的文件,这个文件在多人协作开发的时候就不建议加入在.gitignore中,因为这个文件会锁定当前各依赖库的版本,就算之后再pod install也不会更改版本,不提交上去的话就可以防止第三方库升级后造成大家各自的第三方库版本不同

### **CocoaPods原理** 

- Pods项目最终会编译成一个名为libPods.a的文件,主项目只需要依赖这个.a文件即可 
- 对于资源文件,CocoaPods提供了一个名为Pods-resources.sh的bash脚本,该脚本在每次项目编译的时候都会执行,将第三方的各种资源文件复制到目标目录中
- CocoaPods通过一个名为Pods.xcconfig的文件在编译时设置所有的依赖和参数

</details>


## 使用SDWebImage内存爆涨的问题

<details>
<summary> 参考内容 </summary>

- 产生的源码部分如下

	```objc
	- (UIImage *)diskImageForKey:(NSString *)key {
	    NSData *data = [self diskImageDataBySearchingAllPathsForKey:key];
	    if (data) {
	        UIImage *image = [UIImage sd_imageWithData:data];
	        image = [self scaledImageForKey:key image:image];
	        if (self.shouldDecompressImages) {
	            image = [UIImage decodedImageWithImage:image];
	        }
	        return image;
	    }
	    else {
	        return nil;
	    }
	}
	```

- 在某个VC出栈的时候清除比较合适，因为有可能用户不会再去显示那些图片，但是这些图片依旧占着内存 

	- `[[SDImageCache sharedImageCache] setValue:nil forKey:@"memCache"];` 

</details>


## isa指针的作用
<details>
<summary> 参考内容 </summary>

- 对象的isa指向类，类的isa指向元类（meta class），元类isa指向元类的根类。isa帮助一个对象找到它的方法

- 是一个Class 类型的指针. 每个实例对象有个isa的指针,他指向对象的类，而Class里也有个isa的指针, 指向meteClass(元类)。元类保存了类方法的列表。当类方法被调用时，先会从本身查找类方法的实现，如果没有，元类会向他父类查找该方法。同时注意的是：元类（meteClass）也是类，它也是对象。元类也有isa指针,它的isa指针最终指向的是一个根元类(root meteClass).根元类的isa指针指向本身，这样形成了一个封闭的内循环。


</details>

## 测试都有哪些方式?优缺点呢

<details>
<summary> 参考内容 </summary>

- 联机调试 (一般而言适用于开发人员)
	- 之前需要插线
	- 后期版本可以无线调试！

- 蒲公英等分发平台(就是需要提供参与app测试人员的设备UDID) 缺点:开发者需要将这些设备的UDID添加到开发者中心，每次有新的测试人员加入，需要重新生成profiles

- TestFlight进行App Beta版测试 (apple 官方,iOS8及以上版本的iOS设备才能运行):
	-  优点: 无需UUID,**外部测试人员的上限是10000人**（2018年后又扩大了测试上限）,只需要参与app测试人员提供一个邮箱 
	-  

</details>

## Xcode8开始后自动配置开发证书过程?

<details>
<summary> 参考内容 </summary>

- Xcode会在本机钥匙串寻找team对应的开发证书，如果本地钥匙串存在该证书则加载使用
- 不存在：则从开发者中心寻找本机对应的开发证书，如果开发者中心没有则自动生成一个并下载到钥匙串使用

</details>


## 项目中的图片上传功能如何实现，为什么使用队列上传，为什么不用异步上传

<details>
<summary> 参考内容 </summary>

- image -> data 
- 如果是所有图片一起传,可以使用异步,但是如果是有按照顺序的需求,则需要按照用户的已知顺序传

</details>

## 项目中你是怎么处理网络速度慢、中断抖动等网络请求中的问题?

<details>
<summary> 参考内容 </summary>

> dns 和 ping 探测当前网络情况

- UI -> loading/Tips 
- netWorking ->cdn ,IP 直链

</details>

## 链接

- [面试题系列目录](../README.md)
- **上一份**: [interview-iOS-3](03interview-iOS-3.md)
- **下一份**: [宝库iOS开发笔试题2017年:参考答案已更新完毕](05iOS宝库iOS开发笔试题2017年.md)

## 赞赏一下旺仔(收集整理不易，且赞且珍惜)

</p>
<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/18ff90e4c8344f86aa69c34065bb379a~tplv-k3u1fbpfcp-zoom-1.image" width="300" height="300">
<img src="../images/wechat.JPG" width="300" height="300">
</p>
