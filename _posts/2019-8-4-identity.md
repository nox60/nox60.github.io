# Identity

## What is an Identity?


The different actors in a blockchain network include peers, orderers, client
applications, administrators and more. Each of these actors --- active elements
inside or outside a network able to consume services --- has a digital identity
encapsulated in an X.509 digital certificate. These identities really matter
because they **determine the exact permissions over resources and access to
information that actors have in a blockchain network.**

区块链网络中的不同actor包括peer节点，排序节点，客户端程序，管理员等，这些actor组成了所有的内部和外部的能够消费服务的活跃单元，这些actor都有X.509证书。这些身份非常重要，因为他们决定了这些actor的具体权限，包含能够在区块链网络中使用哪些资源或者接触哪些信息。

A digital identity furthermore has some additional attributes that Fabric uses
to determine permissions, and it gives the union of an identity and the associated
attributes a special name --- **principal**. Principals are just like userIDs or
groupIDs, but a little more flexible because they can include a wide range of
properties of an actor's identity, such as the actor's organization, organizational
unit, role or even the actor's specific identity. When we talk about principals,
they are the properties which determine their permissions.

一个数字身份包含了较多可以用于确定权限的信息，并且给与了单元一个身份，和参数关联起来的具体名字，当事人。当事人就像是用户ID或者组ID，但是有一点灵活的是因为他们包含一个广泛的用户身份的参数范围，比如这个actor的组织，组织单位，角色或者这个参与者的特殊身份，当我们讨论一个主角的时候，更多的是讨论其属性和权限。

For an identity to be **verifiable**, it must come from a **trusted** authority.
A [membership service provider](../membership/membership.html)
(MSP) is how this is achieved in Fabric. More specifically, an MSP is a component
that defines the rules that govern the valid identities for this organization.
The default MSP implementation in Fabric uses X.509 certificates as identities,
adopting a traditional Public Key Infrastructure (PKI) hierarchical model (more
on PKI later).

关于身份是可变化的，是因为其来自一个可信的授权：MSP。MSP是实现身份认证机制的核心。更特殊的是，MSP是一个定义了规则并且控制组织合法信息的机制。默认的MSP实现是使用X.509的证书签发机制，实现了传统的PKI公钥机制。

## A Simple Scenario to Explain the Use of an Identity

一个简单的使用身份的场景。

Imagine that you visit a supermarket to buy some groceries. At the checkout you see
a sign that says that only Visa, Mastercard and AMEX cards are accepted. If you try to
pay with a different card --- let's call it an "ImagineCard" --- it doesn't matter whether
the card is authentic and you have sufficient funds in your account. It will be not be
accepted.

当消费者在超市购物结束需要结账的时候，会拿出自己的信用卡进行结账，而该超市只接受VISA，Mastercard或者AMEX卡进行支付，如果尝试使用一张不同的卡，假设其名字叫'ImagineCard'，不管这个卡是谁授权的以及是否你又足够的钱在账户里，这张卡都将是不被接受的。

![Scenario](./identity.diagram.6.png)

*Having a valid credit card is not enough --- it must also be accepted by the store! PKIs
and MSPs work together in the same way --- a PKI provides a list of identities,
and an MSP says which of these are members of a given organization that participates in
the network.*

上面的例子说明，有一张合法的信用卡是不够的，重要的是还得具有一张能够被超市所接受的信用卡。PKI体系和MSP体系协作完成实现了这一约束体系。PKI提供了身份说明，而MSP确认了成员所属的组织以及参加了哪些网络。

PKI certificate authorities and MSPs provide a similar combination of functionalities.
A PKI is like a card provider --- it dispenses many different types of verifiable
identities. An MSP, on the other hand, is like the list of card providers accepted
by the store, determining which identities are the trusted members (actors)
of the store payment network. **MSPs turn verifiable identities into the members
of a blockchain network**.

PKI身份体系和MSP结合起来提供认证：PKI就像发卡方，发型了多种不同的身份，而MSP就想商店所列出的接受的卡类型的列表，确认了哪些身份是该商店的支付网络中的受信成员。MSP将可验证的身份转化为区块链网络的成员列表。

Let's drill into these concepts in a little more detail.

## What are PKIs?

**A public key infrastructure (PKI) is a collection of internet technologies that provides
secure communications in a network.** It's PKI that puts the **S** in **HTTPS** --- and if
you're reading this documentation on a web browser, you're probably using a PKI to make
sure it comes from a verified source.

PKI是一套互联网安全体系，其提供安全的网络通信技术。如果你正在通过浏览器在线阅读本文档，你也可能正在使用PKI保证你正在阅读可信的资源。

![PKI](./identity.diagram.7.png)

*The elements of Public Key Infrastructure (PKI). A PKI is comprised of Certificate
Authorities who issue digital certificates to parties (e.g., users of a service, service
provider), who then use them to authenticate themselves in the messages they exchange
with their environment. A CA's Certificate Revocation List (CRL) constitutes a reference
for the certificates that are no longer valid. Revocation of a certificate can happen for
a number of reasons. For example, a certificate may be revoked because the cryptographic
private material associated to the certificate has been exposed.*

PKI是由发行数字证书的发行机构CA构建的，CA将数字证书签发给需要进行加密通信的机构。CA有一个吊销列表用来记录哪些机构不再使用CA证书。吊销可能有多种原因：比如一个数字证书的私有相关信息可能被泄漏。

Although a blockchain network is more than a communications network, it relies on the
PKI standard to ensure secure communication between various network participants, and to
ensure that messages posted on the blockchain are properly authenticated.
It's therefore important to understand the basics of PKI and then why MSPs are
so important.

尽管区块链网络是在普通通信网络之上的。但是区块链网络也依赖于PKI标准来确保在多方交换信息的时候的通信安全。并确保消息在区块链网络中是经过合理的授权了的。因此理解PKI体系和MSP非常重要。

There are four key elements to PKI:

 * **Digital Certificates**
 * **Public and Private Keys**
 * **Certificate Authorities**
 * **Certificate Revocation Lists**

Let's quickly describe these PKI basics, and if you want to know more details,
[Wikipedia](https://en.wikipedia.org/wiki/Public_key_infrastructure) is a good
place to start.



## Digital Certificates

A digital certificate is a document which holds a set of attributes relating to
the holder of the certificate. The most common type of certificate is the one
compliant with the [X.509 standard](https://en.wikipedia.org/wiki/X.509), which
allows the encoding of a party's identifying details in its structure.

一个数字证书其实是一份保存了一系列用户属性的文件。用的最多的数字证书格式是X.509格式。该格式允许结构性的编码？

For example, Mary Morris in the Manufacturing Division of Mitchell Cars in Detroit,
Michigan might have a digital certificate with a `SUBJECT` attribute of `C=US`,
`ST=Michigan`, `L=Detroit`, `O=Mitchell Cars`, `OU=Manufacturing`, `CN=Mary Morris /UID=123456`.
Mary's certificate is similar to her government identity card --- it provides
information about Mary which she can use to prove key facts about her. There are
many other attributes in an X.509 certificate, but let's concentrate on just these
for now.

比如Mary Morris在底特律汽车的工厂部门。该证书可能有多个字段比如：

 `C=US`, `ST=Michigan`, `L=Detroit`, `O=Mitchell Cars`, `OU=Manufacturing`, `CN=Mary Morris /UID=123456`.

Mary的证书和她的身份证很类似，都是提供了一系列的和她有关的信息，她可以证明这些信息是她本人的。X.509有非常多的属性在该证书里，但是本文目前只关注上述这些。

![DigitalCertificate](./identity.diagram.8.png)

*A digital certificate describing a party called Mary Morris. Mary is the `SUBJECT` of the
certificate, and the highlighted `SUBJECT` text shows key facts about Mary. The
certificate also holds many more pieces of information, as you can see. Most importantly,
Mary's public key is distributed within her certificate, whereas her private signing key
is not. This signing key must be kept private.*

数字证书描述了Mary Morris的部分信息。Mary是证书的`SUBJECT`，`SUBJECT`高亮的信息代表了Mary的关键信息。这份证书因此也持有了更多的信息，更重要的是，Mary的公钥描述了她的数字签名，相反的，她的私钥则不是，私钥需要妥善的私有保存。

What is important is that all of Mary's attributes can be recorded using a mathematical
technique called cryptography (literally, "*secret writing*") so that tampering will
invalidate the certificate. Cryptography allows Mary to present her certificate to others
to prove her identity so long as the other party trusts the certificate issuer, known
as a **Certificate Authority** (CA). As long as the CA keeps certain cryptographic
information securely (meaning, its own **private signing key**), anyone reading the
certificate can be sure that the information about Mary has not been tampered with ---
it will always have those particular attributes for Mary Morris. Think of Mary's X.509
certificate as a digital identity card that is impossible to change.

最重要的是Mary的信息可以被一种称为cryptography（数字水印？）的技术进行记录，所以无法篡改。Cryptography 的方式使的Mary在展现其相关信息给对 Cryptography加密方式信任的第三方时，可以获得信任。因为CA是过山保管了Mary证书签发的信息了的，第三方在阅读证书的时候，可以确保Mary的证书是没有被篡改过的，这一结论可以找CA证实。可以把Mary的X.509证书信息试为一种无法被修改的加密电子身份证卡。

## Authentication, Public keys, and Private Keys

Authentication and message integrity are important concepts in secure
communications. Authentication requires that parties who exchange messages
are assured of the identity that created a specific message. For a message to have
"integrity" means that cannot have been modified during its transmission.
For example, you might want to be sure you're communicating with the real Mary
Morris rather than an impersonator. Or if Mary has sent you a message, you might want
to be sure that it hasn't been tampered with by anyone else during transmission.

授权和消息完整性在安全通信中是非常重要的概念。授权需要通信双方能够互相信任并且交换。。。。一个消息是否被整合的意思是在传输过程中不会被篡改。比如，你需要首先相信你正在和真正的Mary Morris通信而不是一个伪造的对象，还有就是Mary发了一个消息给你，你要能够相信该消息是没有被在发送途中被别人篡改过的。

Traditional authentication mechanisms rely on **digital signatures** that,
as the name suggests, allow a party to digitally **sign** its messages. Digital
signatures also provide guarantees on the integrity of the signed message.

传统的授权方式使用的是数字签名方式，就想这个名字所提到的，允许发送双方在发送信息的时候对消息体进行签名，然后数字签名提供了消息完整性的保证。

Technically speaking, digital signature mechanisms require each party to
hold two cryptographically connected keys: a public key that is made widely available
and acts as authentication anchor, and a private key that is used to produce
**digital signatures** on messages. Recipients of digitally signed messages can verify
the origin and integrity of a received message by checking that the
attached signature is valid under the public key of the expected sender.

从技术上说，数字签名机制需要双方都有加密的连接key，比如一个公钥用于扮演验证定位器？的作用，而私钥则用于产生信息的数字签名。在收到信息一端使用发送方的公钥来检查信息是否完整，？？？

**The unique relationship between a private key and the respective public key is the
cryptographic magic that makes secure communications possible**. The unique
mathematical relationship between the keys is such that the private key can be used to
produce a signature on a message that only the corresponding public key can match, and
only on the same message.

公钥和私钥之间的特殊关系是cryptographic的魔力，它使得安全通信成为了一种可能，这种特殊的关系使得私钥对消息进行的签名只有对应的公钥才能匹配解出。



![AuthenticationKeys](./identity.diagram.9.png)

In the example above, Mary uses her private key to sign the message. The signature
can be verified by anyone who sees the signed message using her public key.

在上面的例子中，Mary使用他的私钥对信息进行了签名，这个签名可以使用对应的公钥验证。

## Certificate Authorities

As you've seen, an actor or a node is able to participate in the blockchain network,
via the means of a **digital identity** issued for it by an authority trusted by the
system. In the most common case, digital identities (or simply **identities**) have
the form of cryptographically validated digital certificates that comply with X.509
standard and are issued by a Certificate Authority (CA).

一个参与者或者一个节点是可以加入区块链网络的，只要具备合理授权机构签发的数字身份。在很多情况下，数字身份 有 加密可验证的数字格式 符合 X.509标准 由 CA签发的。

role = 角色
actor = 
user = 用户

CAs are a common part of internet security protocols, and you've probably heard of
some of the more popular ones: Symantec (originally Verisign), GeoTrust, DigiCert,
GoDaddy, and Comodo, among others.

作为一套安全协议，经常能听说的CA有，赛门铁克，GeoTrust.....

![CertificateAuthorities](./identity.diagram.11.png)

*A Certificate Authority dispenses certificates to different actors. These certificates
are digitally signed by the CA and bind together the actor with the actor's public key
(and optionally with a comprehensive list of properties). As a result, if one trusts
the CA (and knows its public key), it can trust that the specific actor is bound
to the public key included in the certificate, and owns the included attributes,
by validating the CA's signature on the actor's certificate.*

CA  分发数据证书给不同的参与者，这些由CA签发的证书和参与者的公钥绑定，如果用户信任CA，则用户则信任CA签发的公钥。

Certificates can be widely disseminated, as they do not include either the
actors' nor the CA's private keys. As such they can be used as anchor of
trusts for authenticating messages coming from different actors.

证书可以被广泛的分发，分发时候没有包含参与者或者CA的私钥，他们可以作为来自不同参与者的消息可信度验证方


CAs also have a certificate, which they make widely available. This allows the
consumers of identities issued by a given CA to verify them by checking that the
certificate could only have been generated by the holder of the corresponding
private key (the CA).

CA自己也有证书。这使得CA的用户有机会验证CA签发的证书是否是CA自己的私钥签发的。

In a blockchain setting, every actor who wishes to interact with the network
needs an identity. In this setting, you might say that **one or more CAs** can be used
to **define the members of an organization's from a digital perspective**. It's
the CA that provides the basis for an organization's actors to have a verifiable
digital identity.

在区块链的设置中，所有希望和网络有交互的参与者都必须有认证，一个或者多个CA可以用于证实一个组织的成员的数字身份，因为CA提供了组织成员可验证的数字身份。

### Root CAs, Intermediate CAs and Chains of Trust

CAs come in two flavors: **Root CAs** and **Intermediate CAs**. Because Root CAs
(Symantec, Geotrust, etc) have to **securely distribute** hundreds of millions
of certificates to internet users, it makes sense to spread this process out
across what are called *Intermediate CAs*. These Intermediate CAs have their
certificates issued by the root CA or another intermediate authority, allowing
the establishment of a "chain of trust" for any certificate that is issued by
any CA in the chain. This ability to track back to the Root CA not only allows
the function of CAs to scale while still providing security --- allowing
organizations that consume certificates to use Intermediate CAs with confidence
--- it limits the exposure of the Root CA, which, if compromised, would endanger
the entire chain of trust. If an Intermediate CA is compromised, on the other
hand, there will be a much smaller exposure.


https://www.jianshu.com/p/27d6853d6c4c

CA有两种类型：根CA或者中级CA，因为根CA要分发成千山万的证书给用户，所以使用了中级CA来帮助分发证书。中级CA有根CA签发的证书或者其他中级CA签发的证书，这样任何一个中级CA的证书都能追溯到其根CA上，形成所谓的信任链。这种追溯信任链的能力不光 能让用户对中级CA有足够的信任，还能降低根CA出问题的风险：一个中级CA泄漏仅仅是泄漏自己，而不会影响到根CA。

![ChainOfTrust](./identity.diagram.1.png)

*A chain of trust is established between a Root CA and a set of Intermediate CAs
as long as the issuing CA for the certificate of each of these Intermediate CAs is
either the Root CA itself or has a chain of trust to the Root CA.*

信任链建立在根CA和一系列中级CA上，只要。。。。

Intermediate CAs provide a huge amount of flexibility when it comes to the issuance
of certificates across multiple organizations, and that's very helpful in a
permissioned blockchain system (like Fabric). For example, you'll see that
different organizations may use different Root CAs, or the same Root CA with
different Intermediate CAs --- it really does depend on the needs of the network.

在一个权限管理的区块链系统中，当给很多组织提供证书安全时，中级CA提供了很大量的灵活性，比如，你会看到不同的组织可能会使用不同的根CA，或者同一个根CA，但是使用不同的中级CA，这完全取决于对网络的需求。

### Fabric CA

It's because CAs are so important that Fabric provides a built-in CA component to
allow you to create CAs in the blockchain networks you form. This component --- known
as **Fabric CA** is a private root CA provider capable of managing digital identities of
Fabric participants that have the form of X.509 certificates.
Because Fabric CA is a custom CA targeting the Root CA needs of Fabric,
it is inherently not capable of providing SSL certificates for general/automatic use
in browsers. However, because **some** CA must be used to manage identity
(even in a test environment), Fabric CA can be used to provide and manage
certificates. It is also possible --- and fully appropriate --- to use a
public/commercial root or intermediate CA to provide identification.

因为CA机制非常重要，所以Fabric提供了内建的CA组件，这样可以在区块链网络中使用自签发的CA。这个组件被称为
Fabric CA，是私有的根CA提供者，可以实现区块链网络内部的基于X.509标准的证书签发。因为Fabric CA是一种自定义来实现Fabric网络里面的根的CA，它并不适合用来提供网络上使用的，可以用于浏览器的ssl加密的证书。无论如何，因为CA必须用于管理身份，Fabric CA可以用于提供和管理证书。所以也可以也完全适合使用商业伤的CA根证书或者中级证书来解决首Fabric网络的授权问题。

If you're interested, you can read a lot more about Fabric CA
[in the CA documentation section](http://hyperledger-fabric-ca.readthedocs.io/).

如果有兴趣，可以阅读Fabric CA相关文档。

## Certificate Revocation Lists

A Certificate Revocation List (CRL) is easy to understand --- it's just a list of
references to certificates that a CA knows to be revoked for one reason or another.
If you recall the store scenario, a CRL would be like a list of stolen credit cards.

CRL作为证书的吊销记录，就是一个因为某些被吊销的证书的列表。回忆一下商店那个应用场景，CRL就像一个已挂失信用卡的列表。

When a third party wants to verify another party's identity, it first checks the
issuing CA's CRL to make sure that the certificate has not been revoked. A
verifier doesn't have to check the CRL, but if they don't they run the risk of
accepting a compromised identity.



![CRL](./identity.diagram.12.png)

*Using a CRL to check that a certificate is still valid. If an impersonator tries to
pass a compromised digital certificate to a validating party, it can be first
checked against the issuing CA's CRL to make sure it's not listed as no longer valid.*

Note that a certificate being revoked is very different from a certificate expiring.
Revoked certificates have not expired --- they are, by every other measure, a fully
valid certificate. For more in-depth information about CRLs, click [here](https://hyperledger-fabric-ca.readthedocs.io/en/latest/users-guide.html#generating-a-crl-certificate-revocation-list).

Now that you've seen how a PKI can provide verifiable identities through a chain of
trust, the next step is to see how these identities can be used to represent the
trusted members of a blockchain network. That's where a Membership Service Provider
(MSP) comes into play --- **it identifies the parties who are the members of a
given organization in the blockchain network**.

To learn more about membership, check out the conceptual documentation on [MSPs](../membership/membership.html).


<!---
Licensed under Creative Commons Attribution 4.0 International License https://creativecommons.org/licenses/by/4.0/
-->
