# 什么是webSocket

WebSocket 是一种在单个 TCP 连接上进行**全双工通信**的协议。它提供了浏览器与服务器之间的双向通信通道，使得服务器可以主动向客户端推送数据。

## WebSocket 特点

- **建立在 TCP 协议之上**
- 与 HTTP 协议有着良好的兼容性
- 数据格式轻量，性能开销小
- 可以发送文本，也可以发送**二进制数据**
- **没有同源限制，客户端可以与任意服务器通信**
- **协议标识符是 ws（非加密）和 wss（加密）**



## WebSocket 工作原理

### 连接建立过程

客户端发起 HTTP 请求，请求头中包含：

```tcl
GET /chat HTTP/1.1
Host: server.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
```

### 服务器响应

```tcl
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
```

### 数据传输

建立连接后，双方可以进行双向数据传输：

- 使用帧（frames）进行数据传输
- 支持文本和二进制数据
- 保持连接活跃



## WebSocket API 使用

1. 安装声明文件

   `pnpm i @types/ws -D`

2. 引入ws

   ```typescript
   import ws from 'ws'
   ```

3. 创建socket服务 8080端口

   ```typescript
   const wss=new ws.Server({port:8080},()=>{console.log('socket start on 8080')})
   ```

4. 监听客户端的连接

   ```typescript
   wss.on('connection',(ws)=>{
    //监听客户端的信息
      console.log('客户端连接成功')
   })
   ```

当前流行版本的websocket连接建立:
```JS
const socket = new WebSocket('ws://example.com/socketserver');

// 连接建立时触发
socket.onopen = function(event) {
    console.log('WebSocket连接已建立');
};

// 接收到消息时触发
socket.onmessage = function(event) {
    console.log('收到消息：', event.data);
};

// 连接关闭时触发
socket.onclose = function(event) {
    console.log('WebSocket连接已关闭');
};

// 发生错误时触发
socket.onerror = function(error) {
    console.error('WebSocket错误：', error);
};
```

### 发送信息

```typescript
// 发送文本消息
socket.send('Hello, Server!');

// 发送JSON数据
socket.send(JSON.stringify({
    type: 'message',
    content: 'Hello, Server!'
}));

// 发送二进制数据
socket.send(new Blob(['Hello'], {type: 'text/plain'}));
```





## 心跳机制

websocket长时间不使用或者存在网络波动，为保持连接活跃,那就要考虑一下心跳检测。

保持连接活跃：

```tcl
[客户端] <---> [路由器/防火墙] <---> [服务器]
```



- 许多网络设备(如路由器、防火墙、负载均衡器)会自动关闭一段时间内没有数据传输的TCP连接
- 这些设备通常有一个"超时"设置，如果一个连接在特定时间内(比如60秒)没有任何数据传输，就会被关闭
- 心跳检测通过定期发送小数据包，确保连接不会因为不活跃而被这些中间设备关闭

检测连接状态：

```tcl
客户端 <--- X (网络中断) ---> 服务器
```



- 在网络环境中，连接可能因为各种原因意外断开(网络波动、服务器重启等)
- 客户端可能并不知道连接已经断开(半开连接)
- 心跳检测可以主动发现连接问题，而不是等到需要发送重要数据时才发现连接已断开

触发重连机制：

```tcl
                  [重连机制]
                       |
                       v
[客户端] --心跳失败--> [检测断开] --触发--> [重新连接]
```



- 当心跳检测失败时，可以立即启动重连流程
- 这样可以最小化连接中断对用户体验的影响

减少资源浪费：

- 服务器可以通过心跳检测识别出"僵尸"连接(客户端已经离线但连接仍保持)
- 服务器可以主动关闭这些连接，释放资源

代码中的心跳检测实现：

- 定期(每5秒)向服务器发送一个类型为HEART的消息
- 发送前检查连接状态，只有在连接打开的情况下才发送
- 如果连接已关闭，则清除定时器，不再尝试发送心跳

</think>

```typescript
function heartbeat() {
    if (!socket) return;
    if (socket.readyState !== 1) return;
    socket.send('ping');
    setTimeout(heartbeat, 30000);
}
```



![image-20250225172623312](D:\HeinrichHu\resource\computer network knowledge\06_websocket.assets\image-20250225172623312.png)



我们需要使用心跳检测来进行 **保活**的机制:

1. 每5秒向服务器发送一条特殊的心跳消息(type:1)
2. 通过type:1标识，接收方可以识别这是心跳包而非业务数据
3. 前端收到type:1的消息时通常不做UI处理，而是直接回复或忽略

```typescript
const state = {HEART:1, MESSAGE:2};

// 监听客户端消息
socket.on('message',(e)=>{
    console.log(e.toString());
    // 广播消息给所有客户端
    wss.clients.forEach((client)=>{
        client.send(JSON.stringify({
            type: state.MESSAGE,
            message: e.toString()
        }));
    });
});

// 心跳检测机制（应放在connection事件中）
let heartInterval = null;

function startHeartbeat() {
    // 先清除可能存在的旧定时器
    if (heartInterval) clearInterval(heartInterval);
    
    // 创建新的心跳定时器
    heartInterval = setInterval(() => {
        if (socket.readyState === ws.OPEN) {
            socket.send(JSON.stringify({
                type: state.HEART,
                message: '心跳检测'
            }));
        } else {
            clearInterval(heartInterval);
        }
    }, 5000);
}

// 开始心跳检测
startHeartbeat();
```

分析上述代码:

1. const state={HEART:1,MESSAGE:2} - 定义了消息类型的常量，HEART 表示心跳消息，MESSAGE 表示普通消息。

2. socket.on('message', (e) => { ... }) - 监听客户端发送的消息事件，当服务器接收到客户端的消息时触发。

3. console.log(e.toString()) - 将接收到的消息转换为字符串并打印到控制台。

4. wss.clients.forEach((client) => { ... }) - 遍历所有连接到服务器的客户端。

5. 内部的 client.send(...) - 向每个客户端发送消息，包含类型和内容，这实现了广播功能。

6. let heartInreaval=null - 定义心跳检测的定时器变量。

7. const heartCheck=() => { ... } - 定义心跳检测函数。

8. 在心跳检测函数中:

   检查 socket.readyState === ws.OPEN - 判断 WebSocket 连接是否处于打开状态

   如果连接开着，发送一个类型为 HEART 的心跳消息

   如果连接不是开着的，清除定时器

   设置定时器每 5 秒执行一次心跳检测



### 广播式传送消息

广播式传送消息意味着所有客户端都可以收到消息，我们可以使用wss.client进行遍历发送。

```js
const state={HEART:1,MESSAGE:2}
socket.on('message',(e)=>{
console.log(e.toString())
wss.clients.forEach((client)=>{
client.send('我是服务端 发送给客户端的消息是'+e.toString())
	})
})

```



1. 只有socket状态等于ws.OPEN的时候，我们才会给前端发送心跳检测:`socket.readyState===ws.OPEN)`
2. 如果当前状态不处于ws.OPEN,那么则清理对应的计时器

### 断线重连

实现自动重连机制：

```typescript
function connect() {
    socket = new WebSocket('ws://example.com/socketserver');
    
    socket.onclose = function() {
        console.log('连接断开，正在尝试重连...');
        setTimeout(connect, 5000);
    };
}
```



### 前端代码

如果是1的话前端不做处理，此时正在做心跳检测，如果是2的话前端才进行数据增加。

```typescript
ws.addEventListener('message',(e)=>{
    let li=document.createElement('li')

    let data=JSON.parse(e.data)
    if(data.type===2){
            li.innterText=e.data.message;
            list.appendChild(li)
    }

})
```



## 应用场景

- 实时聊天应用
- 在线游戏
- 实时数据展示（股票行情、体育比分）
- 协同编辑
- 实时监控系统



## 安全考虑

- 使用 wss:// 而不是 ws:// 进行加密通信
- 实现身份验证机制
- 防止 XSS 和注入攻击
- 实现消息验证
- 限制连接数和消息大小



## 最佳实践

### 错误处理

```typescript
socket.onerror = function(error) {
    console.error('WebSocket错误：', error);
    // 实现错误处理逻辑
};
```

### 关闭连接

```typescript
// 正常关闭连接
socket.close(1000, "正常关闭");

// 清理资源
window.onbeforeunload = function() {
    if (socket) {
        socket.close();
    }
};
```

### 消息队列

```typescript
const messageQueue = [];
function sendMessage(message) {
    if (socket.readyState === WebSocket.OPEN) {
        socket.send(message);
    } else {
        messageQueue.push(message);
    }
}
```





## 前端构建

```typescript
const ws=new WebSocket('ws://localhost:8080')
//ws就是协议，和http是一样的，如果有证书就是wss
ws.addEventListener('open',()=>{
    console.log('连接成功')
})
```

前后端都是一样的，通过 `ws.send()`来发送消息:
![image-20250225171632057](D:\HeinrichHu\resource\computer network knowledge\06_websocket.assets\image-20250225171632057.png)

![image-20250225171722963](D:\HeinrichHu\resource\computer network knowledge\06_websocket.assets\image-20250225171722963.png)

前端也是通过以下方式进行接收服务端返回的信息:
```js
ws.addEventListener('message',(e)=>{
    let li=document.createElement('li')
    li.innterText=e.data;
    list.appendChild(li)
})
```

