## PodSpec

> https://guides.cocoapods.org/syntax/podspec.html

- s.static_framework = true 是否是静态库 这个地方很重要 假如不写这句打出来的包 就是动态库 不能使用 一运行会报错 image not found

- spec.swift_version = '3.0', '4.0'

- spec.cocoapods_version = '>= 0.36'

- spec.prepare_command = 'ruby build_files.rb'

- spec.dependency 'AFNetworking', '~> 1.0', :configurations => ['Debug']

- spec.prefix_header_contents = '#import <UIKit/UIKit.h>', '#import <Foundation/Foundation.h>'

```
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

###  匹配

- spec.source_files = 'Classes/**/*.{h,m}', 'More_Classes/**/*.{h,m}'

```
* 将匹配所有文件
c* 将匹配所有以开头的文件 c
*c 将匹配所有以结尾的文件 c
*c*将匹配其中所有文件c（包括开头或结尾）

```

```
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

```
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