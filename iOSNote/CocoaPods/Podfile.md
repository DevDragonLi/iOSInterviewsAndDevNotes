# podFile

## base  

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



### PodFile 

- 指定多个子库


	```
	pod `LFLKit`,:subspec =>['base','LFLTool']
	
	```

- 如果您的Pods文件夹不包含在git中，则您可以添加keep_source_code_for_prebuilt_frameworks!Podfile的头部以加快Pod的安装速度，因为每次预建的Pod发生更改时，它都不会下载所有源代码。

- enable_bitcode_for_prebuilt_frameworks!

```
要使用仓库的另一个分支：

pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git', :branch => 'dev'

要使用master仓库的分支：

pod 'Alamofire', :git => 'https://github.com/Alamofire/Alamofire.git'

/// 指定固定源 
pod 'PonyDebugger', :source => 'https://github.com/CocoaPods/Specs.git'

pod 'QueryKit', :subspecs => ['Attribute', 'QuerySet']

pod "test",:head 一直最新

```

- Build configurations

```
pod 'PonyDebugger', :configurations => ['Debug', 'Beta']

pod 'PonyDebugger', :configuration => 'Debug'

```

### generate_multiple_pod_projects


- `install! 'cocoapods', generate_multiple_pod_projects: true`

- 多个兼容 

```
install! 'cocoapods', 
         disable_input_output_paths: true,
         generate_multiple_pod_projects: true

```

```

post_install do |installer|

  swift_4_0_compatible = [ ... ]
  swift_4_2_compatible = [ ... ]

  installer.pod_target_subprojects.flat_map { |p| p.targets }.each do |t|
    t.build_configurations.each do |c|
      c.build_settings['SWIFT_VERSION'] = '4.0' if swift_4_0_compatible.include? t.name
      c.build_settings['SWIFT_VERSION'] = '4.2' if swift_4_2_compatible.include? t.name
    end
  end
end

```	

