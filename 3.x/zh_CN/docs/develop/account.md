# 3. 账户使用与账户管理

标签：``创建账户`` ``国密账户`` ``密钥文件``

----

FISCO BCOS使用账户来标识和区分每一个独立的用户。在采用公私钥体系的区块链系统里，每一个账户对应着一对公钥和私钥。其中，由公钥经哈希等安全的单向性算法计算后得到地址字符串被用作该账户的账户名，即**账户地址**，为了与智能合约的地址相区别和一些其他的历史原因，账户地址也常被称之**外部账户地址**。而仅有用户知晓的私钥则对应着传统认证模型中的密码。用户需要通过安全的密码学协议证明其知道对应账户的私钥，来声明其对于该账户的所有权，以及进行敏感的账户操作。

```eval_rst
.. important::

    在之前的其他教程中，为了简化操作，使用了工具提供的默认的账户进行操作。但在实际应用部署中，用户需要创建自己的账户，并妥善保存账户私钥，避免账户私钥泄露等严重的安全问题。
```

本文将具体介绍账户的创建、存储和使用方式，要求阅读者有一定的Linux操作基础。

FISCO BCOS提供了脚本和Java SDK用以创建账户，同时也提供了Java SDK和控制台来存储账户私钥。用户可以根据需求选择将账户私钥存储为PEM或者PKCS12格式的文件。其中，PEM格式使用明文存储私钥，而PKCS12使用用户提供的口令加密存储私钥。

## 账户的创建

### 使用脚本创建账户

国密生成账户脚本`get_gm_account.sh`与非国密`get_account.sh`选项和使用方式一致，请参考操作即可，不再赘述。

#### 1. 获取脚本

```shell
curl -#LO https://raw.githubusercontent.com/FISCO-BCOS/console/master/tools/get_account.sh && chmod u+x get_account.sh && bash get_account.sh -h
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试 `curl -#LO https://gitee.com/FISCO-BCOS/console/raw/master/tools/get_account.sh && chmod u+x get_account.sh && bash get_account.sh -h`
```

国密版本请使用下面的指令获取脚本

```shell
curl -#LO https://raw.githubusercontent.com/FISCO-BCOS/console/master/tools/get_gm_account.sh && chmod u+x get_gm_account.sh && bash get_gm_account.sh -h
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试 `curl -#LO https://gitee.com/FISCO-BCOS/console/raw/master/tools/get_gm_account.sh && chmod u+x get_gm_account.sh && bash get_gm_account.sh -h`
```

执行上面的指令，看到如下输出则下载到了正确的脚本，否则请重试。

```shell
Usage: get_account.sh
    default       generate account and store private key in PEM format file
    -p            generate account and store private key in PKCS12 format file
    -k [FILE]     calculate address of PEM format [FILE]
    -P [FILE]     calculate address of PKCS12 format [FILE]
    -a            force script to consider the platform is aarch64
    -h Help
```

#### 2. 使用脚本生成PEM格式私钥

- 生成私钥与地址

```shell
bash get_account.sh
```

执行上面的命令，可以得到类似下面的输出，包括账户地址和以账户地址为文件名的私钥PEM文件与公钥PUB文件。

```shell
[INFO] Account privateHex: 0x3c73c7ee2d549c7a39fddd3e0449677f3c201f0f4c6c540c93ab38f8aab03388
[INFO] Account publicHex : 0x5309fa17ae97f81f80a1da3d6b116377ace351dffdcbfd0e91fbb3bcf0312d363c78b8aaf929b3661c1f02e8b2c318358843de6a2dcc66cc0d5260a0d6874a6e
[INFO] Account Address   : 0x3d1771fb60963fbaf5906c6b2a574c77dae7c136
[INFO] Private Key (pem) : accounts/0x3d1771fb60963fbaf5906c6b2a574c77dae7c136.pem
[INFO] Public  Key (pem) : accounts/0x3d1771fb60963fbaf5906c6b2a574c77dae7c136.pem.pub
```

- 指定PEM私钥文件计算账户地址

```shell
bash get_account.sh -k accounts/0xa04beef19c812628a2aa1f0fc73e0963f84ec75e.pem
```

执行上面的命令，结果如下

```shell
[INFO] Account Address   : 0x3d1771fb60963fbaf5906c6b2a574c77dae7c136
[INFO] Account privateHex: 0x3c73c7ee2d549c7a39fddd3e0449677f3c201f0f4c6c540c93ab38f8aab03388
[INFO] Account publicHex : 0x5309fa17ae97f81f80a1da3d6b116377ace351dffdcbfd0e91fbb3bcf0312d363c78b8aaf929b3661c1f02e8b2c318358843de6a2dcc66cc0d5260a0d6874a6e
```

#### 3. 使用脚本生成PKCS12格式私钥

- 生成私钥与地址

```shell
bash get_account.sh -p
```

执行上面的命令，可以得到类似下面的输出，按照提示输入密码，生成对应的p12文件和pub文件。

```shell
[INFO] Account privateHex: 0x7fe388c7617b7b2241f0a67a789f38f29dd731ae6ce3510302ab26fd429d65ce
[INFO] Account publicHex : 0x8d90d7e3e5b88b62941b23db10f35d13d1a06265e14993d43dff8562d23fded7466c006b74fa04b71c86c20a27ca20915e2f5e01dc466af9e080ff71229957e8
[INFO] Note: the entered password cannot contain Chinese characters!
Warning: -chain option ignored with -nocerts
Enter Export Password:
Verifying - Enter Export Password:
[INFO] Account Address   : 0x6a707388d949ca429c1473e6c5ce334ad9c455be
[INFO] Private Key (p12) : accounts/0x6a707388d949ca429c1473e6c5ce334ad9c455be.p12
[INFO] Public  Key (pem) : accounts/0x6a707388d949ca429c1473e6c5ce334ad9c455be.pem.pub
```

- 指定p12私钥文件计算账户地址，**按提示输入p12文件密码**

```shell
bash get_account.sh -P accounts/0x6a707388d949ca429c1473e6c5ce334ad9c455be.p12
```

执行上面的命令，结果如下

```shell
Enter Import Password:
[INFO] Account Address   : 0x6a707388d949ca429c1473e6c5ce334ad9c455be
[INFO] Account privateHex: 0x7fe388c7617b7b2241f0a67a789f38f29dd731ae6ce3510302ab26fd429d65ce
[INFO] Account publicHex : 0x8d90d7e3e5b88b62941b23db10f35d13d1a06265e14993d43dff8562d23fded7466c006b74fa04b71c86c20a27ca20915e2f5e01dc466af9e080ff71229957e8
```

## 账户的存储

- Java SDK支持通过私钥字符串或者文件加载，所以账户的私钥可以存储在数据库中或者本地文件。
- 本地文件支持两种存储格式，其中PKCS12加密存储，而PEM格式明文存储。
- 开发业务时可以根据实际业务场景选择私钥的存储管理方式。

## 账户的使用

### 控制台加载私钥文件

控制台提供账户生成脚本get_account.sh，生成的的账户私钥文件在accounts目录下，控制台加载私钥时需要指定私钥文件。
控制台启动方式有如下几种：

```shell
bash start.sh
bash start.sh group0
bash start.sh group0 -pem pemName
bash start.sh group0 -p12 p12Name
```

#### 默认启动

控制台随机生成一个账户，使用控制台配置文件指定的群组号启动。

```shell
bash start.sh
```

#### 指定群组号启动

控制台随机生成一个账户，使用命令行指定的群组名启动。

```shell
bash start.sh group0
```

- 注意：指定的群组在控制台配置文件中需要配置bean。

#### 使用PEM格式私钥文件启动

- 使用指定的pem文件的账户启动，输入参数：群组号、-pem、pem文件路径

```shell
bash start.sh group0 -pem accounts/0xebb824a1122e587b17701ed2e512d8638dfb9c88.pem
```

#### 使用PKCS12格式私钥文件启动

- 使用指定的p12文件的账户，需要输入密码，输入参数：群组名、-p12、p12文件路径

```shell
bash start.sh group0 -p12 accounts/0x5ef4df1b156bc9f077ee992a283c2dbb0bf045c0.p12
Enter Export Password:
```

### Java SDK加载私钥文件

如果通过账户生成脚本get_accounts.sh生成了PEM或PKCS12格式的账户私钥文件，则可以通过加载PEM或PKCS12账户私钥文件使用账户。加载私钥有两个类：P12Manager和PEMManager，其中，P12Manager用于加载PKCS12格式的私钥文件，PEMManager用于加载PEM格式的私钥文件。

- P12Manager用法举例：

使用代码加载私钥。

```java
// 初始化BcosSDK
BcosSDK sdk =  BcosSDK.build(configFile);
// 为群组group0初始化client
Client client = sdk.getClient("group0");
// 通过client获取CryptoSuite对象
CryptoSuite cryptoSuite = client.getCryptoSuite();
// 加载pem账户文件
cryptoSuite.loadAccount("p12", p12AccountFilePath, password);
```

- PEMManager使用举例

使用代码加载私钥。

```java
// 初始化BcosSDK
BcosSDK sdk =  BcosSDK.build(configFile);
// 为群组group0初始化client
Client client = sdk.getClient("group0");
// 通过client获取CryptoSuite对象
CryptoSuite cryptoSuite = client.getCryptoSuite();
// 加载pem账户文件
cryptoSuite.loadAccount("pem", pemAccountFilePath, null);
```

## 账户地址的计算

FISCO BCOS的账户地址由ECDSA公钥计算得来，对ECDSA公钥的16进制表示计算keccak-256sum哈希，取计算结果的后20字节的16进制表示作为账户地址，每个字节需要两个16进制数表示，所以账户地址长度为40。FISCO BCOS的账户地址与以太坊兼容。
注意keccak-256sum与`SHA3`**不相同**，详情参考[这里](https://ethereum.stackexchange.com/questions/550/which-cryptographic-hash-function-does-ethereum-use)。

[以太坊地址生成](https://kobl.one/blog/create-full-ethereum-keypair-and-address/)

### 1. 生成ECDSA私钥

首先，我们使用OpenSSL生成椭圆曲线私钥，椭圆曲线的参数使用secp256k1。执行下面的命令，生成PEM格式的私钥并保存在ecprivkey.pem文件中。

```shell
openssl ecparam -name secp256k1 -genkey -noout -out ecprivkey.pem
```

执行下面的指令，查看文件内容。

```shell
cat ecprivkey.pem
```

可以看到类似下面的输出。

```shell
-----BEGIN EC PRIVATE KEY-----
MHQCAQEEINHaCmLhw9S9+vD0IOSUd9IhHO9bBVJXTbbBeTyFNvesoAcGBSuBBAAK
oUQDQgAEjSUbQAZn4tzHnsbeahQ2J0AeMu0iNOxpdpyPo3j9Diq3qdljrv07wvjx
zOzLpUNRcJCC5hnU500MD+4+Zxc8zQ==
-----END EC PRIVATE KEY-----
```

接下来根据私钥计算公钥，执行下面的指令

```shell
openssl ec -in ecprivkey.pem -text -noout 2>/dev/null| sed -n '7,11p' | tr -d ": \n" | awk '{print substr($0,3);}'
```

可以得到类似下面的输出

```shell
8d251b400667e2dcc79ec6de6a143627401e32ed2234ec69769c8fa378fd0e2ab7a9d963aefd3bc2f8f1cceccba54351709082e619d4e74d0c0fee3e67173ccd
```

### 2. 根据公钥计算地址

本节我们根据公钥计算对应的账户地址。我们需要获取keccak-256sum工具，可以从[这里下载](https://github.com/vkobel/ethereum-generate-wallet/tree/master/lib)。

```shell
openssl ec -in ecprivkey.pem -text -noout 2>/dev/null| sed -n '7,11p' | tr -d ": \n" | awk '{print substr($0,3);}' | ./keccak-256sum -x -l | tr -d ' -' | tail -c 41
```

得到类似下面的输出，就是计算得出的账户地址。

```shell
dcc703c0e500b653ca82273b7bfad8045d85a470
```

## 账户管理

```eval_rst
.. important::
   账户的管理冻结、解冻、废止操作，均需要将开启区块链权限模式，详情请参考`权限治理使用指南 <https://fisco-bcos-doc.readthedocs.io/zh_CN/latest/docs/develop/committee_usage.html>`_
```

在开启区块链权限模式之后，每次发起合约调用均会检查账户状态是否正常（tx.origin），账户的状态以存储表的形式记录在BFS `/usr/` 目录下，存储表名为 `/usr/` + 账户地址，如果查不到账户状态默认该账户为正常。BFS `/usr/` 目录底下的账户状态只有在主动设置账户状态时才会创建。**账户状态管理的接口只有治理委员才能操作。**

治理委员可以通过AccountManagerPrecompiled的接口对账户进行操作，固定地址为0x10003。

```solidity
enum AccountStatus{
    normal,
    freeze,
    abolish
}

abstract contract AccountManager {
    // 设置账户状态，只有治理委员可以调用，0 - normal, others - abnormal, 如果账户不存在会先创建
    function setAccountStatus(address addr, AccountStatus status) public virtual returns (int32);
    // 任何用户都可以调用
    function getAccountStatus(address addr) public view virtual returns (AccountStatus);
}
```

### 账户的冻结、解冻、废止

治理委员可以对固定地址0x10003的预编译合约发起交易、对账户的状态进行读写。在执行操作时，将会确定交易发起人msg.sender是否为治理委员会记录中的治理委员，若不是则会拒绝。值得注意的是，治理委员的账户地址不允许修改状态。

治理委员也可以通过控制台对账户进行冻结、解冻、废止等操作，详情请看：[冻结/解冻账户命令](../operation_and_maintenance/console/console_commands.html#freezeaccount-unfreezeaccount)、[废止账户命令](../operation_and_maintenance/console/console_commands.html#abolishaccount)
