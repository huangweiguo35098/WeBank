#Docker 部署 fabric-explorer
用于输出fabric链的数据
##下载blockchain-explorer
    git clone https://gitee.com/qicong-hu/blockchain-explorer.git
##配置fabric2.0

###复制test-network的相关配置文件到explorer目录下
    cd blockchain-explorer/examples/net1/crypto
    cp -r /home/ubuntu/fabric/fabric-samples-2.3.0/test-network/organizations/ordererOrganizations ./
    cp -r /home/ubuntu/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations ./

###修改docker-compose.yaml文件

####打开docker-compose.yaml文件
    cd ~/blockchain-explorer
    sudo vim docker-compose.yaml
        networks:
            mynetwork.com:
                external:
                    name: docker_test

###修改first-network.json

####查看私钥文件的文件名
    cd /home/ubuntu/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore
    ls
    我的结果：priv_sk

####修改first-network.json
    cd ~/blockchain-explorer/examples/net1/connection-profile
    vim first-network.json
    #修改adminPrivateKey，将priv_sk替换为我们上面查到的私钥名,我的已经是priv_sk

####查看用户密码
    first-network.json中的adminCredential即浏览器的用户名和密码

##启动fabric浏览器
    cd ~/blockchain-explorer
    docker-compose up -d
    执行完毕后，通过火狐浏览器访问localhost:8080，即可打开blockchain-explorer

##关闭fabric explorer

###关闭（不删除数据）
    docker-compose down

###彻底删除
    docker-compose down -v

##常见错误

###fabric explorer数据库无法外部连接或是在本地postgresql中无法找到对应数据库
    fabric explorer会在容器中搭建postgresql，默认情况下不会映射到外部端口，需要手动设置
    打开docker-compose.yaml文件，在explorerdb.mynetwork.com中修改：
    explorerdb.mynetwork.com:
        ports:
          - 5432:5432
          
###fabric explorer服务无法启动，提示npm权限拒绝之类的
    确定priv_sk是否替换，建议直接复制名字而非使用*_sk
    还不行可以试着使用unsafe：npm 出于安全考虑不支持以 root 用户运行，即使你用 root 用户身份运行了，npm 会自动转成一个叫 nobody 的用户来运行
    权限不足报错时使用npm的姿势：
    sudo + 命令 + --unsafe-perm