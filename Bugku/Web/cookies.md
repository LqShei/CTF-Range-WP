## 题目： cookies
<img width="1185" height="693" alt="image" src="https://github.com/user-attachments/assets/5213011f-34ac-4144-8340-e68a554b9c32" />
### 1.打开场景
发现主页是一堆乱码，然后链接是 ： http://171.80.2.169:10581/index.php?line=0&filename=a2V5cy50eHQ=
<img width="2519" height="245" alt="image" src="https://github.com/user-attachments/assets/c74f022a-a2ae-46fa-b35b-2b2ff6da960a" />
解码一下filename后面的base64编码，发现是keys.txt ：
<img width="1270" height="983" alt="image" src="https://github.com/user-attachments/assets/5eded5aa-d6f6-4579-8031-96abd2d47357" />

2.将`filename`换成`flag.txt`(ZmxhZy50eHQ=) 试试看，没有什么反应 
<img width="2519" height="492" alt="image" src="https://github.com/user-attachments/assets/f83895c9-ad12-4a30-86e6-fed4f1d163c8" />

换成 `index.php`（aW5kZXgucGhw） 看看
<img width="1916" height="445" alt="image" src="https://github.com/user-attachments/assets/f1bc665e-e105-46de-ad13-f8af434348fd" />
发现还是没有什么反应，之后由于链接中含有两个参数我又将line换成1：http://171.80.2.169:10581/index.php?line=1&filename=aW5kZXgucGhw
<img width="2519" height="264" alt="image" src="https://github.com/user-attachments/assets/a8c55791-b187-49e3-94fb-9a1c71883c55" />
发现了一个error_reporting(0); 
之后又换成line=2,不难猜，应该就是`index.php`的源码内容,之后穷举
<img width="1434" height="206" alt="image" src="https://github.com/user-attachments/assets/acd2a2e5-9715-4626-8c33-4354dbb28e33" />

3.整理出`index.php`的代码如下：
```php
<?php
error_reporting(0); // 关闭错误报告，避免暴露敏感信息

// 从 GET 参数获取 filename，若不存在则默认为空字符串
$file = base64_decode(isset($_GET['filename']) ? $_GET['filename'] : "");

// 从 GET 参数获取 line，若不存在则默认为 0
$line = isset($_GET['line']) ? intval($_GET['line']) : 0;

// 如果 $file 为空，则重定向到 index.php，并附带默认参数（base64 编码的 "index.php"）
if ($file == "") {
    header("location:index.php?line=&filename=a2V5cy50eHQ="); // a2V5cy50eHQ = keys.txt
}

// 定义允许访问的文件列表
$file_list = array(
    '0' => 'keys.txt',
    '1' => 'index.php',
);

// 【关键逻辑】如果 Cookie 中存在 margin=margin，则动态添加 keys.php 到允许列表
if (isset($_COOKIE['margin']) && $_COOKIE['margin'] == 'margin') {     // 看这里
    $file_list[2] = 'keys.php';
}

// 检查请求的文件是否在允许列表中
if (in_array($file, $file_list)) {
    $fa = file($file);           // 读取文件内容为数组（每行一个元素）
    echo $fa[$line];             // 输出指定行的内容
}
?>

```

题目是cookies ，然后代码里面Cookie 控制扩展白名单
设置 Cookie: margin=margin → 可将 keys.php 加入允许列表，攻击者只需设置该 Cookie 即可读取原本不允许访问的 keys.php 文件。
让我们尝试添加 cookie 并访问`keys.php`(a2V5cy5waHA=)试试看
<img width="1966" height="670" alt="image" src="https://github.com/user-attachments/assets/20222e91-b388-4c55-9fb8-46e2ae18867f" />
ok ,得到flag:flag{a277c68a83bb600bac8a5173f1c6f68b}
<img width="1467" height="473" alt="image" src="https://github.com/user-attachments/assets/4c606b0e-84f9-4e50-95e9-c338b91bee3d" />


