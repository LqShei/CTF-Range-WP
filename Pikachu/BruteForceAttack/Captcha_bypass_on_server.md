## 服务端的验证码绕过
<img width="2519" height="679" alt="image" src="https://github.com/user-attachments/assets/2400b347-b23e-4e0b-be2b-2a3cf145ca19" />

### 1.点击提示查看
这个验证码好像一直有效哎!用户还是那三个默认用户.

### 2.使用burpsuite 抓包看一下
<img width="1863" height="1242" alt="image" src="https://github.com/user-attachments/assets/d5b07f45-e1b6-41f5-9218-8987399a983f" />

### 3.发送到Intruder,将密码设为payload,并加载弱口令字典，开始攻击
<img width="2462" height="1366" alt="image" src="https://github.com/user-attachments/assets/f4180019-3403-4525-87df-4323f2542b63" />

### 4.使用过滤器查看含有success字段的数据包
<img width="2499" height="1070" alt="image" src="https://github.com/user-attachments/assets/f8636536-099d-4e75-92d0-06a931f96f98" />

### 5.查看数据包可以发现，123456就是admin的密码
<img width="2496" height="1578" alt="image" src="https://github.com/user-attachments/assets/3ae056bc-fd12-4d40-93c7-319d43c37822" />
