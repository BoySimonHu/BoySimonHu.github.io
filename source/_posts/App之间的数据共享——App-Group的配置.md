---
title: App之间的数据共享——App Group的配置
date: 2017-12-10 16:51:04
tags: iOS日常开发
---
在实际的开发中，我们也许会有这种需求：
  `公司的旗下有两个App，当客户已经登录一个App A的情况下，再登录另一个App B时，B不再需要繁琐的登录过程就可以直接使用A已经登录的信息。但是iOS系统下有这么一个安全机制：每个应用都有自己对应的沙盒，每个沙盒之间都是相互独立的，互不能访问（没有越狱的情况下）。`
这种情况，我们应该怎么处理呢？

# 一、认识App Groups
`AppGroup allows data sharing between two different apps or even app and widgets by creating one common shared path (like document directory). Data saved over there can be accessed by any app which is associated with that particular AppGroup. It is an offline data sharing between apps.`
这是一段关于App groups的一段说明，告诉我们了App Groups可以使两个不同的APP进行数据共享，看起来这个是解决我们刚才那个问题的好方法。那就让我们开启我们的数据共享之旅吧！

# 二、创建APP
- 创建两个app，分别命名为MainApp, SubApp（为了写文章，我重新新建了两个app，只怪自己写Demo命名太乱来 ）。
- 在Apple Developer中配置两个app的App ID：
![](http://upload-images.jianshu.io/upload_images/2540460-be8c3d401499946b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![App ID Screens](http://upload-images.jianshu.io/upload_images/2540460-a40eb0103c3c922a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当我们创建app id的时候不要忘记把App Services选项中的App Groups给勾上哦，这样才能够保证我们接下来可以使用App Groups。
当我们配置完成App ID之后，会发现App Groups是Configurable状态，这是因为咱们还没有配置相应的app groups，别着急，咱们等会再来管它。
![App ID配置完成](http://upload-images.jianshu.io/upload_images/2540460-923bceea86c8e063.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 在Apple Developer中配置两个app的Profile（这里我只配置了dev的profile）：
`一次从简，我这不再赘述profile的配置了，只贴上两张图表示一下。。。`

![](http://upload-images.jianshu.io/upload_images/2540460-69ab6e8d2b3f587f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/2540460-0ef30632e2862dbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

# 三、创建App Groups
接下来就到了我们的重头戏：App Groups
- 还是要打开Apple Developer，在id那一组中又一个App Groups选项，打开就是如下的画面（多么庆幸这一个账号从没设置过任何的组）：

![App Groups初始状态](http://upload-images.jianshu.io/upload_images/2540460-505eea51a1980d70.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 然后我们Continue:
![配置App Groups](http://upload-images.jianshu.io/upload_images/2540460-02b1df304060f905.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
纳尼！！就这两个选项？？？
没错，就这两个选项，第一个是我们这个app group的描述，第二个是我们app group的id。`这个id默认是要group.打头，并且是不能去掉的`

- 还记得咱们刚才App ID的一个Configurable状态吗？咱们现在就去收拾它去~~

![](http://upload-images.jianshu.io/upload_images/2540460-9f42d36844682714.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/2540460-9636821b40d0bd3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/2540460-394b59f154cd802a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](http://upload-images.jianshu.io/upload_images/2540460-3a4d4bdf049ff374.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击Edit---勾选上刚创建好的App Group----配置完成，在返回来看一下咱们的App ID，完美~Enabled状态了。

# 四、在程序中配置App Group
- 分别打开两个程序，切换到Capabilities选项卡，找到App Groups选项，刷新一下，将App developer中的App Groups同步下来，然后勾选上咱们刚才创建的开发组。

![Xcode中配置App Groups](http://upload-images.jianshu.io/upload_images/2540460-38af398ec8a9be4d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

当我们配置完以后，会在文件目录下多出来一个.entitlements的文件。

![工程目录](http://upload-images.jianshu.io/upload_images/2540460-4eb0c58404c44597.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# 五、用代码，创建共享文件
配置工作做完了，接下来，就是我们的代码表现的时间了。
`在这里，我只创建了一个.txt来存储数据，其他类型的持久化存储同理`
废话不多说，贴代码：

- 首先是Main App写入数据

```
//Main App 通过TextField来向共享文件appGroup.txt中写入数据
- (void)textFieldDidEndEditing:(UITextField *)textField {
    //获取App Group的共享目录
    NSURL *groupURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:@"group.com.simon.app.test"];
    NSURL *fileURL = [groupURL URLByAppendingPathComponent:@"appGroup.txt"];
    
    //写入文件
    [textField.text writeToURL:fileURL atomically:YES encoding:NSUTF8StringEncoding error:nil];
}
```

- 接下来是Sub App读数据

```
//Sub App 通过获取appGroup.txt中的数据，展现在label上

//获取App Group的共享目录
    NSURL *groupURL = [[NSFileManager defaultManager] containerURLForSecurityApplicationGroupIdentifier:@"group.com.simon.app.test"];
    NSURL *fileURL = [groupURL URLByAppendingPathComponent:@"appGroup.txt"];
    
    //读取文件
    NSString *str = [NSString stringWithContentsOfURL:fileURL encoding:NSUTF8StringEncoding error:nil];
    self.shareLabel.text = str;
```

`containerURLForSecurityApplicationGroupIdentifier `当当当，没错，就是这个方法，用来在share path中创建share document。
`ps: 抛给大家个问题，感兴趣的同学可以试着找一下分享目录在哪，对于喜欢搞机的朋友来说这个问题so easy`

现在运行一下，来看一下效果：
`Main App输入数据`
![Main App](http://upload-images.jianshu.io/upload_images/2540460-2c4b3dd9a4afbf89.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`Sub App输出数据`


![Sub App](http://upload-images.jianshu.io/upload_images/2540460-a756b64e3f21c7a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可能细心的小伙伴看见了jump to sub和jump to main，这两个button是利用了`URL Types`进行的简单的app跳转，有兴趣的童鞋可以自己尝试一下。

# 六、结语
到现在为止，最简单的两个app之间的数据共享已经说完了，但是这离项目的实际应用还是有段距离的，建议大家可以根据业务自己封装一个`数据共享类`来方便自己正在项目中的使用~~~


[简书地址](https://www.jianshu.com/p/94d4106b9298)