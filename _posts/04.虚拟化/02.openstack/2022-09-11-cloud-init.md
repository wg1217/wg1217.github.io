# 1. Metadata Service

**instance 可以通过 nova-api-metadata 或者 config drive 这两种途径拿到 metadata（元数据）**

## 1.1 nova-api-metadata 

### 1.1.1 架构图



<img src="https://raw.githubusercontent.com/wg1217/picbed/main/202209132325591.png" alt="Metadata Service 的架构图" style="zoom: 40%;" />





### 1.1.2 涉及组件

#### 1.1.2.1 nova-api-metadata

nova-api-metadata 是 nova-api 的一个子服务，它是 metadata 的提供者，instance 可以通过 nova-api-metadata 的 REST API 来获取 metadata 信息。

**nova-api-metadata 运行在控制节点上，服务端口是 8775**

```bash
netstat -anpt |grep 8775 
```

 **查看该启动程序**

```bash
ps -elf |grep `netstat -anpt |grep 8775 |grep -v haproxy | awk '{print$NF}' | awk -F '/' '{print$1}'`
```

![image-20220514175903171](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325592.png)

ps:  **nova-api-metadata** 与 **nova-api** 服务是合并在一起的。在其他OpenStack 的发行版中可能有单独的 nova-api-metadata 进程存在。



**nova.conf 通过参数 enabled_apis 指定是否启用 nova-api-metadata**

[官方文档](https://docs.openstack.org/newton/config-reference/compute/api.html)

```bash
enabled_apis = osapi_compute, metadata

#以上为默认值
#ps: osapi_compute 是常规的 nova-api 服务，metadata 就是 nova-api-metadata 服务
```



#### 1.2.2.2 neutron-metadata-agent 

由于nova-api-metadata 在控制节点上，计算节点上的instance 是无法直接访问 metadata service （通过OpenStack 内部管理网络）

所以需要一个组件进行中转，即 **neutron-metadata-agent**

instance 先将 metadata 请求发给 neutron-metadata-agent（运行在网络节点上）

neutron-metadata-agent 再将请求转发到 nova-api-metadata



#### 1.2.2.3 neutron-ns-metadata-proxy

由于neutron-metadata-agent 是在网络节点上，计算节点上的instance 是无法直接访问  neutron-metadata-agent （通过OpenStack 内部管理网络）

但是网络节点上有另外两个组件，**dhcp agent** 和 **l3 agent**（他们都可以**创建neutron-ns-metadata-proxy**）

如果由 dhcp-agent 创建，neutron-ns-metadata-proxy 就运行在 dhcp-agent 所在的 namespace 中
如果由 l3-agent 创建，neutron-ns-metadata-proxy 就运行在 neutron router 所在的 namespace 中



**neutron-ns-metadata-proxy与instance** 位于同一 OpenStack network 中，计算节点上的**instance** 可以直接和neutron-ns-metadata-proxy通信。



**ps**: 

neutron-ns-metadata-proxy  中间的 ns 就是 namespace 的意思。

neutron-ns-metadata-proxy 与 neutron-metadata-agent 通过 unix domain socket 直接相连。



**总结**：

1. instance 通过 neutron network（Project 网络）将 metadata 请求发送到 neutron-ns-metadata-proxy。

2. neutron-ns-metadata-proxy 通过 unix domain socket 将请求发给 neutron-metadata-agent。

3. neutron-metadata-agent 通过内部管理网络将请求发送给 nova-api-metadata。



## 1.2 访问 Metadata的完整过程

**instance 可以通过 nova-api-metadata 或者 config drive 这两种途径拿到 metadata**

### 1.2.1 nova-api-metadata

[官方文档](https://docs.openstack.org/ocata/config-reference/networking/samples/dhcp_agent.ini.html)

```bash
cat /etc/kolla/neutron-dhcp-agent/dhcp_agent.ini

force_metadata = true  (缺省值为false)
#该值决定获取metada使用dhcp-agent还是l3-agent
#true代表使用dhcp-agent  

enable_isolated_metadata = true   (缺省值为false)
#将169.254.169.254的下一跳设置为dhcp地址
```



#### 1.2.1.1 l3-agent 

```bash
sed -i 's#force_metadata.*#force_metadata = false#g'  /etc/kolla/neutron-dhcp-agent/dhcp_agent.ini
sed -i 's#enable_isolated_metadata.*#enable_isolated_metadata = false#g'  /etc/kolla/neutron-dhcp-agent/dhcp_agent.ini
docker restart neutron_dhcp_agent
```

创建network vlan-1001，子网范围（172.16.11.0/24） 启动DHCP

通过 cirros 镜像部署一个 instance，命名为 vm1，选择网络 vlan-1001



启动过程中，查看 instance 的启动日志，我们看到两个信息：

1. instance 从 DHCP 拿到了 172.16.11.26。
2. instance 会去访问 http://169.254.169.254/2009-04-04/instance-id，尝试了 20 次都失败了。

![image-20220514202627486](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325593.png)



**169.254.169.254** 是 metadata service 的 IP，

这个地址来源于 AWS，当年亚马逊在设计公有云的时候，为了让 instance 能够访问 metadata，就将**169.254.169.254** 这个特殊的 IP 作为 metadata 服务器的地址，instance 启动时就会向 **169.254.169.254** 请求metadata。OpenStack 之后也沿用了这个设计。



instance 首先会将 metadata 请求发送给l3-agent 管理的 neutron-ns-metadata-proxy **169.254.169.254** 



查询neutron-ns-metadata-proxy 的进程

![image-20220514203158684](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325594.png)



当前network  **vlan-1001** 并没有挂在 neutron router 上，所以network **vlan-1001**的网关**172.16.11.1**不存在，instance添加去往**169.254.169.254** 的路由失败。

 

将network  **vlan-1001** 与router **test**相连，重启vm1

![image-20220514204536158](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325595.png)

![image-20220514203932660](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325596.png)



instance成功访问到 **169.254.169.254**，  metadata 服务地址是 **169.254.169.254**，端口为 80。

instance hostname 已经设置为 **vm1**，已经获取到 metadata

![image-20220514204734406](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325597.png)



在**vm1** 中访问一下 metadata

![image-20220514205135204](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325598.png)





从 **vm1** 的路由表得访问 **169.254.169.254** 的请求会走 **172.16.11.1**，**172.16.11.1** 实际上就是 **test** 在 **vlan-1020** 上的 interface IP。

**test** 接收到 **vm1** 的请求，会通过 iptables 规则转发到 9697 端口（neutron-ns-metadata-proxy 的监听端口）。



![image-20220514205153371](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325599.png)



![image-20220514205906377](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325600.png)



总结

1. instance 通过预定义的 169.254.169.254 请求 metadata。

2. 请求被转发到 neutron router。

3. router 将请求转发给 neutron-ns-metadata-proxy。

4. neutron-ns-metadata-proxy 将请求通过 unix domain socket 发给 neutron-metadata-agent，后者再通过管理网络发给 nova-api-metadata。



#### 1.2.1.2 dhcp-agent 

```bash
#确保参数为下值,让 dhcp-agent 来创建和管理 neutron-ns-metadata-proxy

sed -i 's#force_metadata.*#force_metadata = true#g'  /etc/kolla/neutron-dhcp-agent/dhcp_agent.ini
sed -i 's#enable_isolated_metadata.*#enable_isolated_metadata = true#g'  /etc/kolla/neutron-dhcp-agent/dhcp_agent.ini
docker restart neutron_dhcp_agent
```

```bash
 ps -aux |grep metadata-proxy
```

可以看到控制节点 neutron_dhcp_agent容器中 有ns-metadata-proxy 进程

![image-20220514211129747](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325601.png)

![image-20220514211307550](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325602.png)

此进程通过Network-ID 关联到 vlan-1001，用于接收vlan-1020网络上 instance 的 metadata 请求。每一个 network 开启dhcp都有一个与之对应的 ns-metadata-proxy。

重启 instance **vm1**，查看路由表

![image-20220514211402125](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325603.png)



请注意，现在访问 **169.254.169.254** 的路由已变为 **172.16.11.2**。
这里的 **172.16.11.2**是 dhcp-agent 在**vlan-1001**上的IP。这条路由是由 dhcp-agent 添加进去的。

![image-20220514211438574](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325604.png)

正是因为这条路由的存在，即便 l3-agent 与 dhcp-agent 同时提供 neutron-ns-metadata-proxy 服务，metadata 请求也只会发送给 dhcp-agent。



同时我们也看到，dhcp-agent 已经将 IP **169.254.169.254** 配置到了自己身上。也就是说：**vm1** 访问 metadata 的请求 http://169.254.169.254 实际上是发送到了 dhcp-agent 的 80 端口。

而监听 80 端口的正是 dhcp-agent 启动的 neutron-ns-metadata-proxy 进程

![image-20220514211710431](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325605.png)

后面的数据流向与 l3-agent 的场景一样

neutron-ns-metadata-proxy 将请求通过 unix domain socket 发给 neutron-metadata-agent，后者再通过管理网络发给 nova-api-metadata。



 l3-agent和dhcp-agent区别：

对于 **169.254.169.254**

1. l3-agent 用 iptables 规则来转发。

1. dhcp-agent 则是将此 IP 配置到自己的 interface 上。





### 1.2.2 Config Drive

[官方文档](https://docs.openstack.org/nova/ussuri/configuration/config.html#DEFAULT.config_drive_format)

如果 instance 无法通过 metadata service 获取 metadata（**无 DHCP 或者 nova-api-metadata** 服务），instance 还可以通过 config drive 获得 metadata。
config drive 是一个特殊的文件系统，OpenStack 会将 metadata 写到 config drive，并在 instance 启动时挂载给 instance。
如果 instance 安装了 cloud-init，config drive 会被自动 mount 并从中读取 metadata，进而完成后续的初始化工作。



**config drive 默认是 disable 的，所以首先得启用**

有两种方法启用 config drive：

1. 启动 instance 时指定 --config-drive true。
2. 在计算节点的 /etc/nova/nova.conf 中配置 `force_config_drive = true`，这样部署到此计算节点的 instance 都会使用 config drive。

config drive 支持两种格式，iso9660 和 vfat，默认是 iso9660，但这会导致 instance 无法在线迁移，必须设置成`config_drive_format=vfat` 才能在线迁移，这一点需要注意。**（？？？这点有疑问**）





**配置完成后，重启 nova-compute 服务**

**部署一个新的 cirros instance vm2，看看 vm1 与 vm2 的区别，通过`virsh edit` 可以看到 disk.config 已经挂载到 instance 上了**

![image-20220513172333373](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325606.png)

vm2 会多一个 **disk.config** 文件，这就是 config drive，存储在ceph中



**打开 vm2 的控制台，hostname 已经配置好，说明 metadata 拿到了**

**为了确保 metadata 不是从 nova-api-metadata 获取，提前关闭了 DHCP 服务，可以看到当前 vm2 是没有 IP 的。**

![image-20220513172555982](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325607.png)



**lsblk** 查看块设备，iso 设备 **sr0** 就是 config drive，挂载sr0，查看 config drive 的内容

**meta_data.json** 中存放了 ssh public key, hostname 等信息

![image-20220513173114936](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325608.png)





## 1.3 nova-api-metadata返回metadata 



### 1.3.1 l3-agent 

<img src="https://raw.githubusercontent.com/wg1217/picbed/main/202209132325609.png" alt="处理流程图" style="zoom: 40%;" />



instance --> neutron-ns-metadata-proxy --> neutron-metadata-agent --> nova-api-metadata



① neutron-ns-metadata-proxy 接收到请求，在转发给 neutron-metadata-agent 之前会将 instance ip 和 router id 添加到 http 请求的 head 中，这两个信息对于 l3-agent 来说很容易获得。

② neutron-metadata-agent 接收到请求后，会查询 instance 的 id，具体做法是：

1) 通过 router id 找到 router 连接的所有 subnet，然后筛选出 instance ip 所在的 subnet。
2) 在 subnet 中找到 instance ip 对应的 port。
3) 通过 port 找到对应的 instance 及其 id。

③ neutron-metadata-agent 将 instance id 添加到 http 请求的 head 中，然后转发给 nova-api-metadata，这样 nova-api-metadata 就能返回指定 instance 的 metadata 了。



### 1.3.2 dhcp-agent 

<img src="https://raw.githubusercontent.com/wg1217/picbed/main/202209132325610.png" alt="dhcp-agent" style="zoom: 40%;" />

① neutron-ns-metadata-proxy 在转发请求之前会将 instance ip 和 network id 添加到 http 请求的 head 中，这两个信息对于 dhcp-agent 来说很容易获得。

② neutron-metadata-agent 接收到请求后，会查询 instance 的 id，具体做法是：

1) 通过 network id 找到 network 所有的 subnet，然后筛选出 instance ip 所在的 subnet。
2) 在 subnet 中找到 instance ip 对应的 port。
3) 通过 port 找到对应的 instance 及其 id。

③ neutron-metadata-agent 将 instance id 添加到 http 请求的 head 中，然后转发给 nova-api-metadata，
这样 nova-api-metadata 就能返回指定 instance 的 metadata 了。



总结：

不管 instance 将请求发给 l3-agent 还是 dhcp-agent，nova-api-metadata 最终都能获知 instance 的 id，进而返回正确的 metadata。







# 2. Cloud-Init 

[module官方文档](https://cloudinit.readthedocs.io/en/latest/topics/modules.html)

## 2.1 工作原理

cloud-init 要完成的工作 就是如何**使用** instance 通过 nova-api-metadata 或者 config drive 这两种途径拿到 **metadata**。 



传入不同的 **metadata**，就可以完成一些定制化工作，如下

1. 设置时区
2. 设置 hostname
3. 添加 ssh keys到 .ssh/authorized_keys，实现免密登录
4. 设置用户密码
5. 配置网络
6. 安装软件包



cloud-init 会按 4 个阶段执行任务

1. local

2. init

3. config

4. final

   

 cloud-init安装时会将这 4 个阶段执行的任务以服务的形式注册到系统中

1. local 	   cloud-init-local.service
2. init 	     cloud-init.service
3. config     cloud-config.service
4. final        cloud-final.service

![image-20220514214724592](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325611.png)



### **2.1.1. local阶段** 

作为 cloud-init 执行的第一个阶段，cloud-init 的任务配置网卡，这是非常关键的一步，只有当网卡正确配置后，才能获取到 metadata。



- 如果没有 config drive

​     		将所有网卡配置成 dhcp 模式

- 如果有 config drive

​       	从 config drive 中获取配置信息，写入网卡配置文件

​				ubuntu写入 	/etc/sysconfig/network-scripts 目录下 

​				centos 写入 	/etc/sysconfig/network-scripts/ifcfg-xxx文件





### **2.1.2 init config final阶段**

正常情况下，在这三个阶段执行之前 instance 网络已经配置好了，并且已经成功获取到 metadata。

cloud-init 的配置文件 /etc/cloud/cloud.cfg 定义了三个阶段分别要执行的任务，任务以 module 形式指定。

instance 真正的定制工作就是由这些 module 完成的。module 决定做哪些定制化工作，而 metadata 则决定最终定制化的结果。

[module官方文档](https://cloudinit.readthedocs.io/en/latest/topics/modules.html)

![image-20220514215417368](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325612.png)

![image-20220514215438196](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325613.png)

![image-20220514215448738](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325614.png)



## 2.2 instance 的网卡是如何配置的

### 2.2.1 镜像中没有装 cloud-init

此时 instance 启动时网卡能不能被拉起来完全依赖于镜像中 /etc/network/interfaces 原有的配置。

```
auto eth0
iface eth0 inet dhcp
```

**instance 只有满足下面所有条件网卡才能被拉起来**

1. 正好只有一块网卡
2. 正好网卡就叫 eth0
3. 正好 subnet 开了 DHCP



**只要出现下面任意一种情况就会失败**

1. 还有其他网卡，比如 eth1，或者
2. 网卡不叫 eth0 ，比如 ens3，或者
3. 没有 DHCP

不同 instance 的网络配置差别很大，在 image 中写死的方法几乎是无效的，只能依靠 cloud-init 动态写入



### 2.2.2 镜像中安装了 cloud-init

####  2.2.2.1 subnet 开启 DHCP

**使用的测试镜像是 ubuntu 的 cloud image，已经预装了cloud-init**



打开 /var/log/cloud-init.log，分析如下

![image-20220514220724389](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325615.png)

这里可以看到，cloud-init 会做如下工作：

① 扫描出 instance 中的所有网卡（这里是 ens3）

② 获取该网卡的配置信息。 因为没有 config drive，无法得知网卡的详细配置信息，只能采用默认的 fallback 配置，即 dhcp 配置。

③ 将配置信息写入 /etc/network/interfaces.d/50-cloud-init.cfg，内容为：

![image-20220514220824461](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325616.png)

![image-20220514220852005](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325617.png)

这样网卡就以 dhcp 模式拉起来，正好与 subnet 的 dhcp 服务对接上，IP、网关等信息就配上去了。

1. instance 上的每一块网卡都会被 cloud-init 扫描出来。
2. 如果没有 config drive 将采用 fallback 配置，将扫描出来的**第一块** （只有这一块）网卡配置成 dhcp 模式。
   请注意：这是 cloud-init 默认行为，跟这块网卡对应的 subnet 是否开启了 DHCP 没有任何关系。
3. cloud-init 会根据 instance 操作系统类型生成网卡配置文件。例如操作系统是 centos 的话则会将配置写到 /etc/sysconfig/network-scripts 目录下。





####  2.2.2.2 subnet 没开 DHCP

在其他条件不变的情况下，cloud-init 依然会完成那 3 个步骤，也就是说网卡还是会被配置成 dhcp 模式，只是最后网卡没办法获得 IP 而已。

不开 DHCP 也是一个常见的场景，为了让 instance 的网卡在这种情况下也能够被正确配置，我们需要借助 config drive。

**在计算节点  /etc/kolla/nova-compute/nova.conf 中需要添加一个配置，然后重启 nova-compute 服务**

```bash
[DEFAULT]
flat_injected = True

#flat_injected 的作用是让 config drive 能够在 instance 
启动时将网络配置信息动态注入到操作系统中。
```

将当前网络的 DHCP 关闭，instance 部署时指定使用 config drive

打开 /var/log/cloud-init.log，分析如下

![image-20220514222052774](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325618.png)





① 扫描出 instance 中的所有网卡，这一步与不使用 config drive 的情况完全一样。

② 获取该网卡的配置信息。 日志显示配置信息是从 **ds** 获取。ds 是 datasource 的缩写，在这里指的就是 config drive。
（在不使用 config drive 的情况下采用的是 **fallback** 配置）。

网卡配置信息记录在 config drive openstack/latest/network_data.json 文件里，内容如下

![image-20220514222301109](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325619.png)

将配置信息写入 /etc/network/interfaces.d/50-cloud-init.cfg，内容为：

![image-20220514222353791](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325620.png)

可以看到 IP 以 **static** 方式配置。



总结：

在没有使用 config drive 的情况下

cloud-init 只会配置第一块网卡，且设置为 dhcp 模式，所以：

  ① 如果 instance 只有一块网卡，且启用了 DHCP，网卡能够被正常拉起。

  ② 如果 instance 有多块网卡，第一块会尝试以 dhcp 方式拉起，其他网卡不作处理。

使用 config drive 的情况下，

无论是否启用 DHCP，所有网卡都能被正确配置且成功拉起（如果 dhcp 网卡 >= 2，CentOS 还是有问题，可能跟目前所用的 cloud-init 版本较低有关）。

如果可能，尽量使用 config drive。







## 2.3 使用方法

cloud-init 默认会将 instance 的名字设置为 hostname,可利用 cloud-init 的set_hostname 模块指定自己hostname，

官方的 cloud image 默认只能通过 ssh key 登录。我们可以利用set-passwords 模块为用户设置密码并启用密码登录

```bash
#cloud-config
chpasswd:
    list: |
        root:1
        ubuntu:1
    expire: false
ssh_pwauth: yes
disable_root: false
hostname: cloud.yunshan.net
manage_etc_hosts: true 
timezone: 'Asia/Shanghai'
ssh_authorized_keys:
  - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDC2LxfeoP3+P6ZZB6YL1CdIlyri6LtJdloxl+RoPLze/uKsoTShOXK2lm5nK4cJuLU93UqGhGFXVOCkFvoBD3UWElrczHwbkJE8llry6SoHi3hoZi5mqo7BvKolDVMreEMsojSgHRDUXTwaoebtCgzIBKvVfhCbKDNWbpSEN1+MSv2PQQ8nJOafJjWmkUgBugeaTGAOHR0FnZDpD5tCyMscNzs7bPNGdIy41Ud1EyMQC5VUe6uxyfSSnA+Dt3CqU5qJ8ecPfVaSM+kk7CjThFQMiH0DUE4HykLglzXkfMQ2JrBGWUj5sOlpdjz7VNgk29K0w351sHs5H3U9QzYWEaV root@deployment
runcmd:
    - [ mkdir, /yunshan ]
    
    
    
@说明如下

1. cloud-init 只会读取以 #cloud-config开头的数据，所以这一行一定要写对。
2. chpasswd						修改root和ubuntu密码为1
3. ssh_pwauth: yes  			ssh允许密码登录
4. disable_root: false 			ssh允许root用户登录
5. hostname: cloud.yunshan.net 	主机名设置为 cloud.yunshan.net
6. manage_etc_hosts: true 		更新 /etc/hosts 文件。
7.timezone: 'Asia/Shanghai'		修改时区为上海
8.ssh_authorized_keys			推送公钥实现免密登录
9.runcmd						执行命令 mkdir /yunshan
```



### ①web界面创建实例时，加入脚本内容

<img src="https://raw.githubusercontent.com/wg1217/picbed/main/202209132325621.png" alt="image-20220513231006368" style="zoom:33%;" />



**host02上创建vm11**

![image-20220513232122464](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325622.png)

可以 root免密登录，文件夹创建成功，hostname 正确设置，/etc/hosts 随之改变

<img src="https://raw.githubusercontent.com/wg1217/picbed/main/202209132325623.png" alt="image-20220513231930369" style="zoom:33%;" />

<img src="https://raw.githubusercontent.com/wg1217/picbed/main/202209132325624.png" alt="image-20220513232104339" style="zoom:33%;" />



### ②web界面创建实例时，上传脚本文件

<img src="https://raw.githubusercontent.com/wg1217/picbed/main/202209132325625.png" alt="image-20220513232349922" style="zoom: 33%;" />



<img src="https://raw.githubusercontent.com/wg1217/picbed/main/202209132325626.png" alt="image-20220513232418679" style="zoom:33%;" />

**host02上创建vm12**

![image-20220513232746819](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325627.png)

![image-20220513232909432](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325628.png)

### ③命令行创建实例时，使用参数传入脚本

```
openstack server create --image unbuntu --flavor big --network vlan-1020 --availability-zone host02 --user-data user-data.txt  --config-drive true  vm13
```

![image-20220513233205192](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325629.png)

**host02上创建vm13**

![image-20220513233510701](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325630.png)

![image-20220513233652593](https://raw.githubusercontent.com/wg1217/picbed/main/202209132325631.png)



# 3.参考文档

https://www.xjimmy.com/study_openstack

https://cloudinit.readthedocs.io/en/latest/