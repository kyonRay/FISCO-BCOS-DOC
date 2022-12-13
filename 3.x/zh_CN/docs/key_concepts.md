# 关键概念

区块链是由多个学科交叉组合形成的一门技术，本章将阐述区块链相关的基本概念，对涉及的基本理论进行科普介绍。如果您已经对这些基本技术很熟悉，可以跳过本章。

## 区块链是什么

区块链（blockchain）是在比特币之后提出的一个概念，在中本聪关于比特币的[论文](https://bitcoin.org/bitcoin.pdf)中没有直接引入blockchain的概念，而是以chain of block来描述一种数据结构。

Chain of block是指由多个区块通过哈希（hash）串联成一条链式结构的数据组织方式。区块链则是采用多项技术交叉组合，维护管理这个chain of block数据结构，形成一个不可篡改的分布式账本的综合技术领域。

区块链技术是一种在对等网络环境下，通过透明和可信规则，构建不可伪造、难以篡改和可追溯的块链式数据结构，实现和管理可信数据的产生、存取和使用的模式。技术架构上，区块链是由分布式架构与分布式存储、块链式数据结构、点对点网络、共识算法、密码学算法、博弈论、智能合约等多种信息技术共同组成的整体解决方案。

区块链技术和生态起源于比特币，随着金融、司法、供应链、文化娱乐、社会管理、物联网等更多行业对此领域技术的关注，希望将其技术价值应用到更广泛的分布式协作中，区块链技术和产品模式也在持续进化，FISCO BCOS区块链底层平台在区块链技术基础上，专注提升安全、性能、可用性、易用性、隐私保护、合规监管等方面的能力，和业界生态共同发展，体现多方参与、智能协同、专业分工、价值分享的效能。

### 账本

账本顾名思义，用于管理账户、交易流水等数据，支持分类记账、对账、清结算等功能。在多方合作中，多个参与方希望共同维护和共享一份及时、正确、安全的分布式账本，以消除信息不对称，提升运作效率，保证资金和业务安全。而区块链通常被认为是用于构建“分布式共享账本”的一种核心技术，通过链式的区块数据结构、多方共识机制、智能合约、世界状态存储等一系列技术的共同作用，可实现一致、可信、事务安全、难以篡改可追溯的共享账本。

账本里包含的基本内容有区块，交易，账户，世界状态。

#### 区块

区块是按时间次序构建的数据结构，区块链的第一个区块称为“创世块”（genesis block），后续生成的区块用“高度”标识，每个区块高度逐一递增，新区块都会引入前一个区块的hash信息，再用hash算法和本区块的数据生成唯一的数据指纹，从而形成环环相扣的块链状结构，称为“Blockchain”也即区块链。精巧的数据结构设计，使得链上数据按发生时间保存，可追溯可验证，如果修改任何一个区块里的任意一个数据，都会导致整个块链验证不通过，从而篡改的成本会很高。

一个区块的基本数据结构是区块头和区块体，区块头包含区块高度，hash、出块者签名、状态树根等一些基本信息，区块体里包含一批交易数据列表已经相关的回执信息，根据交易列表的大小，整个区块的大小会有所不同，考虑到网络传播等因素，一般不会太大，在1M~几M字节之间。

#### 交易

交易可认为是一段发往区块链系统的请求数据，用于部署合约，调用合约接口，维护合约的生命周期，以及管理资产，进行价值交换等，交易的基本数据结构包括发送者，接受者，交易数据等。用户可以构建一个交易，用自己的私钥给交易签名，发送到链上（通过sendRawTransaction等接口），由多个节点的共识机制处理，执行相关的智能合约代码，生成交易指定的状态数据，然后将交易打包到区块里，和状态数据一起落盘存储，该交易即为被确认，被确认的交易被认为具备了事务性和一致性。

随着交易确认相应还会有交易回执（receipt）产生，和交易一一对应且保存在区块里，用于保存一些交易执行过程生成的信息如结果码、日志、消耗的gas量等。用户可以使用交易hash检查交易回执，判定交易是否完成。

和“写操作”的交易对应，还有一种"只读"调用方式，用于读取链上数据，节点收到请求后会根据请求的参数访问状态信息并返回，并不会将请求加入共识流程，也不会导致修改链上的数据。

#### 账户

在采用账户模型设计的区块链系统里，账户这个术语代表着用户、智能合约的唯一性存在。

在采用公私钥体系的区块链系统里，用户创建一个公私钥对，经过hash等算法换算即得到一个唯一性的地址串，代表这个用户的账户，用户用该私钥管理这个账户里的资产。用户账户在链上不一定有对应的存储空间，而是由智能合约管理用户在链上的数据，因此这种用户账户也会被称为“外部账户”。

对智能合约来说，一个智能合约被部署后，在链上就有了一个唯一的地址，也称为合约账户，指向这个合约的状态位、二进制代码、相关状态数据的索引等。智能合约运行过程中，会通过这个地址加载二进制代码，根据状态数据索引去访问世界状态存储里对应的数据，根据运行结果将数据写入世界状态存储，更新合约账户里的状态数据索引。智能合约被注销时，主要是更新合约账户里的合约状态位，将其置为无效，一般不会直接清除该合约账户的实际数据。

#### 世界状态

FISCO BCOS采用“账户模型”的设计，即除了区块和交易的存储空间外，还会有一块保存智能合约运行结果的存储空间。智能合约执行过程产生的状态数据，经过共识机制确认，分布式的保存在各节点上，数据全局一致，可验证难篡改，所以称为“世界状态”。

状态存储空间的存在，使得区块链上可以保存各种丰富的数据，包括用户账户信息如余额等，智能合约二进制码，智能合约运行结果等相关的各种数据，智能合约执行过程中会从状态存储中获取一些数据参与运算，为实现复杂的合约逻辑提供了基础。

另一方面，维护状态数据需要付出不少存储成本，随着链的持续运行，状态数据会持续膨胀，如采用复杂的数据结构如帕特里夏树（Patricia Tree），状态数据的的容量会进一步扩大，根据不同的场景需要，可对状态数据进行裁剪优化，或采用分布式数据仓库等方案存储，以支持更海量的状态数据容量。

### 共识机制

共识机制是区块链领域的核心概念，无共识，不区块链。区块链作为一个分布式系统，可以由不同的节点共同参与计算、共同见证交易的执行过程，并确认最终计算结果。协同这些松散耦合、互不信任的参与者达成信任关系，并保障一致性，持续性协作的过程，可以抽象为“共识”过程，所牵涉的算法和策略统称为共识机制。

#### 节点

安装了区块链系统所需软硬件，加入到区块链网络里的计算机，可以称为一个“节点”。节点参与到区块链系统的网络通信、逻辑运算、数据验证，验证和保存区块、交易、状态等数据，并对客户端提供交易处理和数据查询的接口。节点的标识采用公私钥机制，生成一串唯一的NodeID，以保证它在网络上的唯一性。

根据对计算的参与程度和数据的存量，节点可分为共识节点和观察节点。共识节点会参与到整个共识过程，做为记账者打包区块、做为验证者验证区块以完成共识过程。观察节点不参与共识，同步数据，进行验证并保存，可以做为数据服务者提供服务。

#### 共识算法

共识算法需要解决的几个核心问题是：

1. 选出在整个系统中具有记账权的角色，做为leader发起一次记账。
2. 参与者采用不可否认和不能篡改的算法，进行多层面验证后，采纳Leader给出的记账。
3. 通过数据同步和分布式一致性协作，保证所有参与者最终收到的结果都是一致的，无错的。

区块链领域常见的共识算法有公链常用的工作量证明（Proof of Work）,权益证明（Proof of Stake），委托权益证明（Delegated Proof of Stake），以及联盟链常用的实用性拜占庭容错共识PBFT（Practical Byzantine Fault Tolerance），Raft等，另外一些前沿性的共识算法通常是将随机数发生器和上述几个共识算法进行有机组合，以改善安全、能耗以及性能和规模问题。

FISCO BCOS共识模块采用插件化的设计，可支持多种共识算法，当前包括PBFT和Raft，后续将会持续实现更大规模，速度更快的共识算法。

### 智能合约

智能合约概念于1995年由Nick Szabo首次提出，指以数字形式定义的能自动执行条款的合约，数字形式意味着合约必须用计算机代码实现，因为只要参与方达成协定，智能合约建立的权利和义务，就会被自动执行，且结果不能被否认。

FISCO BCOS运用智能合约不仅用于资产管理、规则定义和价值交换，还可以用来进行全局配置、运维治理、权限管理等。

#### 智能合约生命周期

智能合约的生命周期为设计，开发,测试,部署,运行,升级,销毁等几个步骤。

开发人员根据需求，进行智能合约代码的编写，编译，单元测试。合约开发语言可包括solidity,C++,java,go,javascript,rust等，语言的选择根据平台虚拟机选型而定。在合约通过测试后，采用部署指令发布到链上，经过共识算法确认后，合约生效并被后续的交易调用。

当合约需要更新升级时，重复以上开发到部署的步骤，发布新版合约，新版合约会有一个新的地址和独立的存储空间，并不是覆盖掉旧合约。新版合约可通过旧合约数据接口访问旧版本合约里保存的数据，或者通过数据迁移的方式将旧合约的数据迁移到新合约的存储里，最佳实践是设计执行流程的“行为合约”和保存数据的“数据合约”，将数据和合约解耦，当业务流程产生改变，而业务数据本身没有改变时，新行为合约直接访问原有的“数据合约”即可。

销毁一个旧合约并不意味着清除合约的所有数据，只是将其状态置为“无效”，该合约则不可再被调用。

#### 智能合约虚拟机

为了运行数字智能合约，区块链系统必须具备可编译、解析、执行计算机代码的编译器和执行器，统称为虚拟机体系。合约编写完毕后，用编译器编译，发送部署交易将合约部署到区块链系统上，部署交易共识通过后，系统给合约分配一个唯一地址和保存合约的二进制代码，当某个合约被另一个交易调用后，虚拟机执行器从合约存储里加载代码并执行，并输出执行结果。

在强调安全性、事务性和一致性的区块链系统里，虚拟机应具有沙盒特征，屏蔽类似随机数、系统时间、外部文件系统、网络等可能导致不确定性的因素，且可以抵抗恶意代码的侵入，以保证在不同节点上同一个交易和同一个合约的执行生成的结果是一致的，执行过程是安全的。

当前流行的虚拟机机制包括EVM， 受控的Docker，WebAssembly等，FISCO BCOS的虚拟机模块采用模块化设计，已经支持受到社区广泛欢迎的EVM，将会支持更多的虚拟机。

#### 图灵完备

图灵机和图灵完备是计算机领域的经典概念，由数学家艾伦·麦席森·图灵（1912～1954）提出的一种抽象计算模型，引申到区块链领域，主要指合约支持判断、跳转、循环、递归等逻辑运算，支持多种数据类型如整形、字符串、结构体的数据处理能力，甚至有一定的面向对象特性如继承、派生、接口等，这样才能支持复杂的业务逻辑和完备的契约执行，与只支持栈操作的简单脚本进行区分。

2014年后出现的区块链大多支持图灵完备的智能合约，使得区块链系统具备更高的可编程性，在区块链既有的基本特性（如多方共识，难以篡改，可追溯等，安全性等）基础上，还可以实现具有一定业务逻辑的业务契约，如李嘉图合约（The Ricardian Contract），也可以使用智能合约来实现。

合约的执行还需要处理“停机问题”，即判断程序是否会在有限的时间之内解决输入的问题，并结束执行，释放资源。想象一下，一个合约在全网部署，在被调用时在每个节点上都会执行，如果这个合约是个无限循环，就意味着可能会耗尽整个体系的资源。所以停机问题的处理也是区块链领域里图灵完备计算体系的一个重要关注点。

## 联盟链概念分析

行业里通常将区块链的类型分为公有链，联盟链，私有链。公有链指所有人都可以随时随地参与甚至是匿名参与的链；私有链指一个主体（如一个机构或一个自然人）所有，私有化的管理和使用的链；联盟链通常是指多个主体达成一定的协议，或建立了一个业务联盟后，多方共同组建的链，加入联盟链的成员需要经过验证，一般是身份可知的。正因为有准入机制，所以联盟链也通常被称为“许可链”。

因为联盟链从组建、加入、运营、交易等环节有准入和身份管理，在链上的操作可以用权限进行管控，共识方面一般采用PBFT等基于多方多轮验证投票的共识机制，不采用POW挖矿的高能耗机制，网络规模相对可控，在交易时延性、事务一致性和确定性、并发和容量方面都可以进行大幅的优化。

联盟链在继承区块链技术的优势的同时，更适合性能容量要求高，强调监管、合规的敏感业务场景，如金融、司法、以及大量和实体经济相关的业务。联盟链的路线，兼顾了业务合规稳定和业务创新，也是国家和行业鼓励发展的方向。

### 性能

#### 性能指标

软件系统的处理性能指标最常见的是TPS（Transaction Per Second），即系统每秒能处理和确认的交易数，TPS越高，性能越高。区块链领域的性能指标除了TPS之外，还有确认时延，网络规模大小等。

确认时延是指交易发送到区块链网络后，经过验证、运算和共识等一系列流程后，到被确认时所用的时间，如比特币网络一个区块是10分钟，交易被大概率确认需要6个区块，即一个小时。采用PBFT算法的话，可以使交易在秒级确认，一旦确认即具有最终确定性，更适合金融等业务需求。

网络规模指在保证一定的TPS和确认时延前提下，能支持多少共识节点的协同工作。业界一般认为采用PBFT共识算法的系统，节点规模在百级左右，再增加就会导致TPS下降，确认时延增加。目前业界有通过随机数算法选择记账组的共识机制，可以改善这个问题。

#### 性能优化

性能的优化有两个方向,向上扩展（Scale up）和平行扩展（Scale out）。向上扩展指在有限的资源基础上优化软硬件配置，极大提升处理能力，如采用更有效率的算法，采用硬件加速等。平行扩展指系统架构具有良好的可扩展性，可以采用分片、分区的方式承载不同的用户、业务流的处理，只要适当增加软硬件资源，就能承载更多的请求。

性能指标和软件架构，硬件配置如CPU、内存、存储规格、网络带宽都密切相关，且随着TPS的增加，对存储容量的压力也会相应增加，需要综合考虑。

### 安全性

安全性是个很大的话题，尤其是构建在分布式网络上多方参与的区块链系统。在系统层面，需要关注网络攻击、系统渗透、数据破坏和泄漏的问题，在业务层面需要关注越权操作、逻辑错误、系统稳定性造成的资产损失、隐私被侵害等问题。

安全性的保障要关注"木桶的短板“，需要有综合性的防护策略，提供多层面，全面的安全防护，满足高要求的安全标准，并提供安全方面的最佳实践，对齐所有参与者的安全级别，保障全网安全。

#### 准入机制

准入机制指在无论是机构还是个人组建和加入链之前，需要满足身份可知、资质可信，技术可靠的标准，主体信息由多方共同审核后，才会启动联盟链组建工作，然后将经过审核的主体的节点加入到网络，为经过审核的人员分配可发送交易的公私钥。
在准入完成后，机构、节点、人员的信息都会登记到链上或可靠的信息服务里，链上的一切行为都可以追溯到机构和人。

#### 权限控制

联盟链上权限控制即不同人员对各种敏感级别的数据读写的控制，细分可以罗列出如合约部署、合约内数据访问、区块数据同步、系统参数访问和修改、节点启停等不同的权限，根据业务需要，还可以加入更多的权限控制点。

权限是分配给角色的，可沿用典型的基于角色的权限访问控制（Role-Based Access Control）设计，一个参考设计是将角色分为运营管理者，交易操作员，应用开发者，运维管理者，监管方，每个角色还可以根据需要细分层级，完备的模型可能会很庞大复杂，可以根据场景需要进行适当的设计，能达到业务安全可控的程度即可。

#### 隐私保护

基于区块链架构的业务场景要求各参与方都输出和共享相关数据，以共同计算和验证，在复杂的商业环境中，机构希望自己的商业数据受控，在越来越被重视的个人数据隐私保护的形势下，个人对隐私保护的诉求也日益增强。如何对共享的数据牵涉隐私的部分进行保护，以及在避免运作过程泄漏隐私，是一个很重要的问题。

隐私保护首先是个管理问题，要求在构建系统开展业务时，把握“最小授权，明示同意的原则”，对数据的收集、存储、应用、披露、删除、恢复全生命周期进行管理，建立日常管理和应急管理制度，在高敏感业务场景设定监管角色，引入第三方检视和审计，从事先事中事后全环节进行管控。

在技术上，可以采用数据脱敏，业务隔离或者系统物理隔离等方式控制数据分发范围，同时也可以引入密码学方法如零知识证明、安全多方计算、环签名、群签名、盲签名等，对数据进行高强度的加密保护。

#### 物理隔离

这个概念主要用于隐私保护领域，“物理隔离”是避免隐私数据泄露的彻底手段，物理隔离指只有共享数据的参与者在网络通信层互通，不参与共享数据的参与者在网络互相都不能通信，不交换哪怕一个字节的数据。

相对而言的是逻辑隔离，参与者可以接收到和自己无关的数据，但数据本身带上权限控制或加密保护，使得没有授权或密钥的参与者不能访问和修改。但随着技术的发展，所受到的权限受控数据或加密数据在若干年后依旧有可能被破解。

对极高敏感性的数据，可以采用“物理隔离”的策略，从根源上杜绝被破解的可能性。相应的成本是需要仔细甄别数据的敏感级别，对隔离策略进行周密的规划，并分配足够的硬件资源承载不同的数据。

### 治理与监管

#### 联盟链治理

联盟链治理牵涉到多参与方协调工作，激励机制，安全运营，监管审计等一系列的问题，核心是理清各参与方的责权利，工作流程，构建顺畅的开发和运维体系，以及保障业务的合法合规，对包括安全性在内的问题能事先防范事后应急处理。为达成治理，需要制定相关的规则且保证各参与方达成共识并贯彻执行。

一个典型的联盟链治理参考模型是各参与方共同组建联盟链委员会，共同讨论和决议，根据场景需要设定各种角色和分配任务，如某些机构负责开发，某些机构参与运营管理，所有机构参与交易和运维，采用智能合约实现管理规则和维护系统数据，委员会和监管机构可掌握一定的管理权限，对业务、机构、人员进行审核和设置，并在出现紧急情况时，根据事先约定的流程，通过共识过的智能合约规则，进行应急操作，如账户重置，业务调整等，在需要进行系统升级时，委员会负责协调各方进行系统更新。

在具备完善治理机制的联盟链上，各参与方根据规则进行点对点的对等合作，包括资产交易、数据交换，极大程度提升运作效率，促进业务创新，同时合规性和安全性等方面也得到了保障。

#### 快速部署

构建一个区块链系统的大致步骤包括：获取硬件资源包括服务器、网络、内存、硬盘存储等，进行环境配置包括选择指定操作系统、开通网络端口和相关策略、带宽规划、存储空间分配等，获取区块链二进制可运行软件或者从源码进行编译，然后进行区块链系统的配置，包括创世块配置、运行时参数配置，日志配置等，进行多方互联配置，包括节点准入配置、端口发现、共识参与方列表等，客户端和开发者工具配置，包括控制台、SDK等，这个过程会包括许多细节，如各种证书和公私钥的管理等，很容易出现环境、版本、配置的差错，导致整个过程复杂、繁琐和反复，形成了较高的使用门槛。

如何将以上步骤简化和加速，使构建和组链过程变得简便，快速，不容易出错，且低成本，需要从以下几方面进行考虑：
首先，标准化目标部署平台，事先将操作系统、依赖软件列表、网络带宽和存储容量、网络策略等关键的软硬件准备好，对齐版本和参数，使得平台可用，依赖完备。当下流行的云服务，docker等方式都可以帮助构建这样的标准化平台。

然后，从使用者的视角出发，优化区块链软件的构建、配置和组链流程，提供快速构建，自动组链的工具，使得使用者不需要关注诸多细节，简单的几步操作即可运行起供开发调试、上线运行的链。

FISCO BCOS非常重视使用者的部署体验，提供了一键部署的命令行，帮助开发者快速搭建开发调试环境，提供企业级搭链工具，面向多机构联合组链的场景，灵活的进行主机、网络等参数配置，管理相关的证书，便于多个企业之间协同工作。经过快速部署的优化，将使用者搭起区块链的时间缩短到几分钟到半小时以内。

#### 数据治理

区块链强调数据层层验证，历史记录可追溯，常见的方案是从创世块以来，所有的数据都会保存在所有的参与节点上（轻节点之外），导致的结果是数据膨胀，容量紧张，尤其是在承载海量服务的场景里，在一定时间之后，一般的存储方案已经无法容纳数据，而海量存储成本很高，另一个角度是安全性，全量数据永久保存，可能面临历史数据泄露的风险，所以需要在数据治理方面进行设计。

数据治理主要是几个策略：裁剪迁移，平行扩容，分布式存储。如何选择需要结合场景分析。

对具有较强时间特征的数据，如某业务的清结算周期是一个星期，那么一个星期前的数据不需要参与在线计算和验证，旧的数据则可以从节点迁移到大数据存储里，满足数据可查询可验证的需求以及业务保存年限的要求，线上节点的数据压力大幅降低，历史数据离线保存，在安全策略上也可以进行更严密的保护。

对规模持续扩大的业务，如用户数或合同存证量剧增，可以针对不同的用户和合同，分配到不同的逻辑分区，每个逻辑分区有独立的存储空间，只承载一定容量的数据，当接近容量的上限，则再分配更多资源容纳新的数据。分区的的设计使得在资源调配，成本管理上都更容易把控。

结合数据裁剪迁移和平行扩容，数据的容量成本，安全级别都得到很好的控制，便于开展海量规模的业务。

#### 运维监控

区块链系统从构建和运行逻辑上都具有较高一致性，不同节点的软硬件系统基本一致。其标准化特性给运维人员带来了便利，可使用通用的工具、运维策略和运维流程等对区块链系统进行构建、部署、配置、故障处理，从而降低运维成本以及提升效率。

运维人员对联盟链的操作会被权限系统控制，运维人员有修改系统配置、启停进程、查看运行日志、排查故障等权限，但不参与到业务交易中，也不能直接查看具有较高安全隐私等级的用户数据，交易数据。

系统运行过程中，可通过监控系统对各种运行指标进行监控，对系统的健康程度进行评估，当出现故障时发出告警通知，便于运维快速反应，进行处理。

监控的维度包括基础环境监控,如CPU占比、系统内存占比和增长、磁盘IO情况、网络连接数和流量等。

区块链系统监控包括如区块高度、交易量和虚拟机计算量，共识节点出块投票情况等。

接口监控包括如接口调用计数、接口调用耗时情况、接口调用成功率等。

监控数据可以通过日志或网络接口进行输出，便于和机构的现有的监控系统进行对接，复用机构的监控能力和既有的运维流程。运维人员收到告警后，采用联盟链提供的运维工具，查看系统信息、修改配置、启停进程、处理故障等。

#### 监管审计

随着区块链技术和业务形态探索的发展，需要在区块链技术平台上提供支持监管的功能，避免区块链系统游离于法律法规以及行业规则之外，成为洗钱、非法融资或犯罪交易的载体。

审计功能主要用于满足区块链系统的审计内控、责任鉴定和事件追溯等要求，需要以有效的技术手段，配合业务所属的行业标准进行精确的审计管理。

监管者可以做为节点接入到区块链系统里，或者通过接口和区块链系统进行交互，监管者可同步到所有的数据进行审计分析，跟踪全局的业务流程，如发现异常，可以向区块链发出具备监管权限的指令，对业务、参与人、账户等进行管控，实现“穿透式监管”。

FISCO BCOS在角色和权限设计，功能接口，审计工具等方面都对监管审计进行了支持。