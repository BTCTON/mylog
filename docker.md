#docker安装

## 原始安装docker

参考https://blog.csdn.net/jeikerxiao/article/details/54632091

curl https://releases.rancher.com/install-docker/18.03.sh | sh

https://www.jianshu.com/p/b3f70bcffe75

	curl https://releases.rancher.com/install-docker/17.12.sh | sh  #最新安装2018-0202
    curl https://releases.rancher.com/install-docker/1.12.sh | sh  #http://52.221.74.151/
	curl https://52.221.74.151/install-docker/1.12.sh 
	sudo apt-mark hold docker-engine # prevent upgrade  sudo apt-mark hold docker-ce
	dpkg -l | grep ^h   #查看哪些软件被Hold
	sudo apt-mark unhold docker-engine
	apt-mark showhold  #另种查看办法
	apt-get -y --allow-downgrades install docker-engine=1.12.6-0~ubuntu-xenial # apt-cache madison docker-engine 直接安装 ubuntu 16注意版本号啊
	curl https://releases.rancher.com/install-docker/17.03.sh | sh   #最新版本

增加源 sudo add-apt-repository "deb http://mirrors.aliyun.com/docker-engine/apt/repo/pool/ $(lsb_release -cs) main"
减少 sudo add-apt-repository --remove "deb http://mirrors.aliyun.com/docker-engine/apt/repo/pool/ $(lsb_release -cs) main"

sudo add-apt-repository --remove "deb https://apt.dockerproject.org/repo/pool/ $(lsb_release -cs) main"

也可以 sudo apt-get install docker-ce=17.12.1~ce-0~ubuntu
sudo apt-get install docker-ce=18.03.0~ce-0~ubuntu

## 国内的安装加速器
```
	sudo vim /lib/systemd/system/docker.service
    echo "DOCKER_OPTS=\"$DOCKER_OPTS --registry-mirror=https://mirror.ccs.tencentyun.com\"" | sudo tee -a /etc/default/docker
    sudo systemctl daemon-reload
    sudo systemctl restart docker

```
## [改变docker文件存储位置](https://forums.docker.com/t/how-do-i-change-the-docker-image-installation-directory/1169)

最简单的办法是在docker执行程序后面加上 “-g /home/docker” 参数。

出现无法删除 device is busy的情况是，用mount命令查看当前挂载的资源情况，然后[umount](http://www.cnblogs.com/xiaouisme/p/4968419.html) 再次删除就可以解决了。
	service docker restart

或者用[老外另外一个办法](https://github.com/moby/moby/issues/9665)
	Then I found out that the consul process was still running:
```
    $ ps aux | grep consul
    root 30879  0.1  2.0 33960432 10160 ?  Ssl  09:14   0:09 /bin/consul agent -config-dir=/config -server -bootstrap
    After killing the process manually I could remove the directory and the container
    
    $ service docker stop
    $ kill 30879
    $ rm -rf /var/lib/docker/aufs/diff/5515181938cf937a979ffd8feefe3cb1e44793840bb59f87be48cf980c9a3076
    $ service docker start
    docker start/running, process 31775
    $ docker ps -a
    CONTAINER IDIMAGE   COMMAND  CREATED STATUS  PORTS  NAMES
    5515181938cfprogrium/consul "/bin/start -server -"   About an hour ago   Dead53/tcp, 0.0.0.0:8300-8302->8300-8302/tcp, 0.0.0.0:8500->8500/tcp, 0.0.0.0:8300-8302->8300-8302/udp, 53/udp, 8400/tcp   nostalgic_darwin
    $ docker rm -f 5515181938cf
    5515181938cf
    $ docker ps -a
    CONTAINER IDIMAGE   COMMAND CREATED STATUS   
    
    ```
