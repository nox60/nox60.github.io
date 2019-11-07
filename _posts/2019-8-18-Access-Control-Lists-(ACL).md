# Access Control Lists (ACL)

## What is an Access Control List?


*Note: This topic deals with access control and policies on a channel
administration level. To learn about access control within a chaincode, check out
our [chaincode for developers tutorial](./chaincode4ade.html#Chaincode_API).*

本文主要讨论关于通道管理的访问控制策略，更多关于在链代码的ACL，查看：

Fabric uses access control lists (ACLs) to manage access to resources by associating
a **policy** --- which specifies a rule that evaluates to true or false, given a set
of identities --- with the resource. Fabric contains a number of default ACLs. In this
document, we'll talk about how they're formatted and how the defaults can be overridden.

Fabric 通过使用指定规则为true活着folse来控制具体的角色是否有权限，来实现管理访问策略。Fabric包含一些默认权限。本文档将讨论这些权限是怎么构成的，并且讨论默认值是怎么被覆盖的。

But before we can do that, it's necessary to understand a little about resources
and policies.

resources: 资源
policy: 策略

在分析实现之前，需要实现知道关于资源和策略的相关知识。



### Resources

Users interact with Fabric by targeting a [user chaincode](./chaincode4ade.html),
[system chaincode](./chaincode4noah.html), or an [events stream source](./peer_event_services.html).
As such, these endpoints are considered "resources" on which access control should be
exercised.

用户和Fabric是通过 targer? 一段用户链代码和系统链代码来实现的，或者，通过事件流代码。


用户和fabric交互的方式有用户链代码、系统链代码、事件流。如上所述，上面这些操作点都要被识为需要管控的资源。

Application developers need to be aware of these resources and the default
policies associated with them. The complete list of these resources are found in
`configtx.yaml`. You can look at a [sample `configtx.yaml` file here](http://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/configtx.yaml).

应用程序开发者应该知道这些资源和默认的策略。完全的资源列表可以在 `configtx.yaml` 配置文件里面看到。通过这个链接可以看到文件模版：



The resources named in `configtx.yaml` is an exhaustive list of all internal resources
currently defined by Fabric. The loose convention adopted there is `<component>/<resourc/
So `cscc/GetConfigBlock` is the resource for the `GetConfigBlock` call in the `CSCC`
component.

这些在`configtx.yaml`中命名的资源是 所有已经定义在Fabric中的内部资源 的 详尽的列表。约定的写法是：`<component>/<resource>`，即组件/资源，因此`cscc/GetConfigBlock`是一个在'CSCC'组件中的GetConfigBlock命令。


### Policies

Policies are fundamental to the way Fabric works because they allow the identity
(or set of identities) associated with a request to be checked against the policy
associated with the resource needed to fulfill the request. Endorsement policies
are used to determine whether a transaction has been appropriately endorsed. The
policies defined in the channel configuration are referenced as modification policies
as well as for access control, and are defined in the channel configuration itself.

Fabric的工作基础是策略，因为策略允许一个和请求所关联的角色，能够被检查是否能够和其希望访问的资源，

因为策略的存在，实现了身份和其访问的资源的匹配检查，

背书策略用来判断一个交易是否被有足够的背书信息。这些在通道配置中定义的策略被引用为修改策略（等同于控制策略），

Policies can be structured in one of two ways: as `Signature` policies or as an
`ImplicitMeta` policy.

策略可以有两种组织方式：`签名`策略或者`隐式元数据`策略

#### `Signature` policies

These policies identify specific users who must sign in order for a policy
to be satisfied. For example:



```
Policies:
  MyPolicy:
    Type: Signature
    Rule: “Org1.Peer OR Org2.Peer”

```

This policy construct can be interpreted as: *the policy named `MyPolicy` can
only be satisfied by the signature of an identity with role of "a peer from
Org1" or "a peer from Org2"*.

策略的结构可以这样解读：

策略被命名为`MyPolicy`仅仅能够被 "a peer from Org1" 或者 "a peer from Org2" 这样的签名所匹配

Signature policies support arbitrary combinations of `AND`, `OR`, and `NOutOf`,
allowing the construction of extremely powerful rules like: "An admin of org A
and two other admins, or 11 of 20 org admins".

签名策略支持 `AND`, `OR`, and `NOutOf` 的任意组合，可以通过这样的方式组装出较强的规则，比如 "  组织A和的一个管理员或者两个管理员，或者 11个或者20个其他管理员  "  "An admin of org A and two other admins, or 11 of 20 org admins" 

#### `ImplicitMeta` policies

`ImplicitMeta` policies aggregate the result of policies deeper in the
configuration hierarchy that are ultimately defined by `Signature` policies. They
support default rules like "A majority of the organization admins". These policies
use a different but still very simple syntax as compared to `Signature` policies:
`<ALL|ANY|MAJORITY> <sub_policy>`.

隐式元数据策略  深度聚合  主要是优化设计的 `Signature` 策略。它支持默认的规则，比如：“一个组织的主要管理者”。 它自己的规则则是使用一种非常简单的语法来对比签名的方式，比如`<ALL|ANY|MAJORITY> <sub_policy>`.



For example: `ANY` `Readers` or `MAJORITY` `Admins`.

比如：任意写入者或者主要的管理员

*Note that in the default policy configuration `Admins` have an operational role.
Policies that specify that only Admins --- or some subset of Admins --- have access
to a resource will tend to be for sensitive or operational aspects of the network
(such as instantiating chaincode on a channel). `Writers` will tend to be able to
propose ledger updates, such as a transaction, but will not typically have
administrative permissions. `Readers` have a passive role. They can access
information but do not have the permission to propose ledger updates nor do can
they perform administrative tasks. These default policies can be added to,
edited, or supplemented, for example by the new `peer` and `client` roles (if you
have `NodeOU` support).*

注意，在默认的策略里面，Admin都是有操作权限的。权限仅仅指派给Admin或者Admin子集的，可以访问一个 敏感或者 XXX 的资源。 `Writers` 一般倾向于能够对账本提交更新操作的，比如交易，一般不用指定管理员权限；`Readers`有被动权限，可以解除到资源信息，不用具有账本的更新和管理员权限。这些默认的权限可以被增加，编辑，或者提供。例如一个新的 `peer` 或者 `client` 角色，如果你支持 `NodeOU` 

Here's an example of an `ImplicitMeta` policy structure:

下面是一个隐式元数据策略例子。

```
Policies:
  AnotherPolicy:
    Type: ImplicitMeta
    Rule: "MAJORITY Admins"
```


Here, the policy `AnotherPolicy` can be satisfied by the `MAJORITY` of `Admins`,
where `Admins` is eventually being specified by lower level `Signature` policy.

此处， `AnotherPolicy` 可以被 `Admins` 的 `MAJORITY` 所满足？

当 `Admins` 。。。。

### Where is access control specified?

Access control defaults exist inside `configtx.yaml`, the file that `configtxgen`
uses to build channel configurations.

ACL的默认配置在  `configtx.yaml` 中，该文件 被 `configtxgen` 工具用来构建通道配置。

Access control can be updated one of two ways, either by editing `configtx.yaml`
itself, which will propagate the ACL change to any new channels, or by updating
access control in the channel configuration of a particular channel.

ACL能够被两种方式更新，一种是修改 `configtx.yaml`，当生成新通道的时候生效；另外一种是更新某个指定通道的ACL配置。

## How ACLs are formatted in `configtx.yaml`

ACLs are formatted as a key-value pair consisting of a resource function name
followed by a string. To see what this looks like, reference this [sample configtx.yaml file](https://github.com/hyperledger/fabric/blob/release-1.2/sampleconfig/configtx.yaml).

ACL是一种特定格式的KV键值对。

Two excerpts from this sample:

```
# ACL policy for invoking chaincodes on peer
peer/Propose: /Channel/Application/Writers
```

```
# ACL policy for sending block events
event/Block: /Channel/Application/Readers
```

These ACLs define that access to `peer/Propose` and `event/Block` resources
is restricted to identities satisfying the policy defined at the canonical path
`/Channel/Application/Writers` and `/Channel/Application/Readers`, respectively.

上面的ACL配置定义了对  `peer/Propose` 和 `event/Block`  两种资源的限制，配置信息在两个绝对路径中。

### Updating ACL defaults in `configtx.yaml`

In cases where it will be necessary to override ACL defaults when bootstrapping
a network, or to change the ACLs before a channel has been bootstrapped, the
best practice will be to update `configtx.yaml`.



Let's say you want to modify the `peer/Propose` ACL default --- which specifies
the policy for invoking chaincodes on a peer -- from `/Channel/Application/Writers`
to a policy called `MyPolicy`.

This is done by adding a policy called `MyPolicy` (it could be called anything,
but for this example we'll call it `MyPolicy`). The policy is defined in the
`Application.Policies` section inside `configtx.yaml` and specifies a rule to be
checked to grant or deny access to a user. For this example, we'll be creating a
`Signature` policy identifying `SampleOrg.admin`.

以上方式通过添加一个`MyPolicy`权限（权限的名字可以自己命名，此处例子中的命名是 `MyPolicy`）。这个策略的定义是在`configtx.yaml`文件的  `Application.Policies`部分，并且指定了一个规则可以用来检测或者赋予或者禁用用户的访问权限。比如，可以创建一个名为 `SampleOrg.admin`的`Signature`类型策略ID。

```
Policies: &ApplicationDefaultPolicies
    Readers:
        Type: ImplicitMeta
        Rule: "ANY Readers"
    Writers:
        Type: ImplicitMeta
        Rule: "ANY Writers"
    Admins:
        Type: ImplicitMeta
        Rule: "MAJORITY Admins"
    MyPolicy:
        Type: Signature
        Rule: "OR('SampleOrg.admin')"
```

Then, edit the `Application: ACLs` section inside `configtx.yaml` to change
`peer/Propose` from this:

现在，编辑`configtx.yaml` 文件中的 `Application: ACLs`部分，编辑内容如下：


`peer/Propose: /Channel/Application/Writers`

To this:

`peer/Propose: /Channel/Application/MyPolicy`


Once these fields have been changed in `configtx.yaml`, the `configtxgen` tool
will use the policies and ACLs defined when creating a channel creation
transaction. When appropriately signed and submitted by one of the admins of the
consortium members, a new channel with the defined ACLs and policies is created.


-- 此处需要验证：
一旦这些在 `configtx.yaml`中的属性发生变化，`configtxgen` 将使用更改过的策略和ACL配置来创建交易。当合适的签名和提交被一个管理员提交时，一个新的通道将会被创建，该通道的ACL和访问策略将会使用新的。

Once `MyPolicy` has been bootstrapped into the channel configuration, it can also
be referenced to override other ACL defaults. For example:

一旦`MyPolicy`在通道配置中被加载起来，它将会覆盖其他默认的ACL配置，比如：

```
SampleSingleMSPChannel:
    Consortium: SampleConsortium
    Application:
        <<: *ApplicationDefaults
        ACLs:
            <<: *ACLsDefault
            event/Block: /Channel/Application/MyPolicy
```

This would restrict the ability to subscribe to block events to `SampleOrg.admin`.

这将严格限制 `SampleOrg.admin` 订阅块事件的能力 

If channels have already been created that want to use this ACL, they'll have
to update their channel configurations one at a time using the following flow:

如果通道已经被创建起来了，他们将每次一个的更新通道的配置

### Updating ACL defaults in the channel config

If channels have already been created that want to use `MyPolicy` to restrict
access to `peer/Propose` --- or if they want to create ACLs they don't want
other channels to know about --- they'll have to update their channel
configurations one at a time through config update transactions.

如果通道已经创建起来，并且希望用  `MyPolicy` 策略来限制访问 `peer/Propose`，这将更新他们的通道配置 

*Note: Channel configuration transactions are an involved process we won't
delve into here. If you want to read more about them check out our document on
[channel configuration updates](./config_update.html) and our ["Adding an Org to a Channel" tutorial](./channel_update_tutorial.html).*

注意，此处没有计划深入分析通道配置交易。如果需要更多的信息请阅读文档： 

After pulling, translating, and stripping the configuration block of its metadata,
you would edit the configuration by adding `MyPolicy` under `Application: policies`,
where the `Admins`, `Writers`, and `Readers` policies already live.

当拉取，翻译，剖析这些元数据配置文件块。当`Admins`, `Writers`, and `Readers`这些策略已经上线的时候， 你会编辑这些配置文件，通过在`Application: policies`下面增加`MyPolicy`

```
"MyPolicy": {
  "mod_policy": "Admins",
  "policy": {
    "type": 1,
    "value": {
      "identities": [
        {
          "principal": {
            "msp_identifier": "SampleOrg",
            "role": "ADMIN"
          },
          "principal_classification": "ROLE"
        }
      ],
      "rule": {
        "n_out_of": {
          "n": 1,
          "rules": [
            {
              "signed_by": 0
            }
          ]
        }
      },
      "version": 0
    }
  },
  "version": "0"
},
```

Note in particular the `msp_identifer` and `role` here.

Then, in the ACLs section of the config, change the `peer/Propose` ACL from
this:

```
"peer/Propose": {
  "policy_ref": "/Channel/Application/Writers"
```

To this:

```
"peer/Propose": {
  "policy_ref": "/Channel/Application/MyPolicy"
```

Note: If you do not have ACLs defined in your channel configuration, you will
have to add the entire ACL structure.

Once the configuration has been updated, it will need to be submitted by the
usual channel update process.

### Satisfying an ACL that requires access to multiple resources

If a member makes a request that calls multiple system chaincodes, all of the ACLs
for those system chaincodes must be satisfied.

For example, `peer/Propose` refers to any proposal request on a channel. If the
particular proposal requires access to two system chaincodes that requires an
identity satisfying `Writers` and one system chaincode that requires an identity
satisfying `MyPolicy`, then the member submitting the proposal must have an identity
that evaluates to "true" for both `Writers` and `MyPolicy`.

例如，`peer/Propose` 引用的是一个在通道上的提议请求。如果这个特殊的请求需要访问两个系统链代码，其中一个需要 满足  `Writers` 身份，另外一个 需要满足 `MyPolicy` 身份，那么这个用户在提交 proposal 请求的时候，必须同时满足 上面两种身份。

In the default configuration, `Writers` is a signature policy whose `rule` is
`SampleOrg.member`. In other words, "any member of my organization". `MyPolicy`,
listed above, has a rule of `SampleOrg.admin`, or "any admin of my organization".
To satisfy these ACLs, the member would have to be both an administrator and a
member of `SampleOrg`. By default, all administrators are members (though not all
administrators are members), but it is possible to overwrite these policies to
whatever you want them to be. As a result, it's important to keep track of these
policies to ensure that the ACLs for peer proposals are not impossible to satisfy
(unless that is the intention).

在默认配置中， `Writers` 是 角色是 `SampleOrg.member` 的签名策略。换句话说，该组织的任何一个成员。 `MyPolicy` 是 `SampleOrg.admin` 的一种角色，或者 组织的任何一个管理员？

要满足上面的ACL，这些成员必须同时拥有 `管理员` 和 `SampleOrg的成员身份` 双重身份。 默认的，所有的管理员都是成员（尽管不是所有的管理员都是成员），但是有可能覆盖这些策略来达到你想要的任何功能。作为一个结果 ， 要对这些策略保持跟踪来确定 这些策略不可能满足    是很重要的

#### Migration considerations for customers using the experimental ACL feature

Previously, the management of access control lists was done in an `isolated_data`
section of the channel creation transaction and updated via `PEER_RESOURCE_UPDATE`
transactions. Originally, it was thought that the `resources` tree would handle the
update of several functions that, ultimately, were handled in other ways, so
maintaining a separate parallel peer configuration tree was judged to be unnecessary.

Migration for customers using the experimental resources tree in v1.1 is possible.
Because the official v1.2 release does not support the old ACL methods, the network
operators should shut down all their peers.  Then, they should upgrade them to v1.2,
submit a channel reconfiguration transaction which enables the v1.2 capability and
sets the desired ACLs, and then finally restart the upgraded peers.  The restarted
peers will immediately consume the new channel configuration and enforce the ACLs as
desired.

<!--- Licensed under Creative Commons Attribution 4.0 International License
https://creativecommons.org/licenses/by/4.0/ -->
