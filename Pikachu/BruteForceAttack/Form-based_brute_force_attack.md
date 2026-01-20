# Pikachu - 基于表单的暴力破解（Brute Force Attack）

## 漏洞描述

在 Pikachu 靶场中，“基于表单的暴力破解”模块是一个典型的弱口令漏洞场景。  
攻击者可以通过不断尝试用户名和密码组合，绕过身份验证系统。

> ⚠️ 注意：本题默认没有限制登录次数或验证码，因此可以被自动化工具快速爆破。

---

## 漏洞位置

- 路径：`/vul/burteforce/bf_form.php#`
- 页面：登录界面（基于表单提交）
- 用户名：`admin`
- 密码：未知（需暴力破解）

---

## 漏洞原理

当服务器未对登录接口进行以下防护时，存在暴力破解风险：
- 无登录失败次数限制
- 无验证码（CAPTCHA）
- 无请求频率限制（Rate Limiting）
- 无账户锁定机制

攻击者可以利用字典 + 自动化工具（如 Hydra、Burp Intruder）枚举出正确密码。

---

## 解题思路

### 1.观察登录页面

<img width="1160" height="398" alt="image" src="https://github.com/user-attachments/assets/ea7cd8d4-9688-4850-a029-efbc8f962621" />

### 2.输入密码`admin`并打开bp进行抓包
<img width="1855" height="636" alt="image" src="https://github.com/user-attachments/assets/71be6365-3594-4583-82ed-139648cc3d21" />

### 3.CTRL + I 放到`Intruder`模块进行暴力破解，将 `passwd` 做为 `payload` 进行爆破
<img width="1675" height="788" alt="image" src="https://github.com/user-attachments/assets/fe669632-e367-4efe-aa87-c32e1bdbea10" />

### 4.加载弱口令字典
<img width="779" height="826" alt="image" src="https://github.com/user-attachments/assets/27703fb2-5469-4130-80d9-c070f8e78d23" />

### 5.开始攻击
<img width="2500" height="1595" alt="image" src="https://github.com/user-attachments/assets/39c48e93-e373-49d6-8342-778eecae346e" />

### 6.使用`View filter` 找出登录成功的数据包，可以看出admin用户的密码是：123456
<img width="2504" height="1552" alt="image" src="https://github.com/user-attachments/assets/a6d45343-2117-4062-bcca-3280cfe70013" />

### 7.当然也可以使用hydra进行密码爆破，命令如下：
```sh
hydra -l admin -P pass1000.txt 117.72.78.253 -s 8080 http-post-form "/vul/burteforce/bf_form.php:username=^USER^&password=^PASS^&submit=Login:F=username or password is not exists"
```
<img width="1276" height="284" alt="image" src="https://github.com/user-attachments/assets/eef69187-6aba-47a1-b332-ef681da010f8" />

```sh
hydra -l admin -P pass1000.txt 117.72.78.253 -s 8080 http-post-form "/vul/burteforce/bf_form.php:username=^USER^&password=^PASS^&submit=Login:S=login success"
```
<img width="1274" height="286" alt="image" src="https://github.com/user-attachments/assets/b0d00b85-12ee-4012-89c8-16c2694e77e7" />

#### 7.1 Hydra 常用参数说明

| **参数**  | **含义**           | **示例**      | **说明**                                       |
| --------- | ------------------ | ------------- | ---------------------------------------------- |
| **`-l`**  | 用户名 (Login)     | `-l admin`    | 指定一个具体的单用户名进行爆破。               |
| **`-L`**  | 用户名列表 (List)  | `-L user.txt` | 指定一个包含多个用户名的字典文件。             |
| **`-p`**  | 密码 (Password)    | `-p 123456`   | 指定一个具体的单密码进行测试。                 |
| **`-P`**  | 密码字典 (Path)    | `-P pass.txt` | 指定用于爆破的密码字典文件路径。               |
| **`-s`**  | 端口 (Server port) | `-s 8080`     | 如果服务没有运行在默认端口，需手动指定。       |
| **`-vV`** | 详细显示 (Verbose) | `-vV`         | 显示爆破过程中的每一组尝试（推荐调试时使用）。 |
| **`-t`**  | 线程数 (Tasks)     | `-t 16`       | 设置并发线程数，默认为 16。                    |

引号里的内容由**三个部分**组成，中间用**冒号 `:`** 分隔：

```
"/vul/burteforce/bf_form.php : username=^USER^&password=^PASS^&submit=Login : F=username or password is not exists"
```

| **序号**     | **示例内容**                                   | **含义说明**                                                 |
| ------------ | ---------------------------------------------- | ------------------------------------------------------------ |
| **第一部分** | `/vul/burteforce/bf_form.php`                  | **表单提交路径**：登录请求发送到的具体页面 URL。             |
| **第二部分** | `username=^USER^&password=^PASS^&submit=Login` | **POST 数据包体**：从 Burp Suite 抓包获取的数据。`^USER^` 和 `^PASS^` 是占位符，Hydra 会自动将字典里的值填入。 |
| **第三部分** | `F=...` 或 `S=...`                             | **验证标志**：Hydra 用来判断登录结果的关键词（见下方详述）。 |

#### F 模式 vs S 模式的区别

这两条命令代表了爆破时的两种逻辑策略：

#### 方案 A：F 模式 (Failure - 找失败)

```
F=username or password is not exists
```

- **逻辑**：Hydra 会检查页面返回的 HTML 源码。
- **含义**：如果页面中**出现了**这段话，说明登录**失败**；如果**没出现**，Hydra 就认为这个密码**正确**。
- **适用场景**：当你明确知道登录失败会提示什么错误（如“密码错误”）时使用。

#### 方案 B：S 模式 (Success - 找成功)

```
S=login success
```

- **逻辑**：Hydra 同样检查页面源码。
- **含义**：只有当页面中**出现了** "login success" 字样时，Hydra 才会记录这个密码。
- **适用场景**：当你明确知道登录成功后的页面特征（如“欢迎登录”、“Success”）时使用。

------

### 
**注意**：在使用 `http-post-form` 时，占位符必须是 `^USER^` 和 `^PASS^`（带脱字符）。如果直接写 `USER` 或 `PASS`，Hydra 会把它当作普通的字符串发送，导致爆破失败。

## 修复建议 (Remediation)

为了防止此类暴力破解攻击，建议开发人员采取以下措施：

1. **引入多因素认证 (MFA)**：除了密码，增加手机验证码或 OTP。
2. **增加图形验证码**：在登录请求前增加验证码校验，防止自动化脚本提交数据。
3. **账户锁定机制**：连续登录失败 N 次后，暂时锁定该账户或 IP 30 分钟。
4. **请求速率限制**：利用 Web 服务器（如 Nginx）限制同一 IP 在短时间内的请求频率。
5. **强口令策略**：在注册时强制要求密码包含数字、大小写字母和特殊符号。






