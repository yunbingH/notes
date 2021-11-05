## 1. 安装并设置OpenVPN的服务器环境

以下步骤将完成服务器端的设置。

### 1.1 OPENVPN和EASYRSA的安装

首先更新Ubuntu的Repository列表，并安装OpenVPN和EasyRSA。

```
#apt-get update
#apt-get install openvpn easy-rsa
```

### 1.2 创建OPENVPN的配置文件SERVER.CONF

DigitalOcean的Droplet初始化之后在**/usr/share/doc/openvpn/example/**这个路径下有示例配置，所以将服务器端的示例配置文件拷贝到openvpn的安装路径下。

```
#gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz &gt; /etc/openvpn/server.conf

```

### 1.3 编辑SERVER.CONF，配置OPENVPN

然后编辑解压出来的**server.conf**文件，可以使用自己喜欢的编辑器，DigitalOcean的指南里推荐了vim。不过nano比较简单，我还是选择了nano。

```
#nano /etc/openvpn/server.conf
```

**server.conf**中有几处需要改的地方，推荐用CTRL+W搜索能够比较快的找到。

如果需要修改OpenVPN的端口号的话可以修改port之后的数字。默认的端口号是1194.

```
# Which TCP/UDP port should OpenVPN listen on?
# If you want to run multiple OpenVPN instances
# on the same machine, use a different port
# number for each one.  You will need to
# open up this port on your firewall.
port 1194
```

如果使用的是ipv6进行连接的话（比如教育网），需要将下面所示的udp修改为udp6。如果使用ipv4进行连接的话，则不用修改此处。

```
# TCP or UDP server?
;proto tcp
proto udp
```

ipv6链接的配置修改之后如下所示。

```
# TCP or UDP server?
;proto tcp
proto udp6
```

之后再找到如下所示的地方。

```
# Diffie hellman parameters.
# Generate your own with:
#   openssl dhparam -out dh1024.pem 1024
# Substitute 2048 for 1024 if you are using
# 2048 bit keys.
dh dh1024.pem
```

将上文中的**dh1024.pem**修改为**dh2048.pem**，此处修改的为RSA的密钥长度，自然是长度越长加密程度越高。修改之后的如下所示。（井号之后的内容都是注释，就不再写第二遍了。）

```
dh dh2048.pem
```

之后仍旧在**server.conf**中找到以下这处。

```
# If enabled, this directive will configure
# all clients to redirect their default
# network gateway through the VPN, causing
# all IP traffic such as web browsing and
# and DNS lookups to go through the VPN
# (The OpenVPN server machine may need to NAT
# or bridge the TUN/TAP interface to the internet
# in order for this to work properly).
;push "redirect-gateway def1 bypass-dhcp"
```

去掉push前的分号，修改之后如下所示。之后OpenVPN的服务端才会使所有的网络流量通过VPN（如同注释所说的一样）。

```
push "redirect-gateway def1 bypass-dhcp"
```

之后仍旧在**server.conf**中找到以下这处。

```
# Certain Windows-specific network settings
# can be pushed to clients, such as DNS
# or WINS server addresses.  CAVEAT:
# http://openvpn.net/faq.html#dhcpcaveats
# The addresses below refer to the public
# DNS servers provided by opendns.com.
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"
```

去掉两个**push**之前的两个分号，修改为以下所示。

```
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
```

修改之后OpenVPN才会通过设置的DNS进行解析。此处使用的208.67.222.222和208.67.220.220为opendns的地址，也可以使用google的8.8.8.8之类的地址。

之后修改**server.conf**，找到下方所示的地方。

```
# You can uncomment this out on
# non-Windows systems.
;user nobody
;group nogroup
```

去掉**user**和**group**之前的分号，最后结果如下。

```
user nobody
group nogroup
```

此处如果不修改的话OpenVPN会以root用户的权限运行。出于安全的考虑，会将其权限限制在用户nobody和用户组nogroup的等级上，而其没有特权，所以能保证安全。

之后找到如下所示的地方。

```
# Select a cryptographic cipher.
# This config item must be copied to
# the client config file as well.
;cipher BF-CBC        # Blowfish (default)
;cipher AES-128-CBC   # AES
;cipher DES-EDE3-CBC  # Triple-DES
```

去掉第二个**cipher**之前的分号，选择使用AES进行加密，修改之后我们之后还会在客户端的配置文件中修改加密方式使其对应。如果不需要加密的话此处也可以不修改。

修改之后的结果为

```
;cipher BF-CBC        # Blowfish (default)
cipher AES-128-CBC   # AES
;cipher DES-EDE3-CBC  # Triple-DES
```

修改完此处之后，**server.conf**已经修改完毕。如果使用的是nano编辑的话，可以通过CTRL+x退出，记得保存。

### 1.4 设置包转发

在终端中输入

```
#echo 1 > /proc/sys/net/ipv4/ip_forward
```

这样服务器才会将客户端的网络流量转发到互联网中，否则流量只会停止在服务器这一点，无法到达用户指定的地址。

然而按照上面所示的方法，在重启之后包转发的设置会被重置为不进行转发。为了重启之后也能正常工作，编辑如下文件。（我还是使用了nano，毕竟简单。）终端中输入

```
#nano /etc/sysctl.conf
```

照旧找到以下所示一处。

```
# Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1
```

将**net.ipv4.ip_forward**前面的井号去掉，取消注释。修改之后如下所示。

```
# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1
```

然而CTRL+x退出，同样要记得保存。

### 1.5 打开UNCOMPLICATED FIREWALL(UFW)防火墙 UFW是UBUNTU 14.04自带的一个防火墙。和它的名字一样，配置这个UFW并不是很复杂。当然如果觉得不需要防火墙保护则可以直接跳过这部分。

在终端输入如下的话，可以完成ipv4的配置。首先让ufw允许ssh，否则无法ssh登录了；之后因为之前在**server.conf**中使用了1194端口和udp协议，所以需要允许1194端口中的udp流量。此处需要注明的是，如果使用的ipv6进行连接的话，可能需要将udp修改为udp6。而且如果同时还跑着其他的程序占用其他端口，同样需要设置对应的端口。比如我的vps中还运行着shadowsocks，一开始开了OpenVPN之后shadowsocks就不能用了，想了一会才发现是ufw的锅。所以干脆我就不用ufw了...

```
#ufw allow ssh
#ufw allow 1194/udp
```

之后修改**/etc/default/ufw**文件修改ufw的转发策略。终端中输入

```
#nano /etc/default/ufw
```

找到**DEFAULT_FORWARD_POLICY="DROP"**，这里的**DROP**必须修改为**ACCEPT**。修改之后如下所示。

```
DEFAULT_FORWARD_POLICY="ACCEPT"
```

之后要为ufw添加额外的规则。终端中输入

```
#nano /etc/ufw/before.rules
```

修改**before.rules**文件，在文件中中间添加一部分。修改完之后样子如下（我已经看不懂在干啥了...总之照做之后能用...）

```
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#
 
# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0] 
# Allow traffic from OpenVPN client to eth0
-A POSTROUTING -s 10.8.0.0/8 -o eth0 -j MASQUERADE
COMMIT
# END OPENVPN RULES
 
# Don't delete these required lines, otherwise there will be errors
*filter
```

设置完ufw之后，我们可以开启ufw了。终端中输入来开启ufw。

```
#ufw enable
```

之后会提示

```
Command may disrupt existing ssh connections. Proceed with operation (y|n)?
```

输入y并回车，此时ufw已启动。

如果需要查看ufw的规则，终端中输入

```
#ufw status
```

如果输出结果类似如下，则配置完成。（视之前的规则设置结果会有不同。）

```
Status: active
 
To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
1194/udp                   ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
1194/udp (v6)              ALLOW       Anywhere (v6)
```

如果觉得ufw好烦没必要，也可以关闭掉ufw，终端中输入

```
#ufw disable
```

即可关闭ufw。

## 2. 创建证书认证和服务器端的证书和密钥

OpenVPN使用证书来加密流量。

### 2.1 设置并创建证书认证

这一步中将设置证书认证，并创建OpenVPN的证书和密钥。OpenVPN可以使用双向的基于证书的认证方法，也就是说在服务端和客户端建立互信关系之前，服务端和客户端需要互相验证证书。这一步中我们将要使用EasyRSA中的脚本来生成证书。

首先将EasyRSA的脚本拷贝到OpenVPN的目录下。终端中输入

```
#cp -r /usr/share/easy-rsa/ /etc/openvpn
```

之后在其下创建一个存放密钥的目录keys。终端中输入

```
#mkdir /etc/openvpn/easy-rsa/keys
```

EasyRSA有一个变量文件**vars**，我们可以编辑它使证书能排除掉我们希望的用户以外的人。这些变量信息会被拷贝到证书和密钥中，之后还会在识别密钥的过程中起作用。终端中输入以下内容来编辑变量文件**vars**。

```
#nano /etc/openvpn/easy-rsa/vars
```

然后在**vars**文件中，以下所示的部分需要按照你的要求进行修改。

```
export KEY_COUNTRY="US"
export KEY_PROVINCE="TX"
export KEY_CITY="Dallas"
export KEY_ORG="My Company Name"
export KEY_EMAIL="sammy@example.com"
export KEY_OU="MYOrganizationalUnit"
```

之后仍然在**vars**这个文件中，找到并修改如下一行，修改之后如下所示。

```
export KEY_NAME="server"
```

出于简单的考虑，此处将使用『server』作为密钥的名字。如果需要使用不同的名字，还需要修改OpenVPN的配置文件中对于**server.key**和**server.crt**的引用。

之后需要生成Diffie-Hellman参数，这个操作会需要一段时间。终端中输入

```
#openssl dhparam -out /etc/openvpn/dh2048.pem 2048
```

之后切换到工作目录，也就是之前拷贝的EasyRSA的脚本所在目录。

```
#cd /etc/openvpn/easy-rsa
```

之后进行PKI（Public Key Infrastructure）的初始化，终端中输入以下命令，需要注意两个点之间有个空格。

```
#. ./vars
```

上面那个命令会输出以下内容。不过因为我们还没在keys目录下生成任何东西，所以并不需要担心这条警告。

```
NOTE: If you run ./clean-all, I will be doing a rm -rf on /etc/openvpn/easy-rsa/keys
```

之后我们会清理当前的工作目录，防止以前的旧密钥或是示例密钥遗留下来。终端中输入

```
#./clean-all
```

之后最后一个命令将会建立证书认证，其会使用一个互动式的OpenSSL的命令（说白了就是你问我答...）。弹出的提示将会提示你确认之前vars文件中输入的那些信息（国家、省份、组织之类的）。

```
#./build-ca
```

简单一些的话，一路留空回车即可。如果需要修改哪一条的话，在这里修改也可以。

### 2.2 生成服务端的证书和密钥

前面说道OpenVPN的证书验证是双向的，服务端和客户端都要有证书和密钥。这一步将会生成服务端的证书和密钥。

现在仍旧在**/etc/openvpn/easy-rsa**这个目录下，终端中输入以下命令来创建服务端的密钥。需要注意的是，这句命令中的参数『server』需要和之前2.1部分中**vars**文件中**export KEY_NAME**后的名字相同。

```
#./build-key-server server
```

然后会输出类似于2.1部分最后一步**./build-ca**之后的提示。照例可以一路回车过去。不过这次会多出两个提示，如下所示。

```
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

两个提示都需要留空，所以直接回车跳过即可。

之后会又有两个询问，分别输入y再回车确认即可。

```
Sign the certificate? [y/n]
1 out of 1 certificate requests certified, commit? [y/n]
```

照着做完之后，会出现如下提示。

```
Write out database with 1 new entries
Data Base Updated
```

### 2.3 移动一下服务端的证书和密钥

按照之前的**server.conf**的设置，我们需要将证书和密钥移动到**/etc/openvpn**这个目录下。终端中输入

```
#cp /etc/openvpn/easy-rsa/keys/{server.crt,server.key,ca.crt} /etc/openvpn
```

这个时候OpenVPN的服务端已经可以运行了，终端中输入以下内容来启动OpenVPN的服务端并检查下运行状态。

```
#service openvpn start
#service openvpn status
```

上面第二条是查询状态的命令，其会输出如下内容。

```
VPN 'server' is running
```

## 3. 生成客户端的证书和密钥

到此为止，我们已经安装并配置了OpenVPN服务端，建立了证书认证，并建立了服务端的证书和密钥。在接下来，我们将使用服务端的证书认证来创建每一个客户端的证书和密钥。这些证书和密钥之后将被安装在客户端所在的设备上（比如笔记本和手机等）。

### 3.1 创建密钥和证书

比较理想的做法是，为每一个连接到VPN的客户端准备一个单独的证书和密钥。另外最好在创建一个通用的证书和密钥供所有的客户端使用。

> 需要注意的是，OpenVPN默认不允许不同的客户端通过同一个证书进行连接。（在**server.conf**的**duplicate-cn**部分可以查看设置）

为每个客户端创建密钥和证书，你需要为每个客户端执行一遍本部分中的命令，不过需要将本部分中的client1的名字改成另外的名字，比如client2、phone2之类的名字。由于使用了不同的证书，所以每个客户端之后都可以从客户端单独的切断连接。本部分剩余部分将使用client1作为示例的客户端名称。

现在首先为client1客户端创建一个密钥，此时你应当仍旧处于**/etc/openvpn/easy-rsa**这个路径下。终端中输入

\#./build-key client1

像上面的一样，你会被提示要求确认vars文件中你输入的国家、省份、组织之类的一些信息，一路留空回车过去即可。同时也会像2.2部分中一样被要求多进行两相确认，提示如下。

```
Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

同2.2部分中一样，上面两个分别留空并回车确认。

之后也会同之前一样有两个要求确认的地方，如下所示。

```
Sign the certificate? [y/n]
1 out of 1 certificate requests certified, commit? [y/n]
```

同样输入y并回车两次分别确认。

如果密钥成功创建了的话，终端中会有以下输出。

```
Write out database with 1 new entries
Data Base Updated
```

类似于服务端的配置文件server.conf，客户端同样在路径**/usr/share/doc/openvpn/examples**下有实例配置文件。我们会将其作为模板，以备下载到客户端之后进行修改。拷贝过程中我们将会把实例配置文件client.conf的扩展名conf修改为ovpn，因为客户端软件需要的文件是ovpn扩展名的。在终端中执行以下命令。

```
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/easy-rsa/keys/client.ovpn

```

到此一个客户端的证书和密钥就配置并创建完毕了。如果需要多组密钥和证书，所需要做的就是重复本部分。当然就像前面提到的，还需要将client1的名字修改为其他的名字。

### 3.2 将证书和密钥拷贝到客户端设备中

*这部分DigitalOcean的指南中介绍了很多方法。不过我觉得准备配置OpenVPN的人中，如果是Linux用户，那这点事情肯定难不住他；如果是windows用户，推荐使用WinSCP，特别傻瓜，全是窗口操作，没啥好说的；没用过mac，所以我也不知道该咋办(= =||)。*

对于每个客户端我们需要从服务端拷贝一份该客户端对应的证书、密钥和客户端示例配置文件。

继续以上文中的client1作为例子，我们的客户端client1的证书和密钥在服务器的以下路径中。

- /etc/openvpn/easy-rsa/keys/client1.crt
- /etc/openvpn/easy-rsa/keys/client1.key

另外对于所有的客户端，证书认证文件和客户端的示例配置文件在如下所示的路径中。需要注意证书认证文件在openvpn的根目录下，跟剩下三个不在一个路径下面。

- /etc/openvpn/easy-rsa/keys/client.ovpn
- /etc/openvpn/ca.crt

本部分完成之后，你的客户端设备中将有以下四个文件。

- client1.crt
- client1.key
- client.ovpn
- ca.crt

## 4. 创建一个OpenVPN的客户端档案

管理客户端文件有许多方法，然而最简单的一种方法就是把所有的文件都放到一个档案文件中。我们只需要修改配置文件模板client.ovpn，使其包含服务端的证书认证、服务端的证书和密钥。修改完毕之后，所需要导入的配置文件就只有client.ovpn这一个文件了。

接下来我们将在客户端设备上修改之前下载的四个文件。之前的客户端配置文件模板client.ovpn需要被复制并重命名，叫什么名字随意。名字并不需要和客户端设备相关，客户端软件将会将这个文件的名字当作VPN连接的名字。所以client.ovpn的命名应当相同与你想要其出现在你操作系统中的名字，比如work就重命名为work.ovpn等。

在这篇指南中，我们将会想要VPN的名字为DigitalOcean，所以client.ovpn将会被复制并重命名为DigitalOcean.ovpn，之后在编辑器中打开DigitalOcean.ovpn。

首先，如果使用的是ipv6连接vpn的话，类似于server.conf中的内容，DigitalOcean中的协议同样需要被修改。找到如下的一段内容

```
# Are we connecting to a TCP or
# UDP server?  Use the same setting as
# on the server.
;proto tcp
proto udp
```

将**proto udp**修改为**proto udp6**。如果使用ipv4连接的话，则不需要修改这一段。修改之后的效果为

```
;proto tcp
proto udp6
```

之后找到如下一段

```
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote my-server-1 1194
```

其中将**my-server-1**修改为你的服务器的地址。如果使用的是ipv4连接VPN，则将**my-server-1**修改为服务器的ipv4地址；同样如果使用ipv6连接VPN则将**my-server-1**修改为ipv6地址（切记看清楚不要设置成网关的，说多了都是泪）

之后找到下面一段，去掉**user**和**group**前面的分号，作用和第1部分中说的一样。客户端如果运行在Windows系统中，那么可以忽略这一步。修改之前为

```
# Downgrade privileges after initialization (non-Windows only)
;user nobody
;group nogroup
```

修改之后为

```
# Downgrade privileges after initialization (non-Windows only)
user nobody
group nogroup
```

之后找到如下一段

```
# SSL/TLS parms.
# . . .
ca ca.crt
cert client.crt
key client.key
```

在**ca、cert、key**前面加上井号将其注释，这样我们之后将证书认证、证书和密钥输入到ovpn文件中才会生效。修改之后的效果如下

```
# SSL/TLS parms.
# . . .
#ca ca.crt
#cert client.crt
#key client.key
```

之前如果在server.conf中设置了加密方式中（比如我们在第1部分中做的那样），那么在DigitalOcean.ovpn中找到如下一段。

```
# Select a cryptographic cipher.
# If the cipher option is used on the server
# then you must also specify it here.
;cipher
```

由于我们之前选择了AES-128-CBC的加密方法，所以我们此处的设置也与server.conf中相同。修改之后的效果如下。

```
# Select a cryptographic cipher.
# If the cipher option is used on the server
# then you must also specify it here.
cipher AES-128-CBC
```

最后在DigitalOcean.ovpn文件的末尾添加以下XML结构的内容。

```
<ca>
(在此处插入ca.crt的内容)
</ca>
<cert>
(在此处插入client1.crt的内容)
</cert>
<key>
(在此处插入client1.key的内容)
</key>
 
 
分别用编辑器打开这三个文件，并插入即可。需要注意的是client1.crt的内容和另外两个有些不同，不过不用管，直接加入进去就好了。修改之后的效果如下。
 
 
<ca>
-----BEGIN CERTIFICATE-----
. . .
-----END CERTIFICATE-----
</ca>
 
<cert>
Certificate:
. . .
-----END CERTIFICATE-----
. . .
-----END CERTIFICATE-----
</cert>
 
<key>
-----BEGIN PRIVATE KEY-----
. . .
-----END PRIVATE KEY-----
</key>
```

至此为止，我们的客户端配置文件已经创建并设置完毕。

## 5. 安装客户端的配置文件

我目前只在windows中设置过openvpn。所以此处只简单叙述下windows中的方法。毕竟之前的才是重头戏，后面的都已经好解决了。

首先将我们的DigitalOcean.ovpn的配置文件拷贝到OpenVPN的安装根目录下的config文件夹下。

然后管理员权限启动OpenVPN GUI。（一定要以管理员权限运行）

然后在系统托盘图标上右键，并点击connect即可。