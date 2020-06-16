# 把 HTTPS 搬进冰箱需要 4 步

本文章的目的是让你在本地创建一个支持 HTTPS 服务的服务端程序.  
最终通过访问的地址是 `https://example.com:8888` 去访问.

本篇文章的操作环境是 linux 中, 不同系统所使用的工具不同但是基本概念是完全一致的.

# 使用 openssl 来创建证书

这个环节你需要了解有关 HTTPS 中的几个基本概念包括:
1. HTTPS SSL 层链接的建立
2. CA 认证
3. 数字签名
4. 非对称加密

这篇[文章详细](https://segmentfault.com/a/1190000014835279#item-1-2)的介绍了上诉关键内容. 如果仅仅是想体验过程, 复杂的内容也可以先跳过.

## 证书

证书是一个文件, 文件格式一般遵循 PEM 规范定义.  
证书用于验证, 所以证书上存在很多有用的信息, 例如网站域名, 例如 `example.com`. 
证书在 HTTPS 链接建立之初会从服务器发送到客户端, 拿上面的例子来说, 我们访问的域名是 `xxxx.com` 而证书中的域名是 `example.com` 不匹配时浏览器会提示安全错误.

证书上存在一个公开的 HASH 值, 是通过证书上公开的信息来计算的.  

为了防止证书被篡改, 证书上存在一个数字签名是由证书颁发机构提供的, 具体做法是证书颁发机构使用自己的私钥和证书上公开的签名算法将之前 HASH 进行加密计算出一个数字签名. 当客户端获取到证书后会通过公开的签名算法以及证书颁发机构提供的公钥来进行解密, 获取 HASH 来和证书公开的 HASH 进行比对.

## 建立自签名证书

在建立私钥的时候需要输入 `passphrase` 这相当于为私钥建立了一个密码, 请牢记这个密码.



## 建立根证书(Root Certificate)

### 添加根证书到浏览器

**注意**: 请优先使用这种方式来注册证书, 而不是注册为系统证书.

在很多软件中都使用的是内置证书数据库而非系统所提供的证书, 例如 firefox 和 google chrome 甚至 nodejs, 所以在注册根证书前一定要查询一下你所使用的软件是否内置证书.


### 添加根证书到系统

具体的方式取决于你使用的操作系统, 在我使用的 linux 中证书 crt 文件可以放置到 `/usr/local/share/ca-certificates` 路径下.

然后运行:
```bash
update-ca-certificates 
```

便可以将证书注册为根证书.

# 修改 hosts

为了成功通过域名来访问我们的服务器而不是 IP 地址, 我们需要在 hosts 文件中添加一条域名 `exmaple.com` 到 IP 的映射.

找到 hosts 文件, 在 linux 上这个文件路径为 `/etc/hosts`, 或者:
```bash
vi /etc/hosts
``` 

在文件的末尾添加一行:
```
192.168.1.106 example.com
```
我使用的是本机局域网 IP 大家可以使用 `127.0.0.1`:
```
127.0.0.1 localhost example.com
```

# 代码

服务端程序使用的是 nodejs ,当然其他语言也是可以的, 在 nodejs 中相较于 HTTP 服务器 HTTPS 只多出了两个参数, 在其他语言中的差异可能和 nodejs 差不多.

请注意代码注释的部分:
```javascript
const HTTPS = require("https");
const FS = require("fs");
const PATH = require("path");

const ProjectRootPath = process.env.PWD;
const PrivateKey = FS.readFileSync(PATH.join(ProjectRootPath, "./cwj.key"));
const Certificate = FS.readFileSync(PATH.join(ProjectRootPath, "./cwj.crt"));

const Server = HTTPS.createServer({
  passphrase: "4012", // 创建私钥时候输入的 passphrase
  key: PrivateKey, // 私钥
  cert: Certificate, // 证书
});

Server.on("request", function (request, response) {
  response.writeHead(200);
  response.end("hello world\n");
});

Server.listen(8888,'192.168.1.106');

```

# 测试

在导入了根证书的浏览器地址栏中输入 `https://example.com:8888` 如果浏览器没有提示错误, 那么恭喜你 HTTPS 服务器已经搭建完成.

# 参考

> https://segmentfault.com/a/1190000014835279#item-1-2