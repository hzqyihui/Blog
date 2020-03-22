## 1.安装
```
sudo apt-get install vsftpd
```
## 2. 配置项
```
anonymous_enable=YES #允许匿名用户
local_enable=YES #允许使用本地用户账号登陆
write_enable=YES #允许ftp用户写数据
connect_from_port_20=YES #通过20端口传输数据
```
## 3. 其他配置项
```
ftpd_banner=welcome to ftp service ：设置连接服务器后的欢迎信息

idle_session_timeout=60 ：限制远程的客户机连接后，所建立的控制连接，在多长时间没有做任何的操作就会中断(秒)

data_connection_timeout=120 ：设置客户机在进行数据传输时,设置空闲的数据中断时间

accept_timeout=60 设置在多长时间后自动建立连接

connect_timeout=60 设置数据连接的最大激活时间，多长时间断开，为别人所使用;

max_clients=200 指明服务器总的客户并发连接数为200

max_per_ip=3 指明每个客户机的最大连接数为3

local_max_rate=50000(50kbytes/sec)  本地用户最大传输速率限制

anon_max_rate=30000匿名用户的最大传输速率限制

pasv_min_port=端口

pasv-max-prot=端口号 定义最大与最小端口，为0表示任意端口;为客户端连接指明端口;

listen_address=IP地址 设置ftp服务来监听的地址，客户端可以用哪个地址来连接;

listen_port=端口号 设置FTP工作的端口号，默认的为21

local_root=path 无论哪个用户都能登录的用户，定义登录帐号的主目录, 若没有指定，则每一个用户则进入到个人用户主目录;

chroot_local_user=yes/no 是否锁定本地系统帐号用户主目录(所有);锁定后，用户只能访问用户的主目录/home/user;

chroot_list_enable=yes/no 启用不锁定用户在主目录的名单

chroot_list_file=/etc/vsftpd/chroot_list指定列表文件

userlist_enable=YES/NO 是否加载用户列表文件;

userlist_deny=YES 表示上面所加载的用户允许登录;

userlist_file=/etc/vsftpd/user_list 指定列表文件
```