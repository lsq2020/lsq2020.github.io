---
layout: post
title: 'Cluster Server'
subtitle: '服务器集群学习'
date: 2017-04-05
categories: 技术
tags: node
---

### 代码位置

名词解释：
&ensp;&ensp;&ensp;** ./ 代表项目目录**
&ensp;&ensp;&ensp;** 主节点 即 server节点 提供http服务**
&ensp;&ensp;&ensp;** 从节点 机 node节点 接收并处理主节点分发的业务逻辑，然后返回**

---
1.&ensp;启动程序：./bin/www
这段代码 通过判断 X_ServerType 为 node 而返回，并且启动成 一个节点服务器
`if (global.X_ServerType == 'node') return tools.execFutureFn(nodeServer.connectHttp);`

X_ServerType 来源于哪里？ 注意第一行就是
`var app = require('../app')`

---

2.&ensp;  解析启动参数  ./app.js

  ```
//-> 解析启动参数,判断自己是 http服务器 还是 服务节点
process.argv.forEach((val, index) => {
    if (index > 1) X_StartArgs[val] = '1';
    if (val == 'server') X_ServerType = 'server';
});
//-> 绑定全局对象
global.X_StartArgs = X_StartArgs;
global.X_ServerType = X_ServerType;

  ```
之所以要把 参数 放在app.js 中解析,是因为 不同的服务 应该有不同名称的日志文件，而区分的方式就是要得到服务器节点类型，所以应该尽可能的在 生成日志前 就解析出 服务节点类型。
  在 解析启动参数后，紧跟着的是这段代码
  ```
  var log = global.loghelper = require('./logHelper');
log.use(app);
  ```
2.1&ensp; 加载日志文件 ./logHelper.js
```
{
    "appenders": [
        {
            "type": "file",
            "filename": webLogPath + "/ainitErp" + (global.X_ServerType == 'node' ? "_" + process.pid.toString() : "") + ".log",
            "category": ["logInfo", "console"],
            "maxLogSize": 2 * 1024 * 1024,//每个文件最大2m
            "backups": 1//备份的日志文件数量
        }
    ]
    , "replaceConsole": true
    , "levels": {"logInfo": "DEBUG"}
}
```
 这里判断服务节点为 node时，取当前进程id 生成 日志文件名

---
3.&ensp; 从节点 与 主节点 连通： ./microService.js
  ```
  /************
   * 连接 http服务器
   ***********/
  function connectHttp() {
      var i = 0;
      //-> 尝试与 服务器建立连接
      while (isConnectHttp == false && i++ < 60) {
          //-> 测试服务器是否打开,由于 1次 测试要花 1秒钟,所以等待60秒 1分钟 就可以了
          if (!testServerOpen()) continue
          else break;
      }
      //-> 与服务器建立连接
      console.info(serverAddress);
      webSocketNode = socketIoClient(serverAddress);
      webSocketNode.on('connect', serverConnect);
      webSocketNode.on('disconnect', serverDisConnect);
  }
  exports.connectHttp = connectHttp;
  ```

这是部分代码，先判断 主节点即 http服务器 是否打开，如果打开的话,就与它建立连接
