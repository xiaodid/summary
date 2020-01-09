# 安装

## 硬件要求

内存：16G以上；网卡：万兆网卡，推荐NIC Bonding。

## segment个数

总共都Segment个数=机器数 * 每台机器都segment实例数

通常，segment数量，由CPU的核数决定，如：在一个3个4核CPU的机器上，可以安装3，6，12个primary（或primary - mirror组）实例

## 安装依赖软件

``` shell
sudo yum install -y apr apr-util bash bzip2 curl krb5 libcurl libevent libxml2 libyaml zlib openldap openssh openssl openssl-libs perl readline rsync R sed tar zip
```

## 安装java

下载 Open JDK8
[OpenJDK](https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u232-b09/OpenJDK8U-jdk_x64_linux_hotspot_8u232b09.tar.gz)

## 编辑```/etc/hosts```文件，确保每台机器，每个nic都出现在这里

## 检查Selinux是否关闭

``` shell
sudo sestatus

# 如果禁用，则有如下输出
SELinuxstatus: disabled
```

## 禁用防火墙

还需要禁用iptable(RHEL 6/CentOS 6), firewalld(RHEL 7/CentOS 7) 或者ufw(Ubuntu)

禁用firewalld：

``` shell
# systemctl status firewalld
# systemctl stop firewalld.service
# systemctl disable firewalld.service
```

## 编辑```/etc/sysctl.conf```文件

### 修改```kernel.shmall```和```kernel.shmmax```

``` shell
# 修改以下
kernel.shmmax = ( _PHYS_PAGES / 2) * PAGE_SIZE
kernel.shmall = ( _PHYS_PAGES / 2)
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
vm.dirty_ratio = 0
# 这个值可以通过这个命令计算
# awk 'BEGIN {OFMT = "%.0f";} /MemTotal/ {print "vm.min_free_kbytes =", $2 * .03;}' /proc/meminfo
vm.min_free_kbytes = 3948375

net.ipv4.ip_local_port_range = 10000  65535
# 添加以下
kernel.shmmni = 4096
vm.overcommit_memory = 2
vm.overcommit_ratio = 95
kernel.sem = 500 2048000 200 40960
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.arp_filter = 1
vm.zone_reclaim_mode = 0
vm.dirty_background_ratio = 0
vm.dirty_ratio = 0
vm.dirty_background_bytes = 1610612736 # 1.5GB
vm.dirty_bytes = 4294967296 # 4GB
vm.dirty_expire_centisecs = 500
vm.dirty_writeback_centisecs = 100
```

这个是基于当前系统修改过的：

``` shell
kernel.sysrq = 1
kernel.core_uses_pid = 1
fs.aio-max-nr = 1048576
kernel.msgmnb = 65536
kernel.msgmax = 65536
#kernel.shmmax = 68719476736
#kernel.shmall = 4294967296
kernel.shmmax = 67385606144
kernel.shmall = 16451564
# added by dxd
kernel.shmmni = 4096
vm.overcommit_memory = 2
vm.overcommit_ratio = 95
kernel.sem = 500 2048000 200 40960
kernel.msgmni = 2048
net.ipv4.tcp_syncookies = 1
net.ipv4.conf.all.arp_filter = 1
vm.zone_reclaim_mode = 0
vm.dirty_background_ratio = 0
vm.dirty_ratio = 0
vm.dirty_background_bytes = 1610612736 # 1.5GB
vm.dirty_bytes = 4294967296 # 4GB
vm.dirty_expire_centisecs = 500
vm.dirty_writeback_centisecs = 100
# end
net.ipv4.ip_forward = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.default.rp_filter = 2
net.ipv4.conf.all.rp_filter = 2
net.ipv4.conf.all.arp_announce = 2
# changed by dxd
net.ipv4.ip_local_port_range = 10000  65535
# net.ipv4.ip_local_port_range = 1024  65535
net.ipv4.tcp_synack_retries = 2
net.ipv4.conf.all.secure_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.rp_filter = 1
net.core.somaxconn = 65535
net.core.rmem_default = 262144
net.core.wmem_default = 262144
# commentted by dxd
# net.core.rmem_max = 16777216
# net.core.wmem_max = 16777216
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152
net.ipv4.tcp_rmem = 8192 87380 16777216
net.ipv4.tcp_wmem = 8192 65536 16777216
net.ipv4.tcp_max_syn_backlog = 16384
net.core.netdev_max_backlog = 10000
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_orphan_retries = 0
net.ipv4.tcp_max_orphans = 131072
#fs.file-max = 65536  #os can config
# commentted by dxd
# vm.min_free_kbytes = 1048576
vm.min_free_kbytes = 3948375
vm.swappiness = 1
# commentted by dxd
# vm.dirty_ratio = 10
vm.vfs_cache_pressure=150
vm.drop_caches = 1
kernel.panic = 60
```

## 修改```/etc/security/limits.conf```文件

``` shell
* soft nofile 524288
* hard nofile 524288
* soft nproc 131072
* hard nproc 131072
```

## 修改```/etc/security/limits.d/20-nproc.conf```

``` shell
*          soft    nproc     131072
root       soft    nproc     unlimited
```

## 修改```/etc/fstab```文件

给xfs文件系统增加以下选项：

``` shell
rw,nodev,noatime,nobarrier,inode64
```

如：

``` shell
UUID="48b8f50b-0a0c-4cd1-9820-7a475fb419c9"  /data  xfs rw,nodev,noatime,nobarrier,inode64 0 0
```

## 设置每个磁盘的```read-ahead(blockdev)```

可以使用```lsblk```命令查看磁盘挂载情况:

``` shell
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda      8:0    0 837.8G  0 disk
├─sda1   8:1    0   500M  0 part /boot
├─sda2   8:2    0 829.3G  0 part /
└─sda3   8:3    0     8G  0 part [SWAP]
sdb      8:16   0   7.3T  0 disk
└─sdb1   8:17   0   7.3T  0 part /data
```

编辑```/etc/rc.d/rc.local```文件，增加：

``` shell
/sbin/blockdev --setra 16384 /dev/sda
/sbin/blockdev --setra 16384 /dev/sdb
```

修改rc.local的权限

``` shell
chmod +x /etc/rc.d/rc.local
```

重启，使用```/sbin/blockdev --getra /dev/sda```查看

## 修改IO Scheduler

修改成deadline

## 禁用Transparent Huge Pages(THP)

禁用THP，因为它会降低Greenplum数据库的性能。
在CentOS 7上，运行以下代码：

``` shell
grubby --update-kernel=ALL --args="transparent_hugepage=never"
```

## 禁用IPC object removal

编辑```/etc/systemd/logind.conf```，增加：

``` shell
RemoveIPC=no
```

## SSH连接阈值

编辑```/etc/ssh/sshd_config```或者```/etc/sshd_config```：

``` shell
MaxStartups 200

# 如果打算使用'start:rate"full'语法，
MaxStartups 10:30:200
```

重启SSH服务：

``` shell
service sshd restart
```

## 同步系统时钟

### 登陆master主机，编辑```/etc/ntp.conf```文件，指定时间服务器

``` shell
server 10.0.11.26
server 10.0.11.27
server 10.0.11.28
```

### 登陆每台segment机器，编辑```/etc/ntp.conf```文件

``` shell
server mdw.veredholdings.cn prefer
server 10.0.11.26
server 10.0.11.27
server 10.0.11.28
```

重启ntpd服务：

``` shell
service ntpd restart
```

## 检查字符集

``` shell
echo $LANG
# en_US.UTF-8
```

否则，修改配置 /etc/sysconfig/language 增加 RC_LANG=en_US.UTF-8

## 创建greenplum管理员用户

通常，这个管理员用户叫gpadmin。

不要使用root用户运行greenplum数据库

``` shell
# 需要指定组id和用户id
# 这次安装是接着最后一个用户id：1002开始，所以指定1003
groupadd gpadmin -g 1003
useradd gpadmin -r -m -g gpadmin -u 1003
passwd gpadmin
```

## 生产gpadmin用户的密钥对

``` shell
ssh-keygen -t rsa -b 4096
```

## 安装greenplum

### 在每台机器上安装greenplum

``` shell
sudo yum install ./greenplum-db-<version>-<platform>.rpm
```

### 修改安装目录权限

``` shell
sudo chown -R gpadmin:gpadmin /usr/local/greenplum*
```

## 启用无密登陆

### 使用gpadmin用户登陆master节点

### 执行greenplum的path文件

``` shell
source /usr/local/greenplum-db-6.2.1/greenplum_path.sh
```

把上面的source命令加入到```.bashrc```文件中

### 使用```ssh-copy-id```命令添加gpadmin用户的公钥

``` shell
ssh-copy-id -p 6666 sdw1.veredholdings.cn
ssh-copy-id -p 6666 sdw2.veredholdings.cn
```

### 创建hostfile_exkeys文件

在gpadmin的用户目录下，创建```hostfile_exkeys```文件，这个文件包含所以greenplum集群中的主机名。
如果机器的网卡没有bond，那么需要把每个网口的名字都列出来
注意，这个文件不能有额外的空行和空格

``` shell
sdw1.veredholdings.cn
sdw2.veredholdings.cn
```

### 运行```gpssh-exkeys```

``` shell
gpssh-exkeys -f hostfile_exkeys
```

## 创建数据存储空间

在master和standby master上执行：

``` shell
mkdir -p /data/master
chown gpadmin:gpadmin /data/master
```

在segment上执行：

``` shell
sudo mkdir -p /data/data1/primary
sudo mkdir -p /data/data1/mirror
sudo mkdir -p /data/data2/primary
sudo mkdir -p /data/data2/mirror
sudo chown -R gpadmin:gpadmin /data/data*
```

## 验证硬件和网络性能

### 验证网络性能

``` shell
gpcheckperf -f hostfile_exkeys -r N -d /tmp > result.out
```

### 验证磁盘性能

``` shell
gpcheckperf -f hostfile_exkeys -r ds -D \
  -d /data/data1/primary -d  /data/data2/primary \
  -d /data/data1/mirror -d  /data/data2/mirror
```

## 初始化数据库

1, 在用户目录创建一个```hostfile_gpinitsystem```文件。这个文件中添加所有segment节点的主机名。一个名字一行，不能有额外的空行和空格

2, 创建一个```gpinitsystem_config```文件的副本

``` shell
cp $GPHOME/docs/cli_help/gpconfigs/gpinitsystem_config \
     /home/gpadmin/gpconfigs/gpinitsystem_config
```

3, 修改这个文件：

``` shell
# FILE NAME: gpinitsystem_config

# Configuration file needed by the gpinitsystem

################################################
#### REQUIRED PARAMETERS
################################################

#### Name of this Greenplum system enclosed in quotes.
ARRAY_NAME="Vered Intelligence GP"

#### Naming convention for utility-generated data directories.
SEG_PREFIX=gpseg

#### Base number by which primary segment port numbers
#### are calculated.
PORT_BASE=6000

#### File system location(s) where primary segment data directories
#### will be created. The number of locations in the list dictate
#### the number of primary segments that will get created per
#### physical host (if multiple addresses for a host are listed in
#### the hostfile, the number of segments will be spread evenly across
#### the specified interface addresses).
# 这个指定了每个segment主机上有几个segment实例。
# 可以之多多个相同的目录
declare -a DATA_DIRECTORY=(/data/data1/primary /data/data2/primary)

#### OS-configured hostname or IP address of the master host.
MASTER_HOSTNAME=mdw.veredholdings.cn

#### File system location where the master data directory
#### will be created.
MASTER_DIRECTORY=/data/master

#### Port number for the master instance.
MASTER_PORT=5432

#### Shell utility used to connect to remote hosts.
TRUSTED_SHELL=ssh

#### Maximum log file segments between automatic WAL checkpoints.
CHECK_POINT_SEGMENTS=8

#### Default server-side character set encoding.
ENCODING=UNICODE

################################################
#### OPTIONAL MIRROR PARAMETERS
################################################

#### Base number by which mirror segment port numbers
#### are calculated.
MIRROR_PORT_BASE=7000

#### File system location(s) where mirror segment data directories
#### will be created. The number of mirror locations must equal the
#### number of primary locations as specified in the
#### DATA_DIRECTORY parameter.
declare -a MIRROR_DATA_DIRECTORY=(/data/data1/mirror /data/data2/mirror)


################################################
#### OTHER OPTIONAL PARAMETERS
################################################

#### Create a database of this name after initialization.
#DATABASE_NAME=name_of_database

#### Specify the location of the host address file here instead of
#### with the the -h option of gpinitsystem.
MACHINE_LIST_FILE=/home/gpadmin/gpconfigs/hostfile_gpinitsystem
```

### 运行初始化工具

``` shell
gpinitsystem -c gpconfigs/gpinitsystem_config -h gpconfigs/hostfile_gpinitsystem -s standby_master_hostname -S
```

工具将询问是否继续：

``` shell
=> Continue with Greenplum creation? Yy/Nn
```

当成功结束：

``` shell
=> Greenplum Database instance successfully created.
```
