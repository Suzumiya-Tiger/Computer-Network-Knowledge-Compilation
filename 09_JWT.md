# jwt(json web token)

token主要是做鉴权用的登录之后存储用户信息

下面这段就是生成的token(令牌) ：

`eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MSwiaWF0IjoxNjg3Njc0NDkyLCJleHAiOjE2ODc3NjA4OTJ9.Y6eFGv4KXqUhlRHglGCESvcJEnyMkMwM1WfICt8xYC4`

## JWT是三部分组成的

### 头部（Header）

头部通常由两部分组成：令牌的类型（即 "JWT"）和所使用的签名算法。头部通常采用 JSON 对象表示，并进行 Base64 URL 编码。

```json
{
  "alg": "HS256",
  "typ": "JWT"
}

```

alg代表所使用的签名算法，例如 HMAC SHA256（HS256）或 RSA 等。 typ：代表令牌的类型，一般为 "JWT"。

### 负载（Payload）

负载包含所要传输的信息，例如用户的身份、权限等。负载也是一个 JSON 对象，同样进行 Base64 URL 编码。

```typescript
{
  "iss": "example.com",
  "exp": 1624645200,
  "sub": "1234567890",
  "username": "johndoe"
}

```

- iss：令牌颁发者（Issuer），代表该 JWT 的签发者。 
- exp：过期时间（Expiration Time），代表该 JWT 的过期时间，以 Unix 时间戳表示。
-  sub：主题（Subject），代表该 JWT 所面向的用户（一般是用户的唯一标识）。 
- 自定义声明：可以添加除了预定义声明之外的任意其他声明。



### 签名（Signature）

签名是使用私钥对头部和负载进行加密的结果。它用于验证令牌的完整性和真实性。

```json
    HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secretKey
    )

```



## express中使用jwt

安装我们需要依赖的插件:

```shell
pnpm i express
pnpm i jsonwebtoken
pnpm i cors
```

cors主要是为了解决跨域使用的。

在对应的代码中：

```typescript
import express from 'express';
import jwt from 'jsonwebtoken';
import cors from 'cors';
const app = express();
const secretKey = 'xmzs' //加盐

app.use(express.urlencoded({ extended: false }));
app.use(express.json());
app.use(cors())

let user = { name: 'admin', password: '123456', id: 1 } //模拟用户信息

app.post('/api/login', (req, res) => {
    console.log(req.body)
    if (req.body.name == user.name && req.body.password == user.password) {
        res.json({
            message: '登录成功',
            code: 200,
            token: jwt.sign({ id: user.id }, secretKey, { expiresIn: 60 * 60 * 24 }) //生成token
        })
    } else {
        res.json({
            message: '登录失败',
            code: 400
        })
    }
})


app.get('/api/list', (req, res) => {
    console.log(req.headers.authorization)
    jwt.verify(req.headers.authorization as string, secretKey, (err, data) => { //验证token
        if (err) {
            res.json({
                message: 'token失效',
                code: 403
            })
        } else {
            res.json({
                message: '获取列表成功',
                code: 200,
                data: [
                    { name: '张三', age: 18 },
                    { name: '李四', age: 20 },
                ]
            })
        }
    })
})


app.listen(3000, () => {
    console.log('server is running 3000');
})

```

在这段代码中，设置了应用使用的中间件，包括处理 URL 编码和 JSON 格式数据的中间件以及跨域资源共享（CORS）中间件。

然后，定义了一个模拟的用户信息对象 user，包含了用户名、密码和用户 ID。

接下来，设置了一个 /api/login 的 POST 路由，用于用户登录验证。当接收到请求时，比较请求体中的用户名和密码与预设的 user 对象中存储的用户名和密码是否匹配。如果匹配，则返回登录成功的 JSON 响应，并使用 jsonwebtoken 的 sign 方法生成一个 JWT，其中包含用户的 ID 信息，并设置了过期时间为 24 小时。

如果用户名和密码不匹配，则返回登录失败的 JSON 响应。

接着，设置了一个 /api/list 的 GET 路由，用于获取列表数据。当接收到请求时，从请求头中获取存储的 JWT（通过 req.headers.authorization），并使用 jsonwebtoken 的 verify 方法验证 JWT 的有效性。如果验证失败，则返回 token 失效的 JSON 响应；如果验证通过，则返回成功获取列表数据的 JSON 响应，并返回一些模拟的数据。

最后，通过调用 app.listen 方法启动应用，监听在 3000 端口上，并输出服务器运行的提示信息。

这段代码实现了基本的用户登录验证和通过 JWT 鉴权的接口，在登录成功后生成的 JWT 中包含了用户的 ID 信息，从而在后续请求中进行验证和授权



## 前端代码

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>

<body>

    <div>
        <div>
            <span>账号</span> <input id="name" type="text">
        </div>
        <div>
            <span>密码</span> <input id="password" type="password">
        </div>
        <button id="btn">登录</button>
    </div>

    <script>
        const btn = document.querySelector('#btn')
        const name = document.querySelector('#name')
        const password = document.querySelector('#password')

        btn.onclick = () => {
            fetch('http://localhost:3000/api/login', {
                body: JSON.stringify({
                    name: name.value,
                    password: password.value
                }),
                headers: {
                    'Content-Type': 'application/json'
                },
                method: 'POST',
            }).then(res => res.json()).then(res => {
                localStorage.setItem('token', res.token)
                location.href = './list.html'
            })
        }
    </script>
</body>

</html>

```

list.html 如果没有token就访问不了

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>List</title>
</head>

<body>
    <script>
        console.log(localStorage.getItem('token'))
        fetch('http://localhost:3000/api/list', {
            headers: {
                'Authorization':`Bearer ${localStorage.getItem('token')}`
            }
        }).then(res => res.json()).then(res => {
            console.log(res)
        })
    </script>
</body>

</html>

```



## JWT的优缺点

### 优点
- 无状态和可扩展性：服务器不需要保存会话信息，便于负载均衡
- 跨域：可以在不同域名的系统中传递
- 自包含：包含了所有用户所需要的信息，避免了多次查询数据库

### 缺点
- 无法撤销：一旦签发，在到期之前无法撤销（除非使用黑名单）
- 数据膨胀：JWT包含了认证信息，如果存储太多信息会导致令牌变大
- 安全性考虑：敏感信息不应存储在JWT的payload中，因为它只是Base64编码，不是加密