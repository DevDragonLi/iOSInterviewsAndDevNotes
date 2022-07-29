## 更新Xcode14后工程报错解决方案（Pod）

> Pod工程中的Bundle target签名报错

- 开启`generate_multiple_pod_projects` 参数工程

```
post_install do |installer|
  installer.generated_projects.each do |project|
    project.targets.each do |target|
        target.build_configurations.each do |config|
            config.build_settings['CODE_SIGN_IDENTITY'] = ''
         end
    end
  end
end

```

- 未开启`generate_multiple_pod_projects` 参数工程

```
post_install do |installer|

  installer.pods_project.targets.each do |target|

    target.build_configurations.each do |config|
       config.build_settings['CODE_SIGN_IDENTITY'] = ''
    end
  end
end

```
