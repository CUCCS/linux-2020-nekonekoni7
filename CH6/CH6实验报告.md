# Chap0x06 
## 实验环境
ubuntu18.04

putty

工作主机host-only:`192.168.56.101`

目标主机host-only:`192.168.56.102`

***
## 工作主机免密SSH登录目标主机
以下在工作主机：
```
# 在工作主机生成ssh-key
ssh-keygen -b 4096

#工作主机通过ssh-copy-id方式导入ssh-key 
ssh-copy-id -i ~/.ssh/id_rsa.pub root@192.168.56.102
```
![免密登录](img/ssh1.png)
------
***
## FTP 
### 实验工具
vsftpd
#### 1.选择原因
+ 安全性高
+ 轻量级
+ 大型站点信任并使用vsftpd
+ CVE漏洞数量位于ProFTPd和PureFTPd之间
#### 2.参考资料
+ [proftpd vs pureftpd vs vsftpd](https://systembash.com/evaluating-ftp-servers-proftpd-vs-pureftpd-vs-vsftpd
)
### 实验过程
- 安装软件（proftpd）

    ```bash
    sudo apt-get update
    sudo apt-get install proftpd
    ```

#### 配置一个提供匿名访问的FTP服务器，匿名访问者可以访问1个目录且仅拥有该目录及其所有子目录的只读访问权限

- 修改/etc/proftpd目录下的proftpd.conf文件

    ```bash
    sudo vim /etc/proftpd/proftpd.conf
    ```

    ![1.1FTP编辑文件](img/ftpp.png)

- 创建共享文件夹

    ```bash
    sudo mkdir ftp
    ```

- 更改用户共享目录

    ```bash
    sudo chown -R ftp:nogroup /home/ftp
    sudo usermod -d /home/ftp ftp
    ```

- 实现效果

    ![1.2FTP实现效果](img/FTP.png)


#### 配置一个支持用户名和密码方式访问的账号，该账号继承匿名访问者所有权限，且拥有对另1个独立目录及其子目录完整读写（包括创建目录、修改文件、删除文件等）权限

- 修改/etc/proftpd目录下的proftpd.conf文件

    ```bash
    sudo vim /etc/proftpd/proftpd.conf

    #添加以下内容
    AuthOrder mod_auth_file.c mod_auth_unix.c
    AuthUserFile /usr/local/etc/proftpd/passwd
    AuthGroupFile /usr/local/etc/proftpd/group
    PersistentPasswd off
    RequireValidShell off
    ```

- 使用ftpasswd创建passwd和group文件

    - 创建了一个user1和user2用户

        ```bash
        sudo ftpasswd --passwd --file=/usr/local/etc/proftpd/passwd --name=user1 --uid=1024 --home=/home/user1 --shell=/bin/false
        ```

    - 创建了一个virtualusers用户组

        ```bash
        sudo ftpasswd --file=/usr/local/etc/proftpd/group --group --name=virtualusers --gid=1024
        ```

    - 把user1和user2加入virtualusers组

        ```bash
        sudo ftpasswd --group --name=virtualusers --gid=1024 --member=user1 --member=user2 --file=/usr/local/etc/proftpd/group
        ```

- 修改/home/user1权限

    ```bash
    sudo chown -R 1024:1024 /home/user1
    sudo chmod -R 700 /home/user1
    ```

- 实现效果

    ![FTP实现效果]![](img/ftp2.png)


#### FTP用户不能越权访问指定目录之外的任意其他目录和文件

- 编辑/etc/proftpd目录下的proftpd.conf文件

    ```bash
    sudo vim /etc/proftpd/proftpd.conf
    ```

- 添加以下内容

    ![FTP编辑文件](img/ftp3.png)


#### 匿名访问权限仅限白名单IP来源用户访问，禁止白名单IP以外的访问

- 编辑 /etc/proftpd目录下的proftpd.conf文件

    ```bash
    sudo vim /etc/proftpd/proftpd.conf
    ```

- 添加以下内容，即IP在白名单中的192.168.243.4可以对ftp服务器进行访问，白名单外的IP不能访问

    ![1.6FTP编辑文件](img/ftp4.png)

***
## NFS
#### 在1台Linux上配置NFS服务，另1台电脑上配置NFS客户端挂载2个权限不同的共享目录，分别对应只读访问和读写访问权限

- Host

    - IP：192.168.56.101

    - 配置NFS服务

        ```bash
        sudo apt-get update
        sudo apt-get install nfs-kernel-server
        ```

    - 创建一个用于挂载的（可读写）文件夹

        ```bash
        sudo mkdir /var/nfs/general -p
        sudo chown nobody:nogroup /var/nfs/general
        #另一个用于挂载的文件夹为/home（不可读写），不需要创建
        ```

    - 修改/etc/exports文件（即NFS服务的主要配置文件），添加以下内容

        ```bash
        /var/nfs/general 192.168.243.4(rw,sync,no_subtree_check)
        /home 192.168.243.4(sync,no_root_squash,no_subtree_check)
        ```

- Client

    - IP：192.168.56.102

    - 配置NFS服务

        ```bash
        sudo apt-get update
        sudo apt-get install nfs-common #创建相应的挂载文件
        sudo mkdir -p /nfs/general
        sudo mkdir -p /nfs/home #挂载文件夹
        sudo mount 192.168.243.3:/var/nfs/general /nfs/general
        sudo mount 192.168.243.3:/home /nfs/home
        ```
    - 执行前启动NFS服务
     ```bash
     sudo service nfs-kernel-server restart  
    ```
- 实现效果

    ![2.1实现效果](img/NFS.png)


-------
### DHCP

### 实验过程
![编辑/etc/netplan/01-netcfg.yaml文件](img/DHCP1.png)
+ client
  ```
  network:
  version: 2
  renderer: networkd
  ethernets:
  enp0s3:
        dhcp4: yes
        dhcp6: yes
    enp0s8:
        dhcp4: yes
        dhcp6: yes
        dhcp-identifier: mac
    enp0s9:
        dhcp4: yes
        dhcp6: yes
  ```
+ server
  ```
  network:
  version: 2
  renderer: networkd
  ethernets:
     enp0s3:
       dhcp4: yes
       dhcp6: yes
     enp0s8:
       dhcp4: yes
       dhcp6: yes
       dhcp-identifier: mac
     enp0s9:
        dhcp4: no
        dhcp6: no
        dhcp-identifier: mac
        addresses: [192.168.57.1/24]
  ```
  `sudo netplan apply`使配置生效
   + server

     运行脚本,修改`/etc/default/isc-dhcp-server`文件
     如下：
     ```
     INTERFACESv4="enp0s9"
     INTERFACESv6="enp0s9"
     ```
   + client

     修改/etc/netplan/01-netcfg.yaml,开启dhcp

  检查dhcp服务是否正确工作`sudo netstat -uap`
  ![dhcp服务正确工作](img/dhcp.png)
--------
## DNS
### 实验过程
+ server
  - 安装bind
     ```
     sudo apt-get install bind9
     ```
  - 修改配置文件/etc/bind/named.conf.options,添加以下： 
     ```
     listen-on { 192.168.57.1; };
     allow-transfer { none; };
     forwarders {
      8.8.8.8;
      8.8.4.4;
     };
     ```
  - 编辑配置文件/etc/bind/named.conf.local
     ```
     zone "cuc.edu.cn" {
     type master;
     file "/etc/bind/db.cuc.edu.cn";
     };
     ```
  - 生成配置文件`db.cuc.edu.cn` 
     ```
     sudo cp /etc/bind/db.local /etc/bind/db.cuc.edu.cn
     ```
  - 编辑配置文件`db.cuc.edu.cn`,添加以下：
     ```
     ;@      IN      NS      localhost.
             IN      NS      ns.cuc.edu.cn.
     ns      IN      A       192.168.57.1
      wp.sec.cuc.edu.cn.      IN      A       192.168.57.1
      dvwa.sec.cuc.edu.cn.    IN      CNAME   wp.sec.cuc.edu.cn.
     @       IN      AAAA    ::1
     ```
+ client
  - 安装resolvconf
     ```
     sudo apt-get update && sudo apt-get install resolvconf
     ``` 
  - 修改配置文件vim /etc/resolvconf/resolv.conf.d/head,添加以下：
     ```
     search cuc.edu.cn
     nameserver 192.168.57.1
     ```
### 实验结果

  ![测试DNS搭建成功](img/DNS.png)
-------
--------

## Samba

### 实验过程
- 安装Samba服务器

    ```bash
    sudo apt-get install samba
    ```

- 创建Samba共享专用的用户

    ```bash
    sudo useradd -M -s /sbin/nologin smbuser
    sudo passwd smbuser
    ```

- 创建的用户必须有一个同名的Linux用户

    ```bash
    sudo smbpasswd -a smbuser
    ```

- 在/etc/samba/smb.conf 文件尾部追加以下“共享目录”配置，guest为匿名用户可以访问的目录（不可写），demo为虚拟用户才能访问的目录（可读写）

    ```bash
    [guest]
    path = /home/samba/guest/
    read only = yes
    guest ok = yes

    [demo]
    path = /home/samba/demo/
    read only = no
    guest ok = no
    force create mode = 0660
    force directory mode = 2770
    force user = smbuser
    force group = smbgroup
    ```

- 恢复一个samba用户

    ```bash
    sudo smbpasswd -e smbuser
    ```

- 创建用户组

    ```bash
    sudo groupadd smbgroup
    sudo usermod -G smbgroup smbuser
    ```


- 创建用于共享的文件夹并修改用户组

    ```bash
    sudo mkdir -p /home/samba/guest/
    sudo mkdir -p /home/samba/demo/
    sudo chgrp -R smbgroup /home/samba/guest/
    sudo chgrp -R smbgroup /home/samba/demo/
    sudo chmod 2775 /home/samba/guest/
    sudo chmod 2770 /home/samba/demo/
    ```

- 启动samba

    ```bash
    sudo smbd
    ```

### 实验结果
- 在Windows上访问`\\192.168.56.104`

    ![共享文件夹](img/smab1.png)

- demo文件夹需要用户名密码

    ![登录](img/smab2.png)

    ![成功登陆](img/smab3.png)
### 参考资料
- [linux-2019-luyj](https://github.com/CUCCS/linux-2019-luyj/blob/Linux_exp0x06/Linux_exp0x06/Linux_exp0x06.md)

- [Ubuntu 18.04安装Samba服务器及配置](https://www.linuxidc.com/Linux/2018-11/155466.htm)

- [第六章：shell脚本编程练习进阶（实验）](https://c4pr1c3.github.io/LinuxSysAdmin/chap0x06.exp.md.html)