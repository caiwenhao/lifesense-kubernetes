# 环境准备

### 服务器准备

> 1. 三台master组成高可用
> 2. centos 7.2 + 4.9.0-1.el7.ucloud.x86\_6内核

### 环境初始化

```
[k8s_19]
10.9.94.239 PS1=lx-k8s-master NUM=A
10.9.173.124 PS1=lx-k8s-master NUM=B
10.9.112.195 PS1=lx-k8s-master NUM=C
```

```bash
#初始化环境
ansible-playbook -i Inventory/k8s Initialize.yml -vs -l k8s_19 -u root -k 
```

### 内核升级

> 对所有服务器进行内核升级

```bash
cd /data
wget http://10.10.119.24/ansible/4.9.0-1.el7.ucloud.x86_64.tar.gz
tar xf 4.9.0-1.el7.ucloud.x86_64.tar.gz 
cd kernel-4.9.0-1.el7.ucloud/
./install.sh 
cat /boot/grub2/grub.cfg |grep menuentry
grub2-set-default 'CentOS Linux (4.9.0-1.el7.ucloud.x86_64) 7 (Core)'
grub2-editenv list
grub2-mkconfig -o /boot/grub2/grub.cfg
```

```bash
#解决升级内核后根目录只读的问题
vim /etc/fstab 
/dev/vda1       /       xfs     defaults        0 0
reboot
```

### anisble部署支持

```bash
easy_install pip
yum install python-netaddr git
pip ansible
git clone https://github.com/caiwenhao/kubespray.git
git checkout -b lifesense origin/lifesense
```

```bash
wget https://pypi.python.org/packages/90/61/f820ff0076a2599dd39406dcb858ecb239438c02ce706c8e91131ab9c7f1/Jinja2-2.9.6.tar.gz#md5=6411537324b4dba0956aaa8109f3c77b
python setup.py install
```



