# vagrant镜像制作

需求: <https://www.vagrantup.com/docs/providers/virtualbox/boxes>

## 开始

生成密码：

```shell
# vagrant
printf 'vagrant' | openssl passwd -6 -salt 'bPQ6FGEIpMN4yrCL' -stdin
```

创建自动化镜像:

```shell
./ubuntu-autoinstall-generator.sh  -a -u user-data/vagrant.example -k -s ubuntu-20.04.4-2022.06.29-live-server-amd64.iso -d ubuntu-autoinstall-vagrant.iso
```

## virtualbox创建虚拟机

<https://www.vagrantup.com/docs/providers/virtualbox/boxes>

创建虚拟机: ubuntu-vagrant

使用virtualbox创建虚拟机，选择自定义镜像。

- 第一张网卡必须是nat模式
- 磁盘大小可以大一点，60G
- 关闭usb(System -> Motherboard -> pointing device记得不选usb)
- 关闭声卡

uefi启动：

- System -> Motherboard -> Extended Features: Enable EFI
- Boot HDD must be attached to SATA controller! IDE, SCSI or SAS will not work (bug report: Ticket #14142)

初始化机器的命令：

`user-data`中已经包含了,有问题的也可以启动镜像后手动调整。

```shell
echo 'vagrant ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers.d/vagrant

# 添加vagrant public key到ssh
sudo su - vagrant
mkdir -m 0700 -p /home/vagrant/.ssh
curl https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub > /home/vagrant/.ssh/authorized_keys
chmod 0600 /home/vagrant/.ssh/authorized_keys
```

关机，下面开始导出镜像。

## 导出box文件

```shell
# base参数指定的是virtualbox中创建的虚拟机的名字
vagrant package --output ubuntu-base.box --base ubuntu-vagrant

# 添加box到本地
vagrant box add meimeitou/ubuntu-20.04 ubuntu-base.box
# 创建一个Vagrantfile
vagrant init meimeitou/ubuntu-20.04
# 启动虚拟机
vagrant up
```
