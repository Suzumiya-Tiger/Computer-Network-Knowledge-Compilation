# 浏览器输入url的流程

URL是由访问协议，服务器名称(域名)，目录名，文件名(目录名和文件名统一称之为请求文件(资源)的路径名)。

![image-20250224153119179](D:\HeinrichHu\resource\computer network knowledge\03_浏览器输入url发生了什么.assets\image-20250224153119179.png)

## DNS的查询规则

DNS查询是为了将域名解析为对应的IP地址。完整的DNS查询顺序如下（按优先级排序）：

1. **浏览器缓存**

   - 浏览器会缓存一定时间的DNS记录
   - 可以通过 `chrome://net-internals/#dns` 查看Chrome的DNS缓存
2. **操作系统缓存**

   - 系统会维护一份DNS缓存
   - Windows可通过 `ipconfig /displaydns` 查看
   - Linux/Mac可通过 `nscd` 服务查看
3. **本地hosts文件**

   - Windows位置：`C:\Windows\System32\drivers\etc\hosts`
   - Linux/Mac位置：`/etc/hosts`
   - 可以手动设置域名和IP的映射关系
4. **DNS服务器递归查询**
   如果前三步都没有找到对应记录，则会进行DNS递归查询：

   a. **本地DNS服务器**

   - 通常是由ISP提供的DNS服务器
   - 也可以是手动配置的DNS服务器（如8.8.8.8）

   b. **根域名服务器**

   - 全球共13组根域名服务器
   - 存储顶级域名服务器的信息

   c. **顶级域名服务器**

   - 管理如.com、.net、.org等顶级域名
   - 存储其下一级域名服务器的信息

   d. **权威域名服务器**

   - 保存实际的域名记录
   - 能够返回最终的IP地址

整个查询过程是递归的，如果在任何一步找到了对应的记录，就会立即返回结果，不再继续后续的查询步骤。

## OPTIONS请求

OPTIONS请求是HTTP协议中的一种预检请求（Preflight Request），主要用于检查实际请求是否可以被服务器所接受。

### 1. 触发条件

OPTIONS请求通常在以下情况下自动触发：

1. **跨域请求**（CORS）且满足以下任一条件：
   - 使用了非简单HTTP方法（PUT、DELETE、CONNECT、OPTIONS、TRACE、PATCH）
   - 包含了非简单HTTP头部字段
   - Content-Type 不是以下之一：
     - application/x-www-form-urlencoded
     - multipart/form-data
     - text/plain

### 2. 主要用途

1. **检查服务器支持的HTTP方法**

   - 服务器会在响应头中返回 `Allow` 字段，列出支持的方法
2. **CORS预检**

   - 检查跨域请求是否被允许
   - 检查允许的请求头和方法
   - 检查是否允许发送身份凭证（cookies）

### 3. 重要的响应头

```http
Access-Control-Allow-Origin: http://example.com
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: Content-Type
Access-Control-Max-Age: 86400
Access-Control-Allow-Credentials: true
```

### 4. 预检请求的缓存

- 通过 `Access-Control-Max-Age` 指定预检请求的缓存时间
- 在缓存有效期内，不会重复发送OPTIONS请求
- 默认缓存时间因浏览器而异

### 5. 示例场景

```javascript
// 这个请求会触发OPTIONS预检
fetch('https://api.example.com/data', {
method: 'POST',
headers: {
'Content-Type': 'application/json',
'Custom-Header': 'value'
},
body: JSON.stringify({ key: 'value' })
});
```

### 6. 性能影响

1. **优点**

   - 提供了安全检查机制
   - 可以预先发现请求问题
   - 支持请求缓存
2. **缺点**

   - 增加了额外的网络请求
   - 可能影响应用性能
   - 增加服务器负载

### 7. 最佳实践

1. 合理设置 `Access-Control-Max-Age` 缓存时间
2. 尽可能使用简单请求
3. 在开发环境中正确配置CORS
4. 监控OPTIONS请求的频率和性能影响

## 浏览器缓存

浏览器缓存分为强缓存和协商缓存两种机制。

### 强缓存

强缓存指的是让浏览器强制缓存服务器提供的资源。

强缓存是利用http响应头中的 `Expires` 和 `Cache-Control` 来控制缓存。

常用于缓存一些静态资源，比如css、图片等资源，我们会在服务器端通过setHeader来设置强缓存和其规则。

![image-20250224160420307](D:\HeinrichHu\resource\computer network knowledge\03_浏览器输入url发生了什么.assets\image-20250224160420307.png)

第二次发送关联强缓存的请求时，浏览器直接返回缓存内容，无需重复对服务端发送请求。

强缓存一般存入硬盘缓存或者内存缓存，内存缓存强用于多次刷新浏览器，第一次刷新浏览器都是读取硬盘缓存。

#### 1.1 Expires（HTTP/1.0）
- 使用绝对时间，如：`Expires: Wed, 21 Oct 2023 07:28:00 GMT`
- 受限于客户端时间，如果客户端时间错误会导致缓存错误
- 现已被 Cache-Control 替代，但为了兼容仍然使用

#### 1.2 Cache-Control（HTTP/1.1）
- 使用相对时间，优先级高于 Expires
- 常用指令：
  - `max-age=<seconds>`: 缓存最大有效期
  - `no-cache`: 强制验证缓存
  - `no-store`: 禁止缓存
  - `private`: 仅浏览器可缓存
  - `public`: 所有中间节点都可缓存
  - `must-revalidate`: 过期后必须验证

![image-20250224160555606](D:\HeinrichHu\resource\computer network knowledge\03_浏览器输入url发生了什么.assets\image-20250224160555606.png)

### 协商缓存

当强缓存失效后，浏览器会发送请求到服务器，验证资源是否真的过期。

#### 2.1 Last-Modified/If-Modified-Since
- `Last-Modified`: 服务器响应资源的最后修改时间
- `If-Modified-Since`: 客户端请求时带上上次响应的 Last-Modified 值
- 缺点：
  - 时间精确度只到秒
  - 文件修改时间变化但内容未变时也会重新下载
  - 服务器动态生成文件时，最后修改时间可能无意义

#### 2.2 ETag/If-None-Match
- `ETag`: 服务器响应资源的唯一标识（通常是文件内容hash值）
- `If-None-Match`: 客户端请求时带上上次响应的 ETag 值
- 优点：
  - 比 Last-Modified 更精确
  - 可以解决 Last-Modified 的所有缺点
- 缺点：
  - 计算 ETag 值会消耗服务器资源
  - 分布式系统中要保证 ETag 的计算方式一致

### 缓存判断流程

1. 浏览器发起请求
2. 检查强缓存是否有效
   - 有效：直接使用缓存（200 from disk/memory cache）
   - 无效：进入协商缓存流程
3. 发送请求到服务器验证协商缓存
   - 资源未修改：返回304，使用本地缓存
   - 资源已修改：返回200和新资源

### 最佳实践

1. **静态资源策略**

```http
Cache-Control: public, max-age=31536000
ETag: "abc123"
```

2. **动态资源策略**

```http
http
Cache-Control: no-cache
ETag: "xyz789"
```

3. **不缓存策略**

```http
Cache-Control: no-store
```

### 常见问题解决

1. **强制刷新缓存**
   - 在文件名中加入版本号或hash值
   - 使用 query string
   - 使用 Service Worker

2. **避免缓存**
   - 开发环境设置 `Cache-Control: no-store`
   - 使用随机参数
   - 设置 `Pragma: no-cache`（兼容HTTP/1.0）





## 浏览器渲染过程

浏览器获取到HTML页面后，会经历以下渲染步骤：

### 1. 构建DOM树（DOM Tree）

1. **解析过程**
   - HTML解析器解析HTML文档
   - 将标签转化为DOM节点
   - 构建节点之间的关系
   - 生成DOM树结构

2. **解析特点**
   - 解析过程是渐进的，可能会多次触发渲染
   - 遇到JavaScript会暂停解析（除非使用async/defer）
   - 错误容忍性强，会尝试修正HTML错误

### 2. 构建CSSOM树（CSS Object Model）

1. **样式来源**
   - 浏览器默认样式
   - 外部样式表
   - 内部样式表
   - 内联样式

2. **计算过程**
   - 格式化CSS代码
   - 标准化属性值
   - 计算每个节点的最终样式
   - 构建CSSOM树

### 3. 构建渲染树（Render Tree）

1. **合并过程**
   - 将DOM树与CSSOM树合并
   - 只包含可见节点
   - 计算样式继承关系
   - 确定每个节点的最终样式

2. **排除元素**
   - display: none 的元素
   - head 标签内容
   - script 标签
   - 不可见的伪元素

### 4. 布局（Layout）与回流（Reflow）

#### 回流触发条件

1. **首次渲染**
   - 页面初始化
   - JavaScript脚本执行完成

2. **元素位置变化**
   - 元素尺寸改变
   - 内容变化导致尺寸改变
   - 元素位置移动
   - 浮动、定位状态改变

3. **视口变化**
   - 浏览器窗口大小改变
   - 字体大小改变
   - 滚动页面

4. **DOM操作**
   - 添加/删除可见DOM元素
   - 元素类名/样式改变
   - 激活CSS伪类（:hover等）

#### 优化回流

1. **批量修改DOM**

```typescript
// 不推荐
element.style.width = '100px';
element.style.height = '100px';
element.style.margin = '10px';
// 推荐
element.style.cssText = 'width: 100px; height: 100px; margin: 10px;';
// 或者
element.classList.add('newStyle');
```

2. **避免频繁读取会引发回流的属性**

```typescript
javascript
// 不推荐
const height = element.offsetHeight;
setInterval(() => {
element.style.top = height + 'px';
}, 100);
// 推荐
const height = element.offsetHeight;
const top = height;
setInterval(() => {
element.style.top = top + 'px';
}, 100);
```

3. **使用绝对定位使元素脱离文档流**
4. **使用document fragment进行批量操作**
5. **对于复杂动画，使用position: fixed/absolute**

### 5. 重绘（Repaint）

1. **触发条件**
   - 修改颜色、背景色
   - 修改边框样式
   - 修改透明度
   - 修改box-shadow
   - visibility改变

2. **优化策略**
   - 使用CSS3硬件加速（transform、opacity、filters）
   - 避免使用table布局
   - 使用visibility替代display: none
   - 降低重绘元素的层级

### 6. 渲染层合并（Composite）

1. **创建新渲染层的条件**
   - 3D转换（transform: 3d）
   - video、canvas元素
   - CSS filters
   - opacity小于1的元素
   - position: fixed

2. **优化建议**
   - 使用transform和opacity实现动画
   - 提升动画元素的层级（will-change）
   - 避免大量层叠元素
   - 合理管理渲染层数量



## V8引擎解析JavaScript流程

V8是Google开发的开源JavaScript引擎，主要用于Chrome浏览器和Node.js。以下是V8引擎执行JavaScript代码的主要流程：

![image-20250224171721777](D:\HeinrichHu\resource\computer network knowledge\03_浏览器输入url发生了什么.assets\image-20250224171721777.png)

### 1. 源码解析阶段

#### 1.1 词法分析（Scanner）
- 将源代码**分解成最小的词法单元（tokens）**
- 识别关键字、标识符、运算符等
- 去除空白符、注释等

#### 1.2 语法分析（Parser）
- **预解析（Pre-parsing）**
  - 不会立即执行的函数只做基础语法检查
  - 减少内存占用和解析时间
  
- **全量解析（Full-parsing）**
  - 生成抽象语法树（AST）
  - 检查语法错误
  - 构建作用域链

### 2. 编译阶段

#### 2.1 生成字节码（Bytecode）
- 通过Ignition解释器将AST转换为字节码
- **相比机器码占用更少内存**
- 可以快速启动执行

```typescript
// 示例：V8如何处理函数
function example() {
// 首次执行时解释执行
for(let i = 0; i < 1000; i++) {
// 多次执行后可能会被优化为机器码
console.log(i);
}
}
```

#### 2.2 生成机器码（Machine Code）
- 通过TurboFan优化编译器将热点代码（hot code）编译为机器码
- 针对性优化，提升执行效率
- 占用更多内存

### 3. 执行阶段

#### 3.1 解释执行
- Ignition解释器逐行执行字节码
- 收集函数执行频率等信息
- 标记潜在的优化机会

#### 3.2 优化执行
- 检测热点代码
- 进行类型推断
- 进行内联优化
- 消除冗余代码

### 4. 优化与去优化

#### 4.1 优化条件
- 函数被多次调用
- 循环被多次执行
- 类型信息稳定

#### 4.2 去优化情况
- 类型发生变化
- 对象结构改变
- 使用了eval或with
- 调试器介入

### 5. 内存管理

#### 5.1 垃圾回收
- 新生代（Minor GC）：Scavenge算法
- 老生代（Major GC）：Mark-Sweep & Mark-Compact

#### 5.2 内存分配
- 新对象优先分配在新生代
- 存活时间长的对象晋升到老生代
- 大对象直接分配在老生代

### 6. 性能优化建议

1. **减少编译时间**

```typescript
// 避免
eval('console.log("Hello")');
// 推荐
console.log("Hello");
```

2. **优化内存使用**

```typescript
// 避免
function createArray() {
return new Array(1000000);
}
// 推荐
const cachedArray = new Array(1000000);
function getArray() {
return cachedArray;
}
```

3. **类型稳定**

```typescript
// 避免
let value = 42;
value = "string"; // 类型改变会导致去优化
// 推荐
let number = 42;
let text = "string";
```

4. **避免复杂的动态特性**

```typescript
// 避免
const obj = {};
for(let i = 0; i < 100; i++) {
obj[prop${i}] = i; // 动态添加属性
}
// 推荐
const obj = Object.create(null);
const properties = Array.from({length: 100}, (, i) => [prop${i}, i]);
Object.assign(obj, Object.fromEntries(properties));

```

