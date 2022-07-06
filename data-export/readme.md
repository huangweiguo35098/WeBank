#Docker 一键部署 Data-Export
##拉取代码，并解压文件
    curl -#LO https://github.com/WeBankBlockchain/Data-Export/releases/download/V1.7.2/data-export-1.7.2.tar.gz
    tar -zxvf data-export-1.7.2.tar.gz && cd data-export-docker && chmod -x build_export.sh

##进入安装路径
    cd data-export-docker

    # config为配置文件目录，使用channel方式连接区块链时，需将证书放至该目录。
    # config包括了abi和bin两个文件夹，用于配置合约信息。
    # 运行生成的sql脚本data_export.sql和可视化脚本default_dashboard.json会保存在config目录下。
    # 运行日志保存在log目录下。
    # data目录为docker安装mysql和es后的数据挂载目录。

##配置证书（channel方式启动）
    cp -r ~/fisco/nodes/0.0.0.0/sdk/* ./config/

##配置文件
    配置文件application.properties位于config目录下
    修改application.properties文件：该文件包含了所有的配置信息。以下配置信息是必须要配置的：
    
    vim config/application.properties

    # Channel方式启动，与java sdk一致，需配置证书
    ## GROUP_ID必须与FISCO-BCOS中配置的groupId一致, 多群组以,分隔，如1,2
    system.groupId=1 
    ##IP为节点运行的IP，PORT为节点运行的channel_port，默认为20200
    system.nodeStr=127.0.0.1:20200
    system.certPath=/home/ubuntu/fisco/nodes/0.0.0.0/sdk 

    ### 数据库的信息，暂时只支持mysql； serverTimezone 用来设置时区
    ### 请确保在运行前创建对应的database，如果分库分表，则可配置多个数据源，如system.db1.dbUrl=\system.db1.user=\system.db0.password=
    system.db0.dbUrl=jdbc:mysql://localhost:3306/data_export?useSSL=false&serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8
    system.db0.user=root
    system.db0.password=123456

    #################################### Export Contract Info Config ####################################
    ###合约存放位置，理论上应该选择webase上部署合约上所生成合约的位置，但没找到，只好将对应合约存于solidity文件夹下
    system.solPath=./config/solidity
    #The options available include: 0.4.25.1;0.5.2.0;0.6.10.0
    system.solcVersion=0.4.25.1

    ### 是否生成grafana可调用的json文件
    system.grafanaEnable=true

##创建数据库
    通过navicat连接数据库并创建url中/localhost:3306/后的对应数据库data_export

##运行程序
    bash build_export.sh

##可视化配置
    sudo docker pull grafana/grafana
    docker run   -d   -p 3000:3000   --name=grafana   -e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource"   grafana/grafana
    grafana安装并启动成功，通过访问[ip]:3000，输入账密admin/admin
    在Configuration下的Data Source里进行数据库连接配置
    Host： ip:3306
    Database ：data_export
    User：root  Password：123456
    最后点击Save&test 出现 Database Connection OK表示连接成功
    出现 db query error: query failed - please inspect Grafana server log for details 可以检查是否mysql不支持远程连接，通过：
    sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf 打开mysqld.cnf文件
    将 bind-address            = 127.0.0.1 改为 bind-address            = 0.0.0.0 保存退出。
    service mysql restart 重启mysql

    通过navicat连接数据库进入mysql数据库，打开user表，将root行的host从localhost改为%，并通过flush privileges; #更新权限 

    通过import导入json文件，理论上直接改变docker grafana里的配置使其默认启动，但还没研究清楚。 
    将json文件中“panels”里的所有“datasource”后内容改为在grafana中数据源的名字
    如：想要访问的grafana数据源名为Mysql 就令“datasource”：“Mysql”
    默认情况下数据源名为MySQL，json中默认为“datasource”：“MySQL”
#Docker 部署 fabric-explorer
用于输出fabric链的数据
##下载blockchain-explorer
    git clone https://gitee.com/qicong-hu/blockchain-explorer.git
##复制test-network的相关配置文件到explorer目录下
    cd blockchain-explorer/examples/net1/crypto
    cp -r /home/ubuntu/fabric/fabric-samples-2.3.0/test-network/organizations/ordererOrganizations ./
    cp -r /home/ubuntu/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations ./
##修改docker-compose.yaml文件
###打开docker-compose.yaml文件
    cd ~/blockchain-explorer
    sudo vim docker-compose.yaml
        networks:
            mynetwork.com:
                external:
                    name: docker_test
##修改first-network.json
###查看私钥文件的文件名
    cd /home/ubuntu/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore
    ls
    我的结果：priv_sk
###修改first-network.json
    cd ~/blockchain-explorer/examples/net1/connection-profile
    vim first-network.json
    #修改adminPrivateKey，将priv_sk替换为我们上面查到的私钥名,我的已经是priv_sk
###查看用户密码
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
