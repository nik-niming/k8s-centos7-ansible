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
#vi /etc/ssh/sshd_config 
    确保如下配置选项的开启与关闭
    PermitRootLogin yes
    PasswordAuthentication yes
    #PasswordAuthentication no
#systemctl restart sshd
```
安装ansible
```
#yum install -y epel-release
#yum install -y python-pip  python-netaddr  ansible git
#pip install --upgrade Jinja2
```
实现ansible master主机对集群其他主机root帐号的免密连接
```
#ssh-copy-id root@192.168.33.100
#ssh-copy-id root@192.168.33.201
#ssh-copy-id root@192.168.33.202
```
配置/opt/data目录下的相关k8s安装资源
```
为了避免每次通过网络下载二进制软件包，这里通过在ansible master主机的/opt/data目录下存放下载好的这些必须文件，在playbook执行过程中将直接通过copy模块实现文件复制，不通过get_url模块在线下载，加快构建速度

[CFSSL]
#mkdir -p /opt/data/cfssl_linux-amd64

#wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
#chmod +x cfssl_linux-amd64

#wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
#chmod +x cfssljson_linux-amd64

#wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64
#chmod +x cfssl-certinfo_linux-amd64

[etcd]
#mkdir -p /opt/data/etcd-v3.1.6-linux-amd64

#wget https://github.com/coreos/etcd/releases/download/v3.1.6/etcd-v3.1.6-linux-amd64.tar.gz
#tar -xvf etcd-v3.1.6-linux-amd64.tar.gz

[flannel]
#mkdir -p /opt/data/flannel
#wget https://github.com/coreos/flannel/releases/download/v0.7.1/flannel-v0.7.1-linux-amd64.tar.gz
#tar -xzvf flannel-v0.7.1-linux-amd64.tar.gz -C /opt/data/flannel

[docker]
#mkdir -p /opt/data/docker
#wget https://get.docker.com/builds/Linux/x86_64/docker-17.04.0-ce.tgz
#tar -xvf docker-17.04.0-ce.tgz

[kubernetes-client]
#mkdir -p /opt/data/kubernetes

#wget https://dl.k8s.io/v1.6.2/kubernetes-client-linux-amd64.tar.gz
#tar -xzvf kubernetes-client-linux-amd64.tar.gz

#wget https://github.com/kubernetes/kubernetes/releases/download/v1.6.2/kubernetes.tar.gz
#tar -xzvf kubernetes.tar.gz

```



