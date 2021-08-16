# hyperledger-fabric_v2.3.2
操作系统：Ubuntu 20.04.2.0

Go：1.15.7

Docker：20.10.7

Docker Compose：1.25.0

Hyperledger Fabric：2.3.2

# 下载VMware，配置Ubuntu系统
Ubuntu镜像：链接：https://pan.baidu.com/s/1eKKctAO75bRtLz3B6ORMfA 
            提取码：z8i9
# 换国内镜像源
系统设置->软件和更新下载自->其他-> "mirrors.aliyun.com"

完成后打开终端

    $ sudo apt update

# 基础工具安装
    $ sudo apt install git
    $ sudo apt install curl
    $ sudo apt install vim

# 安装Docker（这一步操作会比较慢）
    $ sudo apt install docker.io
查看版本
    
    $ docker --version
    Docker version 20.10.7, build 20.10.7-0ubuntu1~20.04.1
设置成非 root 用户也能执行 docker

    $ sudo usermod -aG docker 用户名 
重启

# 安装Docker-Compose
    $ sudo apt install docker-compose
查看版本

    $ docker-compose --version
    docker-compose version 1.25.0, build unknown
允许其他用户执行 compose 相关命令

    $ sudo chmod +x /usr/share/doc/docker-compose
重启

# Go语言安装及环境配置
1.实机下载，将压缩包拖动至虚拟机内

https://studygolang.com/dl/golang/go1.15.7.linux-amd64.tar.gz

在压缩包所在文件夹打开终端

    $ sudo tar -zxvf go1.15.7.linux-amd64.tar.gz -C /usr/local/
2.配置环境变量

    $ sudo gedit /etc/profile
在 profile 文件最后添加如下内容

    export GOROOT=/usr/local/go
    export GOPATH=$HOME/go
    export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
保存后在终端输入

    $ source /etc/profile
3.使用 go version 命令验证是否安装成功

    $ go version
    go version go1.15.7 linux/amd64
重启

# 拉取Fabric源码（建议先在windows环境下拉取后放在虚拟机中）
    $ mkdir -p ~/go/src/github.com/hyperledger 
    $ cd ~/go/src/github.com/hyperledger 
拉取fabric的源码（很慢，有科学上网会快点）

    $ git clone https://github.com/hyperledger/fabric.git
查看并切换当前分支

    $ cd ./fabric
    $ git branch -a  
    $ git checkout v2.3.2 

# 拉取fabric-samples

    $ sudo mkdir -p /etc/docker
1.配置镜像加速器

https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors

替换自己的加速器地址

    $ sudo tee /etc/docker/daemon.json <<-'EOF'
    {
      "registry-mirrors": ["https://eck51glv.mirror.aliyuncs.com"]
    }
    EOF
    $ sudo systemctl daemon-reload
    $ sudo systemctl restart docker
2.下载fabric-samples源码（建议先在windows环境下拉取后放在虚拟机中）

    $ cd ~/go/src/github.com/hyperledger/fabric/scripts
    $ git clone https://github.com/hyperledger/fabric-samples.git
（这一步可能会多次卡住，用ctrl+c退出后重新输入代码即可）
    
    $ cd ./fabric-samples
    $ git branch -a
    $ git checkout v2.3.2(或者2.2.0)
3.下载二进制文件

链接：https://pan.baidu.com/s/1ecCE4nBC8XvBa2NQvGm_hQ 
提取码：65no

下载的hyperledger-fabric-linux-amd64-2.3.2.tar.gz内有bin和config两个文件夹，hyperledger-fabric-ca-linux-amd64-1.5.0.tar.gz内有bin文件夹，将两个bin文件夹内的二进制文件汇总在一个bin文件夹内。最后将bin和config文件夹复制到fabric-samples文件夹内

4.下载Docker镜像

    $ cd ~/go/src/github.com/hyperledger/fabric/scripts
修改脚本

    $ sudo gedit bootstrap.sh
删除文档最后的samplesInstall和binariesInstall步骤，只剩下

    if [ "$DOCKER" == "true" ]; then
      echo
      echo "Installing Hyperledger Fabric docker images"
      echo
     dockerInstall
    fi
执行 bootstrap.sh 脚本

    $ ./bootstrap.sh
完成后会出现

===> List out hyperledger docker images

hyperledger/fabric-tools       2.3.2               18ed4db0cd57        7 weeks ago         1.55GB

hyperledger/fabric-tools       latest              18ed4db0cd57        7 weeks ago         1.55GB

hyperledger/fabric-ca          2.3.2               c18a0d3cc958        7 weeks ago         253MB

hyperledger/fabric-ca          latest              c18a0d3cc958        7 weeks ago         253MB

hyperledger/fabric-ccenv       2.3.2               3d31661a812a        7 weeks ago         1.45GB

hyperledger/fabric-ccenv       latest              3d31661a812a        7 weeks ago         1.45GB

hyperledger/fabric-orderer     2.3.2               b666a6ebbe09        7 weeks ago         173MB

hyperledger/fabric-orderer     latest              b666a6ebbe09        7 weeks ago         173MB

hyperledger/fabric-peer        2.3.2               fa87ccaed0ef        7 weeks ago         179MB

hyperledger/fabric-peer        latest              fa87ccaed0ef        7 weeks ago         179MB

hyperledger/fabric-javaenv     2.3.2               5ba5ba09db8f        2 months ago        1.76GB

hyperledger/fabric-javaenv     latest              5ba5ba09db8f        2 months ago        1.76GB

... ...

5.设置环境变量

    $ sudo gedit /etc/profile
在文件最后添加

    export PATH=$PATH:$GOROOT/bin:$GOPATH/bin:$HOME/go/src/github.com/hyperledger/fabric/scripts/fabric-samples/bin
保存后在终端输入

    $ source /etc/profile
检验环境变量是否成功

    $ fabric-ca-client version
    fabric-ca-client:
    Version: 1.5.0
    Go version: go1.15.7
    OS/Arch: linux/amd64
重启

# 测试网络
    $ cd ~/go/src/github.com/hyperledger/fabric/scripts/fabric-samples/test-network/
    $ ./network.sh createChannel -c <channel name>
    $ ./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go(第一次执行这一步最好挂着docker镜像加速)
进入网络

    $ export PATH=${PWD}/../bin:$PATH
    $ export FABRIC_CFG_PATH=$PWD/../config/
进入Org1

    # Environment variables for Org1
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org1MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
    export CORE_PEER_ADDRESS=localhost:7051
初始化账单

    $ peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
列出账单

    $ peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
转账

    $ peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
进入Org2查看转账记录

    # Environment variables for Org2
    export CORE_PEER_TLS_ENABLED=true
    export CORE_PEER_LOCALMSPID="Org2MSP"
    export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
    export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
    export CORE_PEER_ADDRESS=localhost:9051
查询

    $ peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
查看链码日志

    $ docker ps
    $ docker logs <IMAGE>

# 关闭网络
    $ ./network.sh down
