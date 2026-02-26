# Bugku CTF - POST 题目 WriteUp

## 一、题目信息

|项目|内容|
|---|---|
|题目名称|POST|
|题目分类|Web|
|目标场景地址|[http://171.80.2.169:18380](http://171.80.2.169:18380)|
## 二、解题思路

题目核心是考察 **HTTP POST 请求传参** 的原理，通过构造正确的 POST 请求，向服务器传递参数 `what=flag`，即可触发服务器返回 flag。

## 三、详细解题步骤（Burp Repeater 版）

### 1. 启动靶场

点击题目页面的「启动场景」，获取目标地址 `http://171.80.2.169:18380`。

### 2. 打开 Burp Repeater

- 打开 Burp Suite，切换到 `Repeater` 标签页。

- 点击左上角的 `+` 号，新建一个请求。

### 3. 构造 POST 请求

在请求编辑区输入以下内容（可直接复制修改）：

```HTTP

POST / HTTP/1.1
Host: 171.80.2.169:18380
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:140.0) Gecko/20100101 Firefox/140.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: close
Priority: u=0, i
Content-Type: application/x-www-form-urlencoded
Content-Length: 9

what=flag
```

**关键修改点**：

- 请求方法从 `GET` 改为 `POST`。

- 添加了 `Content-Type: application/x-www-form-urlencoded` 请求头，表明这是一个表单数据请求。

- 在请求头后空一行，添加了 POST 数据 `what=flag`。

### 4. 发送请求并获取 Flag

- 点击 `Send` 按钮发送请求。

- 在右侧的 `Response` 区域查看服务器返回的内容，即可看到 flag。

## 四、最终 Flag

服务器返回的 flag 是：

`flag{5d312c227d67e82e94f4b4d01218a848}` ✅
> （注：文档部分内容可能由 AI 生成）