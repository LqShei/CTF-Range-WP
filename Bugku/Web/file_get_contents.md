### 题目：file_get_contents

<img width="1196" height="817" alt="image" src="https://github.com/user-attachments/assets/70f38425-a86a-4a51-a6e9-d1a98e6c0ffd" />

今天来做一下web题中的file_get_contents，打开题目就是一段PHP代码
<img width="948" height="542" alt="image" src="https://github.com/user-attachments/assets/b9c6770a-8d48-4831-a3bd-8c1ed7a17984" />
#### 代码如下：
```php
<?php
extract($_GET);
if (!empty($ac))
{
$f = trim(file_get_contents($fn));
if ($ac === $f)
{
echo "<p>This is flag:" ." $flag</p>";
}
else
{
echo "<p>sorry!</p>";
}
}
?>
```
#### 代码分析
```tex
1. 使用 `extract($_GET)` 将GET参数提取为变量
2. 如果 `$ac` 变量不为空
3. 从 `$fn` 指定的文件中读取内容（去除首尾空格）
4. 如果 `$ac` 的值与文件内容完全相等（使用严格比较 `===`）
5. 则输出flag

这看起来是一个需要构造特定GET请求的挑战。我们需要设置 `ac` 参数和 `fn` 参数。
```

这是一个典型的PHP extract()漏洞利用。`extract($_GET)`会从GET参数创建变量，但如果有同名的变量存在，它会覆盖。

从代码逻辑看：

1. 我们需要传递两个参数：`ac` 和 `fn`
2. `fn` 应该是一个文件名，PHP会读取这个文件的内容
3. `ac` 需要严格等于从文件中读取的内容

关键点是：**我们需要控制 `$f` 变量的值**。我可以使用 `extract()` 漏洞来覆盖它！

让我们尝试几种方法：

**方法1：直接覆盖 `$f` 变量**
由于 `extract($_GET)` 执行在代码开头，我们可以尝试直接设置 `f` 参数来覆盖 `$f` 变量：

```tex
http://171.80.2.169:13661/?ac=test&fn=test.txt&f=test
```

<img width="1195" height="543" alt="image" src="https://github.com/user-attachments/assets/581eba84-d301-455a-b294-456244da4fc7" />


显示了"sorry!"，说明我们的方法不对。让我们再分析一下：`extract()` 会覆盖已有的变量，但是 `$f = trim(file_get_contents($fn));` 这行在 `extract()` 之后，所以它会覆盖我们设置的 `$f`。

我们需要找到一个方法让 `$fn` 参数指定的文件读取失败，这样 `file_get_contents()` 会返回 `false` 或空值，然后 `trim(false)` 会返回空字符串。让我们试试：

```tex
http://171.80.2.169:13661/?ac=&fn=nonexistentfile123.txt
```

这次没有显示"sorry!"，但也没有输出flag。是因为 `$ac` 为空，所以没有执行 if 块。让我们试试另一个想法：利用 `extract()` 的特性，我们可以传递一个 `fn` 参数来覆盖 `$fn` 变量，但是我们也可以在 `fn` 中传递一个空值或特殊值。

实际上，我意识到另一个关键点：`trim(file_get_contents($fn))` - 如果我们让 `$fn` 是一个空字符串，`file_get_contents('')` 会返回 `false`，然后 `trim(false)` 返回空字符串。

不过，先让我们思考一下：这可能是一个PHP伪协议利用。`file_get_contents()` 可以读取PHP伪协议，比如 `php://input` 来读取POST数据，或者 `data://` 来嵌入数据。

让我们尝试使用 `data://` 伪协议：

```tex
http://171.80.2.169:13661/?ac=test&fn=data://text/plain,test
```

**Well,我们拿到了flag！**

flag是：`flag{e6dcc9ad1a1e4e5e984e06d2270e64bf}`

<img width="1270" height="558" alt="image" src="https://github.com/user-attachments/assets/8b893d48-fb8a-42e6-95e6-0acf9b6aecd2" />


现在让我复盘一下我们是如何解决的：

1. **分析代码逻辑**：

   - `extract($_GET)` 将GET参数提取为变量
   - 如果 `$ac` 不为空，则从 `$fn` 指定的文件中读取内容到 `$f`
   - 如果 `$ac === $f` (严格相等)，则输出flag

2. **漏洞利用**：

   - 关键点是 `file_get_contents($fn)` 可以使用PHP伪协议
   - `data://` 伪协议允许在URL中嵌入数据
   - 我们使用了 `data://text/plain,test` 作为 `fn` 参数，这样 `file_get_contents()` 会读取字符串 `"test"`
   - 同时设置 `ac=test`，这样 `$ac === $f` 的条件就满足了

3. **具体payload**：

   ```
   http://171.80.2.169:13661/?ac=test&fn=data://text/plain,test
   ```

这个CTF题目考察了以下几个知识点：

1. PHP `extract()` 函数的危险使用
2. `file_get_contents()` 支持PHP伪协议
3. `data://` 伪协议的使用
4. 通过控制输入来满足程序逻辑条件

这是一个典型的PHP伪协议利用题目，通过data协议控制文件读取内容，从而满足条件获取flag。
