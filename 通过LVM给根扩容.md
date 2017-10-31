#### 1. zabbix监控到一台测试服的根目录要满了...
#### 2. 所以我登到机器上看了眼， 果然是快满了
```
[root@localhost]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root
                   14G   12G  1.4G  90% /
tmpfs                940M   76K  940M   1% /dev/shm
/dev/sda1             477M   42M  410M  10% /boot 
```
#### 3. 给它加了块硬盘
```
[root@localhost]# fdisk -l
**********************************
**********************************
   Device Boot      Start         End      Blocks   Id  System
/dev/sda1   *           2         501      512000   83  Linux
Partition 1 does not end on cylinder boundary.
/dev/sda2             502       16384    16264192   8e  Linux LVM
Partition 2 does not end on cylinder boundary.
Disk /dev/mapper/VolGroup-lv_root: 14.9 GB, 14935916544 bytes
255 heads, 63 sectors/track, 1815 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000
********************************************************
********************************************************
Disk /dev/sdb: 10.7 GB, 10737418240 bytes
64 heads, 32 sectors/track, 10240 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```
#### 4. 然后分区
```
[root@localhost]# fdisk /dev/sdb
WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').
 
Command (m for help): n
Command action
   e   extended
   p   primary partition (1-4)
p
Partition number (1-4): 1
First cylinder (1-10240, default 1): 
Using default value 1
Last cylinder, +cylinders or +size{K,M,G} (1-10240, default 10240): 
Using default value 10240
Command (m for help): p
Disk /dev/sdb: 10.7 GB, 10737418240 bytes
64 heads, 32 sectors/track, 10240 cylinders
Units = cylinders of 2048 * 512 = 1048576 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0xbf6c7d0a
   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1               1       10240    10485744   83  Linux
 
Command (m for help): t
Selected partition 1
Hex code (type L to list codes): 8e
Changed system type of partition 1 to 8e (Linux LVM)
Command (m for help): w
The partition table has been altered!
Calling ioctl() to re-read partition table.
Syncing disks.
[root@localhost]# kpartx -a /dev/sdb
```
#### 5. 将这个新分区设置为物理卷
```
[root@localhost]# pvcreate /dev/sdb1
  Can't open /dev/sdb1 exclusively.  Mounted filesystem?
[root@localhost]# dmsetup remove sdb1
[root@localhost]# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
[root@localhost]# pvs
  PV         VG       Fmt  Attr PSize  PFree 
  /dev/sda2  VolGroup lvm2 a--  15.51g     0 
  /dev/sdb1           lvm2 ---  10.00g 10.00g
```
#### 6. 把这个物理卷加入卷组中
```
[root@localhost]# vgs
  VG       #PV #LV #SN Attr   VSize  VFree
  VolGroup   1   2   0 wz--n- 15.51g    0 
[root@localhost etc]# vgextend VolGroup /dev/sdb1
  Volume group "VolGroup" successfully extended
[root@localhost etc]# vgs
  VG       #PV #LV #SN Attr   VSize  VFree 
  VolGroup   2   2   0 wz--n- 25.50g 10.00g
```
#### 7. 开始给根目录扩容（警告，对根操作有风险， 请提前做数据备份）
```
[root@localhost]# lvs
  LV      VG       Attr       LSize  Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  lv_root VolGroup -wi-ao---- 13.91g                                                    
  lv_swap VolGroup -wi-ao----  1.60g                                                    
[root@localhost etc]# vgs
  VG       #PV #LV #SN Attr   VSize  VFree 
  VolGroup   2   2   0 wz--n- 25.50g 10.00g
[root@localhost etc]# lvextend -L +9G /dev/mapper/VolGroup-lv_root  
  Size of logical volume VolGroup/lv_root changed from 13.91 GiB (3561 extents) to 22.91 GiB (5865 extents).
  Logical volume lv_root successfully resized
[root@localhost etc]# resize2fs /dev/mapper/VolGroup-lv_root  
resize2fs 1.41.12 (17-May-2010)
Filesystem at /dev/mapper/VolGroup-lv_root is mounted on /; on-line resizing required
old desc_blocks = 1, new_desc_blocks = 2
Performing an on-line resize of /dev/mapper/VolGroup-lv_root to 6005760 (4k) blocks.
The filesystem on /dev/mapper/VolGroup-lv_root is now 6005760 blocks long.
```
#### 8. 查看是否扩容成功
```
[root@localhost]# df -h
Filesystem            Size  Used Avail Use% Mounted on
/dev/mapper/VolGroup-lv_root
                       23G   12G  9.8G  54% /
tmpfs                 940M   76K  940M   1% /dev/shm
/dev/sda1             477M   42M  410M  10% /boot
```
