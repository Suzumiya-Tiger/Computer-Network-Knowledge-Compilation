# HTTPS

![image-20250226153916353](D:\HeinrichHu\resource\computer network knowledge\08_SSL TLS HTTPS.assets\image-20250226153916353.png)

HTTPS是通过http+TLS/SSL证书实现的，TLS是SSL的升级版，广泛使用的是TLS。

TLS/SSL是在应用层和传输层之间进行了一个加密工作。

> **补充说明**: HTTPS (HTTP Secure) 本质上是HTTP协议的安全版本，它通过TLS/SSL协议为数据传输提供了三重保护：加密（防窃听）、数据完整性（防篡改）和身份认证（防冒充）。

## TLS的加密

TLS协议使用了多种加密技术的组合，主要包括对称加密、非对称加密和散列函数。

### 对称加密

**常见的算法有AES DES 加密。**

举例  `麒麟`->`星月`发消息 但是他们的消息不想被别人知道，采用了对称加密，于是他们两个协商了一段密钥，`今生永相随`

```tcl
麒麟：`AES算法 + 密钥（今生永相随）+明文（吃面） = XMZSXMZS==`

星月：`使用AES + 密钥（今生永相随）+密文（ XMZSXMZS==） = 吃面`

```

### 解释

XMZS== 是加密后的密文，只是一个示例表示，实际上是一串看起来随机的字符

在RSA加密中：

- 使用公钥加密的内容只能用私钥解密
- 使用私钥签名的内容可以用公钥验证

### 在HTTPS中的应用

- 客户端使用服务器的公钥加密预主密钥(类似于"吃面"这个明文)
- 加密后变成密文(类似于"XMZS==")
- 服务器用私钥解密这个密文，获得预主密钥
- 这确保了只有持有私钥的服务器才能获取预主密钥

这种方式解决了关键问题：如何在不安全的网络上安全地共享一个秘密(预主密钥)，这个秘密随后被用来生成用于对称加密的会话密钥。

> **补充说明**: 对称加密的优点是速度快、效率高，适合加密大量数据；缺点是密钥分发问题 - 通信双方需要安全地交换密钥，这在不安全的网络中是个挑战。常用的对称加密算法还包括3DES、RC4、ChaCha20等。

### 非对称加密

**常见的算有RSA DSA 加密。**

这里可以把管理员理解为麒麟，私钥永远掌握在管理员手中。

举例 `麒麟`->`星月`发消息，这次使用的是非对称加密，生成了公钥和私钥，公钥可以对外公开，私钥必须只能`麒麟`知道不能泄露。

星月：RSA + 公钥 + 明文（吃面） = XMZS==

麒麟：RSA + 私钥 + 密文（XMZS==） = 吃面

> **补充说明**: 非对称加密解决了密钥分发问题，但计算开销大，不适合加密大量数据。在HTTPS中，非对称加密主要用于安全地交换对称加密的会话密钥。除了RSA和DSA外，现代HTTPS还广泛使用ECC（椭圆曲线加密）算法，如ECDSA和ECDH，它们提供同等安全性但需要更短的密钥长度。

### TLS加密实现原理

TLS融合了对称和非对称加密，还有散列函数，使得其安全性极高。

> **补充说明**: TLS握手过程简述：
> 
> 1. **客户端Hello**: 客户端发送支持的TLS版本、加密算法列表和随机数
> 2. **服务器Hello**: 服务器选择TLS版本和加密算法，发送自己的随机数和数字证书
> 3. **证书验证**: 客户端验证服务器证书的有效性
> 4. **密钥交换**: 客户端生成预主密钥(Pre-Master Secret)，用服务器公钥加密后发送
> 5. **会话密钥生成**: 双方使用随机数和预主密钥生成相同的会话密钥
> 6. **加密通信**: 后续通信使用会话密钥进行对称加密
>
> 这种混合加密系统结合了非对称加密的安全密钥交换和对称加密的高效数据传输。

# 生成HTTPS服务

openSSL 安装

Mac电脑自带了

windows [www.openssl.org/source/](https://link.juejin.cn?target=https%3A%2F%2Fwww.openssl.org%2Fsource%2F)

在 SSL/TLS 加密通信中，一般需要使用三个文件来完成证书相关操作，即：

1. 私钥文件（例如 "private-key.pem"），用于对加密数据进行解密操作。
2. 证书签名请求文件（例如 "certificate.csr"），用于向 CA 申请 SSL/TLS 证书签名。
3. SSL/TLS 证书文件（例如 "certificate.pem"），用于对客户端发送的请求进行验证，以确保通信安全可靠。

私钥文件用于对数据进行解密操作，保证了通信的机密性；证书签名请求文件包含了请求者的身份信息和公钥等信息，需要被发送给 CA 进行签名，从而获取有效的 SSL/TLS 证书；SSL/TLS 证书文件则包含了签名后的证书信息，被用于客户端和服务器之间的身份验证，以确保通信的安全性和可靠性。

通过使用这三个文件进行密钥交换和身份验证，SSL/TLS 可以实现加密通信以及抵御可能的中间人攻击，提高了通信的安全性和保密性。

> **补充说明**: 证书链和信任模型：
>
> 在实际部署中，证书通常形成一个信任链：
> - **根证书(Root CA)**: 由受信任的证书颁发机构创建，预装在操作系统和浏览器中
> - **中间证书(Intermediate CA)**: 由根CA签发，用于签发终端实体证书
> - **终端实体证书(End-entity Certificate)**: 颁发给特定网站或服务
>
> 这种层级结构增强了安全性，因为根CA的私钥可以离线保存，减少被攻击的风险。

## TLS加密流程

### 生成key

执行命令:

```shell
openssl genpkey -algorithm RSA -out private-key.pem -aes256
```

命令解析:

- openssl: OpenSSL 命令行工具的名称。
- genpkey: 生成私钥的命令。
- -algorithm RSA: 指定生成 RSA 私钥。
- -out private-key.pem: 将生成的私钥保存为 private-key.pem 文件。
- -aes256: 为私钥添加 AES 256 位加密，以保护私钥文件不被未经授权的人访问。
- Enter PEM pass phrase qwe123 密码短语生成pem文件的时候需要
- 生成pem 证书文件

回车该命令后，需要输入密码短语，并且需要重复两次，需要自己手动保存。

> **补充说明**: 私钥的安全性至关重要，因为它是整个TLS安全体系的基础。使用`-aes256`参数对私钥进行加密是一种最佳实践，可以防止私钥文件被盗后直接使用。在生产环境中，应当:
> - 限制私钥文件的访问权限
> - 定期轮换密钥
> - 考虑使用硬件安全模块(HSM)存储私钥

### 生成证书文件

```shell
openssl req -new -key private-key.pem -out certificate.csr
```

命令解析:

- "req": 表示使用 X.509 证书请求管理器 (Certificate Request Management) 功能模块。
- "-new": 表示生成新的证书签名请求。
- "-key private-key.pem": 表示使用指定的私钥文件 "private-key.pem" 来加密证书签名请求中的密钥对。
- "-out certificate.csr": 表示输出生成的证书签名请求到文件 "certificate.csr" 中。该文件中包含了申请者提供的一些证书请求信息，例如公钥、授权主体的身份信息等。

回车该命令后，也需要输入密码短语。


完成上述操作后，我们需要按步骤输入一大堆信息:

1. Country Name (2 letter code) []:CN  国家
2. State or Province Name (full name) []:BJ 省份
3. Locality Name (eg, city) []:BJ 城市
4. Organization Name (eg, company)ZMY 组织或者是个人
5. Organizational Unit Name (eg, section) []:XMKJ 机构名称
6. **Common Name (eg, fully qualified host name) []:localhost 域名**
7. Email Address []: 邮箱地址
8. Please enter the following 'extra' attributes
9. to be sent with your certificate request
10. **A challenge password []:  密码加盐 XMZSXMZSXMZS**

> **补充说明**: 证书信息中最关键的是Common Name(CN)，它必须与您网站的域名完全匹配。对于需要保护多个子域名的情况，现代证书支持:
> - 通配符证书: 如`*.example.com`可保护所有一级子域名
> - 主题备用名称(SAN): 在一个证书中包含多个域名
>
> 在生产环境中，CSR通常会发送给受信任的证书颁发机构(CA)进行签名，而不是自签名。

### 生成数字证书

```shell
openssl x509 -req -in certificate.csr -signkey private-key.pem -out certificate.pem
```

命令解析:

- "x509": 表示使用 X.509 证书管理器功能模块。
- "-req": 表示从输入文件（这里为 "certificate.csr"）中读取证书签名请求数据。
- "-in certificate.csr": 指定要读取的证书签名请求文件名。
- "-signkey private-key.pem": 指定使用指定的私钥文件 "private-key.pem" 来进行签名操作。一般情况下，签名证书的私钥应该是和之前生成 CSR 的私钥对应的。
- "-out certificate.pem": 表示将签名后的证书输出到文件 "certificate.pem" 中。该文件中包含了签名后的证书信息，包括签名算法、有效期、公钥、授权主体的身份信息等。
- Enter pass phrase for private-key.pem: 密码短语

回车该命令后，也需要输入一个密码短语进行加密。

> **补充说明**: 上述命令创建的是自签名证书，适用于开发和测试环境，但不适合生产环境，因为:
>
> - 浏览器会显示警告，提示证书不受信任
> - 用户需要手动添加例外才能访问网站
> - 不提供真正的身份验证
>
> 默认情况下，自签名证书的有效期为30天。可以通过添加`-days 365`参数来设置更长的有效期。
>
> 可以使用以下命令查看证书内容:
> ```shell
> openssl x509 -in certificate.pem -text -noout
> ```

到这一步后，https服务就已经创建完成了。

### nodejs接口测试https

引入生成好的两个文件: certificate.pem private-key.pem，开始写代码:

```ts
import https from 'node:https'
import fs from 'node:fs'
https.createServer({
    key: fs.readFileSync('private-key.pem'),
    cert: fs.readFileSync('certificate.pem'),
    //密码短语
    passphrase: 'qwe123'
}, (req, res) => {
    res.writeHead(200)
    res.end('success')
}).listen(443,()=>{
    console.log('server is running')
})

```

1.http默认端口号是80，而https默认端口号是443

解决完之后用`ts-node-esm`启动这个文件，注意在浏览器测试的时候，自己的证书google是不信任的，真正的https证书要从第三方授信机构购买。

> **补充说明**: 在生产环境中，您可以从以下几个来源获取受信任的SSL证书:
>
> 1. **商业CA**: 如DigiCert, Comodo, GlobalSign等，提供不同级别的验证和功能
> 2. **免费证书**: Let's Encrypt提供免费的DV(域名验证)证书，有效期90天，可自动续期
> 3. **云服务提供商**: AWS Certificate Manager, Google Cloud SSL等提供与其云服务集成的证书
>
> 对于Node.js HTTPS服务器，还可以配置其他重要选项:
> ```ts
> https.createServer({
>     key: fs.readFileSync('private-key.pem'),
>     cert: fs.readFileSync('certificate.pem'),
>     passphrase: 'qwe123',
>     // 指定TLS版本，禁用不安全的版本
>     minVersion: 'TLSv1.2',
>     // 指定加密套件优先级
>     ciphers: 'ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384',
>     // 启用HTTP严格传输安全
>     // (需要在响应头中设置Strict-Transport-Security)
> }, (req, res) => {
>     // ...
> })
> ```

# 在nginx使用https

**终极大坑 如果在windows使用nginx配置https 私钥不能设置密码**

1. openssl genrsa -out nginx.key 2048 （生成私钥）

   ![image-20250226161547512](D:\HeinrichHu\resource\computer network knowledge\08_SSL TLS HTTPS.assets\image-20250226161547512.png)

2. openssl req -new -key nginx.key -out nginx.csr（生成签名文件）

   这里会生成前面类似的其他额外信息填写。

   ![image-20250226161556034](D:\HeinrichHu\resource\computer network knowledge\08_SSL TLS HTTPS.assets\image-20250226161556034.png)

3. openssl x509 -req -in nginx.csr -signkey nginx.key -out nginx.crt（生成证书）

![image-20250226161631964](D:\HeinrichHu\resource\computer network knowledge\08_SSL TLS HTTPS.assets\image-20250226161631964.png)

在nginx中配置对应的文件:

```nginx
    server {
       listen       443 ssl;
       server_name  localhost;

       ssl_certificate      nginx.crt;
       ssl_certificate_key  nginx.key;

       ssl_session_cache    shared:SSL:1m;
       ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

       location / {
           root   html;
           index  index.html index.htm;
       }
    }

```

在以下命令行输入对应的文件名，即可完成引入和使用:

```nginx
       ssl_certificate      nginx.crt;
       ssl_certificate_key  nginx.key;
```

重新启动nginx服务后，https证书就可以生效了。

> **补充说明**: 为了提高Nginx HTTPS配置的安全性，建议添加以下配置:
>
> ```nginx
> server {
>     listen 443 ssl http2;  # 启用HTTP/2支持
>     server_name example.com;
>
>     ssl_certificate      nginx.crt;
>     ssl_certificate_key  nginx.key;
>
>     # 优化SSL设置
>     ssl_protocols TLSv1.2 TLSv1.3;  # 只使用安全的TLS版本
>     ssl_prefer_server_ciphers on;   # 优先使用服务器的加密套件
>     ssl_ciphers 'ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384';  # 强加密套件
>     
>     # 启用OCSP Stapling (提高性能和隐私)
>     ssl_stapling on;
>     ssl_stapling_verify on;
>     
>     # 添加安全头
>     add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload";
>     add_header X-Content-Type-Options nosniff;
>     add_header X-Frame-Options DENY;
>     add_header X-XSS-Protection "1; mode=block";
>     
>     # 其他配置...
> }
>
> # 将HTTP请求重定向到HTTPS
> server {
>     listen 80;
>     server_name example.com;
>     return 301 https://$host$request_uri;
> }
> ```
>
> 这些设置可以显著提高您网站的安全评级，在SSL Labs测试中获得A+评分。



# 疑问解决

**HTTPS说到底是一个证书，那么我购买了证书该如何应用？这个证书有什么用？**

##  HTTPS证书的应用与作用

HTTPS不是证书本身，而是一个协议。证书是这个协议中的关键组件。

### 证书的作用

身份验证：证明网站的真实身份，防止中间人攻击

数据加密：提供公钥，用于建立加密通信

建立信任：由受信任的第三方(CA)签发，浏览器信任CA就会信任证书

### 如何应用购买的证书

1. 获取证书文件：购买后，CA会提供几个文件：

   证书文件(.crt/.pem)

   私钥文件(.key)

   可能有中间证书(.ca-bundle)

2. 部署到服务器：

   Web服务器：配置Nginx/Apache等，指向证书文件

   云服务：上传到AWS/Azure等云平台的证书管理器

   CDN：上传到Cloudflare等CDN服务

配置示例（Nginx）：

```nginx
   server {
       listen 443 ssl;
       server_name example.com;
       ssl_certificate /path/to/certificate.crt;
       ssl_certificate_key /path/to/private.key;
   }
```

##  HTTPS的交互流程详解

TTPS交互是一个精心设计的"握手"过程，目的是安全地建立加密通信：

### 详细交互流程

- 客户端Hello

1. 客户端(浏览器)发起请求
2. 发送：支持的TLS版本列表、加密算法列表、随机数A
3. 这是明文传输

- 服务器Hello

1. 服务器接收请求并回应
2. 发送：选择的TLS版本、选择的加密算法、随机数B、服务器证书
3. **证书包含服务器公钥和CA签名**

- 证书验证

1. 客户端验证证书：
2. 检查证书是否由受信任的CA签发
3. 验证证书有效期
4. 确认证书域名与访问的网站匹配
5. 验证证书签名

- 密钥交换

1. 客户端生成一个随机的"预主密钥"(Pre-Master Secret)
2. **使用服务器公钥(从证书获取)加密这个预主密钥**
3. **发送加密后的预主密钥给服务器**
4. **只有持有私钥的服务器才能解密**

- 会话密钥生成

1. **双方各自使用相同的材料生成会话密钥：**
2. 随机数A + 随机数B + 预主密钥
3. **通过特定算法，双方得到相同的会话密钥**
4. 这个会话密钥将用于对称加密

- 加密通信开始

1. 客户端发送"Finished"消息(用会话密钥加密)

2. 服务器回复"Finished"消息(用会话密钥加密)

3. 之后所有通信都使用会话密钥加密

   

