## DVWA 暴力破解
1.首先 用户就用`admin` 然后密码就输入随便输一个，使用Burpsuite 进行抓包
<img width="1353" height="1291" alt="image" src="https://github.com/user-attachments/assets/09a009e3-6bc7-42af-a8d1-34f56bfab31d" />
竟然是`get`请求
<img width="1862" height="837" alt="image" src="https://github.com/user-attachments/assets/71f181f7-b06a-4402-862f-843ca3e9c30a" />
2.Ctrl + i 放到 `Intruder` 模块，并将密码设为payload,并加载弱口令字典
<img width="2458" height="958" alt="image" src="https://github.com/user-attachments/assets/43e78a6d-68b2-46b8-b9a8-c7b9ba2bcc26" />
3.开始攻击
<img width="2508" height="1129" alt="image" src="https://github.com/user-attachments/assets/55d5a698-d12b-419a-9d2e-6575d448fd1f" />
4.使用过滤器找出成功登录的数据包，所以密码就是 : password
<img width="2494" height="1589" alt="image" src="https://github.com/user-attachments/assets/6114fea9-97e5-468d-9949-8e3c619157e2" />

### 源码分析
```php
<?php

if( isset( $_GET[ 'Login' ] ) ) {
    // Get username
    $user = $_GET[ 'username' ];

    // Get password
    $pass = $_GET[ 'password' ];
    $pass = md5( $pass );

    // Check the database
    $query  = "SELECT * FROM `users` WHERE user = '$user' AND password = '$pass';";
    $result = mysqli_query($GLOBALS["___mysqli_ston"],  $query ) or die( '<pre>' . ((is_object($GLOBALS["___mysqli_ston"])) ? mysqli_error($GLOBALS["___mysqli_ston"]) : (($___mysqli_res = mysqli_connect_error()) ? $___mysqli_res : false)) . '</pre>' );

    if( $result && mysqli_num_rows( $result ) == 1 ) {
        // Get users details
        $row    = mysqli_fetch_assoc( $result );
        $avatar = $row["avatar"];

        // Login successful
        echo "<p>Welcome to the password protected area {$user}</p>";
        echo "<img src=\"{$avatar}\" />";
    }
    else {
        // Login failed
        echo "<pre><br />Username and/or password incorrect.</pre>";
    }

    ((is_null($___mysqli_res = mysqli_close($GLOBALS["___mysqli_ston"]))) ? false : $___mysqli_res);
}

?>
```

该脚本接收通过 GET 方法传入的 username 和 password 参数：
1. 对密码进行 MD5 哈希；
2. 构造 SQL 查询语句，在 users 表中查找匹配的用户名和密码；
3. 如果找到唯一匹配记录，则显示欢迎信息和用户头像；
4. 否则提示“用户名或密码错误”。

```php
$result = mysqli_query($GLOBALS["___mysqli_ston"], $query ) 
    or die( 
        '<pre>' . 
        (
            (is_object($GLOBALS["___mysqli_ston"])) 
                ? mysqli_error($GLOBALS["___mysqli_ston"]) 
                : (
                    ($___mysqli_res = mysqli_connect_error()) 
                        ? $___mysqli_res 
                        : false
                )
        ) 
        . '</pre>'
    );
```
这段代码看起来有点复杂，主要是因为它试图在数据库查询失败时安全地输出错误信息，同时兼容连接失败和查询失败两种情况。

第一步：执行查询
```php
mysqli_query(GLOBALS["___mysqli_ston"], query)
```

- 使用全局变量 GLOBALS["___mysqli_ston"]（这是一个 MySQLi 连接对象）执行 SQL 查询 query。
- 如果查询成功，返回结果集（result 是一个 mysqli_result 对象）；
- 如果查询失败（比如 SQL 语法错误），则返回 false。

注意：mysqli_query() 只会在查询本身出错时返回 false，比如 SQL 语法错误。但如果数据库连接根本没建立好，它可能也会失败。

第二步：or die(...) —— 错误处理机制

在 PHP 中，A or B 的意思是：如果 A 为真（成功），就跳过 B；如果 A 为假（失败），就执行 B。

所以：
- 如果 mysqli_query() 成功 → result 被赋值，die() 不执行；
- 如果失败 → 执行 die(...)，直接终止脚本并输出错误信息。

第三步：动态判断错误类型

这里的核心逻辑是：区分“连接已建立但查询失败” vs “连接根本没建立”。

条件判断：
```
(is_object(GLOBALS["___mysqli_ston"]))
```

- 检查 GLOBALS["___mysqli_ston"] 是否是一个有效的对象（即数据库连接成功建立了）。
  - ✅ 如果是对象 → 说明连接是好的，问题出在 SQL 查询本身（比如注入导致的语法错误）。
    - 那就调用 mysqli_error(conn) 获取最近一次查询的错误信息。
  - ❌ 如果不是对象（比如是 false 或 null）→ 说明连接都没成功。
    - 那就调用 mysqli_connect_error() 获取连接阶段的错误信息（如密码错误、数据库不存在等）。

📌 小知识：
- mysqli_error(connection)：获取已连接状态下的 SQL 执行错误。
- mysqli_connect_error()：获取连接数据库时的错误（静态函数，不依赖连接对象）。

第四步：临时变量 ___mysqli_res

```
(___mysqli_res = mysqli_connect_error()) ? ___mysqli_res : false
```

- 这是一个“赋值+判断”的技巧：
  - 先调用 mysqli_connect_error()，把返回值存到临时变量 ___mysqli_res；
  - 如果有错误信息（非空字符串），就用它；
  - 否则返回 false（虽然最后会转成空字符串显示）。

实际上，mysqli_connect_error() 在无错误时返回 null 或空字符串，所以这个三元判断略显冗余，但为了严谨性保留。

✅ 简化理解（等效逻辑）

你可以把整段代码想象成：

```
if (!mysqli_query(GLOBALS["___mysqli_ston"], query)) {
    if (is_object(GLOBALS["___mysqli_ston"])) {
        // 连接成功，但查询失败
        error = mysqli_error(GLOBALS["___mysqli_ston"]);
    } else {
        // 连接本身就失败了
        error = mysqli_connect_error();
    }
    die('' . error . '');
}
```

为什么 DVWA 要这么写？

1. 教学目的：让学习者看到具体的 SQL 错误（比如注入后出现的 syntax error），便于理解漏洞原理。
2. 健壮性：即使数据库连接配置错误（如密码改了），也能显示连接错误，而不是白屏。

但在真实生产环境中，绝对不要把数据库错误直接暴露给用户！这会泄露敏感信息（表结构、SQL 逻辑等）。应该记录日志，并显示通用错误页。

### SQL注入登录
<img width="1353" height="1287" alt="image" src="https://github.com/user-attachments/assets/439570ec-478a-4f95-a45d-0ef87fdd2783" />
<img width="1351" height="1292" alt="image" src="https://github.com/user-attachments/assets/a3a87775-4b36-44c2-a111-86b5c2245804" />





