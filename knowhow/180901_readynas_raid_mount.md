## Readynas RAIDディスクの片方のみをマウントする
### Date: 2018/09/01
#### Version Information
Readynas: v6.9.3
~~~
[root@localhost /]# cat /etc/redhat-release
CentOS Linux release 7.5.1804 (Core)
~~~

#### 手順
##### パーティッションの確認

今回は/dev/sdb3がReadynasのデータ領域。
~~~
[root@localhost mnt]# fdisk -l

Disk /dev/sda: 250.1 GB, 250059350016 bytes, 488397168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O サイズ (最小 / 推奨): 4096 バイト / 4096 バイト
Disk label type: dos
ディスク識別子: 0xf8c72850

デバイス ブート      始点        終点     ブロック   Id  システム
/dev/sda1   *        2048   425480191   212739072    7  HPFS/NTFS/exFAT
/dev/sda2       425480192   427577343     1048576    c  W95 FAT32 (LBA)
/dev/sda3       427577344   428101631      262144   83  Linux
/dev/sda4       428101632   488396799    30147584    5  Extended
/dev/sda5       428103680   488396799    30146560   8e  Linux LVM
WARNING: fdisk GPT support is currently new, and therefore in an experimental phase. Use at your own discretion.

Disk /dev/sdb: 2000.4 GB, 2000398934016 bytes, 3907029168 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O サイズ (最小 / 推奨): 4096 バイト / 33553920 バイト
Disk label type: gpt
Disk identifier: 7641B4BA-488D-440D-8ADE-D9488AD3BE38


#         Start          End    Size  Type            Name
 1           64      8388671      4G  Linux RAID      
 2      8388672      9437247    512M  Linux RAID      
 3      9437248   3907025071    1.8T  Linux RAID      

Disk /dev/mapper/cl-root: 26.6 GB, 26570915840 bytes, 51896320 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O サイズ (最小 / 推奨): 4096 バイト / 4096 バイト


Disk /dev/mapper/cl-swap: 4294 MB, 4294967296 bytes, 8388608 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O サイズ (最小 / 推奨): 4096 バイト / 4096 バイト


~~~
***
##### CentOS起動時謎のRAIDがマウントされているので、それを停止
~~~
[root@localhost /]# cat /proc/mdstat
Personalities :
md125 : inactive sdb1[0](S)
      4190208 blocks super 1.2

md126 : inactive sdb3[0](S)
      1948662840 blocks super 1.2

md127 : inactive sdb2[0](S)
      523760 blocks super 1.2
[root@localhost /]# mdadm --stop --scan
[root@localhost /]# cat /proc/mdstat
Personalities : [raid1]
unused devices: <none>

~~~
***
##### RAIDの構成を行い、マウントする
~~~
[root@localhost /]# mdadm --assemble --run /dev/md0 /dev/sdb3
mdadm: /dev/md0 has been started with 1 drive (out of 2).
[root@localhost /]# cat /proc/mdstat
Personalities : [raid1]
md0 : active raid1 sdb3[0]
      1948662784 blocks super 1.2 [2/1] [U_]

unused devices: <none>
[root@localhost /]# mount /dev/md0 /mnt
~~~
