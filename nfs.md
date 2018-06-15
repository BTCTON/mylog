http://www.chinacion.cn/article/3607.html

# Kubernetes存储系统-NFS Server的Helm部署

在应用商店安装，安装完成后会在存储类出现名称为Nfs.

https://k8w.io/post/54  全系类
[参考一centos下](http://www.cnblogs.com/yinshoucheng-golden/p/6318191.html)

[生产环境下使用nfs](http://blog.csdn.net/weiyuefei/article/details/52130608)

[官方nfs安装指南](http://nfs.sourceforge.net/nfs-howto/ar01s03.html)

[DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-16-04)

[ubuntu官方nfs安装](https://help.ubuntu.com/community/SettingUpNFSHowTo)


## 重点：

1、no_root_squash 参数

访问NFS Server共享目录的用户如果是root的话，它对该共享目录具有root权限。这个配置原本为无盘客户端准备的。用户应避免使用。

2、all_squash

不管访问NFS Server共享目录的用户身份如何，它的权限都被压缩成匿名用户，同时它的UID和GID都会变成nfsnobody账号身份。在早期多个NFS客户端同时读写NFS Server数据时，这个参数很有用。

3、其他参数

rw  read-write，表示可读写权限*

ro  read-only，表示只读权限

sync 请求或写入数据时，数据同步写入到NFS Server的硬盘后才返回。数据安全不会丢，缺点，性能下降。

async 请求或写入数据是，先返回请求，再将数据写入到内存缓存和硬盘中，即异步写入数据。此参数可以提升NFS性能，但是会降低数据的安全。因此，一般情况下建议不用，如果NFS处于瓶颈状态，并且运行数据丢失的话可以打开此参数提升NFS性能。写入时数据会先写到内存缓冲区，等硬盘有空档再写入磁盘，这样可以提升写入效率，风险若服务器宕机或不正常关机，会损失缓冲区中未写入磁盘的数据（解决办法：服务器主板电池或加UPS不间断电源）。（电商秒杀是异步）

4、NFS高并发环境下的服务端重要优化(mount -o 参数）

a.async 异步同步，此参数会提高I/O性能，但会降低数据安全（除非对性能要求很高，对数据可靠性不要求的场合。一般生产环境，不推荐使用）；sync：资料同步写入内存和硬盘

no_subtree_check：不检查父目录的权限。

no_root_squash：root用户具有对根目录的完全管理访问权限。

b.noatime 取消更新文件系统上的inode访问时间,提升I/O性能，优化I/O目的，推荐使用。

c.nodiratime 取消更新文件系统上的directory inode访问时间，高并发环境，推荐显式应用该选项，提高系统性能

d.noexec  挂载的这个文件系统，要不要执行程序（安全选项）

e.nosuid  挂载的这个文件系统上面，可不可以设置UID（安全选项）

f.rsize/wsize 读取（rsize）/写入（wsize）的区块大小（block size），这个设置值可以影响客户端与服务端传输数据的缓冲存储量。一般来说，如果在局域网内，并且客户端与服务端都具有足够的内存，这个值可以设置大一点，比如说32768（bytes）,提升缓冲区块将可提升NFS文件系统的传输能力。但设置的值也不要太大，最好是实现网络能够传输的最大值为限。

## 具体过程

主机端

```
sudo apt-get update
sudo apt-get install nfs-kernel-server
```

客户端（如果安装了主机端就没必要再安装nfs-common了，已经包括了）

```
sudo apt-get update
sudo apt-get install nfs-common
```

主机端先搞一个一般共享的目录


    sudo mkdir /sdb1/nfs -p
    ls -la /sdb1/nfs
	sudo mkdir /sdb1/nfshome -p
    
    sudo chown nobody:nogroup /sdb1/nfs #/sdb1/nfshome 暂时不要考虑修改权限
    # ubuntu 14.04的主机
	sudo mkdir /home/general -p
	sudo mkdir /home/nfs -p
	sudo chown nobody:nogroup /home/general
    
    sudo vim /etc/exports
	# 第一个主机资源，第一个/home/general开放给rancher-nfs
    /home/general	103.84.90.178(rw,sync,no_root_squash,no_subtree_check)	52.77.7.154(rw,async,no_subtree_check)	119.29.185.219(rw,async,no_subtree_check)	118.89.190.210(rw,async,no_subtree_check)	211.159.165.42(rw,async,no_subtree_check)  
    /home/nfs   52.77.7.154(rw,no_root_squash,sync,no_subtree_check)	119.29.185.219(rw,sync,no_root_squash,no_subtree_check)	118.89.190.210(rw,sync,no_root_squash,no_subtree_check)	211.159.165.42(rw,async,no_root_squash,no_subtree_check)
	# 第二个主机资源,全部后台操作
	/sdb1/nfs   103.84.90.134(rw,async,no_subtree_check)  119.29.185.219(rw,async,no_subtree_check)   118.89.190.210(rw,async,no_subtree_check)  
    /sdb1/nfshome   103.84.90.134(rw,no_root_squash,sync,no_subtree_check)	119.29.185.219(rw,sync,no_root_squash,no_subtree_check)	118.89.190.210(rw,sync,no_root_squash,no_subtree_check)
	# BJ主机资源
    /sdb1/nfs   103.89.137.109(rw,async,no_subtree_check)	103.84.90.134(rw,async,no_subtree_check)  119.29.185.219(rw,async,no_subtree_check)   118.89.190.210(rw,async,no_subtree_check)
    /sdb1/nfshome   103.89.137.109(rw,no_root_squash,sync,no_subtree_check)	103.84.90.134(rw,no_root_squash,sync,no_subtree_check)  119.29.185.219(rw,sync,no_root_squash,no_subtree_check) 118.89.190.210(rw,sync,no_root_squash,no_subtree_check)
	/var/balldogory-registry	52.77.7.154(rw,all_squash,async)    

	# 影子主机的从机-新加坡主机设置
	/sdb1/nfs   103.89.137.109(rw,async,no_subtree_check)
	/sdb1/nfshome   103.89.137.109(rw,no_root_squash,sync,no_subtree_check)

    # HK1主机资源，就是为了备份，做其他数据源的备份，提供给老大
    /home/general   119.29.185.219(rw,async,no_subtree_check)   118.89.190.210(rw,async,no_subtree_check)   103.84.90.134(rw,async,no_subtree_check)  
    /home/nfs   119.29.185.219(rw,sync,no_root_squash,no_subtree_check) 118.89.190.210(rw,sync,no_root_squash,no_subtree_check) 103.84.90.134(rw,async,no_root_squash,no_subtree_check)


查看实际使用情况

du -sh /nfs/general

重启nfs服务器

	.84.90.	#ubuntu14.04

	sudo systemctl restart nfs-kernel-server
	sudo /etc/init.d/nfs-kernel-server restart  #ubuntu14.04需要完整目录

可以看到如果no_root_squash参数的话，建立的文件是root，而general的文件拥有者是nobody。

查看客户端挂载的参数：

    grep mnt /proc/mounts

最后优化，[参考](http://note.youdao.com/noteshare?id=7508592d02306bea6e991e646f80c00a)


    vim /etc/hosts
    
    127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
    192.168.200.132 web1.daphne.com web1
    192.168.200.133 web2.daphne.com web2

## 客户端链接
    sudo mkdir -p /nfs/general
	sudo mkdir -p /nfs/home
    sudo mkdir -p /nfs/general2
	sudo mkdir -p /nfs/home2
    sudo mkdir -p /nfs/generalhk1
	sudo mkdir -p /nfs/homehk1

### 测试是否能挂载

	sudo mount -t nfs -o noatime,nodiratime 103.84.90.134:/home/general /nfs/general
	sudo mount -t nfs -o noatime,nodiratime 103.89.137.109:/home/general /nfs/generalhk1  #测试备份
	sudo mount -t nfs -o noatime,nodiratime 52.77.7.142:/sdb1/nfs /nfs/general3
    sudo mount -t nfs -o noatime,nodiratime 211.159.165.42:/sdb1/nfs /nfs/general2
	sudo mount -t nfs -o noatime,nodiratime 211.159.165.42:/var/balldogory-registry /mnt  #临时挂载
	sudo mount -t nfs -o noatime,nodiratime 103.89.137.109:/home/nfs /nfs/homehk1  #测试备份

### 检查与持久化
	grep mnt /proc/mounts #查看挂载别的数据盘情况
	df -h
要持久化，则修改fstab,没有比这家[美国主机商](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-16-04)说的更清楚的了。

	sudo vim /etc/fstab # 别忘记用mount -a 检测。
	# 增加内容
    103.84.90.134:/home/general	/nfs/general   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    103.84.90.134:/home/nfs   /nfs/home  nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
	#可以挂多台机器
	211.159.165.42:/sdb1/nfs	/nfs/general2   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    211.159.165.42:/sdb1/nfshome   /nfs/home2  nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
	# 影子主机需要增加内容
	211.159.165.42:/sdb1/nfs	/nfs/general2   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    211.159.165.42:/sdb1/nfshome   /nfs/home2  nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
	52.77.7.142:/sdb1/nfs	/nfs/general3   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    52.77.7.142:/sdb1/nfshome   /nfs/home3  nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
	# 大陆主机还需增加内容-备用
    103.89.137.109:/home/general /nfs/generalhk1   nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0
    103.89.137.109:/home/nfs   /nfs/homehk1  nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0

## 查看mount历史
	history |grep mount

# [故障修复](http://blog.csdn.net/weiyuefei/article/details/52130608)

# [防火墙安全设置](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-ubuntu-16-04)


# 最后的大餐 来回nfs

## 设置权限和存储位置

	sudo mkdir /nfs/nfshome-bj

## 测试挂载

    sudo mount -t nfs -o noatime,nodiratime 211.159.165.42:/sdb1/nfshome /nfs/nfshome-bj
    
    mount -t nfs -o noatime,nodiratime，rsize=131072,wsize=131072 211.159.165.42:/sdb1/nfshome /nfs/nfshome-bj

内核需要优化才可以用后面两个参数rsize=131072,wsize=131072，或者nfs4系统会自动设置。

grep nfs /proc/mounts  #查看目前mount参数

    211.159.165.42:/sdb1/nfs /nfs/general2 nfs4 rw,noatime,vers=4.0,rsize=524288,wsize=524288,namlen=255,acregmin=1800,acregmax=1800,acdirmin=1800,acdirmax=1800,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=103.84.90.134,local_lock=none,addr=211.159.165.42 0 0`

    none /proc/xen xenfs rw,relatime 0 0
    nfsd /proc/fs/nfsd nfsd rw,relatime 0 0

    103.84.90.134:/home/general/gogs-data /var/lib/rancher/volumes/rancher-nfs/gogs-data nfs4 rw,relatime,vers=4.0,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=103.84.90.134,local_lock=none,addr=103.84.90.134 0 0

    211.159.165.42:/sdb1/nfshome /nfs/nfshome-bj nfs4 rw,noatime,nodiratime,vers=4.0,rsize=524288,wsize=524288,namlen=255,hard,proto=tcp,port=0,timeo=600,retrans=2,sec=sys,clientaddr=103.84.90.134,local_lock=none,addr=211.159.165.42 0 0
    

# 最后进入实用阶段，[cp或者rsync命令对比参考这里](https://serverfault.com/questions/268369/why-rsync-is-faster-than-nfs)

## 解决安装registry时certs文件多主机共享共有的问题

第一步，倒到中间库HK
    
    sudo cp /nfs/nfshome-bj/balldog /home/general/balldogory-registry -r  #这个文件有200M
倒转过来就好了。然后在sg执行

	sudo cp -r /nfs/general/balldogory-registry/certs /var/balldogory-registry

实际上也可以直接在hk挂在BJ,直接倒转
	sudo mount -t nfs -o noatime,nodiratime 211.159.165.42:/var/balldogory-registry /mnt  #临时挂载



