# fetch

fetch是当前流行的网络请求API，用于JS中实现HTTP请求。它是XMLHttpRequest的更好替代品，提供了简单、灵活的接口。

```typescript
fetch(url)
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('Error:', error));
```

## 主要特点

### Promise基础

fetch返回Promise对象

可以使用.then()和async/await语法

更好的错误处理机制

### 请求配置选项

```typescript
fetch(url, {
  method: 'POST', // GET, POST, PUT, DELETE等
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data) // 请求体
})
```

### 常用请求示例

POST请求：

```typescript
async function postData(url, data) {
  try {
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });
    return await response.json();
  } catch (error) {
    console.error('Error:', error);
  }
}
```

## 注意事项

1. fetch只会在网络错误时返回reject，HTTP错误（404、500等）需要在then中处理：

   ```typescript
   fetch(url)
     .then(response => {
       if (!response.ok) {
         throw new Error(`HTTP error! status: ${response.status}`);
       }
       return response.json();
     })
   ```

2. fetch默认不发送cookies，需要设置credentials：

   ```typescript
   fetch(url, {
     credentials: 'include' // 'same-origin' 或 'include'
   })
   ```

### Response对象的常用方法

使用fetch必须对返回方式进行类型制定，response在then中返回，我们需要指定具体的类型如下:

- response.json(): 解析JSON响应
- response.text(): 获取文本响应
- response.blob(): 获取二进制数据
- response.formData(): 获取FormData对象
- response.arrayBuffer(): 获取ArrayBuffer

## 实际应用示例

获取用户数据：

```typescript
async function getUser(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new Error('User not found');
    }
    const userData = await response.json();
    return userData;
  } catch (error) {
    console.error('Error fetching user:', error);
    throw error;
  }
}
```

上传文件：

```typescript
async function uploadFile(file) {
  const formData = new FormData();
  formData.append('file', file);
  
  try {
    const response = await fetch('/api/upload', {
      method: 'POST',
      body: formData
    });
    return await response.json();
  } catch (error) {
    console.error('Error uploading file:', error);
  }
}
```

### 与XMLHttpRequest比较

**fetch的优势：**

- 语法更简洁
- 基于Promise，更好的异步处理
- 更现代的API设计
- 支持请求和响应对象
- 更好的模块化

**fetch的不足：**

- 需要手动检查response.ok
- 不支持直接超时设置（需要配合Promise.race实现）
- 不能直接取消请求（需要使用AbortController）



## post请求

### 1. 基本POST请求

最简单的POST请求示例：

```typescript
fetch('https://api.example.com/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    name: 'John',
    age: 30
  })
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('Error:', error));
```

### 2. 使用 async/await

更现代的写法使用 async/await：

```typescript
async function postData(url, data) {
  try {
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const result = await response.json();
    return result;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// 使用示例
postData('https://api.example.com/data', { name: 'John', age: 30 })
  .then(data => console.log(data));
```

### 3. 不同类型的数据提交

1. 发送JSON数据

```typescript
const data = {
  name: 'John',
  age: 30
};

fetch(url, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify(data)
});
```

2. 发送表单数据

   ```typescript
   const formData = new FormData();
   formData.append('name', 'John');
   formData.append('age', '30');
   
   fetch(url, {
     method: 'POST',
     body: formData  // 不需要设置 Content-Type，浏览器会自动设置
   });
   ```

3. 发送文件

```typescript
const formData = new FormData();
formData.append('file', fileInput.files[0]);

fetch(url, {
  method: 'POST',
  body: formData
});
```

### 常见的配置选项

```typescript
fetch(url, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer token',
    'Accept': 'application/json'
  },
  body: JSON.stringify(data),
  credentials: 'include', // 包含 cookies
  mode: 'cors',          // 跨域设置
  cache: 'no-cache',     // 缓存设置
  redirect: 'follow',    // 重定向设置
  referrerPolicy: 'no-referrer'
});
```



## 实现进度监听

**使用 fetch 配合 ReadableStream（更现代的方式）**

```typescript
async function uploadWithProgress(url, file, progressCallback) {
  // 创建可读流
  const stream = file.stream();
  const reader = stream.getReader();
  const fileSize = file.size;
  let loaded = 0;

  // 创建新的 ReadableStream
  const newStream = new ReadableStream({
    async start(controller) {
      while (true) {
        const {done, value} = await reader.read();
        
        if (done) {
          controller.close();
          break;
        }
        
        loaded += value.length;
        const progress = (loaded / fileSize) * 100;
        progressCallback(progress);
        
        controller.enqueue(value);
      }
    }
  });

  // 发送请求
  try {
    const response = await fetch(url, {
      method: 'POST',
      body: new Response(newStream),
      headers: {
        'Content-Type': file.type,
        'Content-Length': fileSize.toString()
      }
    });

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error('Upload failed:', error);
    throw error;
  }
}

// 使用示例
const fileInput = document.querySelector('input[type="file"]');
const progressBar = document.querySelector('.progress-bar');

fileInput.addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (!file) return;

  try {
    await uploadWithProgress(
      'https://api.example.com/upload',
      file,
      (progress) => {
        console.log(`Upload progress: ${progress.toFixed(2)}%`);
        progressBar.style.width = `${progress}%`;
      }
    );
    console.log('Upload complete!');
  } catch (error) {
    console.error('Upload failed:', error);
  }
});
```

