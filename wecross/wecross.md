#在ubuntu下搭建wecross

    利用wecross可以使得使用者在不同链之间进行数据交流，具体表现为：用户A可以通过部署在本地的跨链路由向用户B所部署的跨链路由发起
    调用数据的请求，其中桥接合约用来将调用请求添加到任务列表，跨链路由会获取该请求并通过路由之间进行数据交换，代理合约则用来调用
    目标合约。
    简单来说对于一条部署在本地的链想要访问其他服务器上的链，只需部署一个跨链路由并将其接入，就可以与其他的跨链路由下的
    链进行数据交流了（个人理解）。需要注意的是跨链路由是通过p2p进行数据交流的，并且只会在具有相同根证书的路由之间进行数据交流。
    
###创建手动组网的操作目录
        mkdir -p ~/wecross-networks && cd ~/wecross-networks
    #下载WeCross
        bash <(curl -sL https://gitee.com/WeBank/WeCross/raw/master/scripts/download_wecross.sh)
    #部署跨链路由
        cd ~/wecross-networks
        vim ipfile
        # 在文件中键入以下内容
        ###ip:rpc端口:p2p端口，一条数据表示生成一个跨链路由###
        127.0.0.1:8250:25500  ###127.0.0.1:8251:25501###
    #生成好ipfile文件后，使用脚本build_wecross.sh生成跨链路由
        bash ./WeCross/build_wecross.sh -n Kt3 -o routers-test1 -f ipfile -c ~/ca

        -n 指定跨链分区标识符(zone id)，跨链分区通过zone id进行区分，可以理解为业务名称。
        -o 指定输出的目录，并在该目录下生成一个跨链路由。
        -f 指定需要生成的WeCross跨链路由的列表，包括ip地址，rpc端口，p2p端口，生成后的router已完成互联配置
        -c 指定生成路由所需的根证书位置
    #接入FISCO BCOS 2.0
        #生成配置框架
            cd ~/wecross-networks/routers-test1/127.0.0.1-8250-25500 ###跨链路由位置
            # -t 链类型，-n 指定链名字，可根据-h查看使用说明
            #### 确保同一跨链分区下不同跨链路由中链名字应当不同，否则无法取得连接的路由下的资源 ####
            bash add_chain.sh -t BCOS2.0 -n bcos-test1
        #拷贝证书
            # 证书目录以实际情况为准
            cp -r ~/fisco/nodes/0.0.0.0/sdk/*   ~/wecross-networks/routers-test1/127.0.0.1-8250-25500/conf/chains/bcos-test1/
        #编辑配置文件
            vi conf/chains/bcos-test1/stub.toml
            [common]                # 通用配置
                name = 'bcos'       # stub配置名称，即 [stubName] = bcos
                type = 'BCOS2.0'    # stub类型，`GM_BCOS2.0`或者`BCOS2.0`，`GM_BCOS2.0`国密类型，`BCOS2.0`非国密类型

            [chain]                 # FISCO-BCOS 链配置
                groupId = 1         # 连接FISCO-BCOS群组id，默认为1
                chainId = 1         # 连接FISCO-BCOS链id，默认为1

            [channelService]        # FISCO-BCOS 配置，以下文件在BCOS链的nodes/127.0.0.1/sdk目录下拷贝
                caCert = 'ca.crt'   # 根证书
                sslCert = 'sdk.crt' # SDK证书
                sslKey = 'sdk.key'  # SDK私钥
                gmConnect = false   # 国密连接开关，若为true则使用BCOS节点SDK的国密证书进行连接，反之则使用非国密证书连接
                gmCaCert = 'gm/gmca.crt'        # 国密CA证书
                gmSslCert = 'gm/gmsdk.crt'      # 国密SDK证书
                gmSslKey = 'gm/gmsdk.key'       # 国密SDK密钥
                gmEnSslCert = 'gm/gmensdk.crt'  # 国密加密证书
                gmEnSslKey = 'gm/gmensdk.key'   # 国密加密密钥
                timeout = 5000                  # SDK请求超时时间
                connectionsStr = ['127.0.0.1:20200']    # 连接列表
        #部署系统合约
            #非国密链
                # 部署代理合约
                bash deploy_system_contract.sh -t BCOS2.0 -c chains/bcos-test1 -P

                # 部署桥接合约
                bash deploy_system_contract.sh -t BCOS2.0 -c chains/bcos-test1 -H
            #国密链
                # 部署代理合约
                bash deploy_system_contract.sh -t GM_BCOS2.0 -c chains/bcos-test1 -P

                # 部署桥接合约
                bash deploy_system_contract.sh -t GM_BCOS2.0 -c chains/bcos-test1 -H


    #接入Hyperledger Fabric 2
        #搭建区块链,基于wecross提供的demo快速搭建
            #检查go,docker,docker Compose
                go version
            #安装go
                # 解压go文件
                tar zxvf go*.tar.gz
                
                # 移动go文件目录
                mv go/ /usr/local/
                
                # 编辑环境配置
                sudo vim /etc/profile

                在文件末尾添加以下内容

                export GOROOT=/usr/local/go
                export GOPATH=$HOME/go
                export PATH=$PATH:$GOROOT/bin:$GOPATH/bin

                保存并退出 vim，使用 source 命令使添加的配置信息生效

                source /etc/profile
                
                #安装完成后建议配置 go 语言使其从公共代理镜像中下载依赖代码
                go env -w GO111MODULE=on
                go env -w GOPROXY=https://goproxy.io,direct

            mkdir -p ~/fabric && cd ~/fabric
            bash <(curl -sL https://gitee.com/WeBank/WeCross/raw/master/scripts/download_demo.sh)
            cp ./wecross-demo/fabric2/* ./
            bash build.sh
            
            #搭建成功，查看Fabric链各个容器运行状态。
                docker ps
        # 配置内置账户
            #生成账户配置框架
                # 切换至对应跨链路由的主目录
                    cd ~/wecross-networks/routers-test1/127.0.0.1-8251-25501/

                # 用脚本生成Fabric账户配置：账户类型（Fabric1.4），账户名（fabric_admin）
                # 接入Fabric链，需要配置一个admin账户
                    bash add_account.sh -t Fabric2.0 -n fabric2_admin 

                # 为Fabric链的每个Org都配置一个admin账户，此处有两个org（Org1和Org2），分别配两个账户

                # 配Org1的admin
                    bash add_account.sh -t Fabric2.0 -n fabric2_admin_org1

                # 配Org2的admin
                    bash add_account.sh -t Fabric2.0 -n fabric2_admin_org2
            #完成配置
                #修改账户配置
                    vim conf/accounts/fabric2_admin_org2/account.toml
                    # 修改mspid，将 'Org1MSP' 更改为 'Org2MSP'
                #拷贝证书文件
                    #Fabric链的证书位于organizations目录，请参考以下命令并根据实际情况完成相关证书的拷贝

                    cd ~/wecross-networks/routers-payment/127.0.0.1-8251-25501
                    # 配置fabric_admin 
                        # 拷贝私钥
                            cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/*_sk conf/accounts/fabric2_admin/account.key
                        # 拷贝证书
                            cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem  conf/accounts/fabric2_admin/account.crt

                    # 配置fabric_admin_org1 
                        # 拷贝私钥
                            cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/keystore/*_sk  conf/accounts/fabric2_admin_org1/account.key
                        # 拷贝证书
                            cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp/signcerts/Admin@org1.example.com-cert.pem   conf/accounts/fabric2_admin_org1/account.crt

                    # 配置fabric_admin_org2 
                        # 拷贝私钥
                            cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/keystore/*_sk conf/accounts/fabric2_admin_org2/account.key
                        # 拷贝证书
                            cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp/signcerts/Admin@org2.example.com-cert.pem conf/accounts/fabric2_admin_org2/account.crt
        #配置插件
            #生成插件配置框架
                进入跨链路由的主目录，用add_chain.sh脚本在conf目录下生成Fabric链的配置框架。

                cd ~/wecross-networks/routers-payment/127.0.0.1-8251-25501

                # -t 链类型，-n 指定链名字，可根据-h查看使用说明
                bash add_chain.sh -t Fabric2.0 -n fabric2
            #完成配置
                #拷贝证书
                    # 拷贝orderer证书
                        cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem conf/chains/fabric2/orderer-tlsca.crt
                    #拷贝org1证书
                        cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt conf/chains/fabric2/org1-tlsca.crt
                    #拷贝org2证书
                        cp ~/fabric/fabric-samples-2.3.0/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt conf/chains/fabric2/org2-tlsca.crt
                #编辑配置文件
                    #配置文件stub.toml配置项
        #部署系统合约
            # 部署代理合约
                bash deploy_system_contract.sh -t Fabric2.0 -c chains/fabric2 -P

            # 部署桥接合约
                bash deploy_system_contract.sh -t Fabric2.0 -c chains/fabric2 -H


    #部署账户服务
        #下载
            cd ~/wecross-networks
            bash <(curl -sL https://gitee.com/WeBank/WeCross/raw/master/scripts/download_account_manager.sh)
        #拷贝证书
            cd ~/wecross-networks/WeCross-Account-Manager/
            cp ~/wecross-networks/routers-test1/cert/sdk/* conf/
        #生成私钥
            bash create_rsa_keypair.sh -d conf/
        #配置
            cp conf/application-sample.toml conf/application.toml
            vim conf/application.toml
            需配置内容包括：

            admin：配置admin账户，此处可默认，router中的admin账户需与此处对应，用于登录账户服务

            db：配置自己的数据库账号密码
        #启动
            bash start.sh
    #启动跨链路由
        cd ~/wecross-networks/routers-test1/127.0.0.1-8250-25500/
        bash start.sh

        启动成功，输出如下：

        WeCross booting up .........
        WeCross start successfully
    #网页管理平台
        ###目前来看，利用脚本build_wecross.sh生成的跨链路由目录下已经包含pages文件夹，直接
        通过[ip]:8250/s/index.html即可访问网页管理平台。###
        #访问网页管理平台被拒绝
            #若需要网页管理平台进行远程访问跨链路由，请将跨链路由所在目录的 conf/wecross.toml 文件，修改[rpc]标签下的address字段为所需IP（如：0.0.0.0）
            #重启跨链路由
                bash stop.sh && bash start.sh
            #仍旧无法访问，或许是对外端口没开放
                #查看端口开放情况
                sudo iptables -vnL
                #开放8250和25500端口
                sudo iptables -I INPUT -p tcp --dport 8250 -j ACCEPT
                sudo iptables -I INPUT -p tcp --dport 25500 -j ACCEPT
                sudo iptables-save
                ####8250端口用于web远程访问，25500端口用于跨链路由之间数据互通，不开放此端口路由将无法被识别###
        #若跨链路由下不存在pages文件夹
            #环境要求
                # node version
                node -v
                结果：v8.16.0
                # npm version
                npm -v
                结果：6.4.1
            #搭建node与npm
                curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash
                sudo apt-get install -y nodejs
                sudo apt-get install npm
            #从远程仓库拉取代码到本地
                git clone https://gitee.com/WeBank/WeCross-WebApp.git
            #npm安装相关依赖，请注意编译环境要求；
                cd WeCross-WebApp
                npm install
            #安装完依赖以后，进行npm编译源代码，编译好的静态文件在dist文件夹中
                npm run build:prod
                cd dist
            #在跨链路由所在文件中，创建pages文件夹
                cd ./routers-test1/127.0.0.1-8250-25500
                mkdir -p pages
            #将dist文件夹中编译好的静态文件全部拷贝至刚创建的pages文件夹中；
                cp -r ./WeCross-WebApp/dist/* ~/wecross-networks/routers-test1/127.0.0.1-8250-25500/pages/
            
            
            






    