### 修改更新源
``` 
sudo sed -i "s/us.archive.ubuntu.com/mirrors.163.com/g" /etc/apt/sources.list
sudo apt-get update
sudo apt-get upgrade
```
### 安装kvm软件及依赖
```
sudo apt-get install kvm qemu -y && \
sudo apt-get install virtinst virt-viewer virt-manager -y
```

### 开机时自动加载IOMMU, intel vt-d
```
sudo vim /etc/default/grub
RUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_iommu=on"
```

### 更新grub文件
```
sudo update-grub
```

### 查看iommu是否启用，如果没有需要重启
```
cat /proc/cmdline
```

### 查看GPU信息, lspci -nnk | grep NV
```
04:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:1b80] (rev a1)
Subsystem: NVIDIA Corporation Device [10de:119e]
04:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:10f0] (rev a1)
Subsystem: NVIDIA Corporation Device [10de:119e]
82:00.0 VGA compatible controller [0300]: NVIDIA Corporation Device [10de:1b80] (rev a1)
Subsystem: NVIDIA Corporation Device [10de:119e]
82:00.1 Audio device [0403]: NVIDIA Corporation Device [10de:10f0] (rev a1)
Subsystem: NVIDIA Corporation Device [10de:119e]
```
> 以上信息中 04:00.0 04:00.1 82:00.0 82:00.1 四组数字分别是两块显卡的PCI插槽序列（宿主机机器不同，数字不同），在启动KVM安装操作系统时，需要用此PCI插槽序列

### 创建虚拟机， 注意--ram 分配给虚拟机的内存不能过大， 否则会报错
```
sudo virt-install --name kvm_gpux2 --boot network,cdrom,menu=on --ram 49152 --vcpus=12 --os-type=linux --accelerate --cdrom /home/iso/ubuntu-16.04.3-server-amd64.iso --disk path=/home/vhost/kvm_gpux2.img,size=200,format=qcow2,bus=ide --bridge=virbr0 --vnc --vncport=5991 --vnclisten=0.0.0.0 --video cirrus --host-device 04:00.0 --host-device 04:00.1 --host-device 82:00.0 --host-device 82:00.1
```
> 创建虚拟机成功后使用vnc客户端连接， 安装系统
关闭虚拟机用于配置文件的更新， 注意这个方法相当于直接断掉虚拟机电源， 最好使用ACPI服务进行开关机
####参考 http://ilanni.blog.51cto.com/526870/1536942/
```
sudo virsh destroy kvm_gpux2
```
### 配置文件
```
sudo vim /etc/libvirt/qemu/kvm_gpux2.xml
此配置文件涵盖了KVM虚拟机所有配置信息，menmory，cpu等等，
打开配置文件，在配置文件的首行添加如下字符
<domain type ='kvm' xmlns:qemu ='http://libvirt.org/schemas/domain/qemu/1.0'>
在添加以上字符之前，删除配置文件第一行<domain type='kvm'>
其作用是允许KVM虚拟机直接发送参数到qemu，进行硬件调用
接在在配置文件的末尾</ domain>之前添加如下信息；
<qemu:commandline>
<qemu:arg value='-cpu'/>
<qemu:arg value='host,hv_time,hv_relaxed,hv_vapic,hv_spinlocks=0x1fff,kvm=off,hv_vendor_id=some4321text'/>
</qemu:commandline>
```

### 应用配置文件
```
virsh define /etc/libvirt/qemu/kvm_gpux2.xml
```

### 启动虚拟机
```
virsh start kvm_gpux2
```
### 在虚拟机中执行 
```
lspci -nnk | grep NV， 查看是否挂载显卡成功
sudo apt-get install nvidia-375 ， 安装显卡驱动
reboot， 重启生效
nvidia-smi， 查看显卡信息
```
