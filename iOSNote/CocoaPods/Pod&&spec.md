# CocoaPods提交私有的框架之流程摘要

## base pod 

- **cocoapods**: **/Users/`LFL`/.cocoapods/repos**

- **cache**: **/Users/`LFL`/Library/Caches/CocoaPods/** 包含如下
	- pods :缓存已经安装的第三方
	- search_index.json 

 
- **资源文件**
	- `Assets` 文件夹内图片资源最终会打包成一个 `bundle` 文件

```
 图片bundle 资源 
 
[[NSbundle bundleForClass :self] pathForResource:"" inDirectory:"当前pod库名.bundle"]


 cell : xib  

 cell = [[[NSBundle bundleForClass:self] loadNibNamed:@"cell 对应 name " owner:nil options:nil] firstObject];
 
 cell 使用图片:`当前pod库名.bundle/图片名`,不能再使用[UIImage imageName:@"名字"],使用 contenofURL 


```
- **xib 处理** 
 - 先编译为 `nib`
 - 读取涉及图片,需要处理对应的`@2`,`@3`
 
 
 
- **子库分离 参考 "AFN"**
	
`s.subspec 'LFLSegumentTool' do |b| 
b.source_files => 'LFLTest/LFLSegumentTool/*'
子库依赖单独处理
b.dependecy = 'GitHubSegumentTool'
end`

### example 

- 私有的远程索引  pod repo add LFLcocoapods URL  部署到`coding` ,`oschina`等

- pod repo remove LFLcocoapods  

- pod repo update LFLcocoapods

- pod spec lint 验证自有库 

- pod repo push LFLTest LFLTest.podspec



### PodFile 

- 指定多个子库

	```
	pod `LFLKit`,:subspec =>['base','LFLTool']
	
	```
	





