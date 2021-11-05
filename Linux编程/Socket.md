# 套接字连接

​		首先，服务器应用程序用系统调用socket来创建一个套接字，它是系统分配给该服务器进程的类似文件描述符的资源，它不能与其他进程共享。

​		接下来，服务器进程会给套接字起个名字。本地套接字的名字是Linux文件系统中的文件名，一般放在/tmp或/usr/tmp目录中。对于网络套接字，它的名字是与客户连接的特定网络有关的服务标识符（端口号或访问点）。这个标识符允许Linux将进入的针对特定端口号的连接转到正确的服务器进程。例如，Web服务器一般在80端口上创建一个套接字，这是一个专用于此目的的标识符。Web浏览器知道对于用户想要访问的Web站点，应该使用端口80来建立HTTP连接。我们用系统调用bind来给套接字命名。然后服务器进程就开始等待客户连接到这个命名套接字。系统调用listen的作用是，创建一个队列并将其用于存放来自客户的进入连接。服务器通过系统调用accept来接受客户的连接。

​		服务器调用accept时，它会创建一个与原有的命名套接字不同的新套接字。这个新套接字只用于与这个特定的客户进行通信，而命名套接字则被保留下来继续处理来自其他客户的连接。Web服务器就会这么做以同时服务来自许多客户的页面请求。对一个简单的服务器来说，后续的客户将在监听队列中等待，知道服务器再次准备就绪。

​		基于套接字系统的客户端更加简单。客户首先调用socket创建一个未命名套接字，然后将服务器的命名套接字作为一个地址来调用connect与服务器建立连接。

​		一旦连接建立，我们就可以像使用底层的文件描述符那样用套接字来实现双向的数据通信。

## 套接字属性

​		套接字的特性由3个属性确定，它们是：域(domain)、类型(type)和协议(protocol)。套接字还用地址作为它的名字。地址的格式随域（又被称为协议族，protocol family）的不同而不同。每个协议族又可以使用一个或多个地址族来定义地址格式。

### 1. 套接字的域

​		域指定套接字通信中使用的网络介质。最常见的套接字域是AF_INET，它指的是Internet网络。其底层的协议--网际协议（IP）只有一个地址族，它使用一种特定的方式来指定网络中的计算机，即人们常说的IP地址。

​		服务器计算机上可能同时有多个服务正在运行。客户可以通过IP端口来指定一台互联网机器上的某个特定服务。在系统内部，端口通过分配一个唯一的16位的整数来标识，在系统外部，则需要通过IP地址和端口号的组合来确定。套接字作为通信的终点，它必须在开始通信之前绑定一个端口。

### 2. 套接字类型

- 流套接字

  流套接字（在某些方面类似于标准的输入/输出流）提供的是一个有序，可靠，双向字节流的连接。因此，发送的数据可以确保不会丢失，重复或乱序到达，并且在这一过程中发生的错误也不会显示出来。大的消息将被分片，传输，再重组。这很像一个文件流，它接收大量的数据，然后以小数据块的行式将它们写入底层磁盘。流套接字的行为是可预见的。

  流套接字由类型SOCK_STREAM指定，它们是在AF_INET域中通过TCP/IP连接实现的。

- 数据报套接字

  与流套接字相反，由类型SOCK_DGRAM指定的数据报套接字不建立和维持一个连接。它对可以发送的数据报的长度由限制。数据报作为一个单独的网络消息被传输，它可能会丢失，重复或乱序到达。

  数据包套接字实在AF_INET域中通过UDP/IP连接实现的，它提供的是一种无序的不可靠服务。但从资源的角度来看，相对来说它们开销比较小，因为不需要维持网络连接。而且因为无需花费时间来建立连接，所以它们的速度也很快。

### 3. 套接字协议

​		只要底层的传输机制允许不止一个协议来提供要求的套接字类型，我们就可以为套接字选择一个特定的协议。网络套接字和文件系统套接字，它们不需要选择一个特定的协议，只需要使用其默认值0即可。

## 创建套接字

​		socket系统调用创建一个套接字并返回一个描述符，该描述符可以用来访问该套接字。

```
#include<sys/types.h>
#include<sys/socket.h>

int socket(int domain,int type,int protocol);
```

​		创建套接字是一条通信线路的一个端点。domain参数指定协议族，type参数指定这个套接字的通信类型，protocol参数指定使用的协议。

| domain  | 说明                             |
| ------- | -------------------------------- |
| AF_UNIX | UNIX域协议（文件系统套接字）     |
| AF_INET | ARPA因特网协议（UNIX网络套接字） |

​		socket函数的参数type指定用于新套接字的通信特性。它的取值包括SOCK_STREAM和SOCK_DGRAM。

​		参数protocol设置为0标识使用默认协议。

## 套接字地址

​		每个套接字域都有其自己的地址格式。

​		对于AF_UNIX域套接字来说，它的地址由结构sockaddr_un来描述，该结构定义再头文件sys/un.h中。sun_family指定地址类型（套接字域），短整型。套接字地址由sun_path成员中的文件名所指定。

```
struct sockaddr_un {
	sa_family_t  sun_family;   /*AF_UNIX*/
	char		 sun_path[];   /*pathname*/
};
```

​		在AF_INET域中，套接字地址由结构socket_in来指定，该结构定义在头文件netinet/in.h中，它至少包含一下几个成员：

```
struct sockaddr_in {
	short int		   sin_family; /* AF_INET */
	unsigned short in  sin_port;   /* Port number */
	struct in_addr	   sin_addr;   /* Internet address */
};
```

​		IP地址结构in_addr被定义为：

```
struct in_addr {
	unsigned long int    s_addr;
};
```

## 命名套接字

​		要想让通过socket调用创建的套接字可以被其他进程使用，服务器程序就必须给该套接字命名。这样AF_UNIX套接字就会关联到一个文件系统的路径名，AF_INET套接字就会关联到一个IP端口号。

```
#include<sys/socket.h>
int bind(int socket,const struct sockaddr *address,size_t address_len);
```

​		bind系统调用把参数address中的地址分配给与文件描述符socket关联的未命名套接字。地址结构的长度由参数address_len传递。地址的长度和格式取决于地址族。bind调用需要将一个特定的地址结构指针转换为指向通用地址类型（struct sockaddr *）。bind调用在成功时返回0，失败时返回-1并设置errno为

| errno值       | 说明                                 |
| ------------- | ------------------------------------ |
| EBADF         | 文件描述符无效                       |
| ENOTSOCK      | 文件描述符对应的不是一个套接字       |
| EINVAL        | 问价描述符对应的是一个已命名的套接字 |
| EADDRNOTAVAIL | 地址不可用                           |
| EADDRINUSE    | 地址已经绑定了一个套接字             |

AF_UNIX域套接字还有其他一些错误代码：

| EACCESS             | 因为权限不足，不能创建文件系统中的路径名 |
| ------------------- | ---------------------------------------- |
| ENOTDIR,ENAMTOOLONG | 表明选择的文件名不符合要求               |

## 创建套接字队列

​		为了能够在套接字上接受进入的连接，服务器程序必须创建一个队列来保存未处理的请求。用listen系统调用来完成这以工作。

```
#include<sys/socket.h>
int listen(int socket,int backlog);
```

​		参数backlog的值表示队列中可以容纳的未处理连接的最大数目。

​		listen函数在成功时返回0，失败时返回-1.

## 接受连接

一旦服务器程序创建并命名了套接字之后，它就可以通过accept系统调用来等待客户建立对该套接字的连接。

```
#include<sys/socket.h>
int accept(int socket,struct sockaddr *address,size_t *address_len);
```

​		accept系统调用只有当由客户程序试图连接到由socket参数指定的套接字上时才返回。这里的客户是指，在套接字队列中派第一个的未处理连接。accept函数将创建一个新套接字来与该客户进行通信，并且返回新套接字的描述符。新套接字的类型和服务器监听套接字类型是一样的。

​		参数address_len指定客户结构的长度。如果客户地址的长度超过这个值，它将被截断。所以在调用accept之前，address_len必须被设置为预期的地址长度。当这个调用返回时，address_len将被设置为连接客户地址结构的实际长度。

​		如果套接字队列中没有未处理的连接，accept将阻塞（程序将暂停）直到有客户建立连接为止。

## 请求连接

​		客户程序通过在一个未命名套接字和服务器套接字之间建立连接的方法来连接到服务器。它们通过connect调用来完成这一工作。

```
#include<sys/socket.h>
int connect(int socket,const struct sockaddr *address,size_t address_len);
```

​		参数socket指定的套接字将连接到参数address指定的服务器套接字，address指向的结构的长度由参数address_len指定。参数socket指定的套接字必须是通过socket调用获得的一个有效的文件描述符。

​		成功时，connect调用返回0，失败时返回-1.可能的错误代码：

| errno值      | 说明                                 |
| ------------ | ------------------------------------ |
| EBADF        | 传递给socket参数的文件描述符无效     |
| EALREADY     | 该套接字上已经有一个正在进行中的连接 |
| ETIMEDOUT    | 连接超时                             |
| ECONNREFUSED | 连接请求被服务器拒绝                 |

## 关闭套接字

​		调用close函数来终止服务器和客户上的套接字连接，就如同对底层文件描述符进行关闭一样。应该在连接的两端都关闭套接字。对于服务器来说，应该在read调用返回0时关闭套接字，但如果套接字是一个面向连接类型的，并且设置了SOCK_LINGER选项，close调用会在该套接字还有未传输数据时阻塞。

# 主机字节序和网络字节序

```
#include <arpa/inet.h>

uint32_t htonl(uint32_t hostlong);
uint16_t htons(uint16_t hostshort);
uint32_t ntohl(uint32_t netlong);
uint16_t ntohs(uint16_t netshort);
```

​		这些函数将16位和32位整数在主机字节序和标准的网络字节序之间进行转换。函数名是与之对应的转换操作的简写行式。例如“host to network，long”（htonl，长整数从主机字节序到网络字节序的转换），”host to network ，short “（htons，短整数从主机字节序到网络字节序的转换）。如果计算机本身的主机字节序与网络字节序相同，这些函数的内容实际上就是空操作。

# IP地址转换函数

```
#include <arpa/inet.h>

int inet_pton(int af, const char *src, void *dst);		//将点分十进制的ip地址转化为用于网络传输的数值格式, 返回值：若成功则为1，若输入不是有效的表达式则为0，若出错则为-1

const char *inet_ntop(int af, const void *src, char *dst, socklen_t size);  //将数值格式转化为点分十进制的ip地址格式,返回值：若成功则为指向结构的指针，若出错则为NULL
```

（1）这两个函数的af参数既可以是AF_INET（ipv4）也可以是AF_INET6（ipv6）。如果，以不被支持的地址族作为af参数，这两个函数都返回一个错误，并将errno置为EAFNOSUPPORT.

（2）第一个函数尝试转换由src指针所指向的字符串，并通过dst指针存放二进制结果，若成功则返回值为1，否则如果所指定的af而言输入字符串不是有效的表达式格式，那么返回值为0.

（3）inet_ntop进行相反的转换，从数值格式（src）转换到表达式（dst)。inet_ntop函数的dst参数不可以是一个空指针。调用者必须为目标存储单元分配内存并指定其大小，调用成功时，这个指针就是该函数的返回值。len参数是目标存储单元的大小，以免该函数溢出其调用者的缓冲区。如果len太小，不足以容纳表达式结果，那么返回一个空指针，并置为errno为ENOSPC。

# 发送和接收数据

```
#include <sys/socket.h>

ssize_t recv(int sockfd, void *buff, size_t nbytes, int flags);
	sockfd: 接收端套接字描述符
	buff：   用来存放recv函数接收到的数据的缓冲区
	nbytes: 指明buff的长度
	flags:   一般置为0
	
ssize_t send(int sockfd, const void *buff, size_t nbytes, int flags);
	sockfd：指定发送端套接字描述符。
	buff：  存放要发送数据的缓冲区
	nbytes: 实际要改善的数据的字节数
	flags：  一般设置为0
```

flag参数值为0或：

| flags         | 说明               | recv | send |
| ------------- | ------------------ | ---- | ---- |
| MSG_DONTROUTE | 绕过路由表查找     |      | √    |
| MSG_DONTWAIT  | 仅本操作非阻塞     | √    | √    |
| MSG_OOB       | 发送或接收带外数据 | √    | √    |
| MSG_PEEK      | 窥看外来消息       | √    |      |
| MSG_WAITALL   | 等待所有数据       | √    |      |

```
int recvfrom(int s,void *buf,int len,unsigned int flags ,struct sockaddr *from ,int *fromlen);
	recv()用来接收远程主机经指定的socket 传来的数据，并把数据存到由参数buf 指向的内存空间，参数len 为可接收数据的最大长度。参数flags 一般设0，其他数值定义请参考recv()。参数from用来指定欲接收的网络地址，结构sockaddr 请参考bind()。参数fromlen为sockaddr的结构长度。
	
int sendto ( int s , const void * msg, int len, unsigned int flags, const 
struct sockaddr * to , int tolen ) 
	sendto() 用来将数据由指定的socket传给对方主机。参数s为已建好连线的socket,如果利用UDP协议则不需经过连线操作。参数msg指向欲连线的数据内容，参数flags 一般设0，详细描述请参考send()。参数to用来指定欲传送的网络地址，结构sockaddr请参考bind()。参数tolen为sockaddr的结果长度
```

# 网络信息

​		主机数据库函数

```
#include<netdb.h>

struct hostent *gethostbyaddr(const void *addr,size_t len,int type);
struct hostent *gethostbyname(const char *name);

这些函数返回的结构中至少会包含以下几个成员
struct hostent {
	char *h_name;		/* name of the host */
	char **h_aliases	/* list of aliases(nicknames) */
	int h_addrtype;		/* address type */
	int h_length;		/* length in bytes of the address */
	char **h_addr_list	/* list of address(network order) */
};
如果没有与我们查询的主机或地址相关的数据项，这些信息函数将返回一个空指针。
```

​		服务及其关联端口号有关的信息

```
#include<netdb.h>

struct srvent *getservbyname(const char *name,const char *proto);
struct servent *getservbyport(int port,const char *proto);
	proto参数指定用于连接该服务的协议，它的两个取值是tcp和udp，前者用于SOCK_STREAM类型的TCP连接，后者用于SOCK_DGRAM类型的UDP数据包。

struct servent {
	char *s_name;		/* name of the service */
	char **s_aliases;	/* list of aliases(alternaative names) */
	int s_port;			/* The IP port number */
	char *s_proto;		/* The service type,usually "tcp" or "udp" */
};
```

​		如果像获得某台计算机的主机数据库类型，可以调用gethostbyname函数并且将结果打印出来。要把返回的地址列表转换为正确的地址类型，并用函数inet_ntoa将它们从网络字节序转换为可打印的字符串。

```
#include<arpa/inet.h>
char *inet_ntoa(struct in_addr in);
```

​		这个函数的作用是，将一个因特网主机地址转换为一个点分四元组格式的字符串。它在失败时返回-1

```
#include<unistd.h>
int gethostname(char *name,int namelength);
```

​		这个函数的作用是，将当前主机的名字写入name指向的字符串中。主机名将以null结尾。参数namelength指定了字符串name的长度，如果返回的主机名太长，它就会被截断。

# 多客户

## fork方式

​		服务器程序在接受来自客户的一个新连接时，会创建出一个新的套接字，而**原先的监听套接字将被保留以继续监听以后的连接**。如果服务武器不能立刻接受后来的连接，它们将被放到队列中以等待处理。原先的套接字仍然可用并且套接字的行为就像文件描述符，服务器调用fork为自己创建第二份副本，打开的套接字就将被新的子进程所继承。新的子进程可以和连接的客户进行通信，而主服务器进程可以继续接受以后的客户连接。

```
#include<sys/types.h>
#include<sys/docket.h>
#include<stdio.h>
#include<netinet/in.h>
#include<signal.h>
#include<unistd.h>
#include<stdlib.h>

int main(){
	int server_sockfd,client_sockfd;
	int server_len,client_len;
	struct sockaddr_in server_address;
	struct sockaddr_in client_address;
	
	server_sockfd = sockt(AF_INET,SOCK_STREAM,0);
	
	server_address.sin_family = AF_INET;
	server_address.sin_addr.s_addr = htonl(INADDR_ANY);
	server_address.sin_port = htons(9734);
	server_len = sizeof(server_address);
	bind(server_sockfd,(struct sockaddr *)&server_address,server_len);
	
	//创建一个连接队列，忽略子进程的退出细节，等待客户的到来：
	listen(server_sockfd,5);
	
	signal(SIGCHLD,SIG_LGN);
	
	while(1){
		char ch;
		
		printf("server waiting\n");
		//接受连接：
		client_len = sizeof(client_address);
		client_socketfd = accept(server_sockfd,(struct sockaddr *)&client_address,&client_len);
		//通过fork调用为这个客户创建一个子进程，然后测试你是在父进程中还是子进程中：
		if(fork() == 0){
			//如果你是在子进程中，就可以对client_sockfd上的客户执行读写操作。
			read(client_sockfd,&ch,1);
			ch++;
			write(client_sockfd,&ch,1);
			close(client_sockfd);
			exit(0);
		}else { //否则你一定实在父进程中，你只需关闭这个客户：
			close(client_sockfd);
		}
	}
}
```

## select系统调用

​		select系统调用允许程序同时在多个底层文件描述符上等待输入的到达（或输出的完成）。服务器可以通过同时**在多个打开的套接字上等待请求到来**的方法处理多个客户。

​		select函数对数据结构fd_set进行操作，它是由打开的文件描述符构成的集合。有一组定义好的宏可以用来控制这些集合：

```
#include<sys/type.h>
#include<sys/time.h>

void FD_ZERO(fd_fst *fdset);
void FD_CLR(int fd,fd_set *fdset);
void FD_SET(int fd,fd_set *fdset);
void FD_ISSET(int fd,fd_set *fdset);

FD_ZERO用于将fd_set初始化为空集合。
FD_SET用于在集合中设置由参数fd传递的文件描述符。
FD_CLR用于在集合中清楚由参数fd产地的文件描述符。
FD_ISSET判断参数fd指向的文件描述符是否是 由参数fdset指向的fd_set集合 中的一个元素。是则返回一个非零值。
```

超时值由一个timeval结构给出，用来防止无期限的阻塞。

```
#include<sys/time.h>
struct timeval{
	time_t tv_sec;		/* 秒，time_t在头文件sys/types.h中被定义为一个整数类型。 */
	long tv_usec;		/* 微秒 */
}
```

select系统调用原型：

```
#include<sys/types.h>
#include<sys/time.h>

int select(int nfds,fd_set *readfds,fd_set *writefds,fd_set *errorfds,struct timeval *timeout);
		参数nfds指定需要测试的文件描述符数目，测试的描述符范围从0到nfds-1。3个描述符集合都可以被设为空指针，这表示不执行响应的测试。fd_set结构中可以容纳的文件描述符的最大数目由常量FD_SETSIZE指定。
	nfds: 		监控的文件描述符集里最大文件描述符加1，因为此参数会告诉内核检测前多少个文件描述符的状态
	readfds：	监控有读数据到达文件描述符集合，传入传出参数
	writefds：	监控写数据到达文件描述符集合，传入传出参数
	exceptfds：	监控异常发生达文件描述符集合,如带外数据到达异常，传入传出参数
	timeout：	定时阻塞监控时间，3种情况
				1.NULL，永远等下去
				2.设置timeval，等待固定时间
				3.设置timeval里时间均为0，检查描述字后立即返回，轮询

```

​		select调用用于测试文件描述符集合中，是否有一个描述符**已处于可读状态或可写状态或错误状态**，它将阻塞以等待某个文件描述符进入上述这些状态。

​		select函数会在发生一下情况时返回：readfds集合中有描述符可读、writefds集合中有描述符可写或errorfds集合中有描述符遇到错误条件。如果这3种情况都没有发生，select将在timeout指定的超时时间经过后返回。如果timeout参数是一个空指针并且套接字上也没有任何活动，这个调用将一直阻塞下去。

​		select调用返回状态发生变化的描述符总数。

## poll

```
#include <poll.h>
int poll(struct pollfd *fds, nfds_t nfds, int timeout);

	struct pollfd {
		int fd; /* 文件描述符 */
		short events; /* 监控的事件 */
		short revents; /* 监控事件中满足条件返回的事件 */
	};
	POLLIN			普通或带外优先数据可读,即POLLRDNORM | POLLRDBAND
	POLLRDNORM		数据可读
	POLLRDBAND		优先级带数据可读
	POLLPRI 		高优先级可读数据
	POLLOUT		普通或带外数据可写
	POLLWRNORM		数据可写
	POLLWRBAND		优先级带数据可写
	POLLERR 		发生错误
	POLLHUP 		发生挂起
	POLLNVAL 		描述字不是一个打开的文件

	nfds 			监控数组中有多少文件描述符需要被监控

	timeout 		毫秒级等待
		-1：阻塞等，#define INFTIM -1 				Linux中没有定义此宏
		0：立即返回，不阻塞进程
		>0：等待指定毫秒数，如当前系统时间精度不够毫秒，向上取值

```

## epoll

​		epoll是Linux下多路复用IO接口select/poll的增强版本，它能显著提高程序在大量并发连接中只有少量活跃的情况下的系统CPU利用率，因为它会复用文件描述符集合来传递结果而不用迫使开发者每次等待事件之前都必须重新准备要被侦听的文件描述符集合，另一点原因就是获取事件的时候，它无须遍历整个被侦听的描述符集，只要遍历那些被内核IO事件异步唤醒而加入Ready队列的描述符集合就行了。

​		epoll除了提供select/poll那种IO事件的水平触发（Level Triggered）外，还提供了边沿触发（Edge Triggered），这就使得用户空间程序有可能缓存IO状态，减少epoll_wait/epoll_pwait的调用，提高应用程序效率。

```
1.	创建一个epoll句柄，参数size用来告诉内核监听的文件描述符的个数，跟内存大小有关。
	#include <sys/epoll.h>
	int epoll_create(int size)		size：监听数目
2.	控制某个epoll监控的文件描述符上的事件：注册、修改、删除。
	#include <sys/epoll.h>
	int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)
		epfd：	为epoll_creat的句柄
		op：		表示动作，用3个宏来表示：
			EPOLL_CTL_ADD (注册新的fd到epfd)，
			EPOLL_CTL_MOD (修改已经注册的fd的监听事件)，
			EPOLL_CTL_DEL (从epfd删除一个fd)；
		event：	告诉内核需要监听的事件

		struct epoll_event {
			__uint32_t events; /* Epoll events */
			epoll_data_t data; /* User data variable */
		};
		typedef union epoll_data {
			void *ptr;
			int fd;
			uint32_t u32;
			uint64_t u64;
		} epoll_data_t;

		EPOLLIN ：	表示对应的文件描述符可以读（包括对端SOCKET正常关闭）
		EPOLLOUT：	表示对应的文件描述符可以写
		EPOLLPRI：	表示对应的文件描述符有紧急的数据可读（这里应该表示有带外数据到来）
		EPOLLERR：	表示对应的文件描述符发生错误
		EPOLLHUP：	表示对应的文件描述符被挂断；
		EPOLLET： 	将EPOLL设为边缘触发(Edge Triggered)模式，这是相对于水平触发(Level Triggered)而言的
		EPOLLONESHOT：只监听一次事件，当监听完这次事件之后，如果还需要继续监听这个socket的话，需要再次把这个socket加入到EPOLL队列里
3.	等待所监控文件描述符上有事件的产生，类似于select()调用。
	#include <sys/epoll.h>
	int epoll_wait(int epfd, struct epoll_event *events, int maxevents, int timeout)
		events：		用来存内核得到事件的集合，
		maxevents：	告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，
		timeout：	是超时时间
			-1：	阻塞
			0：	立即返回，非阻塞
			>0：	指定毫秒
		返回值：	成功返回有多少文件描述符就绪，时间到时返回0，出错返回-1

```

