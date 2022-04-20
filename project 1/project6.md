# WEB SOLUTION WITH WORDPRESS

Spin-up 2 server, one for wordpress web server and the other for MySQL database server, note the availability zones of the servers.

Goto volumes and create 3 volume instances and attach them to the web-server.

then we can do `lsblk` on our terminal to view the attached volumes and `df -h` to view all mounts and free space on the server.

use command `gdisk /dev/xvdf` to initiate the partition process. it gives a prompt and we type `n` for new partition 

we then respond `1` for just a single partition (we can have more that 1, based on preference).

we then press enter twice to use up the whole space available

the current type of the system is usually "linux filesystem" we need to change to logical volumes by selecting (typing) `8e00` 
and then `p` to view what we have configured so far.

we then use `w` to write and confirm the command with a yes `Y`,

at this stage, it should show us that the operation is successfull.

So, we repeat the process for "xvdg and xvdh"

we can then run `lsblk` to view our volumes, each should now carry the number of partitions configured.

we now install "LVM2" `yum install lvm2 -y`  (note, the -y flag is to auto respond yes to all promted questions)

to confirm that it was successfully installed, we can perform `which lvm` and see it in the installed directory

### Create physical volumes

we use the command `pvcreate /dev/xvdf1 /dev/xvdg1 /dev/xvdh1` to create physical volumes and use `pvs` to check the physical volumes

### Volume group

we use the volume group to put the physical volume resource into a single group from which a logical volume can then be created

run `vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1` to create a volume group called "webdata-vg" (in the future we can use vgextend to add more volume to the group)

we can do `vgs` to view the volume group (it should show us the sum of all the physical volumes we added)

### Logical volume

This helps us to manage our volume easily without any downtime.

We use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size

`lvcreate -n apps-lv -L 14G webdata-vg` for the application volume and

`lvcreate -n logs-lv -L 14G webdata-vg` for the logs, we can do `lvs` to see the logical volumes

if we exhust the volume, we can create an additional physical volume, add it to our volume group by `vgextend` and add it to our logical volume by `lvextend` .

## View Setup

use `vgdisplay -v` you get the output below

`
  --- Volume group ---
  VG Name               webdata-vg
  System ID
  Format                lvm2
  Metadata Areas        3
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               0
  Max PV                0
  Cur PV                3
  Act PV                3
  VG Size               <29.99 GiB
  PE Size               4.00 MiB
  Total PE              7677
  Alloc PE / Size       7168 / 28.00 GiB
  Free  PE / Size       509 / <1.99 GiB
  VG UUID               0ecHjq-gzeZ-bQR3-c3pF-PkWe-Mx3U-pa1IJv

  --- Logical volume ---
  LV Path                /dev/webdata-vg/apps-lv
  LV Name                apps-lv
  VG Name                webdata-vg
  LV UUID                2gdJ8n-OxDK-fkvW-PZ1r-9zVK-bF0q-hnh6oA
  LV Write Access        read/write
  LV Creation host, time ip-172-31-22-251.ec2.internal, 2022-04-20 16:46:45 +0000
  LV Status              available
  # open                 0
  LV Size                14.00 GiB
  Current LE             3584
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0

  --- Logical volume ---
  LV Path                /dev/webdata-vg/logs-lv
  LV Name                logs-lv
  VG Name                webdata-vg
  LV UUID                C1LSBC-JWzc-6ZJz-WinM-XS5y-NraO-4cC49X
  LV Write Access        read/write
  LV Creation host, time ip-172-31-22-251.ec2.internal, 2022-04-20 16:47:16 +0000
  LV Status              available
  # open                 0
  LV Size                14.00 GiB
  Current LE             3584
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:1

  --- Physical volumes ---
  PV Name               /dev/xvdh1
  PV UUID               1Hmlvc-uU4t-x736-7awt-RJ7M-f8Fv-0L3sfs
  PV Status             allocatable
  Total PE / Free PE    2559 / 0

  PV Name               /dev/xvdg1
  PV UUID               h1OkeY-zfrf-scVq-qFir-1AUE-Vt1q-YvANo5
  PV Status             allocatable
  Total PE / Free PE    2559 / 509

  PV Name               /dev/xvdf1
  PV UUID               yakDdd-EtJl-iDyk-d0WQ-ETcC-yVjc-W2Jp63
  PV Status             allocatable
  Total PE / Free PE    2559 / 0
`






