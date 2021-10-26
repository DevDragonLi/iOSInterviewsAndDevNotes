# CocoaPods Hook

> CocoaPods 会在其安装生命周期中提供各种 hook。这使用户可以自定义安装过程的多个时间点对其项目进行更改。

> hook方法还可以做一些其他语言脚本的执行，比如执行shell脚本，使用exec 后面跟上具体命令即可
sh命令文件是一样的
 


##  pre_install

>  pod install 之前可以做一些其他处理

```
pre_install do |installer|
    puts "pre install hook"
    Pod::UI.puts "pre install hook "
end

```

##  post_install 

- pod install 之后可以做一些其他处理

```
post_install do |installer|
  puts "post install hook"
  exec 'pwd'

end



post_install do |installer|
  # 遍历项目中所有target
  installer.pods_project.targets.each do |target|
    # 遍历build_configurations
    target.build_configurations.each do |config|
      # 修改build_settings中的ENABLE_BITCODE
      config.build_settings['ENABLE_BITCODE'] = 'NO'
    end
  end
end


# 开启 generate_multiple_pod_projects

post_install do |installer|
  installer.generated_projects.each do |project|
    project.targets.each do |target|
      target.build_configurations.each do |config|
        config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '11.0'
      end
    end
  end
end


```

## post_integrate 

> **1.10新增** 集成是安装过程的最后一步，它负责将生成的 Pods.xcodeproj 与用户的项目集成。这个 hook 将在完成后立即执行。

- 注意：这个 hook 在所有 .xcodeproj 保存（写入磁盘）之后执行。对 Pods.xcodeproj 进行更改将需要额外的 save 操作，但这可能会很慢。如果您预计在Pods.xcodeproj 保存到磁盘之前对其进行更改，则建议使用 post_install hook。


```
post_integrate do |installer|

  puts "post_integrate hook"

end

```
