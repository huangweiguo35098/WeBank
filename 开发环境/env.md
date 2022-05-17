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

