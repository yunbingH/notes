

## qumu-img create

用于创建 磁盘文件 使用的命令，
`qemu-img create [-q] [-f fmt] [-o options] filename [size]`
其中`-f`用于指定磁盘的格式，常用的格式有`raw, cow, qcow, qcow2`

> [raw,cow,qcow,qcow2镜像的比较]https://www.cnblogs.com/billowsand/p/3509545.html

如果要查看create子命令还有哪些选项可用，可以使用`-o ?`

```
[root@node1 ~]# qemu-img create -f qcow2 -o ? /kvm/images/test.img
Supported options:
size             Virtual disk size
compat           Compatibility level (0.10 or 1.1)
backing_file     File name of a base image
backing_fmt      Image format of the base image
encryption       Encrypt the image
cluster_size     qcow2 cluster size
preallocation    Preallocation mode (allowed values: off, metadata, falloc, full)
lazy_refcounts   Postpone refcount updates
```

以上实例表示要创建 /kvm/images/test.img 磁盘文件格式为qcow2它有哪些选项；在linux系统上文件的后缀只是起给人看的，系统不以后缀来确定它的格式；
size：用于指定创建磁盘文件的大小。
compat：用于指定兼容性。
backing_file：用于指定备份文件名称。
backing_fmt：用于指定备份文件的格式。
encryption：用于指定是否加密，true表示加密，false表示不加密，默认false。
cluster_size：用于指定磁盘的簇大小。
preallocation：用于指定磁盘预分配策略，off表示不预分配，metadata表示只预分配元数据大小，falloc表示随文件的增加而分配，full表示立即分配所有磁盘空间。默认是off。

例：创建一个2G的磁盘，分别用不同的预分配策略机制

```
[root@node1 ~]# qemu-img create -f qcow2 -o preallocation=off,size=2G /kvm/images/a1.img
Formatting '/kvm/images/a1.img', fmt=qcow2 size=2147483648 encryption=off cluster_size=65536 preallocation='off' lazy_refcounts=off
[root@node1 ~]# ll -h /kvm/images/a1.img
-rw-r--r-- 1 root root 193K 8月  18 22:17 /kvm/images/a1.img
[root@node1 ~]# qemu-img info /kvm/images/a1.img
image: /kvm/images/a1.img
file format: qcow2
virtual size: 2.0G (2147483648 bytes)
disk size: 196K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 ~]# qemu-img  create -f qcow2 -o preallocation=metadata,size=2G /kvm/images/a2.img
Formatting '/kvm/images/a2.img', fmt=qcow2 size=2147483648 encryption=off cluster_size=65536 preallocation='metadata' lazy_refcounts=off
[root@node1 ~]# ll -h /kvm/images/a2.img
-rw-r--r-- 1 root root 2.1G 8月  18 22:18 /kvm/images/a2.img
[root@node1 ~]# qemu-img info /kvm/images/a2.img
image: /kvm/images/a2.img
file format: qcow2
virtual size: 2.0G (2147483648 bytes)
disk size: 708K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 ~]# qemu-img  create -f qcow2 -o preallocation=falloc,size=2G /kvm/images/a3.img              
Formatting '/kvm/images/a3.img', fmt=qcow2 size=2147483648 encryption=off cluster_size=65536 preallocation='falloc' lazy_refcounts=off
[root@node1 ~]# ll -h /kvm/images/a3.img
-rw-r--r-- 1 root root 2.1G 8月  18 22:19 /kvm/images/a3.img
[root@node1 ~]# qemu-img info /kvm/images/a3.img
image: /kvm/images/a3.img
file format: qcow2
virtual size: 2.0G (2147483648 bytes)
disk size: 2.0G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 ~]# qemu-img  create -f qcow2 -o preallocation=full,size=2G /kvm/images/a4.img             
Formatting '/kvm/images/a4.img', fmt=qcow2 size=2147483648 encryption=off cluster_size=65536 preallocation='full' lazy_refcounts=off
[root@node1 ~]# ll -h /kvm/images/a4.img                                                 
-rw-r--r-- 1 root root 2.1G 8月  18 22:21 /kvm/images/a4.img
[root@node1 ~]# qemu-img info /kvm/images/a4.img                                         
image: /kvm/images/a4.img
file format: qcow2
virtual size: 2.0G (2147483648 bytes)
disk size: 2.0G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
```

