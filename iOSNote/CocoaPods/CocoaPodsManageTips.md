
## **CocoaPods**管理库的使用技巧

- pod install --verbose --no-repo-update  只查找本地,而且不联网更新库,快!

- 使用shell的命名别名来简化

```
setup pod update alias name

alias pod_update='pod update --verbose --no-repo-update'

alias pod_install='pod install --verbose --no-repo-update'

```