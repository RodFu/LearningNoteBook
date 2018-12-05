# Linux自动挂载U盘和SD卡

## mdev工具的简单理解

> mdev是busybox中的一个udev管理程序的一个精简版，他也可以实现设备节点的自动创建和设备的自动挂载，只是在实现的过程中有点差异，在发生热插拔时间的时候，mdev是被hotplug直接调用，这时mdev通过环境变量中的 ACTION 和 DEVPATH，来确定此次热插拔事件的动作以及影响了/sys中的那个目录。接着会看看这个目录中是否有“dev”的属性文件，如果有就利用这些信息为这个设备在/dev 下创建设备节点文件。



> mdev扫描/sys/block是为了实现向后兼容)和/sys/class两个目录下的dev属性文件，从该dev 属性文件中获取到设备编号(dev属性文件以"major:minor\n"形式保存设备编号)，并以包含该dev属性文件的目录名称作为设备名 device_name(即包含dev属性文件的目录称为device_name，而/sys/class和device_name之间的那部分目录称为 subsystem。也就是每个dev属性文件所在的路径都可表示为/sys/class/subsystem/device_name/dev)，在 /dev目录下创建相应的设备文件。例如，cat /sys/class/tty/tty0/dev会得到4:0，subsystem为tty,device_name为tty0。



## 实现方法如下：

### 1、在/rootfs/etc/init.d/rcS文件中添加如下命令：

```bash
mount -t tmpfs mdev /dev
mount -t sysfs sysfs /sys
echo /sbin/mdev > /proc/sys/kernel/hotplug
/sbin/mdev -s
```

### 2、在/rootfs/etc/中增加mdev.conf文件，文件内容如下：

```bash
mmcblk[0-9]p[0-9] 0:0 666   * (/etc/hotplug/automount "/mnt/sdcard" $MDEV $ACTION)
sd[a-z][0-9]      0:0 666   * (/etc/hotplug/automount "/mnt/udisk"  $MDEV $ACTION)
```

> 其中，*表示创建设备节点后和删除设备节点前都去执行后面的命令；如果使用@，则表示在创建设备节点后去执行后面的命令；如果使用$，则表示在删除设备节点前去执行后面的命令。

### 3、在/rootfs/etc/hotplug中增加automount文件，文件内容如下：

```bash
#!/bin/sh

if [ "$1" = "" -o "$2" = "" -o "$3" = "" ]; then
	exit 1
fi

mountPath="$1"

if [ "$3" = "add" ]; then
	blockid=`blkid /dev/$2`
	if [ "$blockid" = "" ]; then
		exit 1
	fi

	tmp='"'
	labelname="${blockid#*"LABEL=$tmp"}"
	labelname="${labelname%%"$tmp UUID="*}"
	
	if [ "$labelname" = "$blockid" ]; then
		echo "labelname is null" > /dev/ttyS0
		exit 1
	fi

	if [ -b /dev/$2 ]; then
		if [ ! -d $mountPath/$2 ]; then
			mkdir -p $mountPath/$2
		fi
		mount /dev/$2 $mountPath/$2
	fi
elif [ "$3" = "remove" ]; then
	sync
	umount $mountPath/$2
	rm -rf $mountPath/$2
else
	exit 1
fi
```

### 4、设置/rootfs/etc/hotplug/automount文件的权限，使其可以被执行

```bash
chmod 777 automount
```

### 5、制作system.img

```bash
sudo make_ext4fs -s -l 768M system.img rootfs/
```

