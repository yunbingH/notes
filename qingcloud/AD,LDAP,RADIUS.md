#### RADIUS服务器

FW与RADIUS服务器之间使用RADIUS协议通信，RADIUS协议使用UDP协议作为传输协议，具有良好的实时性；同时也支持重传机制和备用服务器机制，从而具有较好的可靠性。FW与RADIUS服务器之间使用共享密钥对传输的报文进行加密，具有较好的安全性。

RADIUS协议的实现比较简单，适用于大用户量时服务器端的多线程结构。



#### LDAP服务器

FW与LDAP服务器之间使用LDAP协议通信。LDAP是轻量级目录访问协议的简称，是一种基于TCP/IP的访问在线目录服务的协议。LDAP协议的典型应用是用来保存系统的用户信息，用于用户登录时的认证和授权。LDAP的目录服务功能建立在Client/Server的基础之上，所有的目录信息存储在LDAP服务器上。

目录是一组具有类似属性、以一定逻辑和层次组合的信息。LDAP协议中目录是按照树型结构组织，目录由条目（Entry）组成，条目是具有区别名DN（Distinguished Name）的属性（Attribute）集合。属性由类型（Type）和多个值（Values）组成。LDAP目录树的最顶部就是根，根的区别名称为“Base DN”。

如[图1](https://51hcie.com/p6/file/admin/sec_admin_auth_server_0002.html#sec_admin_auth_server_0002__fig01)所示，通过LDAP Client来查看LDAP协议的目录结构。LDAP服务器所管辖域svn.com中存在组织单元“ou_test”，其中包括aa、bb、dd和ff条目，条目“aa”的DN为：“CN=aa,OU=ou_test,dc=svn,dc=com”。

**图1** LDAP目录结构
![image-20210421111409434](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210421111409.png)

LDAP认证基本过程如下：

1. LDAP客户端使用具有足够目录资源访问权限的用户DN（一般使用LDAP服务器管理员DN）与LDAP服务器进行绑定，获得查询权限。
2. LDAP客户端使用待认证用户名构造查询条件，在LDAP服务器指定根目录下查询此用户，获取用户DN。
3. LDAP客户端使用该用户DN和密码与LDAP服务器进行绑定，验证用户密码是否正确。

#### AD服务器

AD是Windows Server域环境中提供目录服务的组件，可以将AD理解为LDAP在微软平台的一种实现方式。

活动目录将登录身份验证以及目录对象的访问控制集成在一起，管理员可以管理分散在网络各处的目录数据和组织单位，经过授权的网络用户可以访问网络任意位置的资源。

如[图2](https://51hcie.com/p6/file/admin/sec_admin_auth_server_0002.html#sec_admin_auth_server_0002__fig02)所示，AD服务器所管辖域cce.com中包括“研发部”和“市场部”两个部门，则“研发部”的DN为：“OU=研发部,DC=cce,DC=com”。

**图2** AD目录结构
![image-20210421111431669](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210421111431.png)

AD服务器认证与LDAP服务器认证类似，差别在于AD服务器认证包含了一个Kerberos认证和一个标准的LDAP认证过程，LDAP服务器认证只有LDAP认证过程。