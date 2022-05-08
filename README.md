#Ubuntu-WeBASE-DATAEXPORT

#通过Webase一键部署实现在Ubuntu上的Webase的搭建

    #Java环境部署
    sudo apt install -y default-jdk
    若是在调用python3 deploy.py installAll启动服务时报错 java_home相关可以通过：
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

    #获取部署安装包及解压
    wget https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/WeBASE/releases/download/v1.5.4/webase-deploy.zip
    unzip webase-deploy.zip

    #修改配置
    cd webase-deploy
    vi common.properties
        docker.mysql=0
        将数据库port，user，password改成mysql对应的port，user，password
        if.exist.fisco=yes
        fisco.dir= /home/ubuntu/fisco/nodes/127.0.0.1/  本地fisco包含sdk的文件夹的绝对地址

    # 部署并启动所有服务
    python3 deploy.py installAll
    启动成功后以ip：5000访问webase 默认账号admin 默认密码Abcd1234
    可能遇到验证码无法出现，以及系统错误的提示，可以通过：
        netstat -anlp | grep 5001 检查是否webase-node-mgr未能正常启动
        cd webase-deploy/webase-node-mgr/conf
        vi application.yml
        将# database connection configuration下的
            url改为jdbc:mysql://localhost:3306/webasenodemanager?autoReconnect=true&useSSL=false&serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8
        保存文件后回到webase-deploy/webase-node-mgr 通过bash start.sh 启动服务

#通过服务方式启动实现在Ubuntu上的Data-Export的搭建   

    #拉取代码
    git clone https://github.com/WeBankBlockchain/Data-Export.git 

    #进入安装路径
    cd Data-Export/tools
    
    #配置文件设置
    vi config/application.properties

        # Channel方式启动，与java sdk一致，需配置证书
        ## GROUP_ID必须与FISCO-BCOS中配置的groupId一致, 多群组以,分隔，如1,2
        system.groupId=1 
        ##IP为节点运行的IP，PORT为节点运行的channel_port，默认为20200
        system.nodeStr=127.0.0.1:20200
        system.certPath=/home/ubuntu/fisco/nodes/127.0.0.1/sdk 

        ### 数据库的信息，暂时只支持mysql； serverTimezone 用来设置时区
        ### 请确保在运行前创建对应的database，如果分库分表，则可配置多个数据源，如system.db1.dbUrl=\system.db1.user=\system.db0.password=
        system.db0.dbUrl=jdbc:mysql://localhost:3306/data_export?useSSL=false&serverTimezone=Asia/Shanghai&useUnicode=true&characterEncoding=UTF-8
        system.db0.user=root
        system.db0.password=123456

        ### 是否生成grafana可调用的json文件
        system.grafanaEnable=true
    
    #创建数据库
    通过navicat连接数据库并创建url中/localhost:3306/后的对应数据库data_export

    #运行程序
    chmod +x start.sh
    bash start.sh

    #可视化配置
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

    通过import导入json文件，理论上直接改变docker grafana里的配置使其默认启动，或是不通过docker，直接使用grafana会更好，
    但还没研究清楚。 
    将json文件中“panels”里的所有“datasource”后内容改为在grafana中数据源的名字
    如：想要访问的grafana数据源名为Mysql 就令“datasource”：“Mysql”
    默认情况下数据源名为MySQL，json中默认为“datasource”：“MySQL”








