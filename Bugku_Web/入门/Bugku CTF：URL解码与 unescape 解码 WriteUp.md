# Bugku CTF：Web 源代码与 unescape 解码 WriteUp

# Bugku CTF - 源代码（WEB）WriteUp

## 一、题目信息

- **平台**：Bugku CTF

- **题目名称**：源代码

- **分值**：20

- **题目类型**：WEB（前端代码分析）

- **解题核心**：网页源代码分析 + JS `unescape` 解码

## 二、解题思路

题目提示“看看源代码？”，核心突破口为**查看网页源代码**，分析其中隐藏的 JavaScript 代码逻辑，通过执行 `unescape` 解码操作，还原出题目所需的 Flag。

## 三、详细解题步骤

### 步骤1：访问题目场景并查看源代码

1. 点击题目页面的「启动场景」，进入靶场页面（`171.80.2.169:10063`），页面仅显示“看看源代码？”提示语和 Flag 输入框。

2. **查看网页源代码**：

    - 方式1：在页面空白处右键，选择「查看页面源代码」；

    - 方式2：按下浏览器快捷键 `F12`，切换到「源代码/Source」标签页，直接查看当前页面 HTML 源码。

### 步骤2：分析源代码中的关键逻辑

在源代码的 `<script>` 标签中，找到核心 JS 代码：

```JavaScript
截图中的核心JS代码
已准备好填充，替换为你截图中的真实JS代码
示例格式（请替换为截图内容）：
var p0 = "截图中的p0字符串";
var p1 = unescape("截图中的unescape参数") + p0;
var p = unescape(p1);
              
```

核心逻辑：对 `p1` 执行 `unescape` 解码，`p1` 由**解码后的** **截图中unescape内的参数** + 字符串 `p0` 拼接而成，最终 `p` 即为解码后的结果。

### 步骤3：执行 `unescape` 解码获取 Flag

我们分步对代码进行解码（可通过浏览器控制台、Python 脚本或在线解码工具执行）：

#### 方法1：浏览器控制台直接执行（推荐）

1. 按下 `F12` 打开浏览器开发者工具，切换到「控制台/Console」标签页；

2. 复制源代码中的 JS 变量定义与解码代码，粘贴到控制台并回车执行；

3. 执行 `console.log(p)`，控制台会输出解码后的结果，即为 Flag。

#### 方法2：Python 脚本解码（适配本地验证）

利用 Python 模拟 `unescape` 逻辑（`unescape` 对应 Python 的 `urllib.parse.unquote`）：

```Python
对应截图代码的Python解码脚本from urllib.parse import unquote

# 请复制截图中的对应内容替换下方，无需修改其他代码
part1 = unquote("截图中unescape内的参数")  # 直接粘贴截图中unescape括号内的内容
p0 = "截图中的p0完整字符串"  # 直接粘贴截图中p0赋值的字符串
p1 = part1 + p0
flag = unquote(p1)

print("Flag:", flag)
              
```

运行脚本，输出的结果即为最终 Flag。

### 步骤4：提交 Flag 完成题目

将解码得到的 Flag 输入到页面的「请输入flag」输入框，点击「提交」，提示“已解决”即完成解题。
> （注：文档部分内容可能由 AI 生成）