拿下 DVWA (Damn Vulnerable Web Application) 的 **Command Injection（命令注入）** Low 等级其实非常简单，这主要是因为它完全没有对用户输入进行任何过滤。
<img width="1357" height="558" alt="image" src="https://github.com/user-attachments/assets/316b9c4f-49eb-482b-8e4b-2a62d1be81c5" />


这一关的设计初衷是让你理解：**当程序直接将用户输入拼接进系统底层命令时，会发生什么。**

---

### 1. 核心原理

在 Low 等级下，后端代码长这样：

```php
<?php

if( isset( $_POST[ 'Submit' ]  ) ) {
    // Get input
    $target = $_REQUEST[ 'ip' ];

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}

?>

```

由于程序只是简单地把你的输入拼接到 `ping` 命令后面，你可以利用操作系统的**命令连接符**来执行你自己的命令。

### 2. 常用命令连接符

根据你服务器的操作系统（通常是 Linux 或 Windows），你可以使用以下符号：

| 符号 | 作用 | 示例 |
| --- | --- | --- |
| **`;`** | **执行完第一个再执行第二个** (Linux) | `127.0.0.1; ls` |
| **`&`** | **后台运行/并行执行** | `127.0.0.1 & whoami` |
| **`&&`** | **逻辑与**（前一个执行成功才执行下一个） | `127.0.0.1 && cat /etc/passwd` |
| **`\|`** | **管道符**（前一个的输出作为后一个的输入） | `127.0.0.1` |
| **`\|\|`** | **逻辑或**（前一个执行失败才执行下一个） | `invalid_ip` |

---

### 3. 通关步骤

1. **输入正常 IP**：先输入 `127.0.0.1` 确认功能正常，你会看到正常的 Ping 结果。
2. **注入命令**：在 IP 后面加上连接符和你想要执行的命令。
* **查看当前用户**：
输入 `127.0.0.1 && whoami`
<img width="1358" height="456" alt="image" src="https://github.com/user-attachments/assets/f4092521-8068-4fcd-a56d-7501a97b5f37" />

* **列出目录文件**：
输入 `127.0.0.1; ls` (Linux) 或 `127.0.0.1 & dir` (Windows)
<img width="1352" height="495" alt="image" src="https://github.com/user-attachments/assets/1c344fd0-9399-4b99-85b2-9a45299e755e" />

* **读取敏感文件 (Linux)**：
输入 `127.0.0.1 && cat /etc/passwd`
<img width="1358" height="939" alt="image" src="https://github.com/user-attachments/assets/27900c0b-ecd9-4531-9273-6db29b6e73c5" />



---

### 4. 为什么这很危险？

在这个等级下，攻击者拥有了服务器的 **Shell 权限**（权限等级取决于运行 Web 服务的用户，通常是 `www-data` 或 `apache`）。

**攻击者可以利用这一点：**

* 下载并安装后门（木马）。
* 读取数据库配置文件，获取数据库密码。
* 以内网为跳板，攻击其他内网机器。

---

### 源码分析 : php代码如何判断在什么系统上运行的DVWA
```php
<?php

    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

?>

```

这段代码判断系统的秘密就在这一行：

```php
if( stristr( php_uname( 's' ), 'Windows NT' ) )

```

我们可以把它拆解成三个部分来理解，就像剥洋葱一样：

### 1. `php_uname( 's' )` —— 获取系统“真名”

这个函数是 PHP 内置的，专门用来获取运行 PHP 的服务器操作系统信息。参数 `'s'` 代表 **Operating System Name**。

* 如果在 **Windows** 上运行，它通常会返回类似 `"Windows NT"` 的字符串。
* 如果在 **Linux** 上运行，它通常返回 `"Linux"`。
* 如果在 **macOS** 上运行，它会返回 `"Darwin"`。

### 2. `stristr( ... , 'Windows NT' )` —— 关键词搜索

`stristr` 是一个字符串处理函数，它的作用是在一段文字里**找关键词**。

* 它的特点是 **不区分大小写**（这就是那个 "i" 的含义，Case-**i**nsensitive）。
* 它会检查 `php_uname` 返回的结果里是否包含 `"Windows NT"` 这个词组。

### 3. `if` 逻辑判断

* **命中**：如果找到了 "Windows NT"，函数会返回找到的这部分字符串，在 PHP 的 `if` 判断中这被视为 `true`（真），于是执行 Windows 版的 ping 命令。
* **未命中**：如果没找到（比如返回的是 "Linux"），函数会返回 `false`（假），程序就会跳到 `else` 部分，执行 Linux 版的 ping 命令（带 `-c 4` 参数，表示只 ping 4 次，否则 Linux 会一直 ping 下去）。

---

###  为什么代码里要写 `Windows NT` 而不直接写 `Windows`？

这是一个很有年代感的写法。
`Windows NT`（New Technology）是 Windows 内核的核心族群。从早期的 Windows XP、7、10 到现在的 Windows 11，甚至 Windows Server，它们的系统名称中都包含这个标识。这样写可以确保涵盖几乎所有现代的 Windows 服务器环境。

---

如果你想知道你使用的是什么系统，可以使用以下php代码：
```php
<?php
$one = php_uname('s');
echo "<h1>你的系统是： " . $one . "</h1>";

?>
```
<img width="1779" height="335" alt="image" src="https://github.com/user-attachments/assets/e6c82310-bdaf-46b7-9aed-fb7e960bf536" />

