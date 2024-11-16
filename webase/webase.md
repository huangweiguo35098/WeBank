# Ubuntu-WeBASE

## 通过Webase一键Docker部署实现在Ubuntu上的Webase的搭建

### 检查Docker
    docker --version
    结果：Docker version 20.10.0, build 7287ab3

### 配置Docker国内镜像源
    cat /etc/docker/daemon.json
    若提示“目录不存在”、“该文件不存在”或“文件内容为空”属于正常现象，则说明未配置过Docker镜像源

    #新建/修改Docker镜像源配置
        mkdir -p /etc/docker #目录不存在

    # 创建/修改daemon.json配置文件
        vi /etc/docker/daemon.json

    # 配置内容如下：
        {
        "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn"]
        }

    #重新加载配置文件，重启docker服务
        systemctl daemon-reload
        systemctl restart docker.service

### 检查Docker-Compose
    docker-compose --version
    结果：docker-compose version 1.29.2, build 5becea4c

### Docker-Compose安装
    sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    sudo chmod +x /usr/local/bin/docker-compose

### 注意：确保Docker免sudo执行
    # 创建docker用户组
        sudo groupadd docker
    # 将当前用户添加到docker用户组
        sudo gpasswd -a $USER docker
        newgrp docker 
    # 如果还是不能免sudo重启docker服务
        sudo systemctl restart docker

### Ubuntu安装mysql数据库
### PyMySQL部署

    与Webase一键部署相同


### 获取部署安装包及解压
    wget https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/WeBASE/releases/download/v1.5.4/webase-deploy.zip
    unzip webase-deploy.zip

### 修改配置
    cd webase-deploy
    vi common.properties

        与Webase一键部署相同

### 拉取镜像
    python3 deploy.py pullDockerAll

### 部署并启动所有服务
    python3 deploy.py installDockerAll
    启动成功后以ip：5000访问webase 默认账号admin 默认密码Abcd1234

### 服务部署后，需要对各服务进行启停操作，可以使用以下命令：
#### 一键部署
    部署并启动所有服务        python3 deploy.py installDockerAll
    停止一键部署的所有服务    python3 deploy.py stopDockerAll
    启动一键部署的所有服务    python3 deploy.py startDockerAll
#### 节点的启停
    启动所有FISCO-BCOS节点:      python3 deploy.py startNode
    停止所有FISCO-BCOS节点:      python3 deploy.py stopNode
#### WeBASE服务的启停
    启动所有WeBASE服务:      python3 deploy.py dockerStart
    停止所有WeBASE服务:      python3 deploy.py dockerStop



## 通过Webase一键部署实现在Ubuntu上的Webase的搭建

### Java环境部署
    sudo apt install -y default-jdk
    若是在调用python3 deploy.py installAll启动服务时报错 java_home相关可以通过：
    sudo vi /etc/profile 打开文件在最后添加
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
    export PATH=$PATH:$JAVA_HOME/bin
    保存后再 source  /etc/profile

### 检查mysql
    mysql --version

#### Ubuntu安装mysql数据库
    sudo apt install mysql-server
    apt-get install mysql-client

#### PyMySQL部署
    sudo apt-get install -y python3-pip
    sudo pip3 install PyMySQL

#### 获取部署安装包及解压
    wget https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/WeBASE/releases/download/v1.5.4/webase-deploy.zip
    unzip webase-deploy.zip

#### 修改配置
    cd webase-deploy
    vi common.properties
        docker.mysql=0
        #将数据库port，user，password改成mysql对应的port，user，password
        # Mysql database configuration of WeBASE-Node-Manager
        mysql.ip=localhost
        mysql.port=3306
        mysql.user=root
        mysql.password=123456
        mysql.database=webasenodemanager

        # Mysql database configuration of WeBASE-Sign
        sign.mysql.ip=localhost
        sign.mysql.port=3306
        sign.mysql.user=root
        sign.mysql.password=123456
        sign.mysql.database=webasesign
        if.exist.fisco=yes
        fisco.dir= /home/ubuntu/fisco/nodes/127.0.0.1/  本地fisco包含sdk的文件夹的绝对地址

## 部署并启动所有服务
    python3 deploy.py installAll
    启动成功后以ip：5000访问webase 默认账号admin 默认密码Abcd1234
    可能遇到验证码无法出现，以及系统错误的提示，可以通过：
        netstat -anlp | grep 5001 检查是否webase-node-mgr未能正常启动
        cd webase-deploy/webase-node-mgr/conf
        vi application.yml
        将# database connection configuration  下的
            url改为jdbc:mysql://localhost:3306/webasenodemanager?autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8
        保存文件后回到webase-deploy/webase-node-mgr 通过bash start.sh 启动服务
