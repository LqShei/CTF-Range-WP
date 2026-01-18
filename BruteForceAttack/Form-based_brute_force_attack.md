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








