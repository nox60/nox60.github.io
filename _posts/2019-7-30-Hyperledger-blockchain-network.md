![avatar](../images/posts/hyperledger/network.diagram.1.png)

 Channel C1 is governed according to the policy rules specified in channel configuration CC1; the channel is under the control of organizations R1 and R2. Channel C2 is governed according to the policy rules specified in channel configuration CC2; the channel is under the control of organizations R2 and R3. There is an ordering service O4 that services as a network administration point for N, and uses the system channel. The ordering service also supports application channels C1 and C2, for the purposes of transaction ordering into blocks for distribution. Each of the four organizations has a preferred Certificate Authority.

图中有四个组织组成了一个区块链网络。
- R4负责网络最早的初始化。R4在网络中没有参与具体的业务逻辑。
- R1和R2需要有私密的通信支持，R2和R3也如此。
- R1有一个客户端程序可以通过通道C1执行业务逻辑。R2有一个类似的客户端程序，可以通过C1和C2执行业务逻辑。R3有一个客户端程序，可以通过通道C2执行业务程序。
- 节点P1维护了账本L1，该账本关联到C1。节点P2维护了一个L1账本，毫无疑问，该账本同样也是关联到C1的，同时，P2还维护了一个关联到C2的L2账本。节点P3维护了一个关联到C2的L2账本副本。
- 整个网络的相关规则来自NC4(Network Configuration)的配置信息，而NC4是由R1和R4所组成的，所以网络是由R1和R4联合控制的。
- 通道C1