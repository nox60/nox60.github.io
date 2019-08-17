![avatar](/images/posts/hyperledger/network.diagram.1.png)

There is an ordering service O4 that services as a network administration point for N, and uses the system channel. The ordering service also supports application channels C1 and C2, for the purposes of transaction ordering into blocks for distribution. Each of the four organizations has a preferred Certificate Authority.

图中有四个组织组成了一个区块链网络。
- R4负责网络最早的初始化。R4在网络中没有参与具体的业务逻辑。
- R1和R2需要有私密的通信支持，R2和R3也如此。
- R1有一个客户端程序可以通过通道C1执行业务逻辑。R2有一个类似的客户端程序，可以通过C1和C2执行业务逻辑。R3有一个客户端程序，可以通过通道C2执行业务程序。
- 节点P1维护了账本L1，该账本关联到C1。节点P2维护了一个L1账本，毫无疑问，该账本同样也是关联到C1的，同时，P2还维护了一个关联到C2的L2账本。节点P3维护了一个关联到C2的L2账本副本。
- 整个网络的相关规则来自NC4(Network Configuration)的配置信息，而NC4是由R1和R4所组成的，所以网络是由R1和R4联合控制的。
- 通道C1是由通道配置信息CC1所控制的，CC1是由R1和R2管理的。
- 通道C2是由通道配置信息CC2所控制的，CC2是由R2和R3管理的。
- 整个区块链网络N的管理权限由交易排序服务O4所有，并作为系统通道。
- 交易排序服务O4同时为C1和C2两个通道提供服务。最终负责打包所有的交易到区块中。
- 四个组织都充当CA的角色？（此处需要确定）。

# 创建网络

![avatar](/images/posts/hyperledger/network.diagram.2.png)

区块链网络的创建是伴随orderer节点的启动而创建的。在上面的例子中的N，就是这个网络。Ordering 服务组成了一个单orderer节点O4，该节点配置了网络的相关设置（network configuration NC4），并且给与了机构R4管理权限。 At the network level, Certificate Authority CA4 is used to dispense identities to the administrators and network nodes of the R4 organization.

从上图可以看到定义网络N的第一件事情是创造orderer节点O4。也可以理解为orderer节点的初始化，是整个网络初始化的第一步。如之前所说，O4节点的初始化和启动是靠机构R4的管理员做的，并且由R4监管。NC4的配置信息里包含了网络和管理的相关信息。这些管理职能在后面可以发生变更，但是目前R4是网路里的唯一成员，自然只有R4来管理。

# CA证书

同时我们可以看到图中的证书签发机构CA4，该机构用于给其他管理员和节点签发证书。

在网络中不同的机构使用不同的CA来签发证书，以保证识别网络中不同的组件的来源。在本例中，会有R1-R4四个机构的存在，我们将定义四个CA在网络里，每个机构一个CA服务，用于支持不同的机构证书的签发。

证书到机构的映射过程是由MSP(Membership Service Provider)完成的。Network configuration NC4 uses a named MSP to identify the properties of certificates dispensed by CA4 which associate certificate holders with organization R4. NC4 can then use this MSP name in policies to grant actors from R4 particular rights over network resources. An example of such a policy is to identify the administrators in R4 who can add new member organizations to the network. We don’t show MSPs on these diagrams, as they would just clutter them up, but they are very important.

Secondly, we’ll see later how certificates issued by CAs are at the heart of the transaction generation and validation process. Specifically, X.509 certificates are used in client application transaction proposals and smart contract transaction responses to digitally sign transactions. Subsequently the network nodes who host copies of the ledger verify that transaction signatures are valid before accepting transactions onto the ledger.

Let’s recap the basic structure of our example blockchain network. There’s a resource, the network N, accessed by a set of users defined by a Certificate Authority CA4, who have a set of rights over the resources in the network N as described by policies contained inside a network configuration NC4. All of this is made real when we configure and start the ordering service node O4.

# 加入网络管理员

NC4一开始只允许R4管理整个网络。在后面，我们会允许R1的管理员也具有对网络的管理权，我们看下面的网络结构：

![avatar](/images/posts/hyperledger/network.diagram.2.1.png)
*R4升级了NC(network configuration)以使得R1具有了管理权限，这样R1和R4就具备了同等的网络配置和管理能力。*

我们看到了R1具有了和R4一样的网络管理权限能力的同时，也看到了添加了一个签发机构CA1，CA1将用于识别来自R1的用户。

In its simplest form, the ordering service is a single node in the network, and that’s what you can see in the example. Ordering services are usually multi-node, and can be configured to have different nodes in different organizations. For example, we might run O4 in R4 and connect it to O2, a separate orderer node in organization R1. In this way, we would have a multi-site, multi-organization administration structure.

We’ll discuss the ordering service a little more later in this topic, but for now just think of the ordering service as an administration point which provides different organizations controlled access to the network.

# 定义联盟

尽管已经有了一个由于R1和R4管理的网络，但是目前这个网络能做的事情太少，所以接下来首先要做的事情是定一个联盟(consortium)，这个词的字面含义是"命运共同体组织 a group with a shared destiny"，所以需要合理的选择一个联盟里的机构。

![avatar](/images/posts/hyperledger/network.diagram.3.png)
*网络管理员定义了一个具有两个成员的联盟X1，这两个组织成员分别是R1和R2，这个联盟的定义保存在NC4中，CA1和CA2将分别作为这两个组织的签发机构。*

因为NC4控制了网络配置权限，所以R1和R4可以创建新的联盟。上图中显示了新增的联盟X1，其有两个成员组织R1和R2。同时我们可以看到CA2将作为R2的CA服务，用于识别R2的账户。一个联盟可以具有任意数量的成员。

联盟的意义在哪里？联盟是联盟链的核心实现，一个联盟里面的所有组织将同步该联盟里面发生的所有交易，所以当有部分组织需要保持账本同步的时候，他们就应该被组成为一个联盟。

接下来我们将为联盟X1创建一个重要的组件：通道 channel

# 为联盟创建通道

接下来将会为联盟创建通道，通道是实现联盟成员之间互相通信的重要组成部分。在一个网络中可能有多个通道组成。

下图展示了我们增加的第一个通道C1

![avatar](/images/posts/hyperledger/network.diagram.4.png)
*通道C1的创建是实现联盟X1里面的成员通信，目前该联盟里面只有R1和R2，该通道的配置由CC1(channel configuration)定义。CC1和NC4是完全独立的。CC1是由R1和R2在进行管理的。R4虽然是NC4的管理员，但是对CC1没有任何管理权限*

C1使的X1里面的成员能够互相通信。我们可以注意到，C1连接了orderer服务O4，但是but that nothing else is attached to it. 在下一个阶段我们可以连接一个客户端应用程序以及peer节点。

Even though channel C1 is a part of the network N, it is quite distinguishable from it. Also notice that organizations R3 and R4 are not in this channel – it is for transaction processing between R1 and R2. In the previous step, we saw how R4 could grant R1 permission to create new consortia. It’s helpful to mention that R4 also allowed R1 to create channels! In this diagram, it could have been organization R1 or R4 who created a channel C1. Again, note that a channel can have any number of organizations connected to it – we’ve shown two as it’s the simplest configuration.

另外要注意到的是，C1是有一个完全独立的配置信息CC1的，和整个网络的配置信息NC4独立开来。CC1由R1和R2进行管理和配置。R3和R4对CC1没有权限。R3和R4如果想和C1通信，需要R1或者R2在CC1里面赋予合理的权限之后。另外需要注意的是,R4虽然是NC4的管理员，但是他却无法把自己加入到C1中，这个操作只能由R1和R2完成。

通道的重要性体现在，他实现了联盟内成员的通信以及数据共享。通道也可以实现和其他联盟的数据通信和共享。

We can also see that once a channel has been created, it is in a very real sense “free from the network”. It is only organizations that are explicitly specified in a channel configuration that have any control over it, from this time forward into the future. Likewise, any updates to network configuration NC4 from this time onwards will have no direct effect on channel configuration CC1; for example if consortia definition X1 is changed, it will not affect the members of channel C1. Channels are therefore useful because they allow private communications between the organizations constituting the channel. Moreover, the data in a channel is completely isolated from the rest of the network, including other channels.

As an aside, there is also a special system channel defined for use by the ordering service. It behaves in exactly the same way as a regular channel, which are sometimes called application channels for this reason. We don’t normally need to worry about this channel, but we’ll discuss a little bit more about it later in this topic.

# 节点和账本 Peers and Ledgers
接下来我们将用通道来真正连接区块链网络中的各个组织。我们将加入两种新的重要组件节点和账本

![avatar](/images/posts/hyperledger/network.diagram.5.png)

*节点P1加入了通道C1，节点从物理上掌握账本L1，P1和O4讲通过C1通信*

节点从物理上讲，具有账本的副本(节点的存储磁盘上有账本的拷贝)，但是从逻辑上讲，账本是被通道掌握的：这是因为一个通道上只有一个账本，所有的节点都同步了这同一个账本。

节点的一个非常重要的属性就是基于X509的CA证书身份，以此来保证节点和所属机构的关联。一旦节点创立，它就可以通过orderer节点O4来假如C1，当O4收到这个加入请求之后，他将使用CC1的配置来确定P1具有相关权限。例如：CC1会确定P1是否有在账本L1上的读写权限。

注意，决定节点是怎样加入通道的由管理通道的组织决定的，尽管我们目前只加入了一个节点，后面我们会在更多的通道上加入更多的节点。

# 应用程序、智能合约、链代码

![avatar](/images/posts/hyperledger/network.diagram.6.png)

*一个智能合约S5安装到P1节点，R1组织下属的应用程序A1可以使用S5来通过P1访问P1持有的账本L1。*

A1可以通过C1来连接特定的网络资源，在上图的例子中，A1可以连接P1节点和排序节点O4，另外要注意到，通道在通信过程中承担消息中心的职责。尽管A1应用程序在网络之外，依然可以连接到通道C1上。

从上面图中，看起来A1可以直接通过P1连接到L1，但是实际上，所有的程序连接都是通过智能合约代码来实现的：上图中是S5，S5给A1提供了对账本进行查询和更新的接口，简而言之，S5是客户端程序A1和账本L1之间的中介。

智能合约代码是由各机构的开发人员为了实现业务需求专门开发的业务逻辑代码，智能合约代码的目标是生成交易信息，并且分发到区块链网络中，最终形成区块链上的记录信息。智能合约代码需要安装，然后实例化，才能生效。

# 安装智能合约
当开发工程师开发完智能合约代码之后，需要机构R1管理员安装到节点P1上。

当一个机构在同一个通道上有多个节点时，只需要选择一个节点安装智能合约，不用在所有节点上都进行安装。

# 实例化智能合约

当智能合约安装之后，并没有真正的启用，只有实例化之后，智能合约才真正的能够被调用。实例化操作需要机构管理员进行操作。

需要注意的是，尽管通道上的所有节点都能接触到S5智能合约，但是并不是所有节点都能看到其业务逻辑(具体实现的)，这只能是安装该智能合约的节点才具有的权限。智能合约的安装和节点有关(物理持有)，智能合约的实例化(逻辑持有)和通道有关。

# 背书策略

一个智能合约产生的交易需要被其他机构认可的先决条件是，通道中有机构为该合约背书。比如在C1中，S5产生的任何结果需要进入L1账本，则需要R1或者2为其背书。

# 调用智能合约

当智能合约被节点安装并且在一个通道上实力化之后，则可以被客户端应用程序调用。

# 完成网络

![avatar](/images/posts/hyperledger/network.diagram.7.png)

从上图中可以看到机构R2加入了P2节点到C1通道中，P2同时也持有了一个L1账本和智能合约代码S5的副本。同时R2也增加了一个应用程序A2，A2可以通过C1连接到网络内。

此时此刻，这个网络已经成为了一个可以操作的网络，目前R1和R2已经能够同步交易数据，也就是受，A1或者A2调用智能合约代码产生的交易信息，已经能够通过通道C1同步到账本L1中。

# 产生和接受交易

# 节点类型

- 提交节点

所有节点在通道中都是提交节点，当接收到交易产生的区块信息的时候，都要验证区块的合法性然后将区块链到账本上。

- 背书节点

所有安装有智能合约(回忆我们提到的智能合约的安装和初始化，物理持有和逻辑持有概念)的节点，都需要为其安装的智能合约背书，以保证其合法性。However, to actually be an endorsing peer, the smart contract on the peer must be used by a client application to generate a digitally signed transaction response. The term endorsing peer is an explicit reference to this fact.

上面是两种主要类型的节点，同时，节点还有两种其他的职责：

- 主节点

- 锚节点

如果一个节点需要和其他机构通信，则通过通道中的锚节点进行。一个组织可以没有锚节点或者多个锚节点。锚节点是实现垮组织（联盟？）通信的基础。

需要注意的是，一个节点可以同时具有以上四种职能！但是只有锚节点是可选的。主节点，背书节点，提交节点这三种节点在网络中是必须有的。

# 安装和初始化

# Simplifying the visual vocabulary 简化图中的概念

![avatar](/images/posts/hyperledger/network.diagram.8.png)

上图取消了通道的划线，而是用蓝色的圈表示通道编号。这样我们在描述更大的网络结构时将更容易。

# 加入另外一个联盟

到目前为止，我们只有X1一个联盟，下图中，我们将加入X2这个联盟

![avatar](/images/posts/hyperledger/network.diagram.9.png)

上图中，我们加入了X2这个由R2和R3组成的新联盟。新的联盟依然只能由具备网路管理权限的NC4配置文件来定义，所以能够做到以上工作的只有R1或者R4(具备配置权限)。

# 加入新通道

此时我们已经加入了一个新的联盟X2，我们需要加入一个新的通道C2，C2在图中将用红色圈表示。

![avatar](/images/posts/hyperledger/network.diagram.10.png)

此时，通道的配置信息CC2决定了通道内的权限，CC2的管理员是R2和R3。和CC1一样，CC2和NC4是完全隔离开的。没有关系。


