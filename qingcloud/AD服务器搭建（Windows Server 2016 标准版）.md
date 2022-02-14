# 创建虚拟机

登录http://console.qingdesktop.com/

计算->主机->创建

## 1. 选择系统

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402093404.png" alt="image-20210402093337826" style="zoom:67%;" />

## 2.连接桌面

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402093635.png" alt="image-20210402093635670" style="zoom:67%;" />



# 配置AD服务

## 1.启动server服务

server 服务务必须运行，否则虚机无法加域
在安装AD域服务器之前，务必开启**计算机管理**--服务--server
启动类型改成**自动**，启动。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402094136.png" alt="image-20210402094136495" style="zoom: 67%;" />



2）portal 若使用对 AD 的增删改，务必开启 ldap 协议，步骤如下

直接搜索  __组件服务__

或者 在开始-运行中执行 dcomcnfg 打开配置

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210408151732.png" style="zoom:67%;" />

打开我的电脑属性-默认属性，勾选启用分布式COM

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402095613.png" alt="image-20210402095613225" style="zoom: 67%;" />

默认协议

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402095733.png" alt="image-20210402095733864" style="zoom:67%;" />

编辑访问权限，启动和访问权限。



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402095832.png" alt="image-20210402095832874" style="zoom:67%;" />

勾选允许Everyone本地访问和远程访问。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402095905.png" alt="image-20210402095905382" style="zoom:67%;" />

勾选允许Everyone本地激活和远程激活

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100022.png" alt="image-20210402100022022" style="zoom:67%;" />



## 2.角色安装

打开服务器管理器，添加角色和功能。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100214.png" alt="image-20210402100214183" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100250.png" alt="image-20210402100250207" style="zoom:67%;" />

下一步

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100330.png" alt="image-20210402100330332" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100354.png" alt="image-20210402100354900" style="zoom:67%;" />

勾选域服务

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100612.png" alt="image-20210402100612510" style="zoom:67%;" />

点击添加功能

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100637.png" alt="image-20210402100637291" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100726.png" alt="image-20210402100726714" style="zoom:67%;" />

下一步

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100746.png" alt="image-20210402100746118" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100814.png" alt="image-20210402100814709" style="zoom:67%;" />

点击安装

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402100848.png" alt="image-20210402100848130" style="zoom:67%;" />

安装完成

## 3.AD域控制器部署向导

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402101412.png" alt="image-20210402101412564" style="zoom:67%;" />

如果点击了关闭，也可在服务器管理器仪表盘找到部署向导

![image-20210402115627394](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402115627.png)

因为是第一个AD控制器，点添加新林，填入根域名，点下一步。

> 将域控制器添加到现有域：在现有的域控制器中添加新的域控制器
>
> 将新域添加到现有林：在现有的林中新建域，与林中现有的域不同

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402101947.png" alt="image-20210402101947859" style="zoom:67%;" />

输入目录服务还原密码，其他默认，点下一步。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402102051.png" alt="image-20210402102051290" style="zoom:67%;" />

下一步

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402102141.png" alt="image-20210402102141576" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402102203.png" alt="image-20210402102203651" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402102226.png" alt="image-20210402102226650" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402102320.png" alt="image-20210402102320340" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402103200.png" alt="image-20210402103200098" style="zoom:67%;" />

点击安装，会自动重启。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402103820.png" alt="image-20210402103820171" style="zoom:67%;" />

重启后进入系统。此时AD部署成功，点工具-Active Directory用户和计算机，查看部署的域。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402104733.png" alt="image-20210402104733085" style="zoom:67%;" />

## 4.新建组织单位，用户。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402104838.png" alt="image-20210402104838429" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402104911.png" alt="image-20210402104911633" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402104947.png" alt="image-20210402104947676" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105032.png" alt="image-20210402105032083" style="zoom:67%;" />

写入密码后，下一步

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105124.png" alt="image-20210402105124614" style="zoom:67%;" />

完成。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105208.png" alt="image-20210402105208431" style="zoom:67%;" />

**AD认证服务配置：**

> 认证服务名称:atroot.cn
> 认证服务类型:ad
> 认证域:atroot.cn
> 认证 IP 地址:10.11.13.81
> 认证端口:389
> 双因子认证:已关闭
> 认证服务管理员:Administrator@atroot.cn
> 认证服务管理员密码:Zhu88jie
> 安全认证端口:636
> 基础组织单元:ou=qingcloud,dc=atroot,dc=cn
> 认证服务模式:读写模式
>
> 备注：
>
> > 基础组织单元:ou=qingcloud,dc=atroot,dc=cn 说明：
> >
> > AD域服务器里面查看 前面是qingcloud组织单元 后面这个相当于绑定的根目录，创建的用户和机器都在这个qingcloud下
> >
> > atroot.cn/qingcloud

## 5.安装认证服务证书

AD域服务器还需要安装认证服务证书，否则桌面云Portal-认证服务--无法
新建用户。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105323.png" alt="image-20210402105323775" style="zoom:67%;" />





<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105358.png" alt="image-20210402105358209" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105418.png" alt="image-20210402105418411" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105453.png" alt="image-20210402105453683" style="zoom:67%;" />

打勾

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105519.png" alt="image-20210402105519693" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105609.png" alt="image-20210402105609460" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402105946.png" alt="image-20210402105946573" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110007.png" alt="image-20210402110007153" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110042.png" alt="image-20210402110042090" style="zoom:67%;" />

点是，安装。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110105.png" alt="image-20210402110104987" style="zoom:67%;" />

完成。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110453.png" alt="image-20210402110453912" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110520.png" alt="image-20210402110520199" style="zoom:67%;" />



<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110632.png" alt="image-20210402110632195" style="zoom:67%;" />![image-20210402110654538](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110654.png)

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110654.png" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110715.png" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110734.png" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110752.png" alt="image-20210402110752459" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110832.png" alt="image-20210402110831969" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110906.png" alt="image-20210402110906181" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402110922.png" alt="image-20210402110922615" style="zoom:67%;" />

<img src="../AppData/Roaming/Typora/typora-user-images/image-20210402111007866.png" alt="image-20210402111007866" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402111853.png" alt="image-20210402111853253" style="zoom:67%;" />



# Portal端配置

## 创建认证服务器

![image-20210402112543312](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402112543.png)



## 创建区域

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402112320.png" alt="image-20210402112319869" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402112340.png" alt="image-20210402112340877" style="zoom:67%;" />

## 认证服务器配置

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402112638.png" alt="image-20210402112638851" style="zoom:67%;" />

## 云平台配置

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402112709.png" alt="image-20210402112709377" style="zoom:67%;" />

点击修改，填入信息。

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402112942.png" alt="image-20210402112942075" style="zoom:67%;" />

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210402113009.png" alt="image-20210402113009242" style="zoom:67%;" />



# 常见问题

##　１．桌面加域失败，手动加域提示错误：

<img src="https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210414101328.png" style="zoom:67%;" />



### 解决办法：
1）确定AD域服务器的防火墙都是出于关闭状态

![](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210414101303.png)

2）确定AD域服务器的 Microsoft网络的文件和打印机共享是开启状态
服务管理里面的server进程是启动状态



![](https://yunbingh.oss-cn-beijing.aliyuncs.com/img/20210414101306.png)