# 1.LDAP协议基础概念

## 1.1 从用途上阐述LDAP

它是一个存储静态相关信息的服务，适合“一次记录多次读取”。常用LDAP服务存储的信息：

> 公司的物理设备信息（如打印机，它的IP地址、存放位置、厂商、购买时间等）
> 公开的员工信息（地址、电话、电子邮件…）
> 合同和账号信息（客户信息、产品交付日期、投标信息、项目信息…） 凭证信息（认证凭证、许可证凭证…）

## 1.2 从数据结构上阐述LDAP

它是一个树型结构，能有效明确的描述一个组织结构特性的相关信息。在这个树型结构上的每个节点，我们称之为“条目（Entry）”，每个条目有自己的唯一可区别的名称（Distinguished Name ，DN）。条目的DN是由条目所在树型结构中的父节点位置（Base DN）和该条目的某个可用来区别身份的属性（称之为RDN如uid , cn）组合而成。对Full DN ：“shineuserid=linly , ou=Employee , dc=jsoso , dc=net”而言，其中Base DN：“ou=Employee , dc=jsoso , dc=net”，RDN：“shineuserid=linly”下面是一个LDAP服务器的数据结构图：



## 1.3 从协议衍化上阐述LDAP

它是“目录访问协议DAP——ISO X.500”的衍生，简化了DAP协议，提供了轻量级的基于TCP/IP协议的网络访问，降低了管理维护成本，但保持了强壮且易于扩充的信息框架。LDAP的应用程序可以很轻松的新增、修改、查询和删除目录内容信息。

# 2.LDAP目录条目（Directory Entry）简述

## 2.1 从Object Classes谈起

在LDAP目录数据库中，所有的条目都必须定义objectClass这个属性。这有点像Java语言里说阐述的“一切皆对象”的理念，每个条目（LDAP Entry）都要定义自己的Object Classes。Object Class可以看作是LDAP Entry的模板，它定义了条目的属性集，包括必有属性（requited attribute）和可选属性（option attribute）。这里要着重指出的是，在LDAP的Entry中是不能像关系数据库的表那样随意添加属性字段的，一个Entry的属性是由它所继承的所有Object Classes的属性集合决定的，此外可以包括LDAP中规定的“操作属性”（操作属性是一种独立于Object Class而存在的属性，它可以赋给目录中的任意条目）。如果你想添加的属性不在Object Classes定义属性的范畴，也不是LDAP规定的操作属性，那么是不能直接绑定（在LDAP中，给Entry赋予属性的过程称为绑定）到条目上的，你必须自定义一个含有你需要的属性的Object Class，而后将此类型赋给条目。

Object Class是可以被继承的，这使它看上去真的很像Java语言中的POJO对象。继承类的对象实例也必须实现父 类规定的必有属性（requited attribute），同时拥有父类规定的可选属性（option attribute）。继承类可以扩展父类的必有属性和可选属性。由于Object Class的继承特性，因此在一个LDAP Entry上，objectClass属性是一个多值属性，它涵盖了Object Class的完整继承树，如用户条目uid=Linly , ou=People, dc=jsoso , dc=net，它直接实现了inetorgperson这个对象类，那么它的objectClass属性值为inetorgperson，organizationalPerson，person，top。

## 2.2 从Object Classes到Directory Server Schema

上一章节中，我们了解了LDAP条目都要遵守的一个最重要的规定Object Classes，而实际上，对Entry更多更细的规范被涵盖在了Directory Server Schema（目录服务模式）中。Directory Schema声明了完整的LDAP数据的存储规范，这包括数据的字节大小、数值范围和格式定义。

默认的，在一个LDAP服务器上，都定义有一套标准的Schema和一套为服务器功能定制的Schema。用户在需要的时候，是可以定制自己的LDAP属性和Object Class，以扩展标准Schema的功能。在Sun Directory Server中，使用了标准LDAPv3 Schema，并在此基础上做了轻微的扩展。

在Schema中的标准属性（Standard Attributes）是一个键-值对,如：cn：linly ，属性ID（属性名）为cn，属性值为linly 。事实上，一个完整的条目就是由一系列的键-值对组成的。以下是一个完整的LDAP Entry：
引用：

```text
dn: uid=Linly,ou=People, dc=jsoso,dc=net 
telephoneNumber: 13950491407 
mail: linliangyi2005@gmail.com 
objectClass: top 
objectClass: person 
objectClass: organizationalPerson 
objectClass: inetorgperson 
cn: LinLiangyi 
userPassword: {SSHA}aPTgP47LeziVGqjPBI8343FwkcL3QgQQ9kirXw== 
creatorsName: uid=admin,ou=administrators,ou=topologymanagement,o=netscaperoot 
createTimestamp: 20080219070003Z 
nsUniqueId: 2deb0d01-deb811dc-8055dc88-5f880db9 
nsRoleDN: cn=MyAdminRole,ou=People,dc=jsoso,dc=net 
nsRoleDN: cn=secondRole,ou=People,dc=jsoso,dc=net 
cn;phonetic;lang-zh:: IA== 
preferredLanguage: zh-CN 
cn;lang-zh:: 5p6X6Imv55uKICA= 
givenName: liangyi 
givenName;lang-zh:: 6Imv55uK 
sn: lin 
sn;lang-zh:: 5p6X 
uid: linly 
manager: cosTemplateForPostalCode 
modifiersName: uid=admin,ou=administrators,ou=topologymanagement,o=netscaperoot 
modifyTimestamp: 20080227083015Z
```

在Schema中，对属性的定义包含以下内容：

> 一个唯一的属性名称
> 一个属性的OID（object identifier）
> 一段属性的文本描述信息
> 一个关联属性文法定义的OID
> 属性的单值/多值描述；属性是否是目录自有的；属性的由来；附加的一些匹配规则

此外Schema中最重要的部分就是我们上面提到的Object Classes，它实际上是预定义的一套捆绑成套的属性集合。在Schema定义中，Object Classes要包含以下内容：

> 一个唯一的名字
> 一个object identifier (OID) 定义Object Class
> 一个必有的属性集合
> 一个可选的属性集合

## 2.3 高级LDAP条目

在目录服务中，信息是以条目的形式被分层次的组织在一起的。LDAP提供了几种分组机制，使得信息管理更富有弹性。

## 2.3.1 静态组和动态组

组(Group)，声明一个目录条目的集合；

静态组（Static Group）：显式声明了一个它的集合成员，这种方式适用于少量明确的成员组合。

动态组（Dynamic Group）：它定义了一个过滤条件，所有匹配条件的条目都是组的成员。所以称之为动态组，是因为每次读取其组员名单时，要动态计算过滤条件。

使用组的优点是能够快速的查找所属的成员；缺点是，给出任意的成员，无法获知它所属的组。因此从数据关联关系上看，Group适合一对多的查询。

## 2.3.2 受管角色、过滤器角色和嵌套角色

角色（Role），它是条目的另一种集合形式。它与组不同的在于，给定一个任意的成员条目，我们能立刻获知它所属的角色。因此从数据关联关系上看，Role适合多对一的查询。角色定义仅对它们的父节点子树下面的目录条目有效。

受管角色（Managed Role），它等价于Group中的静态组，不同的是，Role不是把组员信息添加到自身属性中，而是将自身的DN添加到组员条目的nsroledn属性中。

过滤器角色（Filtered Role），它与动态组相似，通过定义条目过滤器来确定组员。

嵌套角色（Nested Role），它是对角色定义的一种嵌套形式。可以嵌套其他的嵌套角色的。嵌套角色的成员，是其包含的所有角色成员的合集。嵌套角色通过包含从属于其它子树下的角色，可以扩展其搜索的scope。

## 2.3.3 服务类CoS

服务类实际上是一种属性的共享机制，它无须定义条目间的关联关系，却可以做到数据同步和空间优化。例如，在一个公司目录下，拥有上千个员工，他们拥有相同的公司地址属性；在传统的条目中，地址属性分别存贮在员工条目里，这样不但浪费存储空间，一旦地址变更，则要对员工条目进行逐一修改。采用CoS机制后，公司地址属性被存放在一个对象内，员工条目通过引用这个对象来获得地址信息，从而缩小的存储空间损耗，并方便了信息的修改。

CoS仅对其父节点子树下面的目录条目有效。CoS机制包含两个部分，CoS 定义条目和CoS模板条目。定义条目描述了属性是如何被引用的；模板条目描述了属性的值。CoS机制包含3种类型：

## 2.3.3.1 指针服务类(Pointer CoS)

在Pointer CoS中，CoS包含一个定义Definition Entry，它指定了两个属性：1.共享属性的名称；2.提供共享数据的模板DN。 另外CoS还要有一个Template Entry，它要提供共享的数据。

在定义了Definition Entry和Template Entry后，Pointer CoS将为其父节点子树下面的所有条目（目标条目Target Entry）分配共享属性和模板所定义的值。示意图如下：



> Definition Entry：cn=PointerCoS , dc=example , dc= com定义了CoS的共享属性cosAttribute：postalCode，同时定义了CoS的模板DN cosTemplateDN：cn=cosTemplateForPostalCode，cn=data。
> Template Entry: 它是CoS的模板，定义了共享属性值 postalCode：45773。
> Target Entry：它是目标条目，因为它位于dc=example , dc=com的子树下。所以它获得了共享属性postalCode：45773。

## 2.3.3.2 间接服务类(Indirect CoS)

在使用间接服务类时，在Definition Entry条目中定义了CoS的共享属性cosAttribut和一个用来间接指向模板的属性cosIndirectSpecifier。

首先，我们需要用cosIndirectSpecifier的值A作为属性名，来检索CoS父节点子树中所有拥有A属性的条目，作为目标条目Target Entry。

其次，根据找到的Target Entry条目中A属性的值来定位模板对象。

最后，再分别根据找到的模板对象中拥有的共享属性值赋给对应的Target Entry。

例如，定义如下Definition Entry：

```text
dn: cn=generateDeptNum,ou=People,dc=example,dc=com 
objectclass: top 
objectclass: LDAPsubentry 
objectclass: cosSuperDefinition 
objectclass: cosIndirectDefinition 
cosIndirectSpecifier: manager 
cosAttribute: departmentNumber
```

该CoS定义对条目ou=People,dc=example,dc=com下的子树中所有具有manager属性的条目有效，同时设定其CoS模板指向manager属性的值所指向的条目。

又假定有如下的Template Entry条目，它具有属性departmentNumber：

```text
dn: cn=Carla Fuentes,ou=People,dc=example,dc=com 
… 
objectclass: cosTemplate 
… 
departmentNumber: 318842
```

同时在ou=People,dc=example,dc=com下有Target Entry如下：

```text
dn: cn=Babs Jensen,ou=People,dc=example,dc=com 
cn: Babs Jensen 
... 
manager: cn=Carla Fuentes,ou=People,dc=example,dc=com 
departmentNumber: 318842
```

因为该Entry具有manager属性，且在ou=People,dc=example,dc=com子树下，所以它成为了Target Entity。并且由于其manager的值指向模板cn=Carla Fuentes,ou=People,dc=example,dc=com，因此，它的departmentNumber为 318842。

## 2.3.3.3 经典服务类(Classic CoS)

经典服务类同间接服务类有点相似，它也是对属性的间接应用。在Classic CoS的定义条目中，除了共享属性定义外，还有两个定义，一个是cosTemplateDn，它指向模板条目的父节点；另一个是cosSpecifier，它的值指向目标条目的属性A。由目标条目的属性A的值来代替模板条目的RND。则目标条目的属性A的值加上cosTemplateDn的值恰好定义一个唯一的模板条目。

举例如下，首先是一个经典服务类的定义条目：

```text
dn: cn=classicCoS,dc=example,dc=com 
objectclass: top 
objectclass: LDAPsubentry 
objectclass: cosSuperDefinition 
objectclass: cosClassicDefinition 
cosTemplateDn: ou=People,dc=example,dc=com 
cosSpecifier: building 
cosAttribute: postalAddress
```

该定义条目指明了3个参数：

> 要共享的属性是postalAddress
> 模板条目的上下文前缀是ou=People,dc=example,dc=com
> 模板条目的RDN存储于目标条目的building属性中

其次，假定有如下模板条目：

```text
dn: cn=B07,ou=People,dc=example,dc=com 
objectclass: top 
objectclass: LDAPsubentry 
objectclass: extensibleobject 
objectclass: cosTemplate 
postalAddres: 7 Old Oak Street$Anytown, CA 95054
```

最后，我们假设有以下目标条目

```text
dn: cn=Babs Jensen,ou=People,dc=example,dc=com 
cn: Babs Jensen 
... 
building: B07 
postalAddres: 7 Old Oak Street$Anytown, CA 95054
```

由于目标条目中，building属性的值是B07，因此该条目的模板定义DN = B07加上ou=People,dc=example,dc=com ，即cn=B07,ou=People,dc=example,dc=com，因此目标条目的postalAddres 引用模板的值7 Old Oak Street$Anytown, CA 95054。

## 2.4 LDAP 目录搜索

LDAP搜索是目录服务最常用的功能之一。在LDAP服务中搜索要用到相应的Filter语句。Filter语句由3个部分组成

> 属性，如：cn，uid，操作属性如:objectClass,nsroledn
> 比较操作符，如 <, >,=,…
> 逻辑预算符，如: 与操作& , 或操作|, 非操作！

关于Filter语句组成的详细参数表如下：



## 2.4.1 搜索过滤器示例：

> 下列过滤器将搜索包含一个或多个 manager 属性值的条目。这也称为存在搜索：manager=
> *下列过滤器将搜索包含通用名 Ray Kultgen 的条目。这也称为等价搜索：cn=Ray Kultgen*
> *下列过滤器返回所有不包含通用名 Ray Kultgen 的条目：(!(cn=Ray Kultgen))*
> *下列过滤器返回的所有条目中都有包含子字符串 X.500 的说明属性：description=*X.500
> *下列过滤器返回所有组织单元为 Marketing 且说明字段中不包含子字符串 X.500 的条目：(&(ou=Marketing)(!(description=*X.500*)))
> 下列过滤器返回所有组织单元为 Marketing 且 manager 为 Julie Fulmer 或 Cindy Zwaska 的条目：(&(ou=Marketing)(|(manager=cn=Julie Fulmer,ou=Marketing,dc=siroe,dc=com)(manager=cn=Cindy Zwaska,ou=Marketing,dc=siroe,dc=com)))
> 下列过滤器返回所有不代表人员的条目：(!(objectClass=person))
> 下列过滤器返回所有不代表人员且通用名近似于 printer3b 的条目：(&(!(objectClass=person))(cn~=printer3b))

## 2.4.2 ldapsearch指令参数：

> -b 搜索的起始上下文
> -D 绑定搜索的账号Distinguished Name
> -h 主机名。地址
> -p LDAP服务端口
> -l 搜索的最大耗时
> -s 从上下文开始的搜索范围，有三个常量base（表示仅当前根对象）/one（当前根对象及下一级）/sub（当前根对象的全部子树）
> -W 绑定账号密码
> -z 返回结果的最大数量

## 2.4.3 搜索“操作属性”

在LDAP搜索中，操作属性在默认情况下是不会跟随搜索结果返回的。必须在搜索中明确显示的指定操作属性，如：

```text
ldapsearch -h linly.jsoso.net -p 5201 -D "cn=directory manager" -w password "objectclass=*" aci=accounts。
```

## 2.4.4 搜索“操作对象类”的条目

在LDAP中Role、CoS等对象被定义为特殊的Object Class——操作对象类（operational object class），在一般的搜索中，这类对象是不会作为结果返回给用户的。要想查找这些对象，必须在filter中显式定义要找这个对象类。例如：(objectclass=ldapsubentry)。

# 3.ACI权限控制

ACI（Access Control Instruction）访问控制指令是LDAP 服务中用以控制用户访问权限的有力手段。在目录的Entry中，aci属性记录了对该条目的多条访问控制指令。（aci属性是一个多值操作属性，可以赋予任意的LDAP条目）。

ACI的语法格式如下：

```text
aci: (target)(version 3.0;acl "name";permission bind_rules;)
```

target 指定了ACI要控制访问的目标属性（集合）或条目（集合）。target可以用DN，一个或多个属性，或者一个filter来定义。它是一个可选项。target语法是：

```text
关键字 <op> 表达式
```

target关键字表

version 3.0 这是一个必须的常量字窜，用以识别ACI的版本。

name 指定ACI的名称，可以使任意的字窜，只要区别于同一个条目aci属性下的其他ACI，这是一个必须属性。

permission 指定权限许可。权限包括：read、write、add、delete、search、compare、selfwrite、 proxy 或 all，其中all表示出了proxy之外的所有操作。权限语法：

```text
allow | deny (权限)
```

bind_rules 绑定规则。绑定规则定义了何人、何时，以及从何处可以访问目录。绑定规则可以是如下规则之一：

> 被授予访问权限的用户、组以及角色
> 实体必须从中绑定的位置
> 绑定必须发生的时间或日期
> 绑定期间必须使用的验证类型

绑定规则语法：

```text
keyword  = 或者 != "expression"; （注：timeofday 关键字也支持不等式<、<=、>、>=）
```

LDIF 绑定规则关键字表

ACI样例

> 1.用户 bjensen 具有修改其自己的目录条目中所有属性的权限。 aci:(target="ldap:///uid=bjensen,dc=example,dc=com")(targetattr=*)(version 3.0; acl "aci1"; allow (write) userdn="ldap:///self";)*
> *2.允许 Engineering Admins 组的成员修改 Engineering 业务类别中所有条目的 departmentNumber 和 manager 属性 aci:(targetattr="departmentNumber || manager")(targetfilter="(businessCategory=Engineering)") (version 3.0; acl "eng-admins-write"; allow (write) groupdn ="ldap:///cn=Engineering Admins, dc=example,dc=com";)*
> *3.允许匿名用户对o=NetscapeRoot下的条目读取和搜索 aci:(targetattr="*")(targetfilter=(o=NetscapeRoot))(version 3.0; acl "Default anonymous access"; allow (read, search) userdn="ldap:///anyone";)
> 4.向所有经过验证的用户授予对整个树的读取访问，可以在dc=example,dc=com 节点创建下列 ACI：aci:(version 3.0; acl "all-read"; allow (read)userdn="ldap:///all";)
> 5.允许对整个 http://example.com 树进行匿名读取和搜索访问，可以在dc=example,dc=com 节点创建下列 ACI：aci:(version 3.0; acl "anonymous-read-search";allow (read, search) userdn = "ldap:///anyone";)
> 6.授予Administrators 组对整个目录树写入的权限，则可以在 dc=example,dc=com 节点创建下列 ACI：aci:(version 3.0; acl "Administrators-write"; allow (write) groupdn="ldap:///cn=Administrators,dc=example,dc=com";)