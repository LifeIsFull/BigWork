# 基于属性的访问控制  Fabric实现

## 零、摘要

物联网设备：具有移动性、性能受限、分布式部署等特点，使得传统的集中式访问控制方法难以在当前大规模物联网环境下支持访问控制。

针对这些挑战，本文提出了一种基于Hyperledger结构区块链框架和基于属性的访问控制（ABAC）的物联网访问控制系统 **fabric-IoT**。该系统包含三种智能合约：设备合约（DC）、策略合约（PC）和访问合约（AC）。

DC提供了一种存储设备产生的资源数据的URL的方法，以及一种查询方法。

PC为管理员用户提供管理ABAC策略的功能。

AC是实现普通用户访问控制方法的核心程序。

fabric iot结合ABAC和区块链技术，可以在iot中提供分散、细粒度和动态的访问控制管理。为了验证该系统的性能，设计了两组仿真实验。结果表明，fabric-iot能够在大规模请求环境下保持较高的吞吐量，并在分布式系统中有效地达成一致，保证数据的一致性。

## 一、说明

随着互联网和计算机硬件的发展，越来越多的设备通过无线网络相互连接，使得物联网的规模越来越大。物联网是由大量传感器和网关组成的分布式网络。物联网设备一直与环境交互，产生不同类型的数据资源，如图像、音频、视频、数字信号等，物联网的所有系统和应用都可以接入互联网，实现资源和信息的高效共享[1]。

也就是说，我们生活在一个万物相连的世界。然而，由于物联网设备的分布式部署和规模庞大，设备资源的访问控制面临着巨大的挑战。物联网设备所产生的资源往往含有隐私和敏感数据，一旦被非法获取将产生严重后果。访问控制技术是保护资源的重要手段，已广泛应用于各种系统和环境中[3]。
传统的访问控制方法包括自主访问控制（DAC）、基于身份的访问控制（IBAC）、强制访问控制（MAC），但这些方法都是集中式设计，存在单点失效、扩展困难、可靠性低、吞吐量低等缺点。事实上，物联网设备可能属于不同的组织或用户，并且可能具有移动性和有限的性能，这使得集中式访问控制难以满足物联网环境下访问控制的要求。

基于属性的访问控制（ABAC）是一种逻辑访问控制模型，它根据条目、操作和相关环境的属性来控制主体和对象之间的访问[4]。ABAC首先分别提取用户（主体）、资源（对象）、权限和环境的属性，然后灵活组合这些属性之间的关系，最后将权限管理转化为属性管理，提供了一种细粒度的动态访问管理方法。
区块链[5]是另一种新兴的数据管理技术，它通过分布式存储来保证数据的可靠性。将数据的读取或修改记录作为事务写入一个块中，并通过哈希算法将这些块作为链进行链接，以保证数据的完整性。通过P2P网络和共识算法实现节点间的数据同步，使区块链网络中的各方达成共识，以确保数据一致性。Hyperledger Fabric[6]是一个开源的区块链开发平台，它不仅具有区块链的分散式账本、不可变、群体共识等特点，还提供了更高效的共识机制、更高的吞吐量、智能合约以及对多个组织和账本的支持。
本文将区块链技术应用于物联网访问控制，设计并实现了一个基于Hyperledger Fabric和ABAC的物联网访问控制系统fabic IoT。fabric-iot采用分布式体系结构，可以跟踪记录，提供动态的访问控制管理，解决物联网中的访问控制问题。

本文的主要贡献有：
1）根据物联网设备在现实生活中产生的数据，定义了一个设备资源共享模型。该模型使设备生成的数据资源与URL一一对应，大大简化了设备资源的共享方式和存储结构。
2） 提出了一种基于区块链的物联网访问控制系统fabric-IoT，并详细描述了其工作流程和体系结构。系统采用分布式体系结构将用户和设备分开，实现了权限的动态管理，支持高效的访问。
3） 基于Hyperledger Fabric平台，我们设计了**三种智能合约**。第一个实现了ABAC模型。第二个实现ABAC策略管理。最后一个实现了设备资源管理。
4） 详细介绍了物联网的网络初始化、链码安装和智能合约调用。
5） 我们设计了两组对比实验来验证系统的性能和一致性速度。

本文的剩余部分组织如下。
第二节介绍了相关工作。第三节详细介绍了Hyperledger Fabric的结构和运行机制，以及ABAC模型。第四节介绍了资源模型、策略模型、系统结构、工作流程以及智能合约的实现。第五节介绍了如何建立fabric物联网系统以及如何利用它来管理物联网资源的访问控制。我们设两组对比实验，然后对结果进行分析。第六部分对本文进行了总结，并对下一步的工作进行了展望。

## 二、近期工作

物联网的快速发展对分布式访问控制提出了更高的要求。区块链技术具有分散性、数据加密性、可扩展性和非篡改性四大优势。目前，区块链技术已经发展到3.0版本。智能合约作为其核心技术，为应用构建了安全可靠的运行环境，赋予区块链更强大的功能。因此，许多学者在现有访问控制方法的基础上，结合区块链和智能合约，提出了多种物联网访问控制方法。
文献[7]提出了一种基于安全结构的数据传输技术，实现了工业物联网中的电力交易中心，解决了安全性低、管理成本高、监管难度大的问题。文献[8]提出了一种物联网设备的分散访问控制方案。该方案采用单个智能合约来降低节点间的通信开销。
名为管理中心的节点旨在与设备进行交互，避免区块链和物联网设备之间的直接交互。它具有移动性、可访问性、并发性、轻量级、可扩展性和透明性六大优点。文献[9]提出了一种基于以太坊智能合约的访问控制方案，包括三种智能合约：访问控制合约（ACC）、判断合约（JC）和注册合约（RC）。ACC通过检查对象的行为实现了基于策略的授权。JC被用来判断错误行为并返回相应的惩罚。RC用于注册上述两个智能合约，并提供更新、删除等操作。最后，用一台PC、一台笔记本和两个Raspberry-Pi实现了该体系结构。
文献[10]提出了一种基于以太坊的分布式应用（DAPP）。它将SaaS商业模式与区块链相结合，区块链用于买卖传感器数据。参考文献[11]提出了一种改进的区块链结构。在企业网络中，采用专用链对物联网设备配置文件进行分布式管理。它将设备配置文件存储在区块链上，并通过智能合约监控操作。
在参考文献[12]中，路由器节点的路由信息被保存到区块链中，以确保路由信息不被篡改和追踪。文献[13]提出了一种基于属性的物联网访问控制方法，通过区块链保存属性数据。该方法避免了数据篡改，简化了访问控制协议，满足了物联网设备的计算能力和能量约束。文献[14]提出了一种多区块链分布式密钥管理体系结构（BDKMA），并引入了一种雾计算方法来减少多链操作的延迟，可以更好地保证用户的隐私性和安全性。参考文献[15]中，区块链用于车联网的信任管理。通过将工作证明（PoW）和利害关系证明（PoS）两种共识机制相结合，可以动态地改变挖掘节点的难度，从而更有效地达成共识。文献[16]提出了一种基于雾计算的高效、安全的路况监测认证方案。文献[17]提出了一种边缘链，利用区块链的货币体系将边缘计算资源与账户和物联网设备之间的资源使用联系起来。边缘链建立了基于设备行为的信任模型，控制物联网设备从边缘服务器获取资源。在参考文献[18]中，针对无线传感器网络设计了一种可扩展的物联网安全管理体系结构，并与现有的物联网管理方案进行了性能比较。文献[19]提出了一种基于比特币的权限管理系统，允许用户发布和传输访问策略。其优点是访问策略将对所有用户开放，并防止任何一方否认策略的真实性。参考文献[20]将区块链与RBAC相结合，实现了基于以太坊的跨组织RBAC，允许小组织参与，用户可以完全控制自己的角色。参考文献[21]提出了公平访问隐私保护和权限管理框架，该框架使用智能合约对访问策略进行细粒度管理，并检查策略重用。在文献[22]中，提出了EduRSS方案。它使用基于以太坊的区块链来存储和共享教育记录。
文献[23]提出了一种新的基于信任的推荐方案（TBRS），以保证车载CPS网络中数据传输的安全性和实时性。文献[24]提出了一种基于超图的区块链模型。该模型采用超边缘结构组织存储节点，将整个网络的数据存储转化为局域网存储，降低了存储消耗，提高了安全级别。

## 三、预备知识

### Hyperledger Fabric

以比特币为代表的数字货币取得了巨大的成功，并引起了全球对区块链技术的关注。然而，这种公共链存在以下缺点：
1）事务吞吐量低。每秒只能接受大约7个事务。
2） 交易确认时间长。每笔交易大约需要1个小时才能最终确认。
3） 浪费资源。PoW机制消耗了大量的计算资源和能量。
4） 一致性问题。区块链容易形成分支。当分支形成时，只有最长的链生效，其他链中的所有事务都无效。
5） 隐私问题。由于比特币的账本是开放的，因此交易中没有隐私。

为了解决这些问题，Linux基金会在2015年启动了Hyperledger项目，构建了一个企业区块链开发平台。作为其中的一个程序：Hyperledger Fabric采用模块化结构提供可扩展的组件，包括加密、身份验证、共识算法、智能合约、数据存储等服务。超分类帐结构的所有程序都运行在docker容器中。容器提供了一个沙盒环境，将应用程序与物理资源分开，并将容器相互隔离，以确保应用程序的安全性。Hyperledger Fabric是一种联盟链，其中所有节点都需要经过授权才能加入区块链网络。在此基础上，Fabric提供了一种基于Kafka消息队列的一致性机制，可以在大规模应用场景中快速达成一致。Hyperledger结构克服了公共链的这些缺点。

1） 主要组件：CA、CLIENT、PEER、order 
CA对成员节点的数字证书进行统一管理，生成或取消成员的身份证书。
CLIENT用于与PEER节点交互，同时操作区块链系统。操作分为两类。第一类是管理类，主要用于管理节点，包括启动、停止、配置节点等；第二类是链码类，主要用于链码的生命周期管理，包括链码的安装、实例化、升级和执行等，客户端一般是命令行客户端或客户端由SDK开发的应用程序。
PEER节点是分布式系统中的一个平等节点，该组件存储区块链的账本和链码。
应用程序通过调用链码连接到PEER并查询或更新分类账。有两种类型的PEER：背书人和提交人。背书人节点负责验证、模拟和背书交易。提交人负责验证交易的合法性以及更新区块链和分类账状态。
order负责接受PEER节点发送的事务，按照一定的规则对事务进行排序，将事务按一定的顺序打包成块，并发送给PEER节点。对等节点用新的块更新本地账本，最终达成共识。
2） 通道
大多数企业应用程序的一个重要要求是数据的隐私性和机密性。Fabric设计了一个通道系统来隔离不同组织的区块链数据。在每个通道中，都有一个独立的私人账本和一个区块链。因此，Fabric是一个多渠道、多账本、多区块链的系统。
3） 分类帐
Fabric的数据以分布式分类帐的形式存储，该分类帐中的数据项以键值对的形式存储。所有键值对构成分类帐的状态，称为“世界状态”，如图1所示。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413122407714.png" alt="image-20210413122407714" style="zoom:80%;" />

4） 链码
Fabric中的智能合约称为链码。链码是用golang（支持其他编程语言，如Java）编写的程序，实现预定义的接口。交易可以通过链码生成，这是外部与区块链系统交互的唯一方式。通过提交或评估事务，外部人员可以更改或读取状态数据库（SDB）的数据，而事务将被写入Fabric的分类帐。
业务逻辑可以通过编写链码来实现，因此，开发人员可以编写不同的链码来实现不同的应用程序。

### ABAC Model

基于属性的访问控制（ABAC）是一种综合考虑主体、客体、权限、环境等属性的访问控制技术。它通过判断请求是否包含正确的属性来确定是否授予请求者访问权。由于主体和客体的属性是分开定义的，ABAC可以有效地将策略管理和访问控制分开。策略可以根据实际情况进行更改，如增加或减少策略的属性，以达到可扩展的目的。此外，还可以从多个角度定义实体的属性，实现细粒度的访问控制。属性是ABAC的核心，可以定义为包含四个元素的集合：A∈{S,O,P,E}，每个字段的含义解释如下：

A表示属性，A∈{fname：value}。属性的值具有一个键名称。
S表示主体属性，表示发起访问请求的主体的身份和特征，如身份证、年龄、姓名、职务等。
O表示对象的属性，表示被访问资源的属性，如资源类型、服务IP地址、网络协议等。
P表示权限的属性，表示主体对客体的操作，如读、写、执行等。
E表示环境的属性，表示生成访问请求时的环境信息，如时间、位置等。

ABACR（Attributed based access control request）定义为：ABACR={AS ^AO ^AP ^AE}，它是包含上述四个属性的集合。在AE（Attributes of Environment）下，表示AS（Attributes of Subject）在AO（Attributes of Object）上的AP（Attributes of Operation）。

ABACP（Attributed-based access control policy）定义为：ABACP={AS与或AO与或AP与或AE}，表示主体对客体的访问控制规则。它表示访问受保护资源所需的属性集。

## 四、系统模型和设计

### 资源和策略模型

物联网设备产生的数据种类繁多，其中大部分是非结构化数据[25]。例如，摄像头可以捕捉真实世界的图像并生成图片或视频数据，麦克风可以捕捉外部声音并生成音频数据。传感器可以捕捉温度、湿度、光线等物理信号，并将其转换为数字信号数据。这些数据大多是非结构化的，因此不能直接存储在关系数据库中。而且由于它们都是实时数据，需要及时推送到授权用户手中。一般来说，语音和视频数据都是流数据。设备采集的数据经过编码后，通过WiFi或4G推送到云服务器，最后生成资源URL。
用户可以根据HLS、RTMP等视频传输协议，通过URL来获取流媒体数据。
对于传感器数据，设备通过基于MQTT的[26]服务或其他协议将数据发送到主题。客户端授权后，订阅相应的主题（可以用URL表示），服务器将主题下的消息推送到客户端。此外，这类数据主要用于控制物联网设备进行绑定、解绑、开启、关闭、调整等操作。
通常，客户端可以通过基于HTTP的restfulapi向服务器发送请求。在服务器验证权限之后，它可以通过MQTT或其他协议将控制信号发送回设备。表1显示了几种不同类型资源的URL格式。综上所述，本文定义了一个设备资源模型：{device} $\longrightarrow${resource}$\longrightarrow${url}

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413125431672.png" alt="image-20210413125431672" style="zoom: 80%;" />

主体（用户）对对象（资源）的访问权限由访问策略定义。用户没有直接向设备请求资源，而是通过权限验证，根据区块链系统的URL获取资源数据。
设备资源和用户之间的连接如图2所示。简要工作流程如图中1-5序列号所示。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413125537997.png" alt="image-20210413125537997" style="zoom:67%;" />

1） 设备将资源分发到Internet并生成资源URL。
2） 设备将资源URL保存到区块链系统。
3） 用户通过属性请求区块链系统获得授权。
4） 区块链向授权用户发送URL。
5） 用户根据URL在Internet上下载或拉取资源数据。

结合ABAC模型和物联网设备生成数据的特点，定义设备访问控制策略模型如下： 
<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413125648097.png" alt="image-20210413125648097" style="zoom:80%;" />

**P**（Policy）：表示属性化访问控制策略。这个集合包含四个元素：AS、AO、AP和AE。
**AS**（Attribute of Subject）：表示主体（用户）的属性，包括三种类型：userID（唯一标识用户）、role（用户角色）、group（用户组）。
**AO**（Attribute of Object）：表示对象（资源）的属性，由设备ID或设备的MAC地址组成。在这个模型中，我们不直接将资源URL作为一个属性。相反，我们使用唯一标识作为设备的属性。因为在现实中，设备的网络是可变的，设备产生的数据是动态的。我们假设系统中设备的功能是单一的，每个ID或MAC一次只能对应一个资源URL。
**AP**（Arribute of permisson）：表示用户是否有权访问资源。值1表示“允许”，2表示“拒绝”。初始化AP时，默认值为1。管理员可以根据情况通过将AP的值设置为0来撤销访问授权。
**AE**（Arribute of Environment）：表示访问控制所需的环境属性。AE有三种属性：time、end time和allowed IP。Time代表策略的创建时间。End time表示策略的过期时间。当当前时间晚于结束时间时，策略将无效。允许IP的目的是防止网段外的IP地址访问系统。

### 系统结构

Fabric iot是一个基于区块链的iot访问控制系统，由用户、区块链、智能网关和设备四部分组成，如图3所示。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413130141710.png" alt="image-20210413130141710" style="zoom:80%;" />

**1） 用户**
这个系统将用户分为两类：管理员和普通用户。
管理员负责区块链系统的管理和智能网关程序的维护。管理员需要提供证书才能访问区块链系统。允许的具体操作如下。
① 添加新的智能合约。管理员可以通过API在区块链上部署新的智能合约。
② 升级合同。管理员可以上传一个新的智能合约到节点并安装它，而旧的将升级到新版本。
普通用户是指设备的所有者，通过向区块链系统发送基于属性的授权请求来获取资源URL。

**2） 区块链**
是系统的核心。所有节点在加入区块链系统之前都需要获得CA认证。区块链基于Hyperledger Fabric开发，通过智能合约实现访问控制。区块链系统公开API供用户和智能网关进行访问。它主要实现以下三个功能。
① 设备资源URL数据存储。
② 基于属性的用户权限管理。
③ 用户访问资源的身份验证。

**3） 智能网关**
作为设备与区块链系统之间的桥梁，能够接收来自设备的消息并把消息包含的URL放到区块链中，避免设备直接接入给区块链系统带来的压力。

**4） 物联网设备**
作为最大的群体，物联网设备一般不具备强大的计算能力、足够的存储空间和耐用的电池。
因此，不可能直接将物联网设备部署为区块链的对等节点。物联网设备具有唯一的MAC地址或产品ID，可以与其他设备区分开来。通常，设备可能同时属于某些用户或组。每当设备生成新资源时，都会向智能网关发送一条包含资源URL的消息。该系统采用MQTT协议作为消息传输协议。

### 工作流程

如图4所示，整个系统的工作流程主要包括四个部分。本节详细介绍了每个部分的中间步骤。使用的符号如表2所示。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413130939484.png" alt="image-20210413130939484" style="float:left;zoom:80%;" /><img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413153528824.png" alt="image-20210413153528824" style="zoom:75%;" />



**Part1**  区块链网络初始化和链码安装是系统的基本过程。这些操作需要管理员在内部网中工作。part1主要包括三个步骤。

***步骤1***  在设置Hyperledger结构网络之前，我们需要为所有成员（如peer节点、order节点、channels、users等）创建证书。所有证书都由CA生成。

CA$\longrightarrow${$Cert_{peer}$,$Cert_{order}$,$Cert_{channel}$,$Cert_{user}$}

peer节点和order节点在docker容器中运行。证书在运行之前需要打包到docker映像中。

Build(conf,Cert)$\longrightarrow^{build}$ Image$\longrightarrow^{run}$ Container                          conf:config file of the node

在所有对等节点和订购方节点都成功设置之后，我们开始创建通道。每个通道加入了一个独立的区块链和分类账。

{blockchain,ledger}$\longrightarrow^{join}$Channel

***步骤2***  到目前为止，已经建立了一个基本的Hyperledger结构网络。为了构建应用程序，我们需要设计链码。我们的链码源代码是用Golang编写的。

Code(F(x)...)$\longrightarrow$CC

管理员使用Hyperledger Fabric SDK或客户端安装CC（链码）。所有CC都将安装到对等节点。

Install(CC)$\longrightarrow^{SDK/Client}$ Peer

***步骤3***  安装链码后，需要对其进行初始化。调用函数用于初始化链码。每个实例化的链码都将作为背书保存在容器中。

Invoke(Init) $\longrightarrow^{SDK/Client}$ Peer

**part2**   制定访问控制策略并保存到区块链系统。这个过程需要用户和管理员共同决定和定制访问策略，并由管理员上传到区块链系统。

***步骤1***  管理员和用户共同制定访问策略，根据主体（用户）、对象（设备资源）、操作和环境的属性进行定义。

Decide(AS,AO, AE, AP)$\longrightarrow$ ABACP

***步骤2***   定义访问策略后，管理员将其上传到区块链网络。

Upload(ABACP)$\longrightarrow$contract

***步骤3***   管理员通过运行PolicyContract连接到区块链以添加、修改和删除策略。策略的值保存在SDB中，操作记录写入分类帐。

PolicyContract(ABACP)$\longrightarrow$ {Ledger,SDB} 

**Part3**  设备向智能网关上报资源URL，智能网关将资源URL上传到区块链系统。

***步骤1***  设备生成包含设备ID和URL的消息，并通过MQTT发送给智能网关。

{deviceId,URL} $\longrightarrow^{}$ Msg $\longrightarrow^{MQTT}$ SG                      SG:Smart Gateway

***步骤2***  智能网关解析消息并为区块链生成操作。

Translate(Msg)$\longrightarrow^{}$BA                                                   BA:Blockchain Action

***步骤3***  智能网关连接到区块链客户端以运行操作。

Run(BA)$\longrightarrow^{Cli}$DeviceContract

***步骤4***  区块链通过调用DC函数保存设备资源的URL。 DC:Device Contract

DeviceContract(deviceId,URL)$\longrightarrow^{}${Ledger,SDB}   

**Part4**  基于属性的资源获取过程是系统的核心。它包括以下三个步骤：

***步骤1***  用户发起基于属性的请求。

userId$\longrightarrow^{}$Request{AS ^ AO ^ AP}

***步骤2***  接收到请求后调用AC函数。                                AC:Access Contract

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413145929245.png" alt="image-20210413145929245" style="float: left; zoom: 67%;" />

***步骤3***  如果验证通过，区块链系统调用DC中的函数查询URL并返回给用户。如果失败，403错误（403代表HTTP状态码中的禁止）将返回给用户。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413150234326.png" alt="image-20210413150234326" style="float:left;zoom:70%;" />

### 智能合约设计

智能合约是访问控制实现的核心。
系统中有三种智能合约：策略合约（PC）、设备合约（DC）和访问合约（AC）。

#### 1）策略合约（PC）

它提供了以下操作ABACP的方法。

Auth（）：Admin为用户定义ABACP，并向区块链系统发送添加ABACP的请求。ABACPR（基于属性的访问控制策略请求）的示例如表3所示。Admin使用PC节点的公钥对数据进行加密，然后使用私钥对请求进行签名。PC调用Auth（）用admin的公钥验证其身份并用自己的私钥解密数据。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413150800788.png" alt="image-20210413150800788" style="zoom:80%;" />

CheckPolicy（）：如算法1所示。PC需要检查ABACP的有效性。一个合法的ABACP需要包含上述四个属性，每个属性的类型也需要满足要求。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413150926665.png" alt="image-20210413150926665" style="zoom: 67%;" />

AddPolicy（）：如算法2所示。在CheckPolicy（）验证ABACP合法后，PC调用AddPolicy（）将ABACP添加到SDB中，同时，所有操作记录都将写入分类账。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413151716166.png" alt="image-20210413151716166" style="zoom:67%;" />

UpdatePolicy（）：在某些情况下，管理员需要修改ABACP。函数UpdatePolicy（）实现了更新SDB的接口，更新的操作记录也会写入区块链。UpdatePolicy（）类似于AddPolicy（），它还调用应用程序接口的put方法来覆盖旧值。

DeletePolicy（）：ABACP有一个过期时间，管理员可以取消它。有两种情况会发生删除。一种情况是管理员通过调用此函数主动删除策略。另一个发生在CheckAccess（）方法执行时，如果属性“endTime”已过期，则它将在PC中调用此函数以删除相关策略。如算法3所示。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413152009403.png" alt="image-20210413152009403" style="zoom:67%;" />

QueryPolicy（）：实现数据库查询接口，提供获取其他链码ABACP的功能。我们选择CouchDB作为SDB。尽管CouchDB是一个键值文档数据库，但它支持类似于mongoDB的复杂查询。在这种情况下，QueryPolicy（）支持通过AS或AO查询ABACP。

#### 2）设备合约（DC）

DC主要负责将设备的资源URL存储到SDB中。DC有两个输入参数{DeviceId,URL}。提供的功能如下。

AddURL（）：它使用DeviceId作为键，URL作为值存储在SDB中。

GetURL （）：根据DeviceId从SDB查询相应的URL值。

#### 3）访问合约（AC）

它验证用户的ABACR是否与ABAC策略匹配。与PC一样，请求数据由用户的私钥签名，然后AC验证签名，用用户的公钥验证用户的身份。提供的方法如下。

Auth（）：与同名的PC方法类似，它使用用户的公钥验证请求，并检查其身份的真实性。

GetAttrs（）：验证签名后解析属性数据字段。ABACR中只包含部分ABAC属性:{AS,AO}，而AE需要由AC确定。最后，这些属性组合为{AS,AO,AE}。

CheckAccess（）：实现访问控制管理的核心功能，如算法4所示。首先，它获取GetAttrs（）设置的属性。其次，调用PC的QueryPolicy（）方法，根据AS和AO查询相应的ABACP。如果返回的结果为空，表示没有支持请求的策略，则直接返回403错误（表示没有权限）。如果返回的结果不为空，则表示将获得一个或多个ABACP。第三，开始逐一判断请求的AE是否与ABACP的AE匹配，AP的值是否为1（表示allow）。如果所有属性都与策略匹配，则验证通过。最后调用DC的GetURL（）函数获取资源的URL并返回给用户，否则返回403错误。

<img src="C:\Users\91349\Desktop\BigWork\交流与论文笔记\论文笔记\image-20210413153118567.png" alt="image-20210413153118567" style="zoom: 67%;" />

## 五、实验与比较

本部分介绍了实验过程和结果对比，以展示我们提出的fabric-IOT的功能和性能。在第一部分中，我们列出了实验所使用的硬件和软件。第二部分介绍了系统的建立过程和基于该系统的访问控制的实现。最后，进行了性能测试、对比实验和结果分析。
fabric-iot项目的源代码在GitHub上是开源的：https://github.com/newham/fabric-iot.

