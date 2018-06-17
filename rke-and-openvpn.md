
# rke安装


参考 https://blog.csdn.net/CSDN_duomaomao/article/details/79317846

# openvpn安装

## 如何解决docker

[基于kubernetes的openvpn的部署配置](http://eco.hand-china.com/community/t/topic/125/2)

[解决docker-compose 和 openvpn共存](https://www.aityp.com/%e8%a7%a3%e5%86%b3docker-compose-%e5%92%8c-openvpn%e5%85%b1%e5%ad%98/)

### [安装vpn](https://www.wilean.com/archives/382)

[如何在Ubuntu 16.04上设置一套OpenVPN服务器](https://blog.csdn.net/zstack_org/article/details/69228386)

[官方-上篇的原文](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04)

***我们一定要按英文原版操作！

[windows端openvpn设置]（https://blog.csdn.net/xiaohai7521s/article/details/77684893）

### 解决无法上网的问题

增加

···

dhcp-option DNS 223.5.5.5
dhcp-option DNS 223.6.6.6

···

### 固定ip的问题

https://openvpn.net/index.php/open-source/documentation/manuals/65-openvpn-20x-manpage.html

https://openvpn.net/index.php/component/content/article/65-open-source/general/89-2xhowto.html

http://dnaeon.github.io/static-ip-addresses-in-openvpn/

[阿里云openvpn](https://yq.aliyun.com/ziliao/65785)

[最专业的了](https://blog.csdn.net/cai742925624/article/details/45483571
）
sudo nano /etc/openvpn/server.conf   

sudo nano /etc/openvpn/ipp.txt
sudo chown -R nobody:nogroup /etc/openvpn/ccd

cd ~/openvpn-ca

source vars

./build-key conohajp







