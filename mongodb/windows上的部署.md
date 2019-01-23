## 前言

由于是第一次安装`mongodb`还是在不常配置的win平台上相对于`linux`来说还是稍显复杂一些.

采用`mongodb v4.0.3`版本,使用win下的zip包进行安装.

## 与mysql区别

和`mysql`相同的是`mongodb`也同样的启动一个本地的服务器允许多个客户端进行请求.

在win下`bin`目录下的两个文件对应着不同的关系:

- mongod.exe == 服务端
- mongo.exe == 客户端

## 配置

新建一个目录后将我们解压出的内容放入,和常见的软件包配置一样,`mongodb`服务端拥有两种配置模式.

1. 启动时输入命令参数
2. 使用配置文件

这里我们使用配置文件的方式.

在存放`mongodb`位置的文件夹中找到与`bin`同目录下新建一个`data`文件夹用于保存数据库,在新建一个`logs`文件夹,内部新建一个`mongo.log`文件用于保存日志.

然后再在`bin`同级别目录下建立一个`mongo.conf`文件用于保存配置文件.


`mongo.conf`填入的内容如下:
```
#数据库路径  
dbpath=D:\mongodb\data
#日志输出文件路径  
logpath=D:\mongodb\logs\mongo.log
#错误日志采用追加模式  
logappend=true  
#启用日志文件，默认启用  
journal=true  
#这个选项可以过滤掉一些无用的日志信息，若需要调试使用请设置为false  
quiet=true  
#端口号 默认为27017  
port=27017   
```

## 环境变量

我的安装路径在:
```
D:\mongodb\
```

然后将
```
D:\mongodb\bin
```

添加到环境变量中.

## 测试

在`cmd`或者`powerShell`中执行`mongod --config "D:\mongodb\mongo.conf"`命令.

`mongodb`应该会执行起来利用我们指定的配置,此时的终端应该无法输入内容.

在浏览器中输入`localhost:27017`应该会有提示信息,这个信息是由`mongod`服务器程序显示的.

## 简化启动

网上有很多帖子将`mongodb`设置成了一项系统服务开机自启动,好处是你不必在手动输入启动时候的参数了,也不必使用终端.

但是我这里给出一个简单粗暴的方式,不过前提是你按照上一步配置了环境变量.

在`bin`目录下新建一个`mongodstart.bat`的文件.

`mongodstart.bat`文件内容如下:
```
mongod --config "D:\mongodb\mongo.conf"
```
保存就可以了.

然后你就直接可以在终端中输入`mongodstart`命令来运行`mongodb`的服务器程序.

## 使用客户端

如果你是本地启动的服务器程序且没有修改端口,那么在命令行中直接键入`mongo`就可以运行客户端,此时进入的模式就类似于`mysql`的命令行.

如果客户端和服务器程序是分开的那么需要指定地址和端口号,格式也很简单我们可以指定地址和端口:
```
mongo 127.0.0.1:27017
```

## 帮助

`mongod`和`mongo`命令都拥有`--help`选项使用它就可以获取命令及帮助.


