### 限制IP登陆ssh

1. vim /etc/hosts.allow

   ```bash
   sshd:36.110.228.:allow  #允许 36.110.228. 这个 IP段 地址 ssh 登录
   ```

2. vim /etc/hosts.deny

   ```bash
   sshd:ALL #拒绝全部IP
   ```

3. service sshd restart

hosts.allow和hosts.deny两个文件同时设置规则的时候，**hosts.allow文件中的规则优先级高**，按照此方法设置后服务器只允许36.110.228.这个IP段地址的SSH登录，其它的IP都会拒绝。

