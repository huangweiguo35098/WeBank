# Ubuntu下各种开发环境的搭建
## Docker
### 检查Docker
docker --version
结果：Docker version 20.10.0, build 7287ab3

### 配置Docker国内镜像源
cat /etc/docker/daemon.json
若提示“目录不存在”、“该文件不存在”或“文件内容为空”属于正常现象，则说明未配置过Docker镜像源

### 新建/修改Docker镜像源配置
    mkdir -p /etc/docker #目录不存在

### 创建/修改daemon.json配置文件
    vi /etc/docker/daemon.json

### 配置内容如下：
    {
    "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
    }

### 重新加载配置文件，重启docker服务
    systemctl daemon-reload
    systemctl restart docker.service

### 注意：确保Docker免sudo执行
### 创建docker用户组
    sudo groupadd docker
### 将当前用户添加到docker用户组
    sudo gpasswd -a $USER docker
    newgrp docker 
### 如果还是不能免sudo重启docker服务
    sudo systemctl restart docker
    
## Docker-Compose
### 检查Docker-Compose
    docker-compose --version
    结果：docker-compose version 1.29.2, build 5becea4c

### Docker-Compose安装
    sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose

## Java环境部署
    sudo apt install -y default-jdk
    sudo vi /etc/profile 打开文件在最后添加
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
    export PATH=$PATH:$JAVA_HOME/bin
    保存后再 source  /etc/profile

## 检查mysql
    mysql --version

## Ubuntu安装mysql数据库
    sudo apt install mysql-server
    apt-get install mysql-client
### MySQL修改密码
    alter user 'root'@'localhost' identified with mysql_native_password by '123456'
### MySQL远程连接设置
    vi /etc/mysql/mysql.conf.d/mysqld.cnf 
    将 bind-address            = 127.0.0.1 改为： bind-address            = 0.0.0.0

## PyMySQL部署
    sudo apt-get install -y python3-pip
    sudo pip3 install PyMySQL

## go部署
### 解压go文件
    tar zxvf go*.tar.gz

### 移动go文件目录
    mv go/ /usr/local/

### 编辑环境配置
    sudo vim /etc/profile

### 在文件末尾添加以下内容
    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

### 保存并退出 vim，使用 source 命令使添加的配置信息生效
    source /etc/profile

### 安装完成后建议配置 go 语言使其从公共代理镜像中下载依赖代码
    go env -w GO111MODULE=on
    go env -w GOPROXY=https://goproxy.io,direct

## 搭建node与npm
### nodejs
    curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash
    sudo apt-get install -y nodejs
### npm
    sudo apt-get install npm

## 端口开放
### 查看端口开放情况
    sudo iptables -vnL

### 开放端口
    sudo iptables -I INPUT -p tcp --dport 8250 -j ACCEPT #可将8250改成想要开放的端口
    sudo iptables-save

## 添加或删除SWAP交换分区
### 显示是否使用SWAP
    sudo swapon --show
### 添加SWAP交换分区
    sudo fallocate -l 8G /swapfile
    sudo chmod 600 /swapfile
    sudo mkswap /swapfile
    sudo swapon /swapfile
    # 保证可持久化
        sudo vi /etc/fstab 
         添加行：/swapfile swap swap defaults 0 0
### 删除SWAP交换分区
    sudo swapoff -v /swapfile
    # 在 /etc/fstab 文件中删除有效 swap 的行
    sudo rm /swapfile
