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