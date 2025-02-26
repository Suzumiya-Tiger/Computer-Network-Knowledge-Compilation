# sendBeacon 的应用业务场景

navigator.sendBeacon() 是一个 Web API，它提供了一种可靠的、非阻塞的方式来发送数据到服务器，主要应用于以下业务场景：

## 页面卸载时的数据发送

这是 sendBeacon 最主要的应用场景。当用户关闭页面、刷新页面或导航到其他页面时，常规的 AJAX 请求可能会被浏览器取消，而 sendBeacon 可以确保数据在页面卸载过程中仍能可靠发送：

```typescript
window.addEventListener('unload', function() {
  navigator.sendBeacon('/analytics', JSON.stringify({
    event: 'page_close',
    timeSpent: calculateTimeSpent()
  }));
});
```

## 分析和统计数据收集(埋点)

- 用户行为跟踪：记录用户在页面上的停留时间、点击行为、滚动深度等
- 性能指标上报：收集页面加载时间、资源加载情况等性能数据
- 错误日志上报：在页面出错时发送错误信息

## 会话结束时的状态保存

当用户离开应用时，可以使用 sendBeacon 保存用户的最后状态：

```typescript
window.addEventListener('visibilitychange', function() {
  if (document.visibilityState === 'hidden') {
    navigator.sendBeacon('/api/save-state', JSON.stringify(userState));
  }
});
```

## 批量数据上报

将多个小型事件数据缓存在客户端，然后在适当时机（如页面卸载前）一次性发送：

```typescript
// 收集用户事件
function logUserEvent(event) {
  if (!window.eventBuffer) window.eventBuffer = [];
  window.eventBuffer.push(event);
}

// 页面卸载时发送
window.addEventListener('unload', function() {
  if (window.eventBuffer && window.eventBuffer.length > 0) {
    navigator.sendBeacon('/log', JSON.stringify(window.eventBuffer));
  }
});
```

## 心跳检测

在单页应用中，可以使用 sendBeacon 定期发送心跳信号，表明用户仍在活跃：

```typescript
setInterval(function() {
  navigator.sendBeacon('/api/heartbeat', JSON.stringify({
    userId: currentUser.id,
    timestamp: Date.now()
  }));
}, 30000); // 每30秒发送一次
```

# sendBeacon 的优势

非阻塞：不会影响页面的卸载过程或用户体验

可靠性高：浏览器会尽最大努力确保数据发送，即使在页面关闭过程中

简单易用：API 简洁，使用方便

不受同源策略限制：可以发送跨域请求

sendBeacon 特别适合那些需要在页面生命周期结束时可靠发送数据，且不需要等待响应的场景。



# sendBeacon 的缺点

虽然 navigator.sendBeacon() 在特定场景下非常有用，但它也存在一些明显的局限性和缺点：

## 无法处理响应

最大的缺点是 sendBeacon 是单向通信 - 它不提供任何获取服务器响应的机制：

```typescript
// 无法获取响应状态码、响应内容或错误信息
navigator.sendBeacon('/api/endpoint', data);
// 没有回调函数、Promise 或其他方式来处理响应
```

## 数据大小限制

浏览器对 sendBeacon 请求的数据大小有限制，通常在几 MB 左右（各浏览器实现不同）：

- Chrome: 约 64KB
- Firefox: 约 64KB
- Safari: 约 64KB

超过限制的数据可能导致请求失败。

## 请求方法和格式限制

- 只能使用 POST 方法，不支持 GET、PUT、DELETE 等其他 HTTP 方法
- 无法自定义请求头（除了 Content-Type）
- 无法设置超时、重试策略等

## 调试困难

由于没有响应和错误处理机制，调试 sendBeacon 请求相对困难：

- 无法在客户端确认请求是否成功
- 必须依赖服务器日志来验证数据是否正确接收
- 开发工具中的网络面板显示的信息有限



# 具体实现

## 后端

无需发送复杂数据，只需要返回简单数据，且只能定义Post请求:


![image-20250226152746217](D:\HeinrichHu\resource\computer network knowledge\07_sendBeacon.assets\image-20250226152746217.png)

## 前端

前端可以给每个按钮点击事件触发埋点，每次点击就通过sendBeacon上报给后端，或者把错误日志上报给后端。

![image-20250226152929155](D:\HeinrichHu\resource\computer network knowledge\07_sendBeacon.assets\image-20250226152929155.png)