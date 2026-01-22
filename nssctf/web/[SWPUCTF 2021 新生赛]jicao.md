## 题目 ：[SWPUCTF 2021 新生赛]jicao



1.开启环境 ： [node7.anna.nssctf.cn:23912](http://node7.anna.nssctf.cn:23912/)

2.看到页面中有一段PHP代码：

```php
<?php
highlight_file('index.php');
include("flag.php");
$id=$_POST['id'];
$json=json_decode($_GET['json'],true);
if ($id=="wllmNB"&&$json['x']=="wllm")
{echo $flag;}
?>
```

### 代码分析：

1. **条件逻辑**：
   - `$id`来自POST参数`id`，必须等于`"wllmNB"`
   - `$json`来自GET参数`json`，JSON解码后，`$json['x']`必须等于`"wllm"`
   - 两个条件都必须满足才能输出`$flag`

2. **参数来源**：
   - POST参数：`id`
   - GET参数：`json`

3. **JSON处理**：
   - `json_decode($_GET['json'], true)` - 第二个参数`true`表示返回数组而不是对象

### 解题步骤：

1. **POST请求设置**：设置了`id=wllmNB`
2. **GET参数设置**：设置了`json={"x":"wllm"}`，JSON解码后得到数组`['x' => 'wllm']`
3. **双重条件满足**：两个条件都满足，所以输出了`$flag`

<img width="1280" height="859" alt="image" src="https://github.com/user-attachments/assets/7b8d8064-4fc1-47e1-8283-2f81f2347d76" />



这是一个常见的安全意识训练题目，考察点包括：

1. **HTTP方法理解**：POST和GET参数的区别
2. **JSON处理**：理解PHP的`json_decode()`函数
3. **条件逻辑**：理解AND条件（两个条件都必须满足）
4. **字符串比较**：PHP的松散/严格比较（这里使用的是`==`相等比较）
