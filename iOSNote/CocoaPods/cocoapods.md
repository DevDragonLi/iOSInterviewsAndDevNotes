
## 2021 install cocoapods 

> sudo gem install -n /usr/local/bin cocoapods -v 1.10.1

-------------
-------------
----------------

# ruby.taobao.org停止更新使用cocoapods更新rubychina源 

## 1. 如何使用？

### 1.1请尽可能用比较新的 RubyGems 版本，建议 2.6.x 以上。

```
$ gem update --system # 这里请翻墙一下
$ gem -v
2.6.3


$ gem sources --add https://gems.ruby-china.org/ --remove https://rubygems.org/
$ gem sources -l
https://gems.ruby-china.org

// 确保只有 gems.ruby-china.org

```
### 1.2如果你使用 Gemfile 和 Bundle (例如：Rails 项目)

  你可以用 Bundler 的[ Gem 源代码镜像命令](http://bundler.io/v1.5/bundle_config.html#gem-source-mirrors)

```
$ bundle config mirror.https://rubygems.org https://gems.ruby-china.org

这样你不用改你的 Gemfile 的 source

source 'https://rubygems.org/'
gem 'rails', '4.2.5'

```

## 2.常见问题？
### 2.1 如果遇到 SSL 证书问题，你又无法解决，请直接用 http://gems.ruby-china.org 避免 SSL 的问题。
### 2.2 如果你在意 Gem 下载的安全问题，请正确安装 Ruby、OpenSSL，建议部署 Linux 服务器的时候采用 这个 RVM 安装脚本 的方式安装 Ruby。
### 2.3 Bundler::GemspecError: Could not read gem at /home/xxx/.rvm/gems/ruby-2.1.8/cache/rugged-0.23.3.gem. It may be corrupted.，这类错误是网络原因下载到了坏掉的文件到本地，请直接删除那个文件。



# 文章内容来自[点我](https://gems.ruby-china.org)