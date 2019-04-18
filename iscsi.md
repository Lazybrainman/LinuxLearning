# iscsi网络磁盘 #
## ##

## 一、在server端 ##

### (防护墙设置为trusted) ###

    [root@server0 ~]# firewall-cmd --set-default-zone=trusted
![](https://i.imgur.com/QDyVQRO.png)

### 1. 创建3G大小的分区 ###
    [root@server0 ~]# lsblk
![](https://i.imgur.com/dnTKMcD.png)

    [root@server0 ~]# fdisk  /dev/vdb  
![](https://i.imgur.com/Pm8MX0l.png)

    [root@server0 ~]# partprobe	


### 2. 安装软件包 ###
     [root@server0 ~]# yum -y install targetcli 
![](https://i.imgur.com/lOhZIUo.png)

### 3. 运行软件 ###
    [root@server0 ~]# targetcli  
![](https://i.imgur.com/h0J2eyX.png)

### 4. 查看磁盘 ###
    /> ls  
![](https://i.imgur.com/ydn4lXl.png)

### 5. 建立后端存储 ###
    /> backstores/block  create iscsi_store  /dev/vdb1  
![](https://i.imgur.com/LKyAAKY.png)

### 6. 建立磁盘阵列 ###
    /> iscsi/ create iqn.2018-06.example.com:server0
![](https://i.imgur.com/QGgApZr.png)

### 7. 配置客户端访问控制名 ###
    /> iscsi/iqn(tab)/tpg1/acls  create  
    iqn.2018-06.example.com:desktop0
![](https://i.imgur.com/ca8a3ln.png)

### 8. 进行lun关联 ###
    /> iscsi/iqn(tab)/tpg1/luns  create /backstores/block/filestores
![](https://i.imgur.com/Qy5YkUO.png)

### 9. 开启服务监听的端口 ###
    /> iscsi/iqn(tab)/tpg1/portals  create  172.25.0.11  3260
![](https://i.imgur.com/cr25gW3.png)

### 10. 最后查看保存并退出 ###
	/> ls  
![](https://i.imgur.com/cgiJtij.png)  

	/> saveconfig  
	/> exit  

![](https://i.imgur.com/ySwcMqn.png)
   
### 11. 重启并开机自启服务 ###
    [root@server0 ~]# systemctl restart target 
    [root@server0 ~]# systemctl enable  target
![](https://i.imgur.com/hXWFy3x.png)

##  ##
## 二、在desktop端 ##

### (防护墙设置为trusted) ###

### 1. 安装软件包 ###
	[root@desktop0 ~]# yum -y install iscsi(tab)  
![](https://i.imgur.com/kZp9Udn.png)

### 2. 指定客户端访问名 ###
	[root@desktop0 ~]#vim /etc/iscsi/inita(tab)   
	InitiatorName=iqn.2018-06.example.com:desktop0  
![](https://i.imgur.com/vdYixVI.png)  
![](https://i.imgur.com/LEJtct0.png)

### 3. 重启iscsi服务刷新名字/开机自启 ##
	[root@desktop0 ~]#systecmtl restart iscsid  
	[root@desktop0 ~]#systemctl enable  iscsid  
![](https://i.imgur.com/io8HBRJ.png)

### 4. 发现磁盘（man  iscsiadm） ###
	[root@desktop0 ~]#iscsiadm --mode discoverydb --type sendtargets --portal 172.25.0.11 --discover
![](https://i.imgur.com/kgY5KiI.png)  
![](https://i.imgur.com/UcvwBvT.png)  
![](https://i.imgur.com/UaskRKj.png)

### 5. 连接磁盘 ###
	[root@desktop0 ~]#iscsiadm -m node  -L all 
![](https://i.imgur.com/l3m8Qul.png)  

### 6. 设置自动连接 ###
	[root@desktop0 ~]#vim /var/lib/iscsi/node(tab)/iqn(tab)...(tab)
	...
	node.conn[0].startup=automatic
	...  

### 7. 重启自启服务 ###
	[root@desktop0 ~]#systemctl restart iscsi
	[root@desktop0 ~]#systemctl enable  iscsi

### 8. 查看多的磁盘 ###
	[root@desktop0 ~]#lsblk  
![](https://i.imgur.com/MVq1z2I.png)
   
### 9. 新建分区 ###
	[root@desktop0 ~]#fdisk   /dev/sda  
![](https://i.imgur.com/SQDcPJ8.png)  
   
### 10. 刷新分区 ###
	[root@desktop0 ~]#partprobe  /dev/sda 

   
### 11. 格式化分区 ###
	[root@desktop0 ~]#mkfs.ext4    /dev/sda1
   
### 12. 创建挂载点 ###
	[root@desktop0 ~]#mkdir  /mnt/data
   
### 13.挂载分区 ###
	[root@desktop0 ~]#vim /etc/fstab  
	...
	/dev/sda1  /mnt/data  ext4    defaults,_netdev  0 0  
![](https://i.imgur.com/wBg3p29.png)


### 14. 全部挂载 ###
	[root@desktop0 ~]#mount -a  
	[root@desktop0 ~]#lsblk  

![](https://i.imgur.com/SdcBlrq.png)
   
### 15.先存盘，再强制重启，避免卡死 ###
	[root@desktop0 ~]#sync ;  reboot -f 



