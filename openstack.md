# openstack image

- official: <https://cloud-images.ubuntu.com/>
- kvm: <https://docs.openstack.org/nova/latest/admin/configuration/hypervisor-kvm.html>
- openstack: <https://docs.openstack.org/image-guide/ubuntu-image.html>

## openstack nova

Nova是一个OpenStack项目，提供了一种发放计算实例(也就是虚拟服务器)的方法。Nova支持创建虚拟机、裸金属服务器(通过使用ironic)，对系统容器的支持有限。Nova作为一组守护进程运行在现有的Linux服务器上，以提供该服务。

## 安装kvm

```shell
sudo apt update
sudo apt install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils

```

## ubuntu20.04镜像制作

```shell
# 下载镜像
sudo wget -O /var/lib/libvirt/boot/focal-mini.iso \
  http://archive.ubuntu.com/ubuntu/dists/focal/main/installer-amd64/current/legacy-images/netboot/mini.iso
sudo chown libvirt-qemu:kvm /var/lib/libvirt/boot/focal-mini.iso
# 创建qcow2格式磁盘
sudo qemu-img create -f qcow2 /var/lib/libvirt/images/focal.qcow2 10G
sudo chown libvirt-qemu:kvm /var/lib/libvirt/images/focal.qcow2

# 启动安装虚拟机
virt-install --virt-type kvm --name focal --ram 1024 \
  --cdrom=/var/lib/libvirt/boot/focal-mini.iso \
  --disk /var/lib/libvirt/images/focal.qcow2,bus=virtio,size=10,format=qcow2 \
  --network network=default \
  --noautoconsole \
  --os-type=linux --os-variant=ubuntu20.04

# 或者使用自定义镜像ubuntu-autoinstall-example.iso，由`README.md`中run命令生成的镜像。
virt-install --virt-type kvm --name focal --ram 1024 \
  --cdrom=ubuntu-autoinstall-example.iso \
  --disk /var/lib/libvirt/images/focal.qcow2,bus=virtio,size=10,format=qcow2 \
  --network network=default \
  --noautoconsole \
  --os-type=linux --os-variant=ubuntu20.04
```

`virt-manager`可以进入管理界面，打开终端来查看或者管理安装过程。

## 手动安装过程配置

如果是自定义的autoinstall镜像，下面的选项都是自动完成。建议选择自动安装。

![image](img/1.png)
![image](img/2.png)

配置语音，键盘布局...

`hostname`默认ubuntu就可以，启动后会安装cloud-init，可以在之后创建实例的时候设置hostname。

`mirror`选择最近的城市或国家。`proxy`默认.

`user and password`使用默认的`ubuntu`就可以,当然自建也行,同hostname，后面都可以定制。

![image](img/3.png)

`磁盘分区`: 可以选择手动或者自动,分区没有好坏之分，按照自己的需求来。

`PAM`: 更新或者不更新。

`software selection`: 选个`openssh-server`就可以了。

`install grub boot loader`: yes。

## 启动与配置

```shell
# 查看虚拟机
virsh list --all
# 看下virt配置文件
# 启动虚拟机
virsh start focal
# 查看配置或自定义一些你需要的软件
# 关机
```

## 安装配置

```shell
# Clean up (remove MAC address details)¶
virt-sysprep -d bionic
# Undefine the libvirt domain¶
virsh undefine bionic
```

镜像制作完成： /var/lib/libvirt/images/bionic.qcow2，is now ready for uploading to the Image service by using the `openstack image create` command. For more information, see the Glance User Guide.
