# my-first-sqli WriteUp

## 题目信息

- **题目名称**：my-first-sqli

- **题目类型**：Web - SQL注入

- **所属赛事**：HackINI 2021

- **题目描述**：My First SQL injection!!

- **目标**：获取flag

---

## 解题思路

这是一道非常经典的SQL注入入门题，目标是绕过登录验证。从页面可以看到，后台执行的SQL语句为：

```SQL

SELECT * FROM USERS WHERE username = [输入] AND password = [输入]
```

我们可以通过构造特殊的用户名，让SQL语句的逻辑恒成立，从而绕过密码验证。

---

## 详细步骤

### 1. 万能密码绕过法（推荐）

这是最直接的方法，利用SQL的注释和逻辑运算来绕过验证。

1. 在 **username** 输入框中输入：

    ```Plain Text
    
    ' OR 1=1#
    ```

2. 在 **password** 输入框中输入任意内容，例如：

    ```Plain Text
    
    123
    ```

3. 点击 `Login` 按钮提交。

**原理分析**：

构造后的SQL语句变为：

```SQL

SELECT * FROM USERS WHERE username = '' OR 1=1#' AND password = '123'
```

- `'` 闭合了前面的单引号，使我们的payload脱离字符串限制。

- `OR 1=1` 是一个恒成立的条件，让整个WHERE子句变为真。

- `#` 是SQL的注释符，将后面的 `AND password = ...` 全部注释掉，使其失效。

成功登录后，页面会显示出flag。

---

### 2. 联合查询注入法（进阶）

如果想更深入地了解数据库结构，可以使用联合查询。

1. **判断字段数**：

在username输入框中输入 `1' order by 2 --+`，页面正常显示；输入 `1' order by 3 --+`，页面报错。这说明查询结果有2个字段。

1. **确认注入点**：

输入 `1' union select 1,2 --+`，页面会显示 `1` 和 `2`，证明可以进行联合查询。

1. **获取flag**：

输入 `1' union select 1, 'shellmates{SQLi_goeS_BrrRrRR}' --+`，即可直接获取到flag。

---

## 最终答案

```Plain Text

shellmates{SQLi_goeS_BrrRrRR}
```

---

## 总结

这道题是SQL注入的“Hello World”，核心知识点是：

- 利用单引号 `'` 闭合SQL语句。

- 使用 `OR 1=1` 构造恒真条件。

- 使用 `#` 或 `--+` 注释掉后续的SQL代码。

掌握这些基础技巧后，就可以应对大部分初级的SQL注入题目了。
> （注：文档部分内容可能由 AI 生成）