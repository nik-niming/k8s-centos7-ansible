参考《和我一步步部署 KUBERNETES 集群》，将其部署步骤通过ansible playbook实现自动化部署。
https://www.gitbook.com/book/opsnull/follow-me-install-kubernetes-cluster/details

1. 环境要求
实验环境通过MBP下的virtualbox和vagrant来提供3台虚拟机(centos 7)进行部署，其中1台配置为k8s master其他2台配置为k8s nodes。可以根据实际环境扩展k8s noded的数量。

2. ansible主机配置
vagrant的配置文件分为2个部分：a.ansible主机 b.cluster主机

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

yum install -y epel-release
yum install -y python-pip  python-netaddr  ansible git
pip install --upgrade Jinja2
```

