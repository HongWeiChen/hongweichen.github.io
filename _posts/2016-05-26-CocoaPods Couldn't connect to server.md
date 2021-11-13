# CDN….Response: Couldn't connect to server

在运行pod install的时候提示
```
CDN….Response: Couldn't connect to server
```

解决方法：

打开podfile文件，在顶部新增
```
source 'https://github.com/CocoaPods/Specs.git'
```
