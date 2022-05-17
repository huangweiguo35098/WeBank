# Ubuntu下各种开发环境的搭建
    #Docker
        #检查Docker
            docker --version
            结果：Docker version 20.10.0, build 7287ab3

        #配置Docker国内镜像源
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

        #注意：确保Docker免sudo执行
            # 创建docker用户组
                sudo groupadd docker
            # 将当前用户添加到docker用户组
                sudo gpasswd -a $USER docker
                newgrp docker 
            # 如果还是不能免sudo重启docker服务
                sudo systemctl restart docker
    #Docker-Compose
        #检查Docker-Compose
            docker-compose --version
            结果：docker-compose version 1.29.2, build 5becea4c

        #Docker-Compose安装
            sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.29.2/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
            sudo chmod +x /usr/local/bin/docker-compose
    #Java环境部署
        sudo apt install -y default-jdk
        sudo vi /etc/profile 打开文件在最后添加
            export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
            export PATH=$PATH:$JAVA_HOME/bin
        保存后再 source  /etc/profile

    #检查mysql
        mysql --version

        #Ubuntu安装mysql数据库
            sudo apt install mysql-server
            apt-get install mysql-client

    #PyMySQL部署
        sudo apt-get install -y python3-pip
        sudo pip3 install PyMySQL
