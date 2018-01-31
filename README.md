<!-- README file for dv-snapshots script -->
# DV Snapshots

This program is a bash script for LVM snapshot automation. The script requires *lvm2* to be instaled in the system. Also this script requires *tar* and *7z* to be installed as well.

## Configuration

`dv-snapshots` reads LVM logical volumes to backup from `/usr/local/etc/dv-lvs.cfg` and `/etc/dv/dv-lvs.cfg` in that order.

Configuration file structure must be as follows:
	volume_group/logical_volume:number_of_extents:tar_options
E.g.
	vg1/lv1:320:--exclude=./lost+found
	vg1/lv2:191:
Which tells to backup logical volumes *lv1* and *lv2* from volume group *vg1*. The first volume requires 320 physical extents for the snapshot and the second volume requires 191 PE for its snapshot, assuming that the volume group *vg1* has 511 free PEs in it. Also for the first logical volume *lv1* tar will avoid *lost+found* directory in the filesystem root.

## Free space in the volume group

LVM requires free space somwhere in the volume group. Consider the following:

```
+------------------------------Volume group-------------------------------+
|                                   vg1                                   |
+----------------------------Physical volumes-----------------------------+
|    pv1 (8190 physical extents)     |     pv2 (8190 physical extents)    |
+------------------+----------Logical volumes-----------------------------+
|  lv1 (7679 PEs)  |  Free (511 PEs) |       lv2 (8190 physical extents)  |
+------------------+-----------------+------------------------------------+
```

Number of extents, required for the snapshot, depends on how intensively the volume is written during backup process. In this case *lv1* is written about twice as much than *lv2*, thus it has 320 PEs for snapshot and *lv2* has 191.

## Free RAM

This question requires testing on different machines with different volume sizes. But in my case it was completely safe to have 2.5GiB free RAM for *tar* + *7z* to finish successfully. Even less - 2GiB was quite ok, but having less than 2GiB resulted in failure due to memory insuffisance.

## Mountpoints

Snapshot filesystems for backup are mounted under `/run/dv/shapshots`. For the example above mountpoints will be as follows:
	/run/dv/snapshots/slv1
	/run/dv/snapshots/slv2

Where `s` is a prefix (can be changed inside the script - PREFIX variable). Also directory name under `/run` is composed upon the script's name, where all the `-` (hyphens) are replaced with `/` (slashes). E.g. for the script named `my-awesome-script` and prefix `snap-` mountpoints will be:
	/run/my/awesome/script/snap-lv1
	/run/my/awesome/script/snap-lv2

<!-- vim: set si et ts=4 sw=4 number syntax=markdown: -->
