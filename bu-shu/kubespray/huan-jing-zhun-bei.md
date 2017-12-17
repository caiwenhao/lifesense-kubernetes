# 环境准备

### 服务器准备

> 1. 三台master组成高可用
> 2. centos 7.2 + 4.9.0-1.el7.ucloud.x86\_6内核

### 内核升级

> 对所有服务器进行内核升级

```bash
curl 10.10.119.24/ansible/4.9.0-1.el7.ucloud.x86_64.tar.gz
tar xf 4.9.0-1.el7.ucloud.x86_64.tar.gz 
cd kernel-4.9.0-1.el7.ucloud/
./install.sh 
cat /boot/grub2/grub.cfg |grep menuentry
grub2-set-default 'CentOS Linux (4.9.0-1.el7.ucloud.x86_64) 7 (Core)'
grub2-editenv list
grub2-mkconfig -o /boot/grub2/grub.cfg
vim /etc/fstab 
reboot
```

```
/dev/vda1       /       xfs     defaults        0 0
```

### 环境初始化

```

```

### anisble部署支持

```bash
easy_install pip
yum install python-netaddr git
pip ansible
git clone https://github.com/caiwenhao/kubespray.git
git checkout -b lifesense origin/lifesense
wget https://pypi.python.org/packages/90/61/f820ff0076a2599dd39406dcb858ecb239438c02ce706c8e91131ab9c7f1/Jinja2-2.9.6.tar.gz#md5=6411537324b4dba0956aaa8109f3c77b
python setup.py install
```



