# Bugku CTF Web —— no select WriteUp

## 一、题目信息

- 题目名称：no select

- 题目类型：Web（SQL注入）

- 题目描述：flag{}

- 核心考点：SQL注入万能密码、堆叠注入、handler绕过select过滤

---

## 二、解题思路

题目提示“no select”，说明后端对`select`关键字进行了过滤，无法直接使用联合查询等常规注入手段。我们可以通过**万能密码**绕过登录验证，或利用**堆叠注入+handler语句**读取数据，最终获取flag。

---

## 三、解题步骤

### 步骤1：启动场景，测试注入点

点击「启动场景」进入靶场，页面为一个简单的查询输入框。

- 输入`1`并提交，观察返回结果，确认存在SQL查询逻辑。

- 构造测试payload：`1' and 1=1#`，提交后返回数据；输入`1' and 1=2#`，提交后无返回，证明存在SQL注入漏洞。

### 步骤2：万能密码直接获取flag

构造万能密码payload，使SQL查询恒成立：

```SQL

1' or 1=1#
```

- 原理：闭合前面的单引号，构造`or 1=1`恒真条件，并用`#`注释后续语句，使查询返回所有数据，直接暴露flag。

- 提交后页面直接返回flag内容。

### 步骤3（进阶）：堆叠注入+handler绕过select过滤

若万能密码未直接出flag，可通过堆叠注入进一步获取：

1. **查看数据库**

    ```SQL
    
    1'; show databases;#
    ```

    提交后可看到所有数据库名称，找到目标库。

2. **查看表名**

    ```SQL
    
    1'; show tables;#
    ```

    提交后发现存在`flag`表。

3. **handler读取表数据（绕过select过滤）**

因`select`被禁，使用MySQL的`handler`语句读取表数据：

```SQL

1'; handler `flag` open as `a`; handler `a` read next;#
```

- `handler ... open`：打开表并命名为别名`a`

- `handler ... read next`：读取下一行数据

执行后即可获取`flag`表中的数据。

---

## 四、最终flag

通过上述任意方法均可得到flag，格式为`flag{xxx}`（具体内容以实际环境为准）。

---

## 五、知识点总结

1. **万能密码原理**：构造`' or 1=1#`类payload，使SQL查询WHERE条件恒为真，绕过验证或获取所有数据。

2. **堆叠注入**：使用`;`分隔多条SQL语句，执行额外查询（如`show databases`、`show tables`）。

3. **handler绕过select**：MySQL中`handler`语句可在不使用`select`的情况下读取表数据，适用于`select`被过滤的场景。

---

## 六、防御建议

- 对用户输入进行严格过滤，禁止`'`、`#`、`;`等特殊字符，或使用预编译语句（PreparedStatement）避免SQL注入。

- 限制数据库用户权限，避免堆叠注入等危险操作。

- 对敏感关键字（如`select`、`union`）进行WAF拦截或代码层面过滤。
> （注：文档部分内容可能由 AI 生成）