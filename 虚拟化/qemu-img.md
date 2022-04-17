

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

> 提示：从上面的示例可以看到除了off指定创建的磁盘，我们看到的大小是一个很小的大小，其他模式在文件系统上表现形式都是我们指定大小的空间；从qemu-img info 命令来看，off和metadata disk size是很小的空间，虚拟空间是我们指定的大小；后者falloc和full disk大小和virtual size大小都是我们指定的大小；在文件系统上看到的磁盘文件之所以要大于我们指定的空间，是因为在文件系统上它作为一个文件形式存在，它也有元素数据信息的



## qemu-img info

用于查看指定磁盘文件的详细信息；用法格式：qemu-img info [-f fmt] [--output=ofmt] [--backing-chain] filename

```
[root@node1 ~]# qemu-img info /kvm/images/centos7.qcow2
image: /kvm/images/centos7.qcow2
file format: qcow2
virtual size: 10G (10737418240 bytes)
disk size: 196K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 ~]# qemu-img info /kvm/images/win7.qcow2
image: /kvm/images/win7.qcow2
file format: qcow2
virtual size: 50G (53687091200 bytes)
disk size: 8.5G
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 ~]#
```



## qemu-img check

对指定磁盘文件做检查;用法格式：qemu-img check [-q] [-f fmt] [--output=ofmt] [-r [leaks | all]] [-T src_cache] filename

```
[root@node1 ~]# qemu-img check /kvm/images/win7.qcow2
No errors were found on the image.
138808/819200 = 16.94% allocated, 27.22% fragmented, 0.00% compressed clusters
Image end offset: 9098887168
[root@node1 ~]# qemu-img check /kvm/images/centos7.qcow2
No errors were found on the image.
Image end offset: 262144
[root@node1 ~]#
```



## qemu-img snapshot

对指定磁盘文件做快照相关操作；语法格式：qemu-img snapshot [-q] [-l | -a snapshot | -c snapshot | -d snapshot] filename

-c：表示创建快照

```
[root@node1 ~]# qemu-img snapshot -c snapshot_centos7_1 /kvm/images/centos7.qcow2
```

-l：查看指定磁盘文件的快照列表

```
[root@node1 ~]# qemu-img snapshot -l /kvm/images/centos7.qcow2                    
Snapshot list:
ID        TAG                 VM SIZE                DATE       VM CLOCK
1         snapshot_centos7_1        0 2017-03-29 01:16:08   00:00:00.000
[root@node1 ~]#
```

-a：应用快照

```
[root@node1 ~]# qemu-img snapshot -a snapshot_centos7_1 /kvm/images/centos7.qcow2
```

-d：删除快照

```
[root@node1 ~]# qemu-img snapshot -l /kvm/images/centos7.qcow2  
Snapshot list:
ID        TAG                 VM SIZE                DATE       VM CLOCK
1         snapshot_centos7_1        0 2017-03-29 01:16:08   00:00:00.000
[root@node1 ~]# qemu-img snapshot -d snapshot_centos7_1 /kvm/images/centos7.qcow2
[root@node1 ~]# qemu-img snapshot -l /kvm/images/centos7.qcow2 
```



## qemu-img convert

镜像格式转换，语法格式：qemu-img convert [-c] [-p] [-q] [-n] [-f fmt] [-t cache] [-T src_cache] [-O output_fmt] [-o options] [-s snapshot_name] [-S sparse_size] filename [filename2 [...]] output_filename

-c：表示压缩输出文件，但只有qcow2和qcow格式的镜像文件才支持压缩，而且这种压缩是只读的，如果压缩的扇区被重写，则会被重写为未压缩的数据。

-p：用于显示转换进度。

-o：用于指定输出文件的选项，比如是否加密，大小，等等。

示例1：不加输出格式直接转换qcow2格式的磁盘文件

```
[root@node1 ~]# qemu-img create -f qcow2 /kvm/images/test1.img 1G
Formatting '/kvm/images/test1.img', fmt=qcow2 size=1073741824 encryption=off cluster_size=65536 lazy_refcounts=off
[root@node1 ~]# qemu-img info /kvm/images/test1.img
image: /kvm/images/test1.img
file format: qcow2
virtual size: 1.0G (1073741824 bytes)
disk size: 196K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 ~]# qemu-img convert /kvm/images/test1.img test1-1.img
[root@node1 ~]# qemu-img info test1-1.img
image: test1-1.img
file format: raw
virtual size: 1.0G (1073741824 bytes)
disk size: 0
```

> 提示：qemu-img convert输入的文件格式它会自动识别，输出格式如果不指定它默认转换为raw格式

示例2：就vmdk格式的文件转换成qcow2格式的磁盘文件

```
[root@node1 ~]# cd /kvm/images/
[root@node1 images]# ls
centos7.qcow2  test1.img
[root@node1 images]# rz
rz waiting to receive.
 zmodem trl+C ȡ
 
  100%     512 KB  512 KB/s 00:00:01       0 Errors01-s001.vmdk...
 
[root@node1 images]# ls
CentOS-6.9-x86_64-000001-s001.vmdk  centos7.qcow2  test1.img
[root@node1 images]# qemu-img info CentOS-6.9-x86_64-000001-s001.vmdk
image: CentOS-6.9-x86_64-000001-s001.vmdk
file format: vmdk
virtual size: 4.0G (4261412864 bytes)
disk size: 512K
cluster_size: 65536
Format specific information:
    cid: 4294967295
    parent cid: 4294967295
    create type: monolithicSparse
    extents:
        [0]:
            virtual size: 4261412864
            filename: CentOS-6.9-x86_64-000001-s001.vmdk
            cluster size: 65536
            format:
[root@node1 images]# qemu-img convert ./CentOS-6.9-x86_64-000001-s001.vmdk -O qcow2 ./centos6.qcow2
[root@node1 images]# ll
总用量 1244560
-rw-r--r-- 1 root root     524288 4月  19 2020 CentOS-6.9-x86_64-000001-s001.vmdk
-rw-r--r-- 1 root root     197120 3月  29 01:33 centos6.qcow2
-rw-r--r-- 1 qemu qemu 1273561600 3月  29 01:32 centos7.qcow2
-rw-r--r-- 1 root root     197120 3月  29 01:24 test1.img
[root@node1 images]# qemu-img info ./centos6.qcow2
image: ./centos6.qcow2
file format: qcow2
virtual size: 4.0G (4261412864 bytes)
disk size: 196K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
```

> 提示：我们可以根据把不同格式的磁盘文件相互转换，从而实现把虚拟机从一个平台迁移到另一个平台；



## qemu-img resize

动态增删磁盘的大小；语法格式：qemu-img resize [-q] filename [+ | -]size

```
[root@node1 images]# ll -h
总用量 1.2G
-rw-r--r-- 1 root root 512K 4月  19 2020 CentOS-6.9-x86_64-000001-s001.vmdk
-rw-r--r-- 1 root root 193K 3月  29 01:33 centos6.qcow2
-rw-r--r-- 1 qemu qemu 1.2G 3月  29 02:03 centos7.qcow2
-rw-r--r-- 1 root root 193K 3月  29 01:24 test1.img
[root@node1 images]# qemu-img info test1.img
image: test1.img
file format: qcow2
virtual size: 1.0G (1073741824 bytes)
disk size: 196K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 images]# qemu-img resize test1.img +1G
Image resized.
[root@node1 images]# qemu-img info test1.img     
image: test1.img
file format: qcow2
virtual size: 2.0G (2147483648 bytes)
disk size: 260K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 images]# ll -h
总用量 1.2G
-rw-r--r-- 1 root root 512K 4月  19 2020 CentOS-6.9-x86_64-000001-s001.vmdk
-rw-r--r-- 1 root root 193K 3月  29 01:33 centos6.qcow2
-rw-r--r-- 1 qemu qemu 1.2G 3月  29 02:04 centos7.qcow2
-rw-r--r-- 1 root root 257K 3月  29 02:04 test1.img
```

> 提示：动态缩减空间必须保证磁盘空间大于里面存储的数据空间，在做删减操作有必要先备份一免磁盘损坏导致数据丢失

```
[root@node1 images]# qemu-img info test1.img
image: test1.img
file format: qcow2
virtual size: 2.0G (2147483648 bytes)
disk size: 200K
cluster_size: 65536
Format specific information:
    compat: 1.1
    lazy refcounts: false
[root@node1 images]# qemu-img resize test1.img -1G
qemu-img: qcow2 doesn't support shrinking images yet
qemu-img: This image does not support resize
[root@node1 images]#
```

> 提示：这里还需要注意一点，qcow2的格式磁盘不支持删减操作；
