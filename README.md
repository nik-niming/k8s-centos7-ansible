参考《和我一步步部署 KUBERNETES 集群》，将其部署步骤通过ansible playbook实现自动化部署。
https://www.gitbook.com/book/opsnull/follow-me-install-kubernetes-cluster/details

1. 环境要求
实验环境通过MBP下的virtualbox和vagrant来提供3台虚拟机(centos 7)进行部署，其中1台配置为k8s master其他2台配置为k8s nodes。可以根据实际环境扩展k8s noded的数量。

假设虚拟机IP信息如下:
```
kube-ansible 192.168.33.10   MEM 2G
kube-master  192.168.33.100  MEM 2G
kube-node1   192.168.33.201  MEM 2G
kube-node2   192.168.33.202  MEM 2G
```
通过vagrant/ansible, vagrant/cluster目录下执行vagrant up来初始化虚拟机环境

2. ansible master主机配置
通过vagrant ssh kube-ansible连接到ansible master主机后，通过执行如下命令开启root用户权限的ssh访问。
```
$sudo passwd root
$su -
#vi /etc/ssh/sshd_config   确保如下配置选项的开启与关闭
    PermitRootLogin yes
    PasswordAuthentication yes
    #PasswordAuthentication no
#systemctl restart sshd
```
实现ansible master主机对集群其他主机root帐号的免密连接
```
#ssh-keygen    创建ssh key
#ssh-copy-id root@192.168.33.100
#ssh-copy-id root@192.168.33.201
#ssh-copy-id root@192.168.33.202
```
安装ansible
```
#yum install -y epel-release
#yum install -y python-pip  python-netaddr  ansible git
#pip install --upgrade Jinja2
```
配置/opt/data目录下的相关k8s安装资源
```
为了避免每次通过网络下载二进制软件包，这里通过在ansible master主机的/opt/data目录下存放下载好的这些必须文件，在playbook执行过程中将直接通过copy模块实现文件复制，不通过get_url模块在线下载，加快构建速度

[CFSSL]
#cd /opt/data/
#mkdir -p /opt/data/cfssl_linux-amd64
#cd /opt/data/cfssl_linux-amd64
#wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
#chmod +x cfssl_linux-amd64

#wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
#chmod +x cfssljson_linux-amd64

#wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
#chmod +x cfssl-certinfo_linux-amd64

[etcd]
#cd /opt/data/
#wget https://github.com/coreos/etcd/releases/download/v3.1.6/etcd-v3.1.6-linux-amd64.tar.gz
#tar -xvf etcd-v3.1.6-linux-amd64.tar.gz
#mv etcd-v3.1.6-linux-amd64/ etcd/

[flannel]
#cd /opt/data/
#mkdir -p /opt/data/flannel
#wget https://github.com/coreos/flannel/releases/download/v0.7.1/flannel-v0.7.1-linux-amd64.tar.gz
#tar -xzvf flannel-v0.7.1-linux-amd64.tar.gz -C /opt/data/flannel

[docker]
#cd /opt/data/
#wget https://get.docker.com/builds/Linux/x86_64/docker-17.04.0-ce.tgz
#tar -xvf docker-17.04.0-ce.tgz

[kubernetes-client]
#cd /opt/data/
#wget https://dl.k8s.io/v1.6.2/kubernetes-client-linux-amd64.tar.gz
#tar -xzvf kubernetes-client-linux-amd64.tar.gz

[kubernetes-server]
#wget https://dl.k8s.io/v1.6.2/kubernetes-server-linux-amd64.tar.gz
#tar -xzvf kubernetes-server-linux-amd64.tar.gz

```
在ansible master主机home目录下下载代码，修改相关配置
```
#cd /root
#git clone https://github.com/nik-niming/k8s-centos7-ansible.git
#cd k8s-centos7-ansible
#vi group_vars/all   全局配置文件，确保相关位置路径，k8s集群参数设置正确
```
关键的全局参数：

集群主机网络主接口(如果使用vagrant，默认eth0为NAT,请调整为eth1)  
```
HOST_INTERFACE: eth1    
```

确保ansible master主机上的文件存储路径正确
```
已下载的第三方软件包，预生成的CA证书，配置文件等目录
CFSSL_DIR: /opt/data/cfssl_linux-amd64 
CA_DIR: /opt/data/ca
ETCD_DIR: /opt/data/etcd-v3.1.6-linux-amd64
FLANNEL_DIR: /opt/data/flannel
DOCKER_DIR: /opt/data/docker
KUBE_CLIENT_DIR: /opt/data/kubernetes/client/bin
KUBE_SERVER_DIR: /opt/data/kubernetes/server/bin 
KUBE_CLIENT_CONFIG: /opt/data/kube_client_config

下载的二进制文件执行目录(ansible 目标主机存储二进制文件的位置)
BIN_DIR: /root/local/bin
```

根据实际虚拟机环境进行ip地址替换
```
集群Master IP
MASTER_IP: 192.168.33.100

etcd 集群所有机器 IP
ETCD_NODE_IPS: 192.168.33.100 192.168.33.201 192.168.33.202

etcd 集群间通信的IP和端口
ETCD_NODES: master=https://192.168.33.100:2380,node1=https://192.168.33.201:2380,node2=https://192.168.33.202:2380

etcd 集群服务地址列表
ETCD_ENDPOINTS: https://192.168.33.100:2379,https://192.168.33.201:2379,https://192.168.33.202:2379
```


