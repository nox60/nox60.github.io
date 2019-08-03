# Membership

If you've read through the documentation on [identity](../identity/identity.html)
you've seen how a PKI can provide verifiable identities through a chain
of trust. Now let's see how these identities can be used to represent the
trusted members of a blockchain network.

This is where a **Membership Service** Provider (MSP) comes into play ---
**it identifies which Root CAs and Intermediate CAs are trusted to define
the members of a trust domain, e.g., an organization**, either by listing the
identities of their members, or by identifying which CAs are authorized to
issue valid identities for their members, or --- as will usually be the case ---
through a combination of both.

The power of an MSP goes beyond simply listing who is a network participant or
member of a channel. An MSP can identify specific **roles** an actor might play either
within the scope of the organization the MSP represents (e.g., admins, or as members
of a sub-organization group), and sets the basis for defining **access privileges**
in the context of a network and channel (e.g., channel admins, readers, writers).

MSP不仅仅能够列出一个网络中有哪些账户或者一个通道有哪些成员，MSP可以确定一个参与者在

MSP不仅仅是列出网络中的账户或者通道里的成员。MSP可以实现权限识别，


The configuration of an MSP is advertised to all the channels where members of
the corresponding organization participate (in the form of a **channel MSP**). In
addition to the channel MSP, peers, orderers, and clients also maintain a **local MSP**
to authenticate member messages outside the context of a channel and to define the
permissions over a particular component (who has the ability to install chaincode on
a peer, for example).

在channel MSP情况下：MSP的配置会被通知到channel，这些channel里面的所有成员。
在local MSP情况下：channel MSP, peers, orders和用户都会维护一个本地的MSP，

In addition, an MSP can allow for the identification of a list of identities that
have been revoked --- as discussed in the [Identity](../identity/identity.html)
documentation --- but we will talk about how that process also extends to an MSP.

We'll talk more about local and channel MSPs in a moment. For now let's see what
MSPs do in general.

### Mapping MSPs to Organizations

An **organization** is a managed group of members. This can be something as big
as a multinational corporation or a small as a flower shop. What's most
important about organizations (or **orgs**) is that they manage their
members under a single MSP. Note that this is different from the organization
concept defined in an X.509 certificate, which we'll talk about later.

一个组织是指一组成员, 有可能是大到一个跨国公司，也有可能小到一个花店。最重要的是一个组织将他们的成员放在同一个单独的msp下。注意，这个和X.509定义的组织概念是不一样的。后面我们会详细的讨论这个。

The exclusive relationship between an organization and its MSP makes it sensible to
name the MSP after the organization, a convention you'll find adopted in most policy
configurations. For example, organization `ORG1` would likely have an MSP called
something like `ORG1-MSP`. In some cases an organization may require multiple
membership groups --- for example, where channels are used to perform very different
business functions between organizations. In these cases it makes sense to have
multiple MSPs and name them accordingly, e.g., `ORG2-MSP-NATIONAL` and
`ORG2-MSP-GOVERNMENT`, reflecting the different membership roots of trust within
`ORG2` in the `NATIONAL` sales channel compared to the `GOVERNMENT` regulatory
channel.

一个组织和他的MSP之间的特别的关系使得在对msp命名的时候，将MSP的名字放在组织后面让人很易懂。比如，组织'ORG1'如果有一个MSP，则很合适将名字取为：ORG1-MSP。在很多情况下，一个组织会有多个会员身份，比如说，当有多个通道被用于在不同的组织之间处理不同的业务逻辑，在这种情况下，就比较合适使用上面的命名方式：'ORG2-MSP-NATIONAL'和'ORG2-MSP-GOVERNMENT'，分别是ORG2的销售通道和处理政府相关事务的常规通道。



![MSP1](./membership.diagram.3.png)

*Two different MSP configurations for an organization. The first configuration shows
the typical relationship between an MSP and an organization --- a single MSP defines
the list of members of an organization. In the second configuration, different MSPs
are used to represent different organizational groups with national, international,
and governmental affiliation.*

一个组织中不同的MSP配置。第一个配置现实了组织和其MSP之间比较典型的关系。一个单独的MSP定义了组织下属的所有会员；在第二个配置中，不同的MSP用来反应不同的组和国家，国际，以及政府部门之间的关系。

#### Organizational Units and MSPs

An organization is often divided up into multiple **organizational units** (OUs), each
of which has a certain set of responsibilities. For example, the `ORG1`
organization might have both `ORG1-MANUFACTURING` and `ORG1-DISTRIBUTION` OUs
to reflect these separate lines of business. When a CA issues X.509 certificates,
the `OU` field in the certificate specifies the line of business to which the
identity belongs.

一个组织经常拆分为不同的组织单元OU，每个不同的组织单元都负责其不同的一些相关职能，。比如说，ORG1有`ORG1-MANUFACTURING` 和 `ORG1-DISTRIBUTION`两个组织单元，横线后面的单元名称体现了其的不同智能，当一个CA签发X.509证书的时候，其`OU`字段体现了相关信息。

We'll see later how OUs can be helpful to control the parts of an organization who
are considered to be the members of a blockchain network. For example, only
identities from the `ORG1-MANUFACTURING` OU might be able to access a channel,
whereas `ORG1-DISTRIBUTION` cannot.

在后面将会看到不同的组织单元划分对于一个在区块链上的组织如何控制其各部分是有帮助的。

在后面将会看到将一个组织的不同单元作为区块链中的不同组织是有意义的。比如，仅仅使`ORG1-MANUFACTURING` 这个单元能够连接到一个通道，而
 `ORG1-DISTRIBUTION` 则不能.

Finally, though this is a slight misuse of OUs, they can sometimes be used by
different organizations in a consortium to distinguish each other. In such cases, the
different organizations use the same Root CAs and Intermediate CAs for their chain
of trust, but assign the `OU` field to identify members of each organization.
We'll also see how to configure MSPs to achieve this later.

尽管这其实是或许是对单元的一种不太正确的使用，很多时候用于不同的组织在一个区块链联盟里面互相区分。在这样的情况下，这种不同的组织使用同一个CA或者Intermediate CA证书来在区块链证书证明身份，而使用`OU`字段来区分不同组织的不同成员。在后面的部分中将展示如何做到这样的配置。

### Local and Channel MSPs

本地和通道MSP

MSPs appear in two places in a blockchain network: channel configuration
(**channel MSPs**), and locally on an actor's premise (**local MSP**). **Local MSPs are
defined for clients (users) and for nodes (peers and orderers)**. Node local MSPs define
the permissions for that node (who the peer admins are, for example). The local MSPs
of the users allow the user side to authenticate itself in its transactions as a member
of a channel (e.g. in chaincode transactions), or as the owner of a specific role
into the system (an org admin, for example, in configuration transactions).

MSP在区块链网络中有两种存储方式，一种是通道中的配置，一种是在参与者的本地配置。本地MSP一般用于定于用户以及其节点（peer节点或者排序节点），节点的本地MSP定义了那个节点的权限：比如谁是那个节点的管理员。本地MSP用于让用户允许用户端授权他们自己成为一个通道的成员。比如，在区块链交易过程中，或者，作为一个特殊权限的所有者进入一个系统（比如一个组织的管理员，可以配置交易）

**Every node and user must have a local MSP defined**, as it defines who has
administrative or participatory rights at that level (peer admins will not necessarily
be channel admins, and vice versa).

所有节点和用户都能够有一个本地MSP定义，就像定义每个人的管理员权限和成员权限一样。节点管理员不需要是通道管理员，反之亦然。


In contrast, **channel MSPs define administrative and participatory rights at the
channel level**. Every organization participating in a channel must have an MSP
defined for it. Peers and orderers on a channel will all share the same view of channel
MSPs, and will therefore be able to correctly authenticate the channel participants.
This means that if an organization wishes to join the channel, an MSP incorporating
the chain of trust for the organization's members would need to be included in the
channel configuration. Otherwise transactions originating from this organization's
identities will be rejected.

对比来看，通道MSP定义了通道层级的管理员和一些特殊权限。所有的加入通道的组织都有这样的MSP定义。而Peer节点和排序节点在通道上将共享这些通道MSP，并且因此能够正确的对通道参与者授权。这就意味着如果有一个机构希望加入这个通道，关于这个组织的会员需要被加入通道MSP配置信息。。。否则该组织在通道上提交的交易请求将会被拒绝。

The key difference here between local and channel MSPs is not how they function
--- both turn identities into roles --- but their **scope**.

本地MSP和通道MSP的关键区别不在于他们的功能，他们都是把身份转化为角色，关键的区别在于他们的范围。

<a name="msp2img"></a>

![MSP2](./membership.diagram.4.png)

*Local and channel MSPs. The trust domain (e.g., the organization) of each
peer is defined by the peer's local MSP, e.g., ORG1 or ORG2. Representation
of an organization on a channel is achieved by adding the organization's MSP to
the channel configuration. For example, the channel of this figure is managed by
both ORG1 and ORG2. Similar principles apply for the network, orderers, and users,
but these are not shown here for simplicity.*

本地和通道MSP。可信的域信息（比如某个组织）对于所有节点的定义，都是保存在节点的本地MSP内。比如，ORG1或者ORG2，代表了通道上的一个组织，要实现这个配置需要把该组织的本地MSP添加到通道的配置中。比如，某个通道的配置是由ORG1或者ORG2控制的。。。。。。

You may find it helpful to see how local and channel MSPs are used by seeing
what happens when a blockchain administrator installs and instantiates a smart
contract, as shown in the [diagram above](#msp2img).

关注本地和通道MSP如何用于区分当区块链管理员安装和实例化智能合约时，是有帮助的。

An administrator `B` connects to the peer with an identity issued by `RCA1`
and stored in their local MSP. When `B` tries to install a smart contract on
the peer, the peer checks its local MSP, `ORG1-MSP`, to verify that the identity
of `B` is indeed a member of `ORG1`. A successful verification will allow the
install command to complete successfully. Subsequently, `B` wishes
to instantiate the smart contract on the channel. Because this is a channel
operation, all organizations on the channel must agree to it. Therefore, the
peer must check the MSPs of the channel before it can successfully commit this
command. (Other things must happen too, but concentrate on the above for now.)

当管理员B通过 'RCA1' 所签发的身份 连接到peer节点

'RCA1' 给管理员B签发了一个身份，管理员B将该身份保存在本地MSP中，然后通过保存在本地MSP中的身份连接到peer节点，然后管理员B尝试在这个peer节点上安装一个智能合约，这个peer节点检查了自己的本地MSP信息'ORG1-MSP', 检查这个管理员B的身份，发现管理员B确实是'ORG1'的一个成员。则允许安装该智能合约。在安装完智能合约后，管理员B如果希望在通道上实例化一个智能合约，但这是一个通道操作，所有在通道上关联的组织将会最终确认这个操作。因此，这peer节点需要确认通道的MSP之后，才能提交这个实例化命令。

**Local MSPs are only defined on the file system of the node or user** to which
they apply. Therefore, physically and logically there is only one local MSP per
node or user. However, as channel MSPs are available to all nodes in the
channel, they are logically defined once in the channel configuration. However,
**a channel MSP is also instantiated on the file system of every node in the
channel and kept synchronized via consensus**. So while there is a copy of each
channel MSP on the local file system of every node, logically a channel MSP
resides on and is maintained by the channel or the network.

本地MSP只是定义在节点或者用户的本地文件环境中。因此，对于一个peer节点或者用户来说，物理上或者逻辑上也只能有一个MSP配置文件存在。无论如何，通道的MSP配置对于所有连接到该通道的节点来说都是可用的，他们？逻辑上都只会定义一次在通道定义的时候。无论如何，一个通道MSP将在所有的文件系统中保存并且依靠共识机制保持同步。因此在所有节点中都有通道MSP的本地拷贝存在。逻辑上讲，一个通道是由该通道维护的。


### MSP Levels

The split between channel and local MSPs reflects the needs of organizations
to administer their local resources, such as a peer or orderer nodes, and their
channel resources, such as ledgers, smart contracts, and consortia, which
operate at the channel or network level. It's helpful to think of these MSPs
as being at different **levels**, with **MSPs at a higher level relating to
network administration concerns** while **MSPs at a lower level handle
identity for the administration of private resources**. MSPs are mandatory
at every level of administration --- they must be defined for the network,
channel, peer, orderer, and users.

通道MSP和本地MSP的区别反应了组织和其管理员在他们本地资源管理上的不同需求，例如一个peer节点或者排序节点，以及他们的通道资源（比如账本，智能合约以及联盟信息）这都是通道或者网络级别的资源；这些资源是不同级别的，高等级的MSP和网络相关的管理有关；而低等级的MSP处理身份权限等信息。MSP是强制的，必须在网络定义，通道，节点，排序节点，以及用户中存在。

![MSP3](./membership.diagram.2.png)

*MSP Levels. The MSPs for the peer and orderer are local, whereas the MSPs for a
channel (including the network configuration channel) are shared across all
participants of that channel. In this figure, the network configuration channel
is administered by ORG1, but another application channel can be managed by ORG1
and ORG2. The peer is a member of and managed by ORG2, whereas ORG1 manages the
orderer of the figure. ORG1 trusts identities from RCA1, whereas ORG2 trusts
identities from RCA2. Note that these are administration identities, reflecting
who can administer these components. So while ORG1 administers the network,
ORG2.MSP does exist in the network definition.*

不同级别的MSP，节点和排序节点的MSP是存在本地的，然而通道节点（包括网络配置通道）的MSP是通道共享给所有加入该通道的节点的。在这些配置中，ORG1在管理通道配置，但是其他的业务通道则可以是ORG1和ORG2共同管理。peer节点是一个ORG2的成员并且是ORG2在管理，然而ORG1 管理了orderer排序节点。ORG1信任来自RCA1的证书，然而ORG2信任来自RCA2的证书。注意到，这些管理类证书。反应了谁可以管理这些组件，因此ORG1管理了网络，而ORG2.MSP存在域网络定义中？

 * **Network MSP:** The configuration of a network defines who are the
 members in the network --- by defining the MSPs of the participant organizations
 --- as well as which of these members are authorized to perform
 administrative tasks (e.g., creating a channel).

网络MSP，配置谁是网络中的成员的配置信息，通过定义组织中的参与者。。就像定义哪些成员是授权可以管理任务的：比如创建一个通道。

 * **Channel MSP:** It is important for a channel to maintain the MSPs of its members
 separately. A channel provides private communications between a particular set of
 organizations which in turn have administrative control over it. Channel policies
 interpreted in the context of that channel's MSPs define who has ability to
 participate in certain action on the channel, e.g., adding organizations, or
 instantiating chaincodes. Note that there is no necessary relationship between
 the permission to administrate a channel and the ability to administrate the
 network configuration channel (or any other channel). Administrative rights
 exist within the scope of what is being administrated (unless the rules have
 been written otherwise --- see the discussion of the `ROLE` attribute below).

 通道MSP，对于将一个通道成员的MSP分开管理是比较重要的一个事情。一个通道为他的参与组织提供了私有的通信方式，

 * **Peer MSP:** This local MSP is defined on the file system of each peer and there is a
 single MSP instance for each peer. Conceptually, it performs exactly the same function
 as channel MSPs with the restriction that it only applies to the peer where it is defined.
 An example of an action whose authorization is evaluated using the peer's local MSP is
 the installation of a chaincode on the peer.

 * **Orderer MSP:** Like a peer MSP, an orderer local MSP is also defined on the file system
 of the node and only applies to that node. Like peer nodes, orderers are also owned by a single
 organization and therefore have a single MSP to list the actors or nodes it trusts.

### MSP Structure

So far, you've seen that the most important element of an MSP are the specification
of the root or intermediate CAs that are used to establish an actor's or node's
membership in the respective organization. There are, however, more elements that are
used in conjunction with these two to assist with membership functions.

![MSP4](./membership.diagram.5.png)

*The figure above shows how a local MSP is stored on a local filesystem. Even though
channel MSPs are not physically structured in exactly this way, it's still a helpful
way to think about them.*

As you can see, there are nine elements to an MSP. It's easiest to think of these elements
in a directory structure, where the MSP name is the root folder name with each
subfolder representing different elements of an MSP configuration.

Let's describe these folders in a little more detail and see why they are important.

* **Root CAs:** This folder contains a list of self-signed X.509 certificates of
  the Root CAs trusted by the organization represented by this MSP.
  There must be at least one Root CA X.509 certificate in this MSP folder.

  This is the most important folder because it identifies the CAs from
  which all other certificates must be derived to be considered members of the
  corresponding organization.


* **Intermediate CAs:** This folder contains a list of X.509 certificates of the
  Intermediate CAs trusted by this organization. Each certificate must be signed by
  one of the Root CAs in the MSP or by an Intermediate CA whose issuing CA chain ultimately
  leads back to a trusted Root CA.

  An intermediate CA may represent a different subdivision of the organization
  (like `ORG1-MANUFACTURING` and `ORG1-DISTRIBUTION` do for `ORG1`), or the
  organization itself (as may be the case if a commercial CA is leveraged for
  the organization's identity management). In the latter case intermediate CAs
  can be used to represent organization subdivisions. [Here](../msp.html) you
  may find more information on best practices for MSP configuration. Notice, that
  it is possible to have a functioning network that does not have an Intermediate
  CA, in which case this folder would be empty.

  Like the Root CA folder, this folder defines the CAs from which certificates must be
  issued to be considered members of the organization.


* **Organizational Units (OUs):** These are listed in the `$FABRIC_CFG_PATH/msp/config.yaml`
  file and contain a list of organizational units, whose members are considered
  to be part of the organization represented by this MSP. This is particularly
  useful when you want to restrict the members of an organization to the ones
  holding an identity (signed by one of MSP designated CAs) with a specific OU
  in it.

  Specifying OUs is optional. If no OUs are listed, all the identities that are part of
  an MSP --- as identified by the Root CA and Intermediate CA folders --- will be considered
  members of the organization.


* **Administrators:** This folder contains a list of identities that define the
  actors who have the role of administrators for this organization. For the standard MSP type,
  there should be one or more X.509 certificates in this list.

  It's worth noting that just because an actor has the role of an administrator it doesn't
  mean that they can administer particular resources! The actual power a given identity has
  with respect to administering the system is determined by the policies that manage system
  resources. For example, a channel policy might specify that `ORG1-MANUFACTURING`
  administrators have the rights to add new organizations to the channel, whereas the
  `ORG1-DISTRIBUTION` administrators have no such rights.

  Even though an X.509 certificate has a `ROLE` attribute (specifying, for example, that
  an actor is an `admin`), this refers to an actor's role within its organization
  rather than on the blockchain network. This is similar to the purpose of
  the `OU` attribute, which --- if it has been defined --- refers to an actor's place in
  the organization.

  The `ROLE` attribute **can** be used to confer administrative rights at the channel level
  if the policy for that channel has been written to allow any administrator from an organization
  (or certain organizations) permission to perform certain channel functions (such as
  instantiating chaincode). In this way, an organizational role can confer a network role.


* **Revoked Certificates:** If the identity of an actor has been revoked,
  identifying information about the identity --- not the identity itself --- is held
  in this folder. For X.509-based identities, these identifiers are pairs of strings known as
  Subject Key Identifier (SKI) and Authority Access Identifier (AKI), and are checked
  whenever the X.509 certificate is being used to make sure the certificate has not
  been revoked.

  This list is conceptually the same as a CA's Certificate Revocation List (CRL),
  but it also relates to revocation of membership from the organization. As a result,
  the administrator of an MSP, local or channel, can quickly revoke an actor or node
  from an organization by advertising the updated CRL of the CA the revoked certificate
  as issued by. This "list of lists" is optional. It will only become populated
  as certificates are revoked.


* **Node Identity:** This folder contains the identity of the node, i.e.,
  cryptographic material that --- in combination to the content of `KeyStore` --- would
  allow the node to authenticate itself in the messages that is sends to other
  participants of its channels and network. For X.509 based identities, this
  folder contains an **X.509 certificate**. This is the certificate a peer places
  in a transaction proposal response, for example, to indicate that the peer has
  endorsed it --- which can subsequently be checked against the resulting
  transaction's endorsement policy at validation time.

  This folder is mandatory for local MSPs, and there must be exactly one X.509 certificate
  for the node. It is not used for channel MSPs.


* **`KeyStore` for Private Key:** This folder is defined for the local MSP of a peer or
  orderer node (or in an client's local MSP), and contains the node's **signing key**.
  This key matches cryptographically the node's identity included in **Node Identity**
  folder and is used to sign data --- for example to sign a transaction proposal response,
  as part of the endorsement phase.

  This folder is mandatory for local MSPs, and must contain exactly one private key.
  Obviously, access to this folder must be limited only to the identities of users who have
  administrative responsibility on the peer.

  Configuration of a **channel MSPs** does not include this folder, as channel MSPs
  solely aim to offer identity validation functionalities and not signing abilities.


* **TLS Root CA:** This folder contains a list of self-signed X.509 certificates of the
  Root CAs trusted by this organization **for TLS communications**. An example of a TLS
  communication would be when a peer needs to connect to an orderer so that it can receive
  ledger updates.

  MSP TLS information relates to the nodes inside the network --- the peers and the
  orderers, in other words, rather than the applications and administrations that
  consume the network.

  There must be at least one TLS Root CA X.509 certificate in this folder.


* **TLS Intermediate CA:** This folder contains a list intermediate CA certificates
  CAs trusted by the organization represented by this MSP **for TLS communications**.
  This folder is specifically useful when commercial CAs are used for TLS certificates of an
  organization. Similar to membership intermediate CAs, specifying intermediate TLS CAs is
  optional.

  For more information about TLS, click [here](../enable_tls.html).

If you've read this doc as well as our doc on [Identity](../identity/identity.html)), you
should have a pretty good grasp of how identities and membership work in Hyperledger Fabric.
You've seen how a PKI and MSPs are used to identify the actors collaborating in a blockchain
network. You've learned how certificates, public/private keys, and roots of trust work,
in addition to how MSPs are physically and logically structured.

<!---
Licensed under Creative Commons Attribution 4.0 International License https://creativecommons.org/licenses/by/4.0/
-->
