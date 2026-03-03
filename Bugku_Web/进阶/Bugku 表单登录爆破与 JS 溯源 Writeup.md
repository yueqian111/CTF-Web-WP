# Bugku 表单登录爆破与 JS 溯源 Writeup

# Bugku 表单登录爆破&JS跳转溯源 Writeup

## 一、题目核心分析

本题为**表单登录爆破类题目**，核心逻辑是：登录请求提交至`check.php`后，后端会生成`code`参数并嵌入前端JS；仅当账号密码正确时，生成的`code`不为默认值`bugku10000`，前端JS会携带该`code`跳转至`success.php`，最终在该页面获取Flag。

## 二、工具准备

- 浏览器（Chrome/Firefox）+ 开发者工具

- Burp Suite Community Edition（抓包、爆破）

- 弱口令字典：`中国网民-弱密码字典 T1000.txt`

- Python3（字典预处理）

## 三、详细解题步骤

### 步骤1：抓包分析业务流程

1. 打开题目页面，随意输入账号密码（如`admin/admin123`）提交，用Burp拦截登录请求。

2. 分析请求：登录请求为`POST`方法，提交至`check.php`，携带`username`和`password`参数。

3. 分析响应：`check.php`返回的HTML中嵌入核心JS代码：

    ```JavaScript
    
    var r = {code: 'bugku10000'};
    if(r.code == 'bugku10000'){
        document.getElementById('d').innerHTML = "wrong account or password!";
    }else{
        window.location.href = 'success.php?code='+r.code;
    }
    ```

4. 访问`success.php?code=bugku10000`，页面仅返回`no!`，验证**默认code无效**，需爆破出密码对应的正确`code`。

### 步骤2：字典预处理（筛选z开头密码）

题目提示密码为**z开头**，用Python脚本从弱口令字典中筛选符合条件的密码，生成专属字典：

```Python

# 字典预处理脚本
f = open("中国网民-弱密码字典 T1000.txt", "r", encoding="utf-8")
wf = open("z开头密码字典.txt", "w", encoding="utf-8")

for item in f:
    item = item.strip()
    if item.startswith("z"):
        print(item)
        wf.write(item + "\n")

f.close()
wf.close()
```

执行后得到仅包含`z`开头密码的字典（如`zxcvbnm`、`zombie`、`zachary`等）。

### 步骤3：Burp Suite 爆破配置

1. **发送请求至Intruder**：将拦截的`check.php`登录请求，右键发送至`Intruder`模块。

2. **清除默认标记，仅标记密码参数**：

    - 进入`Intruder` → `Positions`，点击`Clear §`清除所有默认标记。

    - 选中`password`参数的取值（如`admin123`），点击`Add §`，仅对密码进行爆破。

3. **加载自定义字典**：

    - 进入`Payloads` → `Payload Set 1`，选择`Payload type`为`Simple list`。

    - 点击`Load`，导入步骤2生成的`z开头密码字典.txt`。

4. **关闭多余爆破组**：将`Payload Sets`的数量改为`1`，避免无效爆破。

### 步骤4：结果筛选（核心难点突破）

爆破完成后，发现**所有响应包的长度一致**，无法通过长度直接筛选，需通过`Grep Extract`提取响应中的`code`值进行区分：

1. 进入`Intruder` → `Options` → `Grep - Extract`，点击`Add`。

2. 弹出`Define grep item`窗口，切换到`Response`标签，选中响应中`var r = {code: 'xxx'};`里的`code`值区域（如`bugku10000`），点击`OK`确认提取规则。

3. 重新执行爆破，爆破结果会新增一列`Grep - Extract`，显示每个密码对应的`code`值。

4. 筛选结果：找到`code`值**不为** **`bugku10000`** 的条目，对应的密码为正确密码，提取到的`code`为`hacker1000`。

### 步骤5：获取Flag

1. 方式1（浏览器直接访问）：在地址栏输入`success.php?code=hacker1000`，页面直接显示Flag：`flag{a8920ba08e424b2676547d5bbe681376}`。

2. 方式2（正常登录）：用爆破出的正确密码登录，前端JS会自动跳转至`success.php?code=hacker1000`，获取Flag。

## 四、扩展方法（Grep Match 筛选）

除了`Grep Extract`，也可通过`Grep Match`直接匹配非默认`code`：

1. 进入`Intruder` → `Options` → `Grep - Match`，点击`Add`。

2. 输入匹配规则：`hacker1000`（或排除`bugku10000`），爆破后会自动标记匹配到的结果，快速定位正确密码。

## 五、关键知识点总结

1. **业务流程溯源**：通过抓包分析JS逻辑，明确`code`参数的核心作用。

2. **字典精准化**：通过题目提示缩小爆破范围，提升效率。

3. **Burp 高级筛选**：响应长度相同时，利用`Grep Extract`/`Grep Match`提取/匹配关键内容筛选结果。

4. **JS执行特性**：Burp不会执行前端JS，需手动携带正确`code`访问目标页面。
> （注：文档部分内容可能由 AI 生成）