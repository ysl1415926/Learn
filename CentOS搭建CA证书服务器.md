# Sercer服务器&CA服务器：

## 环境准备：
  Server：192.168.179.136
  
  CA：192.168.179.135

设置IP地址为自动获取

编辑网卡文件：vi /etc/sysconfig/network-scripts/ifcfg-ens33

我这里的网卡是ens33,可以运行ip addr 或者ip a,来查看自己的网卡
![image](https://github.com/ysl1415926/cve/assets/138963581/f557a443-d012-40c9-bdc9-bf2b68a7802a)

将ONBOOT=no修改ONBOOT=yes
![image](https://github.com/ysl1415926/cve/assets/138963581/1b6ea202-5afd-4c64-9f59-e5995fe4a866)

保存退出，重启网络
![image](https://github.com/ysl1415926/cve/assets/138963581/e2d02a8b-580c-4bba-a185-39d56c95eebe)

## 第一步，更换本地源
### 1，备份yum
首先我们对系统本身的yum源进行备份: mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bik

这条命令用于将/etc/yum.repos.d/CentOS-Base.repo文件重命名为/etc/yum.repos.d/CentOS-Base.repo.bik

在Linux系统中，mv命令用于移动文件或重命名文件。在这个命令中，/etc/yum.repos.d/CentOS-Base.repo是要移动或重命名的原始文件路径，/etc/yum.repos.d/CentOS-Base.repo.bik是目标文件路径。

执行该命令后，CentOS-Base.repo文件将被重命名为CentOS-Base.repo.bik，并放置在相同的目录中。这样做可以为CentOS-Base.repo文件创建一个备份，以免不小心修改或删除该文件时丢失重要的配置。

[root@localhost ~]# mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bik

[root@localhost ~]# ls /etc/yum.repos.d/

CentOS-Base.repo.bik  CentOS-CR.repo  CentOS-Debuginfo.repo  CentOS-fasttrack.repo  CentOS-Media.repo  CentOS-Sources.repo  CentOS-Vault.repo  CentOS-x86_64-kernel.repo

可以看到CentOS-Base.repo.bik这个文件，就是我们刚刚备份的文件

![image](https://github.com/ysl1415926/cve/assets/138963581/eed2a499-2fce-4124-829e-11f0efbbd20d)

如果使用yum命令安装东西，出现如下报错，原因是yum仓库没有正常启动

![image](https://github.com/ysl1415926/cve/assets/138963581/d2af200e-5a97-4e15-9e96-5d234e17f769)

我们就需要运行 curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo 来更换本地源

![image](https://github.com/ysl1415926/cve/assets/138963581/2a070fb3-ceb5-4883-b806-3f091cf1c35c)

换源完成后，重启yum仓库，这样yum就可以正常使用了

[root@localhost ~]# yum repolist all

![image](https://github.com/ysl1415926/cve/assets/138963581/5d419961-015f-4af0-85c6-26c283ad9b36)

接下来就是配置服务器了

# CA证书服务器的配置如下：

[root@server ~]# yum install -y openssl     //安装OpenSSL工具默认是安装好了的 

[root@server ~]# vim /etc/pki/tls/openssl.cnf    //查看配置文件

在编辑模式可以输入，set nu 来显示行号

![image](https://github.com/ysl1415926/cve/assets/138963581/28132451-d4d2-4cc8-9653-d4dac8f4bfeb)

42 dir             = /etc/pki/CA           #相关证书的存放的目录

43 certs           = $dir/certs            #存储签发的数字证书

45 database        = $dir/index.txt        # 记录颁发证书的信息

51 serial          = $dir/serial            #记录证书编号

[root@server ~]# cd /etc/pki/CA/    //这个目录是存放证书相关的文件的地方

[root@server CA]# ls

certs  crl  newcerts  private

[root@server CA]# cd private/    //这个目录是存放CA证书服务的私钥的地方

CA证书服务器创建自签名证书并设置权限为600

[root@server ~]# (umask 077;openssl genrsa -out /etc/pki/CA/private/cakey.pem 2048)

Generating RSA private key, 2048 bit long modulus
..............................................+++
.............................................................+++
e is 65537 (0x10001)

![image](https://github.com/ysl1415926/cve/assets/138963581/c12fa0b4-2932-4604-9603-87b540be81e9)

CA证书服务器签发本地自签名证书（需要输入一些基本信息）

[root@server ~]# openssl req -new -x509 -key /etc/pki/CA/private/cakey.pem -out /etc/pki/CA/cacert.pem -days 365

Country Name (2 letter code) [XX]:CN      //国家

State or Province Name (full name) []:SC  //所在省

Locality Name (eg, city) [Default City]:CD   //所在市

Organization Name (eg, company) [Default Company Ltd]:ZABBIX    //单位名称

Organizational Unit Name (eg, section) []:ZABBIX-SERVER  //组织单位名称

Common Name (eg, your name or your server's hostname) []:jw.com    //单位的域名

Email Address []:admin@163.com   邮箱

![image](https://github.com/ysl1415926/cve/assets/138963581/ff57ede4-ba1c-4f20-adf0-a9f0ae8dcb41)

CA证书服务还需要创建两个文件，才可以执行颁发证书操作

[root@server ~]# cd /etc/pki/CA/    //进入这个目录

[root@server CA]# touch index.txt    //创建记录申请证书的文件

[root@server CA]# echo 01 > serial    //证书编号

[root@server CA]# cat serial 

01

![image](https://github.com/ysl1415926/cve/assets/138963581/81e9cbc8-0470-4e47-a6f9-5e95e2863dfc)
# Apache服务器的配置

[root@clinet ~]# yum install -y httpd mod_ssl 

[root@clinet ~]# echo "this is CA " >> /var/www/html/index.html     //写入一个页面，暂时不要启动httpd服务器

创建私钥httpd.key

[root@clinet ~]# mkdir ssl    //创建一个目录

[root@clinet ~]# cd ssl/

[root@clinet ssl]# (umask 077;openssl genrsa -out /root/ssl/httpd.key 2048)

Generating RSA private key, 2048 bit long modulus
......+++
................................................................+++
e is 65537 (0x10001)

![image](https://github.com/ysl1415926/cve/assets/138963581/37007c6e-bf7f-4be2-8b73-400cdd4402a2)

依据私钥生成证书申请文件

[root@clinet ssl]# openssl req -new -key httpd.key -out httpd.csr    //必须和CA一样填写，唯一就是多了一个设置密码，如果不一样，后面签署的时候会报错

![image](https://github.com/ysl1415926/cve/assets/138963581/590bef41-08cc-4310-92df-624b9b7f1972)


[root@clinet ssl]# ls    //最后在这个目录就生成了两个文件了

httpd.csr  httpd.key

![image](https://github.com/ysl1415926/cve/assets/138963581/bfd78917-c3e9-4cf4-9e26-8b18863d1d46)

然后我们将生成的证书申请文件发送到 CA证书服务器进行授权操作

[root@clinet ssl]# scp httpd.csr root@192.168.179.135:/

![image](https://github.com/ysl1415926/cve/assets/138963581/f575e132-8a40-43a5-a0c3-f52ce945de06)
# CA证书服务器的操作
[root@localhost CA]# cd /

[root@localhost /]# ls

bin  boot  dev  etc  home  httpd.csr  lib  lib64  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

可以看见我们刚刚传过来的文件httpd.csr

![image](https://github.com/ysl1415926/cve/assets/138963581/217f429d-0881-492d-8a0a-4de9636d6b7f)

[root@server /]# openssl ca -in httpd.csr -out /etc/pki/CA/certs/httpd.crt -days 365

![image](https://github.com/ysl1415926/cve/assets/138963581/74a34fc0-f278-4283-9d82-e161d505d5ba)


然后我们在将生成的证书文件传送会Apache服务器即可

[root@server /]# scp /etc/pki/CA/certs/httpd.crt root@192.168.179.136:/root/ssl

root@192.168.179.136's password: 

httpd.crt                                                      100% 4571     7.2MB/s   00:00    

![image](https://github.com/ysl1415926/cve/assets/138963581/cf1c1ed7-2dc5-4df2-85d1-1340e88ee5ac)

# Apache服务器的操作
[root@clinet ssl]# ls   然后就会如下的文件

httpd.crt  httpd.csr  httpd.key

![image](https://github.com/ysl1415926/cve/assets/138963581/254735a0-7864-4211-99be-8c55f4642b6f)

[root@clinet ~]# vi /etc/httpd/conf.d/ssl.conf   //然后我们编辑这个文件   添加这两个文件所在的路径即可

   100 SSLCertificateFile /root/ssl/httpd.crt
   
   107 SSLCertificateKeyFile /root/ssl/httpd.key
![image](https://github.com/ysl1415926/cve/assets/138963581/3d85d8c4-d8cb-4495-b39b-069cd67c0f94)

   
先关闭防火墙以及selinux 不然等下启动会出现问题

[root@clinet ssl]# systemctl stop firewalld   

[root@clinet ssl]# setenforce 0

[root@clinet ssl]# systemctl start httpd

[root@clinet ssl]# ss -tan | grep 80

LISTEN     0      128       [::]:80                    [::]:*   

[root@clinet ssl]# ss -tan | grep 443

LISTEN     0      128       [::]:443                   [::]:* 

![image](https://github.com/ysl1415926/cve/assets/138963581/10c40cb5-704b-4965-a3a3-90f3a3406932)
然后我们用谷歌浏览器访问一下server服务器

![image](https://github.com/ysl1415926/cve/assets/138963581/a2149a04-a8db-41c9-ad62-ec839ce74841)

可以看到，成功访问
