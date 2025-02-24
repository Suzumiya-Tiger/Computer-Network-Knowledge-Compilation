![image-20250224172601034](D:\HeinrichHu\resource\computer network knowledge\04_CDN是什么.assets\image-20250224172601034.png)

## CDN（Content Delivery Network）内容分发网络

### 1. 基本概念

![image-20250224172954160](D:\HeinrichHu\resource\computer network knowledge\04_CDN是什么.assets\image-20250224172954160.png)

CDN是一种通过分布在各地的服务器，将内容缓存在用户近距离节点，从而提高访问速度的网络架构。

### 2. 工作原理

1. **用户请求资源**
   - 用户访问网站（如：www.example.com/image.jpg）
   - DNS解析时会将请求指向最近的CDN节点

2. **CDN节点处理**
   - 如果节点有缓存，直接返回
   - 如果没有，向上层节点请求
   - 最终可能请求到源站

### 3. 主要优势

1. **提升速度**
   
   - 就近访问
   - 减少网络跳转
   - 智能路由
   
2. **降低成本**
   - 减少源站带宽压力
   - 降低服务器负载
   - 防止DDOS攻击

3. **提高可用性**
   - 多节点备份
   - 故障迁移
   - 负载均衡
   
   ![image-20250224173109492](D:\HeinrichHu\resource\computer network knowledge\04_CDN是什么.assets\image-20250224173109492.png)

### 4. 应用场景

1. **静态资源加速**
   - 图片、视频
   - CSS、JavaScript文件
   - 下载文件

2. **动态内容加速**
   - 动态页面
   - API接口
   - 实时数据

### 5. 面试要点

1. **技术优势**
   - 提高访问速度
   - 减少服务器压力
   - 提升可用性
   - 节省带宽成本

2. **实现原理**
   - DNS解析
   - 缓存策略
   - 回源机制

3. **使用建议**
   - 适合大文件、静态资源
   - 考虑成本收益
   - 注意缓存更新