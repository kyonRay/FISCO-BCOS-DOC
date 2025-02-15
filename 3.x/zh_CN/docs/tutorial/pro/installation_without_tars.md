# 手动搭建Pro版区块链网络(无tars管理台)

标签：``Pro版区块链网络`` ``部署`` ``不依赖tars网页管理台``

------------

FISCO BCOS 3.x的Pro版本可以不依赖tars网页管理台进行搭建。本文档以在单机上部署2机构2节点区块链服务为例，介绍Pro版本不依赖tars网页管理台的FISCO BCOS搭建部署流程。

```eval_rst
.. note::
   - Pro版本不依赖tars网页管理台搭建FISCO BCOS ``BcosBuilder/pro`` 工具进行建链和扩容等相关操作，该工具的介绍请参考 `BcosBuilder <./pro_builder.html>`_ 
```

**注意:**

## 1. 安装依赖

部署工具`BcosBuilder`依赖`python3, curl`，根据您使用的操作系统，使用以下命令安装依赖。

**安装Ubuntu依赖(版本不小于Ubuntu18.04)**

```shell
sudo apt-get update
sudo apt-get install -y curl python3 wget
```

**安装CentOS依赖(版本不小于CentOS 7)**

```shell
sudo yum install -y curl python3 python3-devel wget
```

**安装macOS依赖**

```shell
brew install curl python3 wget
```

## 2. 下载Pro版区块链构建工具BcosBuilder

```eval_rst
.. note::
   - 部署工具 ``BcosBuilder`` 配置和使用请参考 `这里 <./pro_builder.html>`_
   - 若从github下载部署工具 ``BcosBuilder`` 网速太慢，请尝试: curl -#LO https://github.com/FISCO-BCOS/FISCO-BCOS/releases/download/v3.6.0/BcosBuilder.tgz && tar -xvf BcosBuilder.tgz
```

```shell
# 创建操作目录
mkdir -p ~/fisco && cd ~/fisco

# 下载Pro版区块链构建工具BcosBuilder
curl -#LO https://github.com/FISCO-BCOS/FISCO-BCOS/releases/download/v3.6.0/BcosBuilder.tgz && tar -xvf BcosBuilder.tgz

# Note: 若网速太慢，可尝试如下命令下载部署脚本:
curl -#LO https://github.com/FISCO-BCOS/FISCO-BCOS/releases/download/v3.6.0/BcosBuilder.tgz && tar -xvf BcosBuilder.tgz

# 安装构建工具依赖包
cd BcosBuilder && pip3 install -r requirements.txt
```

## 3. 部署Pro版本区块链节点

Pro版本FISCO BCOS包括RPC服务、Gateway服务以及区块链节点服务BcosNodeService。

- RPC服务：负责接收客户端请求，并将请求转发到节点进行处理， RPC服务可横向扩展，一个RPC服务可接入多个区块链节点服务
- Gateway服务：负责跨机构区块链节点之间的网络通信，Gateway服务横向可扩展，一个Gateway服务可接入多个区块链节点服务
- 区块链节点服务`BcosNodeService`：提供区块链相关的服务，包括共识、执行、交易上链等，节点服务通过接入到RPC服务和Gateway服务获取网络通信功能。

关于Pro版本FISCO BCOS的总体架构设计可参考[这里](../../design/architecture.md)。

本章以在单机上部署2机构2节点区块链服务为例，介绍Pro版本不依赖tars网页管理台的FISCO BCOS搭建部署流程，对应的服务组网模式如下:

![](../../../images/tutorial/pro_topology.png)

### 3.1 下载二进制

构建Pro版本FISCO BCOS前，需要先下载二进制包，`BcosBuilder`的提供了基于linux的静态二进制包下载功能，下载最新二进制的命令如下：

```eval_rst
.. note::
   - 可通过 ``python3 build_chain.py -h`` 查看脚本使用方法
   - 可通过 ``python3 build_chain.py build -h`` 查看构建安装包的使用方法
   - 使用``python3 build_chain.py download_binary``命令下载可执行二进制文件，二进制默认下载到 ``binary`` 目录
   - 若下载二进制比较慢，请尝试: ``python3 build_chain.py download_binary -t cdn``
```

```shell
# 进入操作目录
cd ~/fisco/BcosBuilder/pro

# 运行build_chain.py脚本下载二进制，二进制包默认下载到binary目录
python3 build_chain.py download_binary
```

### 3.2 工具使用

`build_chain.py`提供`build`命令，用来构建不依赖于`tars`网页管理台的安装包，使用方式如下:

```shell
$ python3 build_chain.py build --help
usage: build_chain.py build [-h] [-t TYPE] -c CONFIG [-O OUTPUT]

e.g:
python3 build_chain.py build -c conf/config-build-example.toml -O output_dir
python3 build_chain.py build -c conf/config-build-example.toml -t node -O output_dir
python3 build_chain.py build -c conf/config-build-example.toml -t rpc  -O output_dir
python3 build_chain.py build -c conf/config-build-example.toml -t gateway  -O output_dir

options:
  -h, --help            show this help message and exit
  -t TYPE, --type TYPE  [Optional] specify the type:
                        * type list: rpc, gateway, node, all
  -c CONFIG, --config CONFIG
                        [Required] the config file, default is config.toml:
                         * config to build chain example: conf/config-build-example.toml
  -O OUTPUT, --output OUTPUT
                        [Optional] specify the output dir, default is ./generated
```

参数:
    `-c`: 配置文件，默认值`./config.toml`
    `-O`: 安装包的目录，默认值`./generated`
    `-t`: 指定构建安装包的服务类型: `rpc`、`gateway`、`node`，默认值`all`，构建所有服务的安装包

### 3.3 构建安装包

在建链工具`BcosBuilder`目录，执行如下命令，可构建两个节点服务、两个RPC服务、两个网关服务的安装包，ip均为`127.0.0.1`，占用的监听端口分别:

- RPC服务: `20200`和`20201`
- 网关服务: `30300`和`30301`
- tars端口: `40401` ~ `40407`、`40411` ~ `40417`

**注意:** 搭建不依赖tars网页管理台的环境时，由于没有tars页面管理台的存在，各个微服务中的tars模块监听端口和连接信息需要使用配置文件管理，可以参考tars配置文件说明。

#### 3.3.1 tars配置文件
搭建不依赖tars网页管理台的环境，由于没有tars管理后台的存在，各个服务的tars模块监听和连接信息需要使用配置文件管理。

每个服务会有两个额外的配置文件`tars.conf`和`tars_proxy.ini`

```shell
$ ls -a 127.0.0.1/*/conf/tars.conf
127.0.0.1/gateway_30300/conf/tars.conf          
127.0.0.1/group0_node_40402/conf/tars.conf      
127.0.0.1/rpc_20200/conf/tars.conf
127.0.0.1/gateway_30301/conf/tars.conf          
127.0.0.1/group0_node_40412/conf/tars.conf      
127.0.0.1/rpc_20201/conf/tars.conf
```

```shell
$ ls -a 127.0.0.1/*/conf/tars_proxy.ini
127.0.0.1/gateway_30300/conf/tars_proxy.ini     
127.0.0.1/group0_node_40402/conf/tars_proxy.ini 
127.0.0.1/rpc_20200/conf/tars_proxy.ini
127.0.0.1/gateway_30301/conf/tars_proxy.ini     
127.0.0.1/group0_node_40412/conf/tars_proxy.ini 
127.0.0.1/rpc_20201/conf/tars_proxy.ini
```

##### 3.3.1.1. tars.conf

服务内部tars模块的服务端监听信息

###### 3.3.1.1.1 RPC服务

```shell
$ cat 127.0.0.1/rpc_20200/conf/tars.conf
<tars>
  <application>
    enableset=n
    setdivision=NULL
    <server>
      app=chain0
      server=agencyABcosRpcService
      localip=127.0.0.1
      basepath=./conf/
      datapath=./.data/
      logpath=./log/
      logsize=100M
      lognum=5
      logLevel=INFO
      deactivating-timeout=3000
      activating-timeout=10000
      opencoroutine=0
      coroutinememsize=1G
      coroutinestack=128K
      closecout=0
      netthread=4
      <chain0.agencyABcosRpcService.RpcServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40400 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyABcosRpcService.RpcServiceObj
        threads=8
      </chain0.agencyABcosRpcService.RpcServiceObjAdapter>
    </server>
    <client>
      sync-invoke-timeout=3000
      async-invoke-timeout=5000
      asyncthread=8
      modulename=chain0.agencyABcosRpcService
    </client>
  </application>
</tars>
```

tars配置详情参考 <https://doc.tarsyun.com/#/base/tars-template.md>

如配置示例:

```shell
<chain0.agencyABcosRpcService.RpcServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40400 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyABcosRpcService.RpcServiceObj
        threads=8
      </chain0.agencyABcosRpcService.RpcServiceObjAdapter> 
```

RPC服务内部有一个tars rpc模块，监听端口`40400`

注意: 服务内部的`tars`监听信息修改，可以在构建时修改`config.toml`的`[agency.rpc] tars_listen_ip`和`tars_listen_port`配置

```shell
[agency.rpc]
    deploy_ip=["127.0.0.1"]
    # rpc listen ip
    listen_ip="0.0.0.0"
    # rpc listen port
    listen_port=20201
    thread_count=4
    # rpc tars server listen ip
    tars_listen_ip="0.0.0.0"   # 修改tars监听的IP
    # rpc tars server listen port
    tars_listen_port=40410     # 修改tars监听的端口
```

###### 3.3.1.1.2 Gateway网关服务

```shell
$ cat 127.0.0.1/gateway_30300/conf/tars.conf
<tars>
  <application>
    enableset=n
    setdivision=NULL
    <server>
      app=chain0
      server=agencyABcosGatewayService
      localip=127.0.0.1
      basepath=./conf/
      datapath=./.data/
      logpath=./log/
      logsize=100M
      lognum=10
      logLevel=INFO
      deactivating-timeout=3000
      activating-timeout=10000
      opencoroutine=0
      coroutinememsize=1G
      coroutinestack=128K
      closecout=0
      netthread=4
      <chain0.agencyABcosGatewayService.GatewayServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40401 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyABcosGatewayService.GatewayServiceObj
        threads=8
      </chain0.agencyABcosGatewayService.GatewayServiceObjAdapter>
    </server>
    <client>
      sync-invoke-timeout=3000
      async-invoke-timeout=5000
      asyncthread=8
      modulename=chain0.agencyABcosGatewayService
    </client>
  </application>
</tars>
```

tars配置详情参考 <https://doc.tarsyun.com/#/base/tars-template.md>

如配置示例:

```shell
<chain0.agencyABcosGatewayService.GatewayServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40401 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyABcosGatewayService.GatewayServiceObj
        threads=8
      </chain0.agencyABcosGatewayService.GatewayServiceObjAdapter>
```

Gateway网关服务内部有一个tars gateway模块，监听端口`40401`

注意: 服务内部的`tars`监听信息修改，可以在构建时修改`config.toml`的`[agency.gateway] tars_listen_ip`和`tars_listen_port`配置

```shell
[agency.gateway]
    deploy_ip=["127.0.0.1"]
    # gateway listen ip
    listen_ip="0.0.0.0"
    # gateway listen port
    listen_port=30300
    # gateway connected peers, should be all of the gateway peers info
    peers=["127.0.0.1:30300", "127.0.0.1:30301"]
    # gateway tars server listen ip
    tars_listen_ip="0.0.0.0"        # 修改tars监听IP
    # gateway tars server listen port
    tars_listen_port=40401          # 修改tars监听端口
```

###### 3.3.1.1.3 节点服务

```shell
cat 127.0.0.1/group0_node_40402/conf/tars.conf
<tars>
  <application>
    enableset=n
    setdivision=NULL
    <server>
      app=chain0
      server=agencyAgroup0node0BcosNodeService
      localip=127.0.0.1
      basepath=./conf/
      datapath=./.data/
      logpath=./log
      logsize=100M
      lognum=10
      logLevel=INFO
      deactivating-timeout=3000
      activating-timeout=10000
      opencoroutine=0
      coroutinememsize=1G
      coroutinestack=128K
      closecout=0
      netthread=4
      <chain0.agencyAgroup0node0BcosNodeService.TxPoolServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40402 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.TxPoolServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.TxPoolServiceObjAdapter>
      <chain0.agencyAgroup0node0BcosNodeService.SchedulerServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40403 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.SchedulerServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.SchedulerServiceObjAdapter>
      <chain0.agencyAgroup0node0BcosNodeService.PBFTServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40404 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.PBFTServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.PBFTServiceObjAdapter>
      <chain0.agencyAgroup0node0BcosNodeService.LedgerServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40405 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.LedgerServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.LedgerServiceObjAdapter>
      <chain0.agencyAgroup0node0BcosNodeService.FrontServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40406 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.FrontServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.FrontServiceObjAdapter>
    </server>
    <client>
      sync-invoke-timeout=3000
      async-invoke-timeout=5000
      asyncthread=8
      modulename=chain0.agencyAgroup0node0BcosNodeService
    </client>
  </application>
</tars>
```

如配置示例:

节点服务内部包含5个tars模块: TxPool、Scheduler、PBFT、Ledger和Front。

- TxPool

```shell
<chain0.agencyAgroup0node0BcosNodeService.TxPoolServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40402 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.TxPoolServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.TxPoolServiceObjAdapter>
```

交易池模块监听端口: 40402

- Scheduler

```shell
<chain0.agencyAgroup0node0BcosNodeService.SchedulerServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40403 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.SchedulerServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.SchedulerServiceObjAdapter>
```

Scheduler模块监听端口: 40403

- PBFT

```shell
<chain0.agencyAgroup0node0BcosNodeService.PBFTServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40404 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.PBFTServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.PBFTServiceObjAdapter>
```

PBFT模块监听端口: 40404

- Ledger

```shell
 <chain0.agencyAgroup0node0BcosNodeService.LedgerServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40405 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.LedgerServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.LedgerServiceObjAdapter>
```

Ledger模块监听端口: 40405

- Front

```shell
<chain0.agencyAgroup0node0BcosNodeService.FrontServiceObjAdapter>
        allow
        endpoint=tcp -h 127.0.0.1 -p 40406 -t 60000
        maxconns=100000
        protocol=tars
        queuecap=50000
        queuetimeout=20000
        servant=chain0.agencyAgroup0node0BcosNodeService.FrontServiceObj
        threads=8
      </chain0.agencyAgroup0node0BcosNodeService.FrontServiceObjAdapter>
```

Front模块监听端口: 40406

注意:

- 服务内部的`tars`监听端口修改，可以在构建时修改`config.toml`的`[[agency.group]] [[agency.group.node]] tars_listen_ip`和`tars_listen_port`配置
- 节点服务内部需要分配五个连续的端口，范围是[tars_listen_port, tars_listen_port+4]，请注意端口冲突

```shell
[[agency.group]]
        group_id = "group0"
        [[agency.group.node]]
        # node name, Notice: node_name in the same agency and group must be unique
        node_name = "node0"
        deploy_ip = "127.0.0.1"
        # node tars server listen ip
        tars_listen_ip="0.0.0.0"        # 修改tars监听IP
        # node tars server listen port, Notice: the tars server of the node will cost five ports, then the port tars_listen_port ~ tars_listen_port + 4 should be in free
        tars_listen_port=40402          # 修改tars监听端口
        # enable data disk encryption for bcos node or not, default is false
        enable_storage_security = false
        # url of the key center, in format of ip:port, please refer to https://github.com/FISCO-BCOS/key-manager for details
        # key_center_url =
        # cipher_data_key =
        monitor_listen_port = "3902"
        # monitor log path example:"/home/fisco/tars/framework/app_log/"
        monitor_log_path = ""
```

**tars_proxy.ini**

tars客户端连接信息

```shell
$ cat generated/chain0/agencyA_tars_proxy.ini
[rpc]
proxy.0 = 127.0.0.1:40400

[gateway]
proxy.0 = 127.0.0.1:40401

[txpool]
proxy.0 = 127.0.0.1:40402

[scheduler]
proxy.0 = 127.0.0.1:40403

[pbft]
proxy.0 = 127.0.0.1:40404

[ledger]
proxy.0 = 127.0.0.1:40405

[front]
proxy.0 = 127.0.0.1:40406
```

示例:

```shell
[rpc]
proxy.0 = 127.0.0.1:40400
```

上面的配置表示，服务内部模块如果需要与rpc模块通信，使用的endpoint为`127.0.0.1:40400`

**注意:**

- 机构内部各个服务的`tars_proxy.ini`建议保持一致
- 在扩容时扩容的服务会生成新的文件，需要将新生成的`机构名_tars_proxy.ini`合并到已使用的`tars_proxy.ini`文件，并同步到所有的服务，服务需要重启后生效，否则新扩容的服务无法接入到现有环境

进行本操作前，请确保机器的上述端口没被占用。

```shell
# 进入操作目录
cd ~/fisco/BcosBuilder/pro

# 从conf目录拷贝配置
cp conf/config-build-example.toml config.toml
```

此时拷贝的`config.toml`为整个`BcosBuilder`使用的配置文件，配置详情请参考链接：[配置介绍](./pro_builder.html#id1)。

```shell
python3 build_chain.py build -c config.toml -O ./generated
```

执行上述命令后，输出`* build tars install package output dir : ./generated`并且无其他报错时，安装包构建成功：

```shell
=========================================================
* output dir: ./generated
* Don't load tars token and url
* args type: all
* generate config for the rpc service, build opr: True
----------- * generate config for the rpc service agencyABcosRpcService -----------
* generate config.ini for the rpc service agencyABcosRpcService
* store ./generated/127.0.0.1/rpc_20200/conf/config.ini
* generate config.ini for the rpc service agencyABcosRpcService success
* generate cert for the rpc service agencyABcosRpcService
* generate cert, ip: 127.0.0.1, output path: ./generated/127.0.0.1/rpc_20200/conf
* generate sdk cert, output path: ./generated/127.0.0.1/rpc_20200/conf
* generate cert for the rpc service agencyABcosRpcService success
----------- * generate config for the rpc service successagencyABcosRpcService -----------
* generate tars install package for BcosRpcService:agencyABcosRpcService:agencyA:chain0:rpc:binary/
----------- * generate config for the rpc service agencyBBcosRpcService -----------
* generate config.ini for the rpc service agencyBBcosRpcService
* store ./generated/127.0.0.1/rpc_20201/conf/config.ini
* generate config.ini for the rpc service agencyBBcosRpcService success
* generate cert for the rpc service agencyBBcosRpcService
* generate cert, ip: 127.0.0.1, output path: ./generated/127.0.0.1/rpc_20201/conf
* generate sdk cert, output path: ./generated/127.0.0.1/rpc_20201/conf
* generate cert for the rpc service agencyBBcosRpcService success
----------- * generate config for the rpc service successagencyBBcosRpcService -----------
* generate tars install package for BcosRpcService:agencyBBcosRpcService:agencyB:chain0:rpc:binary/
* generate config for the rpc service success
* generate config for the gateway service, build opr: True
----------- * generate config for the p2p service agencyABcosGatewayService -----------
* generate config.ini for the p2p service agencyABcosGatewayService
* store ./generated/127.0.0.1/gateway_30300/conf/config.ini
* generate config.ini for the p2p service agencyABcosGatewayService success
* generate cert for the p2p service agencyABcosGatewayService
* generate cert, ip: 127.0.0.1, output path: ./generated/127.0.0.1/gateway_30300/conf
* generate cert for the p2p service agencyABcosGatewayService success
----------- * generate config for the p2p service successagencyABcosGatewayService -----------
* generate gateway connection file: ./generated/127.0.0.1/gateway_30300/conf/nodes.json
* generate tars install package for BcosGatewayService:agencyABcosGatewayService:agencyA:chain0:gateway:binary/
----------- * generate config for the p2p service agencyBBcosGatewayService -----------
* generate config.ini for the p2p service agencyBBcosGatewayService
* store ./generated/127.0.0.1/gateway_30301/conf/config.ini
* generate config.ini for the p2p service agencyBBcosGatewayService success
* generate cert for the p2p service agencyBBcosGatewayService
* generate cert, ip: 127.0.0.1, output path: ./generated/127.0.0.1/gateway_30301/conf
* generate cert for the p2p service agencyBBcosGatewayService success
----------- * generate config for the p2p service successagencyBBcosGatewayService -----------
* generate gateway connection file: ./generated/127.0.0.1/gateway_30301/conf/nodes.json
* generate tars install package for BcosGatewayService:agencyBBcosGatewayService:agencyB:chain0:gateway:binary/
* generate config for the gateway service success
----------- generate genesis config for group group0 -----------
* generate pem file for agencyAgroup0node0BcosNodeService
 - pem_path: ./generated/chain0/group0/agencyAgroup0node0BcosNodeService/node.pem
 - node_id_path: ./generated/chain0/group0/agencyAgroup0node0BcosNodeService/node.nodeid
 - node_id: 728cbaebdc8e70ee21d22a9049fe5a7ae73ffe98f8013151c4e826e4f49fb8fc5379617747852449f571ed5fdb64321342baa33cf55f288f1651dd7d67f35374

 - sm_crypto: 0
* generate pem file for agencyBgroup0node0BcosNodeService
 - pem_path: ./generated/chain0/group0/agencyBgroup0node0BcosNodeService/node.pem
 - node_id_path: ./generated/chain0/group0/agencyBgroup0node0BcosNodeService/node.nodeid
 - node_id: ec78eddc1c8bdc109fd3ccf189ade1b4207e2f45bb9de5742cd8881de4348c42c4e1397f9462cb94e23f2130787bdbda7bcd3d73bad84f79b86a8c103bd34d8e

 - sm_crypto: 0
* generate genesis config nodeid
* chain_id: group0
* group_id: chain0
* consensus_type: pbft
* block_tx_count_limit: 1000
* leader_period: 1
* gas_limit: 3000000000
* compatibility_version: 3.6.0
* generate_genesis_config_nodeid success
* store genesis config for chain0.group0
  path: ./generated/chain0/group0/config.genesis
* store genesis config for chain0.group0 success
* store genesis config for agencyAgroup0node0BcosNodeService
  path: ./generated/chain0/group0/agencyAgroup0node0BcosNodeService/config.genesis
* store genesis config for agencyAgroup0node0BcosNodeService success
* store genesis config for agencyBgroup0node0BcosNodeService
  path: ./generated/chain0/group0/agencyBgroup0node0BcosNodeService/config.genesis
* store genesis config for agencyBgroup0node0BcosNodeService success
----------- generate genesis config for group0 success -----------
----------- generate ini config for group group0 -----------
* store ini config for agencyAgroup0node0BcosNodeService
  path: ./generated/chain0/group0/agencyAgroup0node0BcosNodeService/config.ini
* store ini config for agencyAgroup0node0BcosNodeService success
* store ini config for agencyBgroup0node0BcosNodeService
  path: ./generated/chain0/group0/agencyBgroup0node0BcosNodeService/config.ini
* store ini config for agencyBgroup0node0BcosNodeService success
----------- generate ini config for group group0 success -----------
 * generate node install package for deploy_ip: 127.0.0.1:./generated/127.0.0.1/group0_node_40402:agencyAgroup0node0BcosNodeService
=> base_dir: ./generated/127.0.0.1/group0_node_40402
* generate tars node install package service: agencyAgroup0node0BcosNodeService, chain id: chain0, tars pkg dir: binary/
* generate tars install package for BcosNodeService:agencyAgroup0node0BcosNodeService:agencyA:chain0:node:binary/
 * generate node install package for deploy_ip: 127.0.0.1:./generated/127.0.0.1/group0_node_40412:agencyBgroup0node0BcosNodeService
=> base_dir: ./generated/127.0.0.1/group0_node_40412
* generate tars node install package service: agencyBgroup0node0BcosNodeService, chain id: chain0, tars pkg dir: binary/
* generate tars install package for BcosNodeService:agencyBgroup0node0BcosNodeService:agencyB:chain0:node:binary/
* copy tars_proxy.ini: ./generated/chain0/agencyA_tars_proxy.ini ,dir: ./generated/127.0.0.1/rpc_20200/conf
* copy tars_proxy.ini: ./generated/chain0/agencyB_tars_proxy.ini ,dir: ./generated/127.0.0.1/rpc_20201/conf
* copy tars_proxy.ini: ./generated/chain0/agencyA_tars_proxy.ini ,dir: ./generated/127.0.0.1/gateway_30300/conf
* copy tars_proxy.ini: ./generated/chain0/agencyB_tars_proxy.ini ,dir: ./generated/127.0.0.1/gateway_30301/conf
* copy node tars_proxy.ini: ./generated/chain0/agencyA_tars_proxy.ini ,dir: ./generated/127.0.0.1/group0_node_40402
* copy node tars_proxy.ini: ./generated/chain0/agencyB_tars_proxy.ini ,dir: ./generated/127.0.0.1/group0_node_40412
==========================================================
* build tars install package output dir : ./generated
```

构建过程中生成的服务安装包相关的配置位于`generated/`目录，具体如下：

```shell
$ tree generated/
generated
├── 127.0.0.1
│   ├── gateway_30300  # 网关服务目录
│   │   ├── BcosGatewayService  # 可执行程序
│   │   ├── conf                # 配置目录
│   │   │   ├── ca.crt          # 根证书
│   │   │   ├── cert.cnf        # 证书配置文件
│   │   │   ├── config.ini      # 配置文件
│   │   │   ├── nodes.json      # p2p连接配置文件
│   │   │   ├── ssl.crt         # ssl证书，用于网关之间p2p网络连接
│   │   │   ├── ssl.key         # ssl证书私钥
│   │   │   ├── ssl.nodeid     
│   │   │   ├── tars.conf       # tars服务端配置，参考tars.conf配置说明
│   │   │   └── tars_proxy.ini  # tars客户端连接配置，参考tars_proxy.ini配置说明
│   │   ├── start.sh  # 启动脚本
│   │   └── stop.sh   # 停止脚本
│   ├── gateway_30301 # 网关服务目录
│   │   ├── BcosGatewayService  # 可执行程序
│   │   ├── conf                # 配置目录
│   │   │   ├── ca.crt          # 根证书
│   │   │   ├── cert.cnf        # 证书配置文件
│   │   │   ├── config.ini      # 配置文件
│   │   │   ├── nodes.json      # p2p连接配置文件
│   │   │   ├── ssl.crt         # ssl证书，用于网关之间p2p网络连接
│   │   │   ├── ssl.key         # ssl证书私钥
│   │   │   ├── ssl.nodeid     
│   │   │   ├── tars.conf       # tars服务端配置，参考tars.conf配置说明
│   │   │   └── tars_proxy.ini  # tars客户端连接配置，参考tars_proxy.ini配置说明
│   │   ├── start.sh  # 启动脚本
│   │   └── stop.sh   # 停止脚本
│   ├── group0_node_40402   # 节点服务目录
│   │   ├── BcosNodeService     # 可执行程序
│   │   ├── conf                # 配置目录
│   │   │   ├── config.genesis  # 区块链节点创世块文件
│   │   │   ├── config.ini      # 配置文件
│   │   │   ├── node.nodeid     # 节点nodeid
│   │   │   ├── node.pem        # 私钥文件，共识模块用于消息签名、验签
│   │   │   ├── tars.conf       # tars服务端配置，参考tars.conf配置说明
│   │   │   └── tars_proxy.ini  # tars客户端连接配置，参考tars_proxy.ini配置说明
│   │   ├── start.sh  # 启动脚本
│   │   └── stop.sh   # 停止脚本
│   ├── group0_node_40412   # 节点服务目录
│   │   ├── BcosNodeService     # 可执行程序
│   │   ├── conf                # 配置目录
│   │   │   ├── config.genesis  # 区块链节点创世块文件
│   │   │   ├── config.ini      # 配置文件
│   │   │   ├── node.nodeid     # 节点nodeid
│   │   │   ├── node.pem        # 私钥文件，共识模块用于消息签名、验签
│   │   │   ├── tars.conf       # tars服务端配置，参考tars.conf配置说明
│   │   │   └── tars_proxy.ini  # tars客户端连接配置，参考tars_proxy.ini配置说明
│   │   ├── start.sh  # 启动脚本
│   │   └── stop.sh   # 停止脚本
│   ├── rpc_20200           # RPC服务目录 
│   │   ├── BcosRpcService      # 可执行程序
│   │   ├── conf                # 配置目录
│   │   │   ├── ca.crt          # RPC根证书
│   │   │   ├── cert.cnf        # 证书配置文件
│   │   │   ├── config.ini      # 配置文件
│   │   │   ├── sdk             # SDK证书目录，rpc与sdk之间的ssl连接使用，sdk连接rpc服务时需要拷贝这些文件
│   │   │   │   ├── ca.crt
│   │   │   │   ├── cert.cnf
│   │   │   │   ├── sdk.crt
│   │   │   │   ├── sdk.key
│   │   │   │   └── sdk.nodeid
│   │   │   ├── ssl.crt         # ssl证书，用于rpc与sdk的网络连接
│   │   │   ├── ssl.key         # ssl证书私钥
│   │   │   ├── ssl.nodeid
│   │   │   ├── tars.conf       # tars服务端配置，参考tars.conf配置说明
│   │   │   └── tars_proxy.ini  # tars客户端连接配置，参考tars_proxy.ini配置说明
│   │   ├── start.sh    # 启动脚本
│   │   └── stop.sh     # 停止脚本
│   ├── rpc_20201         # RPC服务目录
│   │   ├── BcosRpcService      # 可执行程序
│   │   ├── conf                # 配置目录
│   │   │   ├── ca.crt          # RPC根证书
│   │   │   ├── cert.cnf        # 证书配置文件
│   │   │   ├── config.ini      # 配置文件
│   │   │   ├── sdk             # SDK证书目录，rpc与sdk之间的ssl连接使用，sdk连接rpc服务时需要拷贝这些文件
│   │   │   │   ├── ca.crt
│   │   │   │   ├── cert.cnf
│   │   │   │   ├── sdk.crt
│   │   │   │   ├── sdk.key
│   │   │   │   └── sdk.nodeid
│   │   │   ├── ssl.crt         # ssl证书，用于rpc与sdk的网络连接
│   │   │   ├── ssl.key         # ssl证书私钥
│   │   │   ├── ssl.nodeid
│   │   │   ├── tars.conf       # tars服务端配置，参考tars.conf配置说明
│   │   │   └── tars_proxy.ini  # tars客户端连接配置，参考tars_proxy.ini配置说明
│   │   ├── start.sh    # 启动脚本
│   │   └── stop.sh     # 停止脚本
│   ├── start_all.sh        # 启动脚本，启动所有服务节点
│   └── stop_all.sh         # 停止脚本，停止所有服务节点
├── chain0        
│   ├── agencyA_tars_proxy.ini                # 机构A tars_proxy.ini额外备份，机构内部的各个服务节点的tars_proxy.ini需要保持一致，在扩容或者缩容等服务节点变更的操作后，所有的服务都需要更新该配置文件，然后重启
│   ├── agencyB_tars_proxy.ini                # 机构B tars_proxy.ini额外备份，机构内部的各个服务节点的tars_proxy.ini需要保持一致，在扩容或者缩容等服务节点变更的操作后，所有的服务都需要更新该配置文件，然后重启
│   └── group0                      
│       ├── agencyAgroup0node0BcosNodeService # 节点agencyAgroup0node0BcosNodeService
│       │   ├── config.genesis                  # 节点创世块文件，重要文件，群组中扩容节点时需要该文件
│       │   ├── config.ini                      # 节点配置文件，与节点服务conf/config.ini是同一个文件
│       │   ├── node.nodeid                     # 节点nodeid，节点注册或者退出时使用
│       │   └── node.pem                        # 节点私钥文件，共识模块用于消息签名、验签
│       ├── agencyBgroup0node0BcosNodeService # 节点agencyBgroup0node0BcosNodeService
│       │   ├── config.genesis                  # 节点创世块文件，群组中扩容新节点时需要该文件
│       │   ├── config.ini                      # 节点配置文件，与节点服务conf/config.ini是同一个文件
│       │   ├── node.nodeid                     # 节点nodeid，节点注册或者退出时使用
│       │   └── node.pem                        # 节点私钥文件，共识模块用于消息签名、验签
│       └── config.genesis
├── gateway     # 网关服务根证书目录，扩容新网关服务节点时为节点签发证书，用于网关服务之间p2p网络连接
│   └── chain0
│       └── ca
│           ├── ca.crt
│           ├── ca.key
│           ├── ca.srl
│           └── cert.cnf
└── rpc         # RPC服务根证书目录，扩容新RPC服务节点时为节点签发证书，用于RPC服务与sdk之间网络连接
    └── chain0
        └── ca
            ├── ca.crt
            ├── ca.key
            ├── ca.srl
            └── cert.cnf
```

### 3.4 启动服务

**注意:** 本示例为单机环境搭建，实际环境中，服务分部在不同的机器上，这时首先需要将安装包拷贝至对应的机器，然后再启动服务。

```shell
$ cd generated/127.0.0.1
$ bash start_all.sh
try to start gateway_30300
try to start gateway_30301
try to start group0_node_40402
try to start group0_node_40412
try to start rpc_20200
try to start rpc_20201
 gateway_30300 start successfully pid=32749
 group0_node_40412 start successfully pid=32752
 gateway_30301 start successfully pid=32758
 group0_node_40402 start successfully pid=32760
 rpc_20200 start successfully pid=32759
 rpc_20201 start successfully pid=32761
```

服务启动成功。

## 4. 配置及使用控制台

控制台同时适用于Pro版本和Air版本的FISCO BCOS区块链，且在体验上完全一致。Pro版本区块链体验环境搭建完毕后，可配置并使用控制台向Pro版本区块链发送交易。

### 4.1 安装依赖

```eval_rst
.. note::
   - 控制台的配置方法和命令请参考 `这里 <../../develop/console/console_config.html>`_
```

使用控制台之前，需先安装java环境：

```shell
# ubuntu系统安装java
sudo apt install -y default-jdk

#centos系统安装java
sudo yum install -y java java-devel
```

### 4.2 下载、配置并使用控制台

**步骤1：下载控制台**

```shell
cd ~/fisco && curl -LO https://github.com/FISCO-BCOS/console/releases/download/v3.6.0/download_console.sh && bash download_console.sh
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试 `cd ~/fisco && curl -#LO https://gitee.com/FISCO-BCOS/console/raw/master/tools/download_console.sh && bash download_console.sh`
```

**步骤2：配置控制台**

- 拷贝控制台配置文件

若RPC服务未采用默认端口，请将文件中的20200替换成RPC服务监听端口。

```shell
# 最新版本控制台使用如下命令拷贝配置文件
cp -n console/conf/config-example.toml console/conf/config.toml
```

- 配置控制台证书

```shell
# 可通过命令 find . -name sdk找到所有SDK证书路径
cp BcosBuilder/pro/generated/127.0.0.1/rpc_20200/conf/sdk/* console/conf/
```

**步骤3：启动并使用控制台**

```shell
cd ~/fisco/console && bash start.sh
```

输出下述信息表明启动成功 否则请检查conf/config.toml中节点端口配置是否正确以及是否配置了SDK证书：

```shell
=============================================================================================
Welcome to FISCO BCOS console(3.6.0)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=============================================================================================
```

- 用控制台获取信息

```shell
# 获取网络连接信息：
[group0]: /> getPeers
PeersInfo{
    p2pNodeID='3082010a0282010100ad0098dd754ad939046b84ac83ff699e44f48de82c0d192f9378eb48d04abcc28117c855cc979b0d1381484d8c138ba1d4c52fcbf3724397153aba39b622f20eecd699ddee8afcf52d2ec543dab89cbcb3557b68585874c80b6853d4a827b4dff1d5fa8d4836dd4bf8cff7853d6249bb49ad7f3ba30ebb6298c0e911c4bfd156cabe8007380b295101ebf289d94fb7bc88ad442138cfa912d274fe6ed29e7a895aa3f3d9e98ad1ffdf36f3af5926889897d72adbe64c875bec1199c99f1549cb939053778ed514d774b6c891cdd3b1c47a76920ae1b8430ed6f2b29bb0d5ae649b64a96132696517705e08148913bd8bda35cb1049087862c59ffd3a8a8f69a10203010001',
    endPoint='0.0.0.0:30300',
    groupNodeIDInfo=[
        NodeIDInfo{
            group='group0',
            nodeIDList=[
                17be1d488dc961090110c445f72fd97db655db31738d2c1f63a8f3be809085dacfb4df631e2af1ed086ad3b4c5c228050983b93f5e169ef38400ec08cc88381e
            ]
        }
    ],
    peers=[
        PeerInfo{
            p2pNodeID='3082010a0282010100cc342cc289d60ac66cc0e55e08ccda05ea19f440f667840876d14c9445fdb44d5ba7d9d86c9209d9b61a3915cfd7965b565d6f1500629a14519edbaafe744c11e0ac49f8858668f5c5f5e35f3c10734129308b71129d91218f9673649504ee6f2466ce494695daa120e5cf73785e0251ecd1126a1a7f1fe43842f47644394240e841fe19b73d2b1411c0459cdabc2c3beedd07cdc3acd64615493d9b043bab4c3864486ced0e22c24d82839ef3afc998dec933318ec0e7abb5413fc07638e3cc3f7286fdb274cfe257b07a748485b610971377bd02bb83abab12f4969e8822c73f03f3d0314648812bf20de2ca6bdc2f2c10aaf92bd71a62d392faa3104a2ec10203010001',
            endPoint='127.0.0.1:60765',
            groupNodeIDInfo=[
                NodeIDInfo{
                    group='group0',
                    nodeIDList=[
                        934ddb929c088767fcc0f3b8cf4e5469e46f6d8c33e4c732ef3af8f39940045701e2cea83b0260202361f6e6cb7d5b5a6e2f7d69b5147c03fc79835f1a10ec73
                    ]
                }
            ]
        }
    ]
}

# 获取节点列表信息
[group0]: /> getGroupPeers
peer0: 17be1d488dc961090110c445f72fd97db655db31738d2c1f63a8f3be809085dacfb4df631e2af1ed086ad3b4c5c228050983b93f5e169ef38400ec08cc88381e
peer1: 934ddb929c088767fcc0f3b8cf4e5469e46f6d8c33e4c732ef3af8f39940045701e2cea83b0260202361f6e6cb7d5b5a6e2f7d69b5147c03fc79835f1a10ec73

[group0]: /> getSealerList
[
    Sealer{
        nodeID='17be1d488dc961090110c445f72fd97db655db31738d2c1f63a8f3be809085dacfb4df631e2af1ed086ad3b4c5c228050983b93f5e169ef38400ec08cc88381e',
        weight=1
    },
    Sealer{
        nodeID='934ddb929c088767fcc0f3b8cf4e5469e46f6d8c33e4c732ef3af8f39940045701e2cea83b0260202361f6e6cb7d5b5a6e2f7d69b5147c03fc79835f1a10ec73',
        weight=1
    }
]
```

### 4.3. 部署和调用合约

**步骤1：编写HelloWorld合约**

HelloWorld合约提供了两个接口`get()`和`set()`，用于获取/设置合约变量`name`，合约内容如下：

```c++
pragma solidity >=0.6.10 <0.8.20;
contract HelloWorld {
    string name;

    constructor() public {
        name = "Hello, World!";
    }

    function get() public view returns (string memory) {
        return name;
    }

    function set(string memory n) public {
        name = n;
    }
}
```

**步骤2: 部署HelloWorld合约**

为了方便用户快速体验，HelloWorld合约已经内置于控制台中，位于控制台目录`contracts/solidity/HelloWorld.sol`，参考下面命令部署：

```shell
# 在控制台输入以下指令 部署成功则返回合约地址
[group0]: /> deploy HelloWorld
transaction hash: 0x011602b6376e2bc806f1da91e2c70b0410b4d422fdbd4c346619a75d8e17e9da
contract address: 0x6849f21d1e455e9f0712b1e99fa4fcd23758e8f1
currentAccount: 0xfbb8fed0ce5402b799514c8f0a00661426549623

# 查看当前块高
[group0]: /> getBlockNumber
1
```

**步骤3. 调用HelloWorld合约**

```shell
# 调用get接口获取name变量，此处的合约地址是deploy指令返回的地址
[group0]: /> call HelloWorld 0x6849f21d1e455e9f0712b1e99fa4fcd23758e8f1 get
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return value size:1
Return types: (string)
Return values:(Hello, World!)
---------------------------------------------------------------------------------------------

# 查看当前块高，块高不变，因为get接口不更改账本状态
[group0]: /> getBlockNumber
1

# 调用set方法设置name
[group0]: /> call HelloWorld 0x6849F21D1E455e9f0712b1e99Fa4FCD23758E8F1 set "Hello, FISCO BCOS"
transaction hash: 0x2f7c85c2c59a76ccaad85d95b09497ad05ca7983c5ec79c8f9d102d1c8dddc30
---------------------------------------------------------------------------------------------
transaction status: 0
description: transaction executed successfully
---------------------------------------------------------------------------------------------
Receipt message: Success
Return message: Success
Return value size:0
Return types: ()
Return values:()
---------------------------------------------------------------------------------------------
Event logs
Event: {}

# 查看当前块高，因为set接口修改了账本状态，块高增加到2
[group0]: /> getBlockNumber
2

# 退出控制台
[group0]: /> exit
```

## 5 扩容
在成功搭建不依赖tars管理台的区块链网络基础上，本部分介绍如何对rpc、gateway、node进行扩容。

### 5.1 扩容RPC/Gateway服务（不依赖tars管理台）
通过单机扩容Pro版本FISCO BCOS联盟链的RPC/Gateway服务为例，帮助用户掌握不依赖tars管理台，进行Pro版本FISCO BCOS区块链的服务扩容。

#### 5.1.1. 修改扩容配置

区块链节点服务的扩容配置可参考`BcosBuilder`的扩容模板`conf/config-node-rpc-example.toml`，具体配置步骤如下：

```shell
# 进入操作目录
cd ~/fisco/BcosBuilder
# 扩容配置
cp conf/config-build-expand-rpc.toml config-expand-rpc.toml
```

```shell
cat config-expand-rpc.toml
[tars]
tars_pkg_dir = "binary/"

[chain]
chain_id="chain0"
# the rpc-service enable sm-ssl or not, default disable sm-ssl
rpc_sm_ssl=false
# the gateway-service enable sm-ssl or not, default disable sm-ssm
gateway_sm_ssl=false
# the existed rpc service ca path, will generate new ca if not configured
rpc_ca_cert_path=""
# the existed gateway service ca path, will generate new ca if not configured
#gateway_ca_cert_path="

[[agency]]
name = "agencyA"
# enable data disk encryption for rpc/gateway or not, default is false
enable_storage_security = false
# url of the key center, in format of ip:port, please refer to https://github.com/FISCO-BCOS/key-manager for details
# key_center_url =
# cipher_data_key =

    [agency.rpc]
    deploy_ip=["127.0.0.1"]
    # rpc listen ip
    listen_ip="0.0.0.0"
    # rpc listen port
    listen_port=20200
    thread_count=4
    # rpc tars server listen ip
    tars_listen_ip="0.0.0.0"
    # rpc tars server listen port
    tars_listen_port=40400
```

根据需要修改配置文件:

- RPC根证书路径

```shell
rpc_ca_cert_path="generated/rpc/chain0/ca/"
```

- 部署服务器修改

  ```shell
  deploy_ip = "127.0.0.1"
  ```

- 修改rpc监听信息

  ```shell
    listen_ip="0.0.0.0"
    listen_port=20202
  ```

- tars 监听信息修改

  ```shell
  tars_listen_ip="0.0.0.0"
  tars_listen_port=40420
  ```

#### 5.1.2. 生成安装包

配置修改完成之后，使用如下命令生成安装包

```shell
python3 build_chain.py build -c config-expand-rpc.toml -t rpc -O ./expand/rpc
```

执行上述命令后，输出`* build tars install package output dir : ./expand/rpc`并且无其他报错时，安装包构建成功

```shell
$ python3 build_chain.py build -c config-expand-rpc.toml -t rpc -O ./expand/rpc
=========================================================
* output dir: ./expand/rpc
* Don't load tars token and url
* args type: rpc
* generate config for the rpc service, build opr: True
----------- * generate config for the rpc service agencyABcosRpcService -----------
* generate config.ini for the rpc service agencyABcosRpcService
* store ./expand/rpc/127.0.0.1/rpc_20200/conf/config.ini
* generate config.ini for the rpc service agencyABcosRpcService success
* generate cert for the rpc service agencyABcosRpcService
* generate cert, ip: 127.0.0.1, output path: ./expand/rpc/127.0.0.1/rpc_20200/conf
* generate sdk cert, output path: ./expand/rpc/127.0.0.1/rpc_20200/conf
* generate cert for the rpc service agencyABcosRpcService success
----------- * generate config for the rpc service successagencyABcosRpcService -----------
* generate tars install package for BcosRpcService:agencyABcosRpcService:agencyA:chain0:rpc:binary/
* generate config for the rpc service success
* copy tars_proxy.ini: ./expand/rpc/chain0/agencyA_tars_proxy.ini ,dir: ./expand/rpc/127.0.0.1/rpc_20200/conf
=========================================================
* build tars install package output dir : ./expand/rpc
```

生成的安装包`./expand/rpc/`

```shell
expand/rpc
├── 127.0.0.1 
│   ├── rpc_20202         # RPC服务目录
│   │   ├── BcosRpcService      # 可执行程序
│   │   ├── conf                # 配置目录
│   │   │   ├── ca.crt          # RPC根证书
│   │   │   ├── cert.cnf        # 证书配置文件
│   │   │   ├── config.ini      # 配置文件
│   │   │   ├── sdk             # SDK证书目录，rpc与sdk之间的ssl连接使用，sdk连接rpc服务时需要拷贝这些文件
│   │   │   │   ├── ca.crt
│   │   │   │   ├── cert.cnf
│   │   │   │   ├── sdk.crt
│   │   │   │   ├── sdk.key
│   │   │   │   └── sdk.nodeid
│   │   │   ├── ssl.crt         # ssl证书，用于rpc与sdk的网络连接
│   │   │   ├── ssl.key         # ssl证书私钥
│   │   │   ├── ssl.nodeid
│   │   │   ├── tars.conf       # tars服务端配置，参考tars.conf配置说明
│   │   │   └── tars_proxy.ini  # tars客户端连接配置，参考tars_proxy.ini配置说明
│   │   ├── start.sh    # 启动脚本
│   │   └── stop.sh     # 停止脚本
│   ├── start_all.sh
│   └── stop_all.sh
└── chain0
    └── agencyA_tars_proxy.ini # 新生成tars客户端连接配置
```

#### 5.1.3. 合并tars_proxy.ini文件

使用`merge-config`命令 合并tars_proxy文件

```shell
python3 build_chain.py merge-config --help
usage: build_chain.py merge-config [-h] -t TYPE -c CONFIG [CONFIG ...] -O OUTPUT

e.g:
python3 build_chain.py merge-config -t tars -c tars0.conf tars1.conf -O output_dir

options:
  -h, --help            show this help message and exit
  -t TYPE, --type TYPE  [Required] specify the type:
                        * type list: tars
  -c CONFIG [CONFIG ...], --config CONFIG [CONFIG ...]
                        [Required] the config files to be
  -O OUTPUT, --output OUTPUT
                        [Required] specify the output dir
```

-t/--type   : 合并的配置文件类型，目前只支持`tars`类型
-c/--config : 配置列表，需要合并的配置文件列表
-O/--output : 输出目录

```shell
python3 build_chain.py merge-config -t tars -c generated/chain0/agencyA_tars_proxy.ini expand/rpc/chain0/agencyA_tars_proxy.ini -O agencyA_tars_proxy
```

```shell
cat agencyA_tars_proxy/tars_proxy.ini
[gateway]
proxy.0 = 127.0.0.1:40401

[rpc]
proxy.0 = 127.0.0.1:40400
proxy.1 = 127.0.0.1:40420

[txpool]
proxy.0 = 127.0.0.1:40402
proxy.1 = 127.0.0.1:40422

[scheduler]
proxy.0 = 127.0.0.1:40403
proxy.1 = 127.0.0.1:40423

[pbft]
proxy.0 = 127.0.0.1:40404
proxy.1 = 127.0.0.1:40424

[ledger]
proxy.0 = 127.0.0.1:40405
proxy.1 = 127.0.0.1:40425

[front]
proxy.0 = 127.0.0.1:40406
proxy.1 = 127.0.0.1:40426
```

操作成功之后，所有的tars连接信息会合并入同一个文件

#### 5.1.4. 更新tars_proxy并重启服务

更新机构A所有的服务的tars_proxy.ini文件

包括已有的服务以及扩容的服务。

```shell
cp -f agencyA_tars_proxy/tars_proxy.ini generated/127.0.0.1/gateway_30300/conf/
cp -f agencyA_tars_proxy/tars_proxy.ini generated/127.0.0.1/rpc_20200/conf
cp -f agencyA_tars_proxy/tars_proxy.ini generated/127.0.0.1/group0_node_40402/conf
cp -f agencyA_tars_proxy/tars_proxy.ini expand/rpc/127.0.0.1/rpc_20201/conf # 扩容的服务
```

重启服务

```shell
$ cd ~/fisco/BcosBuilder/pro/
$ bash generated/127.0.0.1/stop_all.sh
try to stop gateway_30300
 stop BcosGatewayService success.
try to stop gateway_30301
 stop BcosGatewayService success.
try to stop group0_node_40402
 stop BcosNodeService success.
try to stop group0_node_40412
 stop BcosNodeService success.
try to stop rpc_20200
 stop BcosRpcService success.
try to stop rpc_20201
 stop BcosRpcService success.
```

```shell
$ cd ~/fisco/BcosBuilder/pro/
$ bash generated/127.0.0.1/start_all.sh
try to start gateway_30300
try to start gateway_30301
try to start group0_node_40402
try to start group0_node_40412
try to start rpc_20200
try to start rpc_20201
 gateway_30300 start successfully pid=75226
 group0_node_40402 start successfully pid=75235
 rpc_20200 start successfully pid=75238
 group0_node_40412 start successfully pid=75237
 gateway_30301 start successfully pid=75230
 rpc_20201 start successfully pid=75239
```

启动扩容节点

```shell
$ cd ~/fisco/BcosBuilder/pro/
bash expand/rpc/127.0.0.1/start_all.sh
try to start rpc_20202
 rpc_20202 start successfully pid=75529
```

#### 5.1.5. 通过控制台连接扩容的RPC服务

##### 5.1.5.1 安装依赖

```eval_rst
.. note::
   - 控制台的配置方法和命令请参考 `这里 <../../operation_and_maintenance/console/console_config.html>`_
```

使用控制台之前，需先安装java环境：

```shell
# ubuntu系统安装java
sudo apt install -y default-jdk

#centos系统安装java
sudo yum install -y java java-devel
```

##### 5.1.5.2 下载、配置并使用控制台

**步骤1：下载控制台**

```shell
cd ~/fisco && curl -LO https://github.com/FISCO-BCOS/console/releases/download/v3.6.0/download_console.sh && bash download_console.sh
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试 `cd ~/fisco && curl -#LO https://gitee.com/FISCO-BCOS/console/raw/master/tools/download_console.sh && bash download_console.sh`
```

**步骤2：配置控制台**

- 拷贝控制台配置文件

将端口替换为扩容的RPC服务的端口

```shell
# 最新版本控制台使用如下命令拷贝配置文件
cp -n console/conf/config-example.toml console/conf/config.toml
```

```shell
peers=["127.0.0.1:20202"]
```

- 配置控制台证书

```shell
# 可通过命令 find . -name sdk找到所有SDK证书路径
cp ~/fisco/BcosBuilder/pro/expand/rpc/127.0.0.1/rpc_20202/conf/sdk/* console/conf
```

**步骤3：启动并使用控制台**

```shell
cd ~/fisco/console && bash start.sh
```

**步骤4: 调用控制台**

```shell
[group0]: /apps> deploy HelloWorld # 部署合约
transaction hash: 0x98bd489a77f9531bc4ccade0a72c6cff6aa0ca1205d6e5fe391b2cc150443277
contract address: 0x33e56a083e135936c1144960a708c43a661706c0
currentAccount: 0x3e00116eaf82e440ef93da1ecc510471cb3c97de
[group0]: /apps> call HelloWorld 0x33e56a083e135936c1144960a708c43a661706c0 set "Hello,Fisco" #调用合约
transaction hash: 0x30ac4d27c35fd3d5ca0f009ad195ec4c98e90bf72503879e7860e6a508acd614
---------------------------------------------------------------------------------------------
transaction status: 0
description: transaction executed successfully
---------------------------------------------------------------------------------------------
Receipt message: Success
Return message: Success
Return value size:0
Return types: ()
Return values:()
---------------------------------------------------------------------------------------------
```

### 5.2 扩容节点服务（不依赖tars管理台）

#### 5.2.1. 修改扩容配置

区块链节点服务的扩容配置可参考`BcosBuilder`的扩容模板`conf/config-node-expand-example.toml`，具体配置步骤如下：

```shell
# 进入操作目录
cd ~/fisco/BcosBuilder
# 扩容配置
cp conf/config-build-expand-node-example.toml config-expand-node.toml
cat config-expand-node.toml
```

```shell
[tars]
tars_pkg_dir = "binary/"

[chain]
chain_id="chain0"
# the rpc-service enable sm-ssl or not, default disable sm-ssl
rpc_sm_ssl=false
# the gateway-service enable sm-ssl or not, default disable sm-ssm
gateway_sm_ssl=false
# the existed rpc service ca path, will generate new ca if not configured
#rpc_ca_cert_path=""
# the existed gateway service ca path, will generate new ca if not configured
#gateway_ca_cert_path="

[[group]]
group_id="group0"
# the genesis configuration path of the group, will generate new genesis configuration if not configured
genesis_config_path = "./generated/chain0/group0/config.genesis"
# VM type, now only support evm/wasm
vm_type="evm"
# use sm-crypto or not
sm_crypto=false
# enable auth-check or not
auth_check=true
init_auth_address="0x241abd724d0d03aa3323679de3ed65a358e5b121"

# the genesis config
# the number of blocks generated by each leader
leader_period = 1
# the max number of transactions of a block
block_tx_count_limit = 1000
# consensus algorithm now support PBFT(consensus_type=pbft), rPBFT(consensus_type=rpbft)
consensus_type = "pbft"
# transaction gas limit
gas_limit = "3000000000"
# compatible version, can be dynamically upgraded through setSystemConfig
compatibility_version="3.6.0"

[[agency]]
name = "agencyA"
# enable data disk encryption for rpc/gateway or not, default is false
enable_storage_security = false
# url of the key center, in format of ip:port, please refer to https://github.com/FISCO-BCOS/key-manager for details
# key_center_url =
# cipher_data_key =

    [[agency.group]]
        group_id = "group0"
        [[agency.group.node]]
        # node name, Notice: node_name in the same agency and group must be unique
        node_name = "node2"
        deploy_ip = "127.0.0.1"
        # node tars server listen ip
        tars_listen_ip="0.0.0.0"
        # node tars server listen port, Notice: the tars server of the node will cost five ports, then the port tars_listen_port ~ tars_listen_port + 4 should be in free
        tars_listen_port=40422
        # enable data disk encryption for bcos node or not, default is false
        enable_storage_security = false
        # url of the key center, in format of ip:port, please refer to https://github.com/FISCO-BCOS/key-manager for details
        # key_center_url =
        # cipher_data_key =
        monitor_listen_port = "3902"
        # monitor log path example:"/home/fisco/tars/framework/app_log/"
        monitor_log_path = ""
```

根据需要修改配置文件:

- 修改节点名

  节点名不要与已有的节点冲突

  ```shell
  node_name = "node2"
  ```

- 部署服务器修改

  ```shell
  deploy_ip = "127.0.0.1"
  ```

- 设置创世块文件路径

  ```shell
  genesis_config_path = "./generated/chain0/group0/config.genesis"
  ```

- tars 监听信息修改

  ```shell
  tars_listen_ip="0.0.0.0"
  tars_listen_port=40422
  ```

#### 5.2.2. 生成安装包

配置修改完成之后，使用如下命令生成安装包

```shell
python3 build_chain.py build -c config-expand-node.toml -t node -O ./expand/node
```

执行上述命令后，输出`* build tars install package output dir : ./expand/node`并且无其他报错时，安装包构建成功

```shell
$ python3 build_chain.py build -c config-expand-node.toml -t node -O ./expand/node
=========================================================
* output dir: ./expand/node
* Don't load tars token and url
* args type: node
----------- generate genesis config for group group0 -----------
* the genesis config file has been set, path: /Users/octopus/fisco/BcosBuilder/pro/generated/chain0/group0/config.genesis
* generate pem file for agencyAgroup0node2BcosNodeService
 - pem_path: ./expand/node/chain0/group0/agencyAgroup0node2BcosNodeService/node.pem
 - node_id_path: ./expand/node/chain0/group0/agencyAgroup0node2BcosNodeService/node.nodeid
 - node_id: 0ebd3916fa9e73736f64c6f84b4f55dfa0a714f26641eb9e171b0aa1975b99848a10805fbb12350434e81311008654d217a7aef9a854283d7868be10c3f21af9

 - sm_crypto: 0
* store genesis config for agencyAgroup0node2BcosNodeService
  path: ./expand/node/chain0/group0/agencyAgroup0node2BcosNodeService/config.genesis
* store genesis config for agencyAgroup0node2BcosNodeService success
----------- generate genesis config for group0 success -----------
----------- generate ini config for group group0 -----------
* store ini config for agencyAgroup0node2BcosNodeService
  path: ./expand/node/chain0/group0/agencyAgroup0node2BcosNodeService/config.ini
* store ini config for agencyAgroup0node2BcosNodeService success
----------- generate ini config for group group0 success -----------
 * generate node install package for deploy_ip: 127.0.0.1:./expand/node/127.0.0.1/group0_node_40422:agencyAgroup0node2BcosNodeService
=> base_dir: ./expand/node/127.0.0.1/group0_node_40422
* generate tars node install package service: agencyAgroup0node2BcosNodeService, chain id: chain0, tars pkg dir: binary/
* generate tars install package for BcosNodeService:agencyAgroup0node2BcosNodeService:agencyA:chain0:node:binary/
* copy node tars_proxy.ini: ./expand/node/chain0/agencyA_tars_proxy.ini ,dir: ./expand/node/127.0.0.1/group0_node_40422
=========================================================
* build tars install package output dir : ./expand/node
```

生成的安装包`./expand/node/`

```shell
tree  expand/node/
expand/node/
├── 127.0.0.1
│   ├── group0_node_40422   # 节点目录
│   │   ├── BcosNodeService     # 可执行程序
│   │   ├── conf                # 配置目录
│   │   │   ├── config.genesis  # 区块链节点创世块文件
│   │   │   ├── config.ini      # 配置文件
│   │   │   ├── node.nodeid     # 节点nodeid
│   │   │   ├── node.pem        # 私钥文件，共识模块用于消息签名、验签
│   │   │   ├── tars.conf       # tars服务端配置，参考tars.conf配置说明
│   │   │   └── tars_proxy.ini  # tars客户端连接配置，参考tars_proxy.ini配置说明
│   │   ├── start.sh    # 启动脚本
│   │   └── stop.sh     # 停止脚本
│   ├── start_all.sh    # 启动脚本，启动所有服务节点
│   └── stop_all.sh     # 停止脚本，停止所有服务节点
└── chain0
    ├── agencyA_tars_proxy.ini  # 新生成tars客户端连接配置
    └── group0
        └── agencyAgroup0node2BcosNodeService
            ├── config.genesis  # 区块链创世块文件
            ├── config.ini      # 扩容节点的配置寄文件
            ├── node.nodeid     # 扩容节点的nodeid
            └── node.pem        # 扩容节点的私钥文件
```

#### 5.2.3. 合并tars_proxy.ini文件

使用`merge-config`命令 合并tars_proxy文件

```shell
python3 build_chain.py merge-config --help
usage: build_chain.py merge-config [-h] -t TYPE -c CONFIG [CONFIG ...] -O OUTPUT

e.g:
python3 build_chain.py merge-config -t tars -c tars0.conf tars1.conf -O output_dir

options:
  -h, --help            show this help message and exit
  -t TYPE, --type TYPE  [Required] specify the type:
                        * type list: tars
  -c CONFIG [CONFIG ...], --config CONFIG [CONFIG ...]
                        [Required] the config files to be
  -O OUTPUT, --output OUTPUT
                        [Required] specify the output dir
```

-t/--type   : 合并的配置文件类型，目前只支持`tars`类型
-c/--config : 配置列表，需要合并的配置文件列表
-O/--output : 输出目录

```shell
python3 build_chain.py merge-config -t tars -c generated/chain0/agencyA_tars_proxy.ini expand/node/chain0/agencyA_tars_proxy.ini -O agencyA_tars_proxy
```

```shell
cat agencyA_tars_proxy/tars_proxy.ini
[gateway]
proxy.0 = 127.0.0.1:40401

[rpc]
proxy.0 = 127.0.0.1:40400

[txpool]
proxy.0 = 127.0.0.1:40402
proxy.1 = 127.0.0.1:40422

[scheduler]
proxy.0 = 127.0.0.1:40403
proxy.1 = 127.0.0.1:40423

[pbft]
proxy.0 = 127.0.0.1:40404
proxy.1 = 127.0.0.1:40424

[ledger]
proxy.0 = 127.0.0.1:40405
proxy.1 = 127.0.0.1:40425

[front]
proxy.0 = 127.0.0.1:40406
proxy.1 = 127.0.0.1:40426
```

操作成功之后，所有的tars连接信息会合并入同一个文件

#### 5.2.4. 更新tars_proxy并重启服务

更新机构A所有的服务的tars_proxy.ini文件

包括已有的服务以及扩容的服务。

```shell
cp -f agencyA_tars_proxy/tars_proxy.ini generated/127.0.0.1/gateway_30300/conf/
cp -f agencyA_tars_proxy/tars_proxy.ini generated/127.0.0.1/rpc_20200/conf
cp -f agencyA_tars_proxy/tars_proxy.ini generated/127.0.0.1/group0_node_40402/conf
cp -f agencyA_tars_proxy/tars_proxy.ini expand/node/127.0.0.1/group0_node_40422/conf # 扩容的服务
```

重启服务

```shell
$ cd ~/fisco/BcosBuilder/pro/
$ bash generated/127.0.0.1/stop_all.sh
try to stop gateway_30300
 stop BcosGatewayService success.
try to stop gateway_30301
 stop BcosGatewayService success.
try to stop group0_node_40402
 stop BcosNodeService success.
try to stop group0_node_40412
 stop BcosNodeService success.
try to stop rpc_20200
 stop BcosRpcService success.
try to stop rpc_20201
 stop BcosRpcService success.
```

```shell
$ cd ~/fisco/BcosBuilder/pro/
$ bash generated/127.0.0.1/start_all.sh
try to start gateway_30300
try to start gateway_30301
try to start group0_node_40402
try to start group0_node_40412
try to start rpc_20200
try to start rpc_20201
 gateway_30300 start successfully pid=75226
 group0_node_40402 start successfully pid=75235
 rpc_20200 start successfully pid=75238
 group0_node_40412 start successfully pid=75237
 gateway_30301 start successfully pid=75230
 rpc_20201 start successfully pid=75239
```

启动扩容节点

```shell
$ cd ~/fisco/BcosBuilder/pro/
bash expand/node/127.0.0.1/start_all.sh
try to start group0_node_40422
 group0_node_40422 start successfully pid=75529
```

#### 5.2.5. 将新扩容节点加入到群组

```eval_rst
.. note::
   扩容新节点时，不建议直接将节点加为共识节点，当扩容节点的块高与链上已有节点最高块高一致时候，才可将其加入为共识节点。
```

**步骤1：获取扩容节点的NodeID**

```shell
$ cd ~/fisco/BcosBuilder/pro/
$ cat expand/node/127.0.0.1/group0_node_40422/conf/node.nodeid
0ebd3916fa9e73736f64c6f84b4f55dfa0a714f26641eb9e171b0aa1975b99848a10805fbb12350434e81311008654d217a7aef9a854283d7868be10c3f21af9
```

新节点扩容成功后，可通过控制台的`getGroupPeers`命令查看新增的节点列表：

```shell
[group0]: /> getGroupPeers
[group0]: /apps> getGroupPeers
peer0: 17be1d488dc961090110c445f72fd97db655db31738d2c1f63a8f3be809085dacfb4df631e2af1ed086ad3b4c5c228050983b93f5e169ef38400ec08cc88381e
peer1: 934ddb929c088767fcc0f3b8cf4e5469e46f6d8c33e4c732ef3af8f39940045701e2cea83b0260202361f6e6cb7d5b5a6e2f7d69b5147c03fc79835f1a10ec73

[group0]: /> getSealerList
[
    Sealer{
        nodeID='17be1d488dc961090110c445f72fd97db655db31738d2c1f63a8f3be809085dacfb4df631e2af1ed086ad3b4c5c228050983b93f5e169ef38400ec08cc88381e',
        weight=1
    },
    Sealer{
        nodeID='934ddb929c088767fcc0f3b8cf4e5469e46f6d8c33e4c732ef3af8f39940045701e2cea83b0260202361f6e6cb7d5b5a6e2f7d69b5147c03fc79835f1a10ec73',
        weight=1
    }
]
[group0]: /> getObserverList  # 获取群组观察节点列表
[]
```

从控制台输出可看出，nodeID为`0ebd3916fa9e73736f64c6f84b4f55dfa0a714f26641eb9e171b0aa1975b99848a10805fbb12350434e81311008654d217a7aef9a854283d7868be10c3f21af9`节点不在群组内，使用控制台`addObserver`命令将其加入到观察节点如下：

**步骤2: 将扩容节点加入为观察节点**

```shell
[group0]: /> addObserver 0ebd3916fa9e73736f64c6f84b4f55dfa0a714f26641eb9e171b0aa1975b99848a10805fbb12350434e81311008654d217a7aef9a854283d7868be10c3f21af9
{
    "code":0,
    "msg":"Success"
}

[group0]: /> getObserverList
[
    0ebd3916fa9e73736f64c6f84b4f55dfa0a714f26641eb9e171b0aa1975b99848a10805fbb12350434e81311008654d217a7aef9a854283d7868be10c3f21af9
]
```

**步骤3：扩容节点同步到最高块后，将扩容节点加入为共识节点**

```shell
[group0]: /> addSealer 0ebd3916fa9e73736f64c6f84b4f55dfa0a714f26641eb9e171b0aa1975b99848a10805fbb12350434e81311008654d217a7aef9a854283d7868be10c3f21af9 1
{
    "code":0,
    "msg":"Success"
}

[group0]: /> getSealerList
[
    [
    Sealer{
        nodeID='17be1d488dc961090110c445f72fd97db655db31738d2c1f63a8f3be809085dacfb4df631e2af1ed086ad3b4c5c228050983b93f5e169ef38400ec08cc88381e',
        weight=1
    },
    Sealer{
        nodeID='934ddb929c088767fcc0f3b8cf4e5469e46f6d8c33e4c732ef3af8f39940045701e2cea83b0260202361f6e6cb7d5b5a6e2f7d69b5147c03fc79835f1a10ec73',
        weight=1
    },
    Sealer{
        nodeID='0ebd3916fa9e73736f64c6f84b4f55dfa0a714f26641eb9e171b0aa1975b99848a10805fbb12350434e81311008654d217a7aef9a854283d7868be10c3f21af9',
        weight=1
    }
]
```

**步骤4：部署和调用合约**

```shell
[group0]: /apps> deploy HelloWorld
transaction hash: 0x2255701e51fd5d38d357a5f294c11ff550ab6cc4a5e1b40237f2130b58e8ac59
contract address: 0x6546c3571f17858ea45575e7c6457dad03e53dbb
currentAccount: 0x3e00116eaf82e440ef93da1ecc510471cb3c97de

[group0]: /apps> call HelloWorld 0x6546c3571f17858ea45575e7c6457dad03e53dbb set "Hello,Fisco"
transaction hash: 0x4025cb4c87ae36f5caa4b6488df5c40530f4e1cb52d1d0abb4cb4242e16a4755
---------------------------------------------------------------------------------------------
transaction status: 0
description: transaction executed successfully
---------------------------------------------------------------------------------------------
Receipt message: Success
Return message: Success
Return value size:0
Return types: ()
Return values:()
---------------------------------------------------------------------------------------------

[group0]: /apps> call HelloWorld 0x6546c3571f17858ea45575e7c6457dad03e53dbb get
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return value size:1
Return types: (STRING)
Return values:(Hello,Fisco)
---------------------------------------------------------------------------------------------
```