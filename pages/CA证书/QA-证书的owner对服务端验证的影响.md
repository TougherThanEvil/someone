---
layout: page
title: "QA-证书的owner对服务端验证的影响"
---

#### 证书的私钥Owner中有一些关键的关于身份的说明元素，比如CN/OU/C，我们公司的ELK服务端直接通过DN（也就是整个owner的string）配置证书的role，导致了新证书中的OU顺序有变化然后匹配到正确的role，访问ELK服务返回403报错。

ELK在kibana的Role mapping页面里配置了角色和用于mapping角色的DN信息：

Roles: client_app_name_xxx_rw

Mapping Rules: dn text = "CN=abc, OU=ou1, OU=ou2, O=xxx, C=CN"

可以看到，ELK这边通过整个证书的owner来匹配我们客户端的然后映射到对应的role上。

我们的Elasticsearch这边证书到期了，需要生成新的证书密钥对。

密钥生成了，根CA证书也导入了，验证链接是好的，就是报403没有权限。。

密钥是自动生成的，owner里面有两个OU，然后这两个OU的顺序变了。

新的证书的Subject："CN=xxx, **OU=ou2, OU=ou1**, O=xxx, C=CN"

原来这导致了mapping匹配不上，我们客户curl测试结果显示403

通过查看ELK的auth接口更直观的看到没有任何role

-bash-xx$ curl https://xx-xxx:9200/_xpack/security/_audithenticate?pretty --key /xx/xx/privatekey.key__--pass xxxx --cert /xx/xx/certificate.cer -k -v

{

"username":"abc",

**"roles":[],**

...

}

我用GPT查了这写身份元素有没有顺序的规定，回答很标准， 如下：

    证书私钥的owner中的CN（Common Name）和OU（Organizational Unit）通常是有序的，但并不是严格要求有序的。根据X.500标准，证书主体的命名应该按照特定的顺序组织，但实际上并不是所有证书都严格遵循这一规定。

通常情况下，CN位于主体名称的最前面，表示最重要的信息，如域名或者个人姓名。而OU通常位于CN之后，表示组织单位。虽然这是常见的顺序，但并不是所有证书都按照这个顺序来排列。

因此，虽然通常情况下CN和OU是有序的，但在实际情况中，不要依赖于它们的顺序。最好的做法是在解析证书时，不要假设它们的顺序，而是根据它们的标识符来正确地解析证书的主体信息。

所以跟我们ELK的同时沟通了，他们是可以修改这个的，需要修改代码区提取CN做判断，而不是拿整个owner/subject去做映射
