# CocoaPods提交开源的框架之流程摘要

> [ScreenAdaptationKit-example](https://github.com/DevDragonLi/ScreenAdaptationKit)

## pod project 
- 以`github`模板为基础,创建项目 
	- `pod lib create nameKit`

- 创建项目的podspec,如果项目已存在,并无 podspec 
	- `pod spec create name.podspec`


## 编辑.podspec

- [guides.cocoapods.org](https://guides.cocoapods.org/syntax/podspec.html)

```
s.name：名称，pod search 搜索的关键词
s.version：版本 (tag)
s.summary：pod search 搜索的关键词
s.homepage：主页地址，例如Github地址
s.license：许可证
s.author：作者
s.social_media_url：社交网址
s.platform：平台
s.source：Git仓库地址，例如在Github地址后边加上 .git 就是Git仓库地址.
s.source_files：需要包含的源文件 ,下文有详细说明
s.resources：需要包含的图片等资源文件
s.dependency：依赖库，如有多个可以这样写,需要用到的frameworks，不需要加.frameworks后缀
s.requires_arc：是否要求ARC

```

- `s.source_files` 常见写法
	
	* `*` 表示匹配所有文件
	* `*.{h,m}` 表示匹配所有以.h和.m结尾的文件
	* `**` 表示匹配所有`子目录`

```
"ProjectKit/*"
"ProjectKit/Classes/*.{h,m}"
"ProjectKit/**/*.h"

```

## 验证 && tag
- 验证podspec文件,编辑完podspec文件后需要验证一下这个文件是否可用podspec文件不允许有任何的Warning或者Error:`pod lib lint ` 
	- `--allow-warnings`  允许有任何的Warning 
	- `--verbose`			 获取更多错误信息
	- `--use-libraries`  包含.a 需要添加此参数:

- `tag` 上传podspec文件中需要指定的tag， 完成上述操作后给项目打tag
	- `git tag -m"update lib podspec" "1.0.0"`
	- `git push --tags`

## 提交 CocoaPods

-  Trunk 服务注册

```
`pod trunk register dragonli_52171@163.com 'DevDragonLi' --description='DevDragonLi' macbook pro' --verbose`
```	
- 发布:`pod trunk push nameKit.podspec`	


## [CocoaPods管理库的使用技巧](./CocoaPods管理库的使用技巧.md)
	