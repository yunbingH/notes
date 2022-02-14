## 安装 Python-LDAP

在 Ubuntu/Debian 下安装 python-ldap 模块：

```
$ sudo apt-get install python-ldap
```

在 CentOS/RHEL 下安装 python-ldap 模块：

```
# yum install python-ldap
```

## 创建

创建一条 LDAP 新纪录。有个要注意的地方，我们的 LDAP 有个属性 active，用来判断用户帐号是否是激活的 attrs[‘active’] = ‘TRUE’，这里的 ‘TRUE’ 不能用小写的 ‘true’，刚开始被 LDAP 管理工具上的小写 ‘true’ 误导，老以为 Python 程序里也应该用小写，结果总报错。

![phpLDAPadmin](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210419164837.png)



```python
def ldap_add(firstname, lastname, username):
    l = ldap.open(LDAP_HOST)
    l.protocol_version = ldap.VERSION3
    l.simple_bind(LDAP_BIND, LDAP_PASS)

    cn = firstname + ' ' + lastname
    addDN = "cn=%s,ou=People,dc=vpsee,dc=com" % cn
    attrs = {}
    attrs['objectclass'] = ['top','person','inetOrgPerson','posixAccount','vpseeAccount']
    attrs['cn'] = cn
    attrs['givenName'] = firstname
    attrs['homeDirectory'] = '/home/people/%s' % username
    attrs['loginShell'] = '/bin/bash'
    attrs['sn'] = lastname
    attrs['uid'] = username
    attrs['uidNumber'] = ldap_newuid()
    attrs['gidNumber'] = ldap_getgid()
    attrs['active'] = 'TRUE'
    ldif = modlist.addModlist(attrs)
    l.add_s(addDN, ldif)
    l.unbind_s()
```

## 查找和读取

查找和读取一条 LDAP 纪录，比如根据 username 查找出 cn：

```python
def ldap_getcn(username):
    try:
        l = ldap.open(LDAP_HOST)
        l.protocol_version = ldap.VERSION3
        l.simple_bind(LDAP_BIND, LDAP_PASS)

        searchScope = ldap.SCOPE_SUBTREE
        searchFilter = "uid=*" + username + "*"
        resultID = l.search(LDAP_BASE, searchScope, searchFilter, None)
        result_set = []
        while 1:
            result_type, result_data = l.result(resultID, 0)
            if (result_data == []):
                break
            else:
                if result_type == ldap.RES_SEARCH_ENTRY:
                    result_set.append(result_data)
        return result_set[0][0][1]['cn'][0]
    except ldap.LDAPError, e:
        print e
```

## 更新

更新一条 LDAP 纪录，比如更新用户状态 active 为 false：

```python
def ldap_deactive(username):
    try:
        l = ldap.open(LDAP_HOST)
        l.protocol_version = ldap.VERSION3
        l.simple_bind(LDAP_BIND, LDAP_PASS)

        deactiveDN = ("cn=%s," + LDAP_BASE) % ldap_getcn(username)
        old = {'active':'TRUE'}
        new = {'active':'FALSE'}
        ldif = modlist.modifyModlist(old, new)
        l.modify_s(deactiveDN, ldif)
        l.unbind_s()
    except ldap.LDAPError, e:
        print e
```

## 删除

删除一条 LDAP 纪录：

```python
def ldap_delete(username):
    try:
        l = ldap.open(LDAP_HOST)
        l.protocol_version = ldap.VERSION3
        l.simple_bind(LDAP_BIND, LDAP_PASS)

        deleteDN = ("cn=%s," + LDAP_BASE) % ldap_getcn(username)
        l.delete_s(deleteDN)
    except ldap.LDAPError, e:
        print e
```