## cocoapods plugins

> sudo gem update --system

> User：【podfile ： plugin 'plugin name'】

> install：`$ sudo gem plugin name`

- binary 
- dependencies 
- cocoapods-user-defined-build-types
- cocoapods-static-swift-framework

### [cocoapods-binary ](https://github.com/leavez/cocoapods-binary/)

> 构建二进制适合本地Debug使用

- remote pod transfer to framework

- sudo gem install cocoapods-binary 

### [cocoapods-pod-merge](https://github.com/grab/cocoapods-pod-merge)

> group Pod组件合并插件

<details>
<summary> 点击展开示例说明 </summary>

> pod install 源码OK，二进制需要对应remote自身再桥接。

- sudo gem install cocoapods-dependencies

- This plugin requires a file called `MergeFile`. This is how it looks:
	- touch Gemfile 
	- gem 'cocoapods-pod-merge' 
	- bundle install
- bundle exec pod install

```

group 'Networking'
	pod 'AFNetworking'
	pod 'SDWebImage'
end

plugin 'cocoapods-pod-merge'

target 'MyApp'
	pod 'Networking', :path => 'MergedPods/Networking'
	pod 'UI', :path => 'MergedPods/UI'
end

```  
</details>


### [pod-dependencies](https://github.com/segiddins/cocoapods-dependencies)

> dependencies:分析当前组件依赖细节

- sudo gem install cocoapods-dependencies


###  [cocoapods-static-swift-framework](https://github.com/leavez/cocoapods-static-swift-framework)

- sudo gem install cocoapods-static-swift-framework	

### [cocoapods-packager](https://github.com/CocoaPods/cocoapods-packager)

> 只适用于远端库（本质从远端clone再处理）

> M1并无适配及产物架构不可自定义，不建议使用！

- sudo gem install  cocoapods-packager

- your Gemfile:gem "cocoapods-packager" && bundle install 

- pod package name.podspec

```
如果需要排除源码中依赖的第三方库，可以在打包命令中添加 --exclude-deps 选项。
//静态framework
pod package name.podspec --force --no-mangle --embedded
//静态.a 
pod package name.podspec --library --force
// .a 调整
s.vendored_libraries = "Path/to/xxx.a"
s.source_files = "xxx/**/*.h"

/强制覆盖之前已经生成过的二进制库
--force

//生成静态.framework
--embedded

//生成静态.a
--library

//生成动态.framework
--dynamic

//动态.framework是需要签名的，所以只有生成动态库的时候需要这个BundleId
--bundle-identifier

//不包含依赖的符号表，生成动态库的时候不能包含这个命令，动态库一定需要包含依赖的符号表。
--exclude-deps

//表示生成的库是debug还是release，默认是release。--configuration=Debug
--configuration

--no-mangle
//表示不使用name mangling技术，pod package默认是使用这个技术的。我们能在用pod package生成二进制库的时候会看到终端有输出Mangling symbols和Building mangled framework。表示使用了这个技术。
//如果你的pod库没有其他依赖的话，那么不使用这个命令也不会报错。但是如果有其他依赖，不使用--no-mangle这个命令的话，那么你在工程里使用生成的二进制库的时候就会报错：Undefined symbols for architecture x86_64。

--subspecs

//如果你的pod库有subspec，那么加上这个命名表示只给某个或几个subspec生成二进制库，--subspecs=subspec1,subspec2。生成的库的名字就是你podspec的名字，如果你想生成的库的名字跟subspec的名字一样，那么就需要修改podspec的名字。
这个脚本就是批量生成subspec的二进制库，每一个subspec的库名就是podspecName+subspecName。

--spec-sources

//一些依赖的source，如果你有依赖是来自于私有库的，那就需要加上那个私有库的source，默认是cocoapods的Specs仓库。--spec-sources=private,https://github.com/CocoaPods/Specs.git。

```

### [cocoapods-user-defined-build-types](https://github.com/joncardasis/cocoapods-user-defined-build-types)

- sudo gem install cocoapods-user-defined-build-types

```
plugin 'cocoapods-user-defined-build-types'

enable_user_defined_build_types!

target "CoffeeApp" do
    pod 'Alamofire'
    pod "SwiftyJSON", :build_type => :dynamic_framework
 
dynamic_library
dynamic_framework
static_library
static_framework   

```