# vsftpd

vsftpd是“very secure FTP daemon”的缩写，是一个完全免费、开源的ftp服务器软件。小巧轻快，安全易用，支持虚拟用户，支持带宽限制等功能。

1. `yum -y install vsftpd`下载安装，默认配置文件在 /etc/vsftpd/vsftpd.conf；

2. 创建虚拟用户：

   - 在根目录或用户目录下创建ftp文件夹：`makdir ftpfile`
   - 添加匿名用户：`useradd ftpuser -d /home/ftpfile -s /sbin/nologin`
   - 修改ftpfile权限：`chown -R ftpuser.ftpuser /home/ftpfile`
   - 重设ftpuser密码：`passwd ftpuser`

3. 配置：

   - `touch /etc/vsftpd/chroot_list`，并将虚拟用户ftpuser直接写入里面

   - `vim /etc/selinux/config`，修改 SELINUX=disabled

     - 如果之后验证时遇到550拒绝访问时执行
     - `getsebool -a|grep ftp`查看状态
     - `setsebool -P ftpd_full_access on`、`setsebool tftp_home_dir on`

   - 重启Linux

   - 配置vsftpd.conf：修改以下（每一项的说明见：[鸟哥的LInux私房菜](http://linux.vbird.org/linux_server/0410vsftpd.php#server_pkg)）

     ```shell
     local_enable=YES
     
     write_enable=YES
     
     local_umask=022
     
     dirmessage_enable=YES
     
     xferlog_enable=YES
     
     connect_from_port_20=YES
     
     ftpd_banner=Welcome to fms5cms FTP service.
     
     local_root=/home/ftpfile
     anon_root=/home/ftpfile
     use_localtime=yes
     
     #会使得ftp用户可以进入上一级目录，这是比较危险的，所以建议设为NO
     chroot_local_user=NO
     chroot_list_enable=YES
     #vsftpd2.3.5后，当用户被限定在其主目录下，则该用户的主目录不能再有写权限了，否则会报错，故配置allow_writeable_chroot=YES
     allow_writeable_chroot=YES
     
     chroot_list_file=/etc/vsftpd/chroot_list
     
     listen=NO
     
     listen_ipv6=YES
     
     pam_service_name=vsftpd
     userlist_enable=YES
     
     tcp_wrappers=YES
     
     pasv_min_port=61001
     pasv_max_port=62000
     pasv_enable=YES
     ```

   - 这里关闭了防火墙所以没有配置防火墙

启动：`systemctl start vsftpd.service`

打开浏览器访问：ftp:ip地址   ip地址可以通过`ifconfig`命令查询，输入创建的ftp匿名用户的账号、密码即可。

可以使用ftp客户端软件(如：filezilla)连接ftp服务器，进行文件上传下载