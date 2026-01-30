### DVWA 暴力破解
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

