## 特殊权限位[suid/sgid/t]

setuid(suid):针对命令和二进制程序的，当普通用户执行某个命令的时候，可以拥有这个命令对应用户的权限，即让普通用户可以以root用户的角色执行程序或命令。

setgid(sgid):希望一个目录被多个用户（同属一个组）共享，同一个组的用户可以处理。

粘滞位(t):把一个文件夹的权限都打开，然后共享文件，像/tmp一样，但是生产环境一般不使用。

### suid

```
chmod u+s a.txt
chmod 4777 a.txt
chmod u-s a.txt
```

```
问题： passwd文件的属组是root,表示只有root用户可以访问的文件，为什么普通用户依然可以使用该命令更改自己的密码？
答案：当普通用户[omd]使用passwd命令的时候，系统看到passwd命令文件的属性有大写s后，表示这个命令的属主权限被omd用户获得,也就是omd用户获得文件/etc/shadow的root的rwx权限
```

![passwd](\img\passwd.png)



当x位没有小写x执行权限的时候，suid的权限显示的就是大S。

```
suid和sudo都可以用于提高用户的操作权限，那他们的区别是什么？
          suid:是让所有用户拥有某个权限可以执行某个程序或命令  -->仅仅是一个工具，仅仅在执行过程中生效，所有人都可用
          sudo:是让某个用户获得特定的某个命令或者执行权限      -->身份提高，终身有效，某个用户才有的权利
```

---

### sgid

**setgid(sgid): 希望一个目录被多个用户(同属于一个组)共享，同一个组的用户可以处理**

```
    chmod g+s /home/omd/h    -->添加sgid
    chmod 2755 /home/omd/h    -->添加sgid
    chmod g-s /home/omd/h     -->取消sgid

```

### 粘滞位

**粘滞位(t): 把一个文件夹的权限都打开，然后共享文件，像/tmp一样，但是 生产环境一般不使用** 

```
chmod o+t h.txt       --> 添加粘滞位
chmod o-t h.txt        --> 取消粘滞位
chmod 1777 /home/omd  –>添加粘滞位
```



## 小结

suid 4000: 权限符s(S),用户位上的x位置 chmod 4777 h.txt
guid 2000: 权限符s(S),用户组位上的x位置 chmod 2777 h.txt
粘滞位 1000: 权限符t(T),其他用户位上的x位置 chmod 1777 /home/omd
  对应位有x则，字符权限小写，否则表现为大写