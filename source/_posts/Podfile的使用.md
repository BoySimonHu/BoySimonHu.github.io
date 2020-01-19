---
title: Podfile 语法以及基本使用
date: 2020-01-19 16:58:09
tags: CocoaPods相关
---
### pod install 的基本流程
- 查看 ~/.cocoapods/repo/master/Specs 是否存在
- 存在，从这个本地三方库信息库中获取 Podfile 中对应三方库的 git 地址
- 不存在，输出 Setting up CocoaPods Master repo，并- 拉取三方库信息库到 ~/.cocoapods/repo/中
- 使用 git 命令从 GitHub 上拉取 Podfile 中对应的三方库源码

### Podfile是什么？
Podfile是一种规范，描述一个或多个xcode项目目标的依赖项

最简单的写法：
```
target 'MyApp'
pod 'AFNetworking', '~> 1.0'
```

一种复杂的写法
```
platform :ios, '9.0'
inhibit_all_warnings!

target 'MyApp' do
  pod 'ObjectiveSugar', '~> 0.5'

  target 'MyAppTests' do
    inherit! :search_paths
    pod 'OCMock', '~> 2.0.1'
  end
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    puts "#{target.name}"
  end
end
```

### 主配置
Podfile的全局可用配置

`install!` CocoaPods的安装方法，用于设置安装方法和安装时的选项，在Podfile的安装时期调用

一共有两个参数，第一个参数是说明安装方法，第二个参数是用来说明安装选项。
例如：
```
install! 'cocoapods',                           // 用来说明使用cocoapods方法进行安装
         :deterministic_uuids => false,         // 忽略uuid重复检查
         :integrate_targets => false            // 是否将已安装的pod集成到工程中
```

涉及到的安装选项：
```
:clean
移除不被podspec用到和工程不支持平台的文件，默认true

:deduplicate_targets
是否对目标库进行去重处理，当不同的target对同一个库有不同的需求时，会在库之后添加相应的后缀，默认true，例如：
target 'MyTargetA' do
  pod 'MyPod/SubA'
end

target 'MyTargetB' do
  pod 'MyPod'
end
结果为 MyPod 和 MyPod-SubA

:deterministic_uuids   ??
uuid重复检查  默认true

:integrate_targets
是否将下载下来的三方库集成到工程中，false则只会讲三方下载到Pods文件夹中，不会集成，默认true

:lock_pod_sources
是否锁定pod下载下来的文件（文件右上角的锁），加锁会明显的影响install速度，默认为true

:warn_for_multiple_pod_sources
当多个pod源存在包含相同库名和版本时，是否警告，默认为true

:share_schemes_for_development_pods
是否允许三方库共享schemes，默认为false

:disable_input_output_paths  --- 需要check
是否禁用CocoaPods脚本阶段的输入和输出路径（复制框架和复制资源），默认为false

:preserve_pod_file_structure
是否保留所有的pod结构，包括额外的pod资源，true时会保存库原有的文件结构, 默认为false

:generate_multiple_pod_projects 
是否生成多个pod的project，true时会在Pods.xcodeproj下生成每个pod的子project，默认为false

:incremental_installation  --- 需要check
是否仅启用重新生成自上次安装以来已更改的目标及其关联项目，默认为false

:skip_pods_project_generation  --- 需要check(和integrate_targets区别？)
是否跳过生成Pods.xcodeproj，仅执行依赖项解析和下载。默认为false

```

### 依赖

##### pod
声明一个工程中所用到的依赖

一个依赖的声明，由库名称和库版本组成，其中库版本为可选项。

不加库版本号：
```
pod 'SSZipArchive'  # 每次install都将会拉取库的最新版本
```

指定库的版本号
```
pod 'SSZipArchive', '0.9'   # 每次install，都将固定拉取库的某个版本
```

除了以上两种方式，还可使用运算符进行版本限制
- `= 1.0`   固定版本号为1.0
- `> 1.0`   大于1.0
- `>= 1.0`  大于1.0，包括1.0
- `< 1.0`   小于1.0
- `<= 1.0`  小于1.0，包括1.0
- `~> 0.1.2`    0.2以下、不包括0.2，等同于 `>= 0.1.2 && < 2.0`
- `~> 0.1.3-beta.0` 测试和发布版本的0.1.3，发布版本在0.2以下，且不包含0.2。用破折号（-）分隔的组件将不考虑版本要求。

##### Build configurations
默认依赖是将会安装在工程的所有target上，但是实际开发中，由于调试或者是其他的要求，需要将依赖只安装在部分target上。
```
pod 'PonyDebugger', :configurations => ['Debug', 'Beta']
将PonyDebugger只安装在Debug和Beta target下
```
或
```
pod 'PonyDebugger', :configuration => 'Debug'
只将PonyDebugger安装在Debug下。
```

##### Modular Headers
某个库需要使用Modular Headers时：
```
pod 'SSZipArchive', :modular_headers => true
```
如果已经使用了`use_modular_headers!`属性，但是某个库又不需要使用Modular Headers：
```
pod 'SSZipArchive', :modular_headers => false
```

##### Source
默认情况下，pod install都将会从`https://github.com/CocoaPods/Specs.git`源下去拉取库。也可指定单个库的源
```
pod 'PonyDebugger', :source => 'https://github.com/CocoaPods/Specs.git'

```
**Note:** 当我们使用某几个库，或者是使用自己制作的私有库时，可以在podfile的开头先指定源
```
source 'https://github.com/CocoaPods/Specs.git'
```

##### Subspecs
一个库中可能会包含多个子库，默认的安装方式，会将所有的子库全部下载下来，若只需要某个子库，则可以使用以下方法：
```
# 下载QueryKit中的Attribute子库
pod 'QueryKit/Attribute'

# 下载QueryKit中的Attribute和QuerySet子库
pod 'QueryKit', :subspecs => ['Attribute', 'QuerySet']
```

##### Test Specs
`:testspecs`选项可用将pod库的测试代码下载下来，默认情况下不会拉取此部分的代码，声明方式如下：
```
pod 'AFNetworking', :testspecs => ['UnitTests', 'SomeOtherTests']
```
若想让pod库支持该选项，需要在Podspec中添加`test_spec`

##### 使用本地路径
在实际开发中，可能会在工程中修改某些库的代码，这种情况下，使用本地路径的方式可以更方便的完成开发:
```
pod 'AFNetworking', :path => '~/Documents/AFNetworking'
```
**Note:** 由于这种方式实际上是链接本地目录，所以使用该方式修改的代码是会被保存下来的。

##### 从远程仓库目录中的podspec获取
我们可以直接拉取远程仓库的某个分支或某个commit的代码
```
# 拉取AFNetworking的master分支代码
pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git'

# 拉取AFNetworking的dev分支代码
pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :branch => 'dev'

# 拉取AFNetworking的0.7.0 tag的代码
pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :tag => '0.7.0'

# 拉取AFNetworking中commit号为082f8319af的代码
pod 'AFNetworking', :git => 'https://github.com/gowalla/AFNetworking.git', :commit => '082f8319af'
```
**Note：** 本地路径中的库，若也是使用pod管理，则同样可以指定分支
```
pod 'AFNetworking', :path => '~/Documents/AFNetworking', :branch => 'dev'
```

##### 从远程source之外的地方获取podspec配置
如果想拉取一个不在source上的podspec配置，则可以使用`:podspec`方式进行拉取。注意podspec需要是一个http的资源
```
pod 'JSONKit', :podspec => 'https://example.com/JSONKit.podspec'
```

##### script_phase
可以通过Podfile先target中添加脚本
```
target 'ZipApp' do
  script_phase :name => 'HelloWorldScript', :script => 'echo "Hello World"'
  script_phase :name => 'HelloWorldScript', :script => 'puts "Hello World"', :shell_path => '/usr/bin/ruby'


  pod 'SSZipArchive'
end
```

##### 抽象target
可以定义`abstract_target`来方便其他的target进行继承
```
# Defining an abstract target
abstract_target 'Networking' do
  pod 'AlamoFire'

  target 'Networking App 1'
  target 'Networking App 2'
end

# Note: There are no targets called "Shows" in any of this workspace's Xcode projects
abstract_target 'Shows' do
  pod 'ShowsKit'

  # The target ShowsiOS has its own copy of ShowsKit (inherited) + ShowWebAuth (added here)
  target 'ShowsiOS' do
    pod 'ShowWebAuth'
  end

  # The target ShowsTV has its own copy of ShowsKit (inherited) + ShowTVAuth (added here)
  target 'ShowsTV' do
    pod 'ShowTVAuth'
  end

  # Our tests target has its own copy of
  # our testing frameworks, and has access
  # to ShowsKit as well because it is
  # a child of the abstract target 'Shows'

  target 'ShowsTests' do
    inherit! :search_paths
    pod 'Specta'
    pod 'Expecta'
  end
end
```

##### inherit!
有三种类型：
- `:complete` 继承父级所有行为；
- `:none` 什么行为都不继承；
- `:search_paths` 继承父级的 search paths

```
target 'App' do
  target 'AppTests' do
    inherit! :search_paths
  end
end
```

### target配置
用来控制CocoaPods最后生成的Project

##### platform
声明库支持的平台，默认为`4.3 for iOS, 10.6 for OS X, 9.0 for tvOS and 2.0 for watchOS`

参数：
- 平台名称
- 指定版本
```
platform :ios, '4.0'
# 或
platform :ios
```

##### project
声明xcode工程应该与哪个target相链接，指定Pod管理的项目路径
```
# This Target can be found in a Xcode project called `FastGPS`
target 'MyGPSApp' do
  project 'FastGPS'
  ...
end

# Same Podfile, multiple Xcodeprojects
target 'MyNotesApp' do
  project 'FastNotes'
  ...
end
```

##### inhibit_all_warnings!
消除编译警告，默认全局。

如果指向消除某个库的编译警告，可使用如下语法:
```
pod 'SSZipArchive', :inhibit_warnings => true
```
或 全局消除下，依然让某个库有警告：
```
pod 'SSZipArchive', :inhibit_warnings => false
```

##### use_modular_headers!
全局使用modular_headers，局部同上

##### use_frameworks!
Pod使用静态库

##### supports_swift_versions
支持的swift版本
```
target 'MyApp' do
  supports_swift_versions '>= 3.0', '< 4.0'
  pod 'AFNetworking', '~> 1.0'
end
```

```
supports_swift_versions '>= 3.0', '< 4.0'

target 'MyApp' do
  pod 'AFNetworking', '~> 1.0'
end

target 'ZipApp' do
  pod 'SSZipArchive'
end
```

### Workspace
Workspace配置以及全局设置

##### workspace
生成的`workspace`路径，可用来修改最后workspace的生成名称，默认与工程名称相同
```
workspace 'MyWorkspace'
```

##### generate_bridge_support!
生成BridgeSupport文件 

##### set_arc_compatibility_flag!
指定-fobjc-arc标志应添加到OTHER_LD_FLAGS

### Hooks
在install过程中调用，Hook是全局的，不会针对某个target进行操作

##### plugin
使用插件进行Hook
```
# 使用插件cocoapods-keys，传递参数Eidolon
plugin 'cocoapods-keys', :keyring => 'Eidolon'
# 使用插件slather
plugin 'slather'
```

##### pre_install
在pod下载后安装之前做一些操作。
```
pre_install do |installer|
  # Do something fancy!
end
```

##### post_install
在生成xcode项目之前，做最后的更改，`installer`是该方法唯一的参数
```
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      config.build_settings['GCC_ENABLE_OBJC_GC'] = 'supported'
    end
  end
end
```
