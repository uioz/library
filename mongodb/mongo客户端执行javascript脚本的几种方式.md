# 直接在shell中运行javascript代码

```javascript
let post = { // 此处回车会进入多行模式,进入多行模式后连续三次换行则放弃执行编辑的脚本
    title:'shell test',
    data:new Date()// 还可以使用javascript标准语法
}
```

# 启动客户端时候禁用自动连接

只有禁用了自动连接后,我们才可以在启动命令行启动客户端的时候指定脚本.

```bash
mongo --nodb
```

# 在启动客户端的时候指定脚本

脚本1:
```javascript
const hello = 'hello';
```

脚本2:
```javascript
const world = 'world';
```

脚本3:
```
print(hello+world);
```

命令行:
```
 mongo .\script1.js .\script2.js .\script3.js --nodb
```

输出:
```
MongoDB shell version v4.0.3
loading file: .\script1.js
loading file: .\script2.js
loading file: .\script3.js
helloworld
```

## 巨坑

javascript中的`console.log`居然不能使用,被换成了`print`真是奇葩.

# 在连接远程服务器(指定服务器)的时候执行脚本

```
mongo --quite server-1:30000/foo script1.js script2.js script3.js
```

**注意**:`--quiet`表示无输出启动.

# 在shell运行时加载外部脚本

```
// -----
load('./script.js');
// -----
```

# 在自启动脚本中执行

在用户目录下创建一个`.mongorc.js`文件后mongo会在启动的时候自动去加载执行它.

**注意**:留意windows和linux用户目录不同.



