## 什么是XSS攻击？

XSS（Cross-Site Scripting，跨站脚本攻击）是一种常见的网络安全漏洞，攻击者通过在受信任的网站上注入恶意代码，当用户浏览该页面时，恶意代码会在用户的浏览器上执行。

### XSS攻击的三种主要类型

存储型XSS（Stored XSS）：

恶意代码被永久存储在目标服务器上（如数据库）

当用户请求包含此恶意代码的页面时，代码会被执行

例如：在论坛发帖中插入恶意JavaScript代码

反射型XSS（Reflected XSS）：

恶意代码包含在URL中，服务器将其"反射"回浏览器

通常需要诱导用户点击特制的链接

例如：搜索功能直接将未过滤的搜索词显示在结果页面

DOM型XSS（DOM-based XSS）：

漏洞存在于客户端代码中，而非服务器端

恶意代码通过修改页面的DOM环境在本地执行

例如：使用document.location.hash直接操作DOM

## XSS攻击的危害

窃取用户Cookie和会话信息

劫持用户账号

修改DOM结构

利用用户浏览器进行恶意操作（如CSRF攻击）

网络钓鱼，欺骗用户提供敏感信息

传播蠕虫病毒

## 如何防御XSS攻击

### 1. 输入验证和过滤

```typescript
// 不安全的代码
document.getElementById("userContent").innerHTML = userInput;

// 安全的做法 - 使用DOMPurify库
import DOMPurify from 'dompurify';
document.getElementById("userContent").innerHTML = DOMPurify.sanitize(userInput);
```

### 2. 输出编码

```typescript
// 编码HTML实体
function encodeHTML(str) {
  return str.replace(/&/g, '&amp;')
            .replace(/</g, '&lt;')
            .replace(/>/g, '&gt;')
            .replace(/"/g, '&quot;')
            .replace(/'/g, '&#39;');
}

// 使用
document.getElementById("userContent").textContent = userInput; // 优先使用textContent
```

### 3. 使用内容安全策略(CSP)

在HTTP响应头中添加：

```tcl
Content-Security-Policy: default-src 'self'; script-src 'self' https://trusted-cdn.com;
```

或在HTML中：

```typescript
<meta http-equiv="Content-Security-Policy" content="default-src 'self'; script-src 'self' https://trusted-cdn.com;">
```

### 5. 使用现代框架

```typescript
// React自动转义
function UserComponent({ userData }) {
  return <div>{userData.name}</div>; // 自动转义
}
```

### 6. 验证URL参数

```typescript
// 检查URL参数是否包含可疑内容
function isSafeUrl(url) {
  return !/javascript:/i.test(url) && !/data:/i.test(url);
}
```

## 面试中的加分点

提及XSS的实际案例：如Twitter的XSS蠕虫、MySpace的Samy蠕虫等

讨论XSS与其他安全问题的关系：如XSS如何促成CSRF攻击

提及现代防御机制：

子资源完整性(SRI)

特权分离原则

沙箱化技术

讨论XSS的检测方法：

静态代码分析

动态扫描工具

渗透测试

针对不同环境的特定防御：

单页应用(SPA)中的XSS防御

在微服务架构中的XSS防御策略

## 代码示例：完整的XSS防御实践

```typescript
// 服务器端 (Node.js/Express)
app.use(helmet()); // 设置安全相关的HTTP头

// 输入验证
const sanitizeInput = (input) => {
  // 移除危险字符和标签
  return input.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');
};

app.post('/api/comments', (req, res) => {
  const sanitizedComment = sanitizeInput(req.body.comment);
  // 存储到数据库...
});

// 客户端
// 使用textContent而非innerHTML
document.getElementById('userComment').textContent = commentData;

// 对于需要HTML的情况，使用DOMPurify
import DOMPurify from 'dompurify';
const sanitizedHTML = DOMPurify.sanitize(htmlContent, {
  ALLOWED_TAGS: ['b', 'i', 'em', 'strong'],
  ALLOWED_ATTR: []
});
document.getElementById('formattedContent').innerHTML = sanitizedHTML;
```

