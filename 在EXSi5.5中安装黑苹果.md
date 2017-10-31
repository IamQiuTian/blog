> 
  * 准备一个os x的iso镜像
  * https://github.com/franciscocpg/install-os-maverick-vmware/archive/master.zip 把这个包下载下来
  * 将里面一个叫 'unlock-all-v130' 的文件夹上传到EXSi主机的某个存储卷上，注意是卷不是根目录
  * 将EXSi设为维护模式
  * 使用ssh进入EXSi系统
  * 进入你上传的'unlock-all-v130'文件夹的目录中，如/vmfs/volumes/55f4555e-c612d421-21b0-44a8423ad8d9/unlock-all-v130 
  * cd unlock-all-v130/exsi
  * chmod +x *
  * ./install.sh
  * 重启EXSi主机
  * 重启完成后退出维护模式就可以安装os x系统了
