---
title: CocoaPods的安装和使用（心得）
date: 2017-06-07 22:18:37
tags: CocoaPods相关
---
`上周五由于项目需要，我把一个有Pods项目改了名，可恨的是电脑上的CocoaPods挂掉了，公司网差。只好周末的时候在家奋斗了一下，功夫不负有心人，最后是被我搞好了，由于出现了好多问题，所以在这把自己的安装心得说一下，仅供大家参考！`
### CocoaPods简介：
 CocoaPods是什么呢？它负责管理iOS项目中第三方开源库的工具，它的项目源码都在Github上管理的，那么问题又来了，Github是什么呢？(### 被扔西红柿)，咳咳~这个我就不在这废话了。我们开发iOS项目时不可避免地要使用第三方开源库（现在第三方开源库的使用率比前些年更多了，建议大家在允许的条件下  最好是能够自己造些轮子），CocoaPods的出现使得我们可以节           省配置和更新第三方开源库的时间和精力。
###安装前奏
首先要提一下CocoaPods的安装顺序:
```
Xcode -> homebrew -> RVM -> Ruby -> CocoaPods
```
- Xcode不用说。。大家也没有不知道的。
- homebrew是什么？接触过Linux的同学应该挺熟悉yum的，没错，homebrew就是OS的yum，一款软件管理工具。
- RVM  它的全程是Ruby Version Manager，大家看名字也应该可以了解到，这是一款命令行管理工具，能够轻松的管理Ruby的版本。
- Ruby 这是一款专门为面向设计编程制作的脚本语言，简单易用，功能强大。

关于这几个工具的安装我就不在这篇文章中赘述了，有需要的我再专门写一篇关于他们的安装。
###安装正式开始
- 首先使用Ruby的gem命令来进行安装
```
$ sudo gem install cocoapods
$ pod setup
```
敲完这些以后，你会突然发现，卡住了。恭喜你，体验了一把被“墙”的感觉。这是因为Ruby的源（安装源）`https://rubygems.org/`是亚马逊的云服务（这个说真的我是之前在唐巧大神的博客中了解到的），这个时候大部分的教程都会叫你换成淘宝的源`淘宝的源:https://ruby.taobao.org/`，不过........

![淘宝Ruby源网站截图](http://upload-images.jianshu.io/upload_images/2540460-310176d63557b3a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
没错，停止维护了，更具淘宝源上的提示，现在源应该使用`http://gems.ruby-china.org/`这个。
好的问题，解决了，那么让我们换一下ruby的源吧
```
gem sources --remove https://rubygems.org/                 //删除
gem sources -a http://gems.ruby-china.org/
gem sources -l
```
你会发现新的ruby源已经安装完毕，完美~
- 继续`pod setup`命令，这个时候终端会出现`Setting up CocoaPods master repo`，这个步骤 Cocoapods 会将它的信息下载到Mac的` ~/.cocoapods`目录下，然后漫长的等待来了~~~（表示楼主在这个阶段过一会就预览一下~/.cocoapods/repos/master的大小，当然也可以cd到目录下用`du -sh *`命令查看进度，速度挺快，一分钟能走1MB了呢>_<||| ）。
####使用 CocoaPods 的镜像索引
是不是等不下去了？好吧，告诉一个可以提高下载速度的方法，那就是使用CocoaPods的镜像索引。
所有的项目的 `Podspec` 文件都托管在`https://github.com/CocoaPods/Specs`，第一次执行`pod setup`时，CocoaPods 会将这些`podspec`索引文件更新到本地的 `~/.cocoapods/`目录下.
一个叫 [akinliu](http://akinliu.github.io/2014/05/03/cocoapods-specs-/) 的朋友在 [gitcafe](http://gitcafe.com/) 和 [oschina](http://www.oschina.net/) 上建立了 CocoaPods 索引库的镜像，因为 `gitcafe` 和 `oschina`的服务器都是在国内，所以在执行索引更新操作时，会快了好多。如下操作可以将 CocoaPods 设置成使用 `gitcafe` 镜像：
```
pod repo remove masterpod repo add master https://gitcafe.com/akuandev/Specs.gitpod repo update
```
使用 `oschina` 镜像：
```
pod repo remove masterpod repo add master http://git.oschina.net/akuandev/Specs.git
```
###CocoaPods使用
安装完成后，我们就可以安心的来使用CocoaPods了
- 使用CocoaPods有一个前提，是我们的项目目录下必须要有一个Profile的文件，那这个文件要怎么创建呢：
```
cd ''项目根目录''
pod init
```
OK，搞定这个时候使用`vim Profile`命令编辑Profile里的内容，将依赖库名字依次列在Profile中，最终格式如下：

![Profile最终格式](http://upload-images.jianshu.io/upload_images/2540460-310a76a5f23350b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其中`target 'Demo' do`和`#Pods for Demo`中的Demo是我的工程名。
- 到现在为止，你的Profile就已经配置好了，然后执行：
```
pod install
```
等待完成后，你的第三方库就已经配置好了，打开`*.xcworkspace`，没错，再也不是`*.xcodeproj`文件了。
链接你的设备，使用配置好的Pods，run一下你的程序吧~
- **Note:**  每次更改了 Podfile 文件，你需要重新执行一次`pod update`命令。

### 三个重要的知识点
- 当执行pod install之后，除了Podfile，还会生成一个名为`Podfile.lock`的文件，它会锁定当前各依赖库的版本，之后即使多次执行`pod install`也不会更改版本，只有执行`pod update`才会改变`Podfile.lock`.在多人协作的时候，这样可以防止第三方库升级时候造成大家各自的第三方库版本不一致，所以在往SVN提交版本的时候不能把它落下。（个人认为可以把它从ignore列表中删掉）
- Pods项目最终会编译成一个名为libPods.a的文件，主项目只需要依赖这个.a文件即可，这个文件一般在项目的frameworks文件夹下。
- CocoaPods通过一个名为Pods.xcconfig的文件来在编译时设置所有的依赖和参数。
###结语
这次重命名项目让我好好的研究了一把CocoaPods，毕竟之前都是直接看教程敲命令行😂。不过还是要吐槽一下`pod setup`的下载速度~~楼主表示，都怪你~让我连输了三把排位！ヽ(ˋДˊ)ノヽ(ˋДˊ)ノヽ(ˋДˊ)ノ

### 其他
[简书地址](https://www.jianshu.com/p/859bda28bd4c)