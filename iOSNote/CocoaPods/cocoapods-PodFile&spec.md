# PodFile&&Podspec

## base  

- **cocoapods**: **/Users/`LFL`/.cocoapods/repos**

- **cache**: **/Users/`LFL`/Library/Caches/CocoaPods/** 包含如下
	- pods :缓存已经安装的第三方
	- search_index.json 


## Podspec

```ruby
s.name：名称，pod search 搜索的关键词
s.version：版本 (tag)
s.summary：pod search 搜索的关键词
s.homepage：主页地址，例如Github地址
s.license：许可证
s.author：作者
s.social_media_url：社交网址
s.platform：平台
s.source：Git仓库地址，例如在Github地址后边加上 .git 就是Git仓库地址.
s.source_files :"ProjectKit/*"
	* `*` 表示匹配所有文件:"ProjectKit/*"
	* `*.{h,m}` 表示匹配所有以.h和.m结尾的文件:"ProjectKit/Classes/*.{h,m}"
	* `**` 表示匹配所有`子目录`:"ProjectKit/**/*.h"

s.resources：需要包含的图片等资源文件
s.dependency：依赖库，如有多个可以这样写,需要用到的frameworks，不需要加.frameworks后缀
s.requires_arc：是否要求ARC

```
 
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

```ruby
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

```ruby
pod 'PonyDebugger', :configurations => ['Debug', 'Beta']

pod 'PonyDebugger', :configuration => 'Debug'

```

### generate_multiple_pod_projects


- `install! 'cocoapods', generate_multiple_pod_projects: true`

- 多个兼容 

```ruby
install! 'cocoapods', 
         disable_input_output_paths: true,
         generate_multiple_pod_projects: true

```

```ruby

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
## PodSpec 扩展

> https://guides.cocoapods.org/syntax/podspec.html

- s.static_framework = true 
	- 是否是静态库 这个地方很重要 假如不写这句打出来的包 就是动态库 不能使用 一运行会报错 image not found

- spec.swift_version = '3.0', '4.0'

- spec.cocoapods_version = '>= 0.36'

- spec.prepare_command = 'ruby build_files.rb'

- spec.dependency 'AFNetworking', '~> 1.0', :configurations => ['Debug']

- spec.prefix_header_contents = '#import <UIKit/UIKit.h>', '#import <Foundation/Foundation.h>'


```ruby

spec.prefix_header_file = 'iphone/include/prefix.pch'
spec.prefix_header_file = false

spec.module_name = 'Three20'

 # 配置 Xcode Build Setting
  s.xcconfig = {
    'HEADER_SEARCH_PATHS' => '$(PODS_ROOT)/',     # 配置 Header 搜索路径 
    'FRAMEWORK_SEARCH_PATHS' => '$(PODS_ROOT)/',    # 配置 Framwork 搜索路径 
    'GCC_PREPROCESSOR_DEFINITIONS' => 'RELEASE COCOAPODS=1' # 配置预编译宏
  }

```

```ruby
spec.ios.vendored_frameworks = 'Frameworks/MyFramework.framework'
spec.vendored_frameworks = 'MyFramework.framework', 'TheirFramework.framework'

spec.ios.vendored_library = 'Libraries/libProj4.a'
spec.vendored_libraries = 'libProj4.a', 'libJavaScriptCo

spec.ios.resource_bundle = { 'MapBox' => 'MapView/Map/Resources/*.png' }
spec.resource_bundles = {
    'MapBox' => ['MapView/Map/Resources/*.png'],
    'MapBoxOtherResources' => ['MapView/Map/OtherResources/*.png']
  }

spec.resource = 'Resources/HockeySDK.bundle'

spec.resources = ['Images/*.png', 'Sounds/*']
  
spec.default_subspec = 'Core'
spec.default_subspecs = 'Core', 'UI'
spec.default_subspecs = :none  

```

```ruby
Pod::Spec.new do |spec|
  spec.name          = 'ShareKit'
  spec.source_files  = 'Classes/ShareKit/{Configuration,Core,Customize UI,UI}/**/*.{h,m,c}'
  # ...

  spec.subspec 'Evernote' do |evernote|
    evernote.source_files = 'Classes/ShareKit/Sharers/Services/Evernote/**/*.{h,m}'
  end

  spec.subspec 'Facebook' do |facebook|
    facebook.source_files   = 'Classes/ShareKit/Sharers/Services/Facebook/**/*.{h,m}'
    facebook.compiler_flags = '-Wno-incomplete-implementation -Wno-missing-prototypes'
    facebook.dependency 'Facebook-iOS-SDK'
  end
  # ...
end

```

