# Yocto2.4 recovery研究

## Service update package checking

### 1. 从服务器下载OTATimeStamp文件

通过wget命令从服务器下载OTATimeStamp文件，文件内容格式为：

```
timestamp 001.001.001
```

### 2.根据OTATimeStamp文件内容判断是否需要下载更新包

将下载的OTATimeStamp文件中的timestamp字段与本地OTATimeStamp文件中的timestmp字段进行比较，判断是否需要去下载更新包

### 3. 下载md5文件

当需要下载更新包时，首先需要下载md5文件，文件名格式为timestamp.md5（xxx.xxx.xxx.md5），其中timestmp为OTATimeStamp文件中提取的timestamp字段值，可使用如下命令生成md5文件：

```bash
md5sum [file name] > xxx.xxx.xxx.md5
```

md5文件内容示例：

```bash
1 c56325fb2a7b561e8447ecc481b65fa8  file_001.tgz
2 fe512d292882ca345d3ff592e7e6575c  zImage
3 4e1bd25c1d24d9a58c7beb12fc193f34  imx6dl-vab820.dtb
4 9efc43e4317b1239977b50ca3755f2dc  imx6q-vab820.dtb
```



### 4. 根据md5文件内容下载更新包

md5文件中列出了能从服务器下载的文件名，service程序会根据解析md5文件内容，逐一下载各个文件



## eMMC partition information

| 分区 | 分区名称 | 文件系统类型 |                    分区作用                     |
| :--: | :------: | :----------: | :---------------------------------------------: |
|  p1  |   boot   |     fat      |        存放系统正常启动的zImage、dtb文件        |
|  p2  |  rootfs  |     ext4     |          存放系统正常启动的rootfs文件           |
|  p3  | recovery |     fat      |  存放recovery模式的zImage、dtb、recovery.img等  |
|  p5  |  image   |     ext4     | 存放OTATimeStamp和下载的zImage、dtb、rootfs文件 |
|  p6  |   data   |     ext4     |              临时空间，可以不需要               |



## Create new partition by using parted tool

**MBR格式的分区：**

```bash
create_new_partitions()
{
	partprobe
	sync
	
	echo "unit MiB"                            >  $cmdfile
	echo "mkpart primary ext4 8 24"            >> $cmdfile
	echo "mkpart primary ext4 25 2073"         >> $cmdfile
	if [ $enable_recovery == "1" ]; then
		echo "mkpart primary ext4 2074 2090"   >> $cmdfile
		echo "mkpart primary ext4 2091 3627"   >> $cmdfile
		echo "mkpart primary ext4 3628 4652"   >> $cmdfile
		echo "mkpart primary ext4 4653 -1"     >> $cmdfile
	else
		echo "mkpart primary ext4 2074 -1"     >> $cmdfile
	fi
	echo "print"                               >> $cmdfile
	echo "quit"                                >> $cmdfile

	partprobe
	sync

	dd if=/dev/zero of=$emmc_dev bs=1k count=1 conv=notrunc
	sync

	partprobe
	sync

	# MBR type paritition
	parted -s $emmc_dev mktable msdos
	partprobe
	sync

	cat $cmdfile | parted $emmc_dev
	partprobe
	sync
	
	partprobe
	mdev -s
	sync
	
	# dump partition table
	printf "unit MiB\nprint\nquit\n" | parted $emmc_dev
	
	# Partprobe will notify udevd to mount partitions.
	# We have to unmount all partitions again.
	umount -f "$emmc_dev"*
	sync
}
```

**GPT格式的分区：**

```bash

create_new_partitions()
{
	partprobe
	sync

	echo "unit MiB"                            >  $cmdfile
	echo "mkpart boot ext4 8 24"               >> $cmdfile
	echo "mkpart rootfs ext4 25 2073"          >> $cmdfile
	if [ $enable_recovery == "1" ]; then
		echo "mkpart recovery ext4 2074 2090"  >> $cmdfile
		echo "mkpart extended ext4 2091 3627"  >> $cmdfile
		echo "mkpart image ext4 3628 4652"     >> $cmdfile
		echo "mkpart data ext4 4653 -1"        >> $cmdfile
	else
		echo "mkpart data ext4 2074 -1"        >> $cmdfile
	fi
	echo "print"                               >> $cmdfile
	echo "quit"                                >> $cmdfile

	partprobe
	sync

	dd if=/dev/zero of=$emmc_dev bs=1k count=1 conv=notrunc
	sync

	partprobe
	sync

	# GPT type paritition
	parted -s $emmc_dev mktable gpt
	partprobe
	sync

	cat $cmdfile | parted $emmc_dev
	partprobe
	sync
	
	partprobe
	mdev -s
	sync
	
	# dump partition table
	printf "unit MiB\nprint\nquit\n" | parted $emmc_dev
	
	# Partprobe will notify udevd to mount partitions.
	# We have to unmount all partitions again.
	umount -f "$emmc_dev"*
	sync
}

```



## Add support for uboot recovery mode

使能Yocto2.4 uboot 自带的recovery功能（即Android recovery功能）的patch如下：

```bash
diff --git a/configs/mx6qvab820_defconfig b/configs/mx6qvab820_defconfig
index c4e3e23..f7f42f9 100644
--- a/configs/mx6qvab820_defconfig
+++ b/configs/mx6qvab820_defconfig
@@ -2,7 +2,8 @@ CONFIG_ARM=y
 CONFIG_ARCH_MX6=y
 CONFIG_TARGET_MX6QVAB820=y
 CONFIG_VIDEO=y
-CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/via/mx6qvab820/vab820.cfg,MX6Q"
+CONFIG_SYS_EXTRA_OPTIONS="IMX_CONFIG=board/via/mx6qvab820/vab820.cfg,MX6Q,ANDROID_SUPPORT"
+CONFIG_EFI_PARTITION=y
 CONFIG_BOOTDELAY=3
 # CONFIG_CONSOLE_MUX is not set
 CONFIG_SYS_CONSOLE_IS_IN_ENV=y
diff --git a/drivers/usb/gadget/command.c b/drivers/usb/gadget/command.c
index e9f7d29..9a20907 100644
--- a/drivers/usb/gadget/command.c
+++ b/drivers/usb/gadget/command.c
@@ -7,7 +7,12 @@
 #include <common.h>
 #include <g_dnl.h>
 #include "bcb.h"
+#ifdef CONFIG_FASTBOOT_STORAGE_NAND
+#include <nand.h>
+#endif
+#include "bcb.h"

+#ifndef CONFIG_FASTBOOT_STORAGE_NAND
 int bcb_read_command(char *command)
 {
        int ret = 0;
@@ -60,3 +65,57 @@ int bcb_write_command(char *bcb_command)
        free(p_block);
        return 0;
 }
+#else
+#define ALIGN_BYTES 64
+#define MISC_PAGES 3
+int bcb_read_command(char *command)
+{
+       char read_cmd[128];
+       char *addr_str;
+       char *nand_str;
+       ulong misc_info_size;
+       struct mtd_info *nand = nand_info[0];
+       if (command == NULL)
+               return -1;
+       memset(read_cmd, 0, 128);
+       misc_info_size = MISC_PAGES * nand->writesize;
+       nand_str = (char *)memalign(ALIGN_BYTES, misc_info_size);
+       sprintf(read_cmd, "nand read 0x%x ${misc_nand_offset} \
+                       0x%x", nand_str, misc_info_size);
+       run_command(read_cmd, 0);
+       /* The offset of bootloader_message is 1 PAGE.
+        * The offset of bootloader_message and the size of misc info
+        * need align with user space and recovery.
+        */
+       addr_str = nand_str + nand->writesize;
+       memcpy(command, (char *)addr_str, 32);
+       free(nand_str);
+       return 0;
+}
+int bcb_write_command(char *command)
+{
+       char cmd[128];
+       char *addr_str;
+       char *nand_str;
+       ulong misc_info_size;
+       struct mtd_info *nand = nand_info[0];
+       if (command == NULL)
+               return -1;
+       memset(cmd, 0, 128);
+       misc_info_size = MISC_PAGES * nand->writesize;
+       nand_str = (char *)memalign(ALIGN_BYTES, misc_info_size);
+       sprintf(cmd, "nand read 0x%x ${misc_nand_offset} \
+                       0x%x", nand_str, misc_info_size);
+       run_command(cmd, 0);
+       /* the offset of bootloader_message is 1 PAGE*/
+       addr_str = nand_str +  nand->writesize;
+       memcpy((char *)addr_str, command, 32);
+       /* erase 3 pages which hold BCB struct.*/
+       sprintf(cmd, "nand erase ${misc_nand_offset} 0x%x",nand->erasesize);
+       run_command(cmd, 0);
+       sprintf(cmd, "nand write 0x%x ${misc_nand_offset} 0x%x",nand_str, misc_info_size);
+       run_command(cmd, 0);
+       free(nand_str);
+       return 0;
+}
+#endif
diff --git a/drivers/usb/gadget/f_fastboot.c b/drivers/usb/gadget/f_fastboot.c
index 7a4b3d7..cb67415 100755
--- a/drivers/usb/gadget/f_fastboot.c
+++ b/drivers/usb/gadget/f_fastboot.c
@@ -1422,6 +1422,7 @@ void board_fastboot_setup(void)
        u32 dev_no;
 #endif
        switch (get_boot_device()) {
+       case SPI_NOR_BOOT:
 #if defined(CONFIG_FASTBOOT_STORAGE_MMC)
        case SD1_BOOT:
        case SD2_BOOT:
@@ -1496,6 +1497,7 @@ void board_recovery_setup(void)
 #endif
        int bootdev = get_boot_device();
        switch (bootdev) {
+       case SPI_NOR_BOOT:
 #if defined(CONFIG_FASTBOOT_STORAGE_MMC)
        case SD1_BOOT:
        case SD2_BOOT:
diff --git a/include/configs/mx6qvab820_common.h b/include/configs/mx6qvab820_common.h
index 39dd5aa..3a5a028 100644
--- a/include/configs/mx6qvab820_common.h
+++ b/include/configs/mx6qvab820_common.h
@@ -447,6 +447,7 @@
 #define CONFIG_SUPPORT_RAW_INITRD
 #define CONFIG_SERIAL_TAG

+/*
 #undef CONFIG_EXTRA_ENV_SETTINGS
 #undef CONFIG_BOOTCOMMAND

@@ -474,6 +475,7 @@
        "boot_normal=run bootargs_all; run bspinst_img; boota mmc0\0" \
        "boot_recovery=run bootargs_all; run bspinst_img; boota mmc0 recovery\0" \
        "bootcmd=run boot_normal\0"
+*/

 #define CONFIG_FASTBOOT_BUF_ADDR   CONFIG_SYS_LOAD_ADDR
 #define CONFIG_FASTBOOT_BUF_SIZE   0x19000000

```



## Run recovery steps in uboot

```c
board_r.c initr_check_fastboot
    |-> f_fastboot.c fastboot_run_mode
            |-> fastboot_get_bootmode
            |       |-> is_recovery_key_pressing
            |       |-> command.c bcd_read_command
            |       |       |-> bcd.c bcd_rw_block // 从misc分区读取数据
            |       |-> bcd_write_command // 清除misc分区的内容，下次启动不会执行recovery
            |-> board_recovery_setup
                    |-> // 设置环境变量，启动recovery系统
```



## Enter recovery mode in uboot

uboot判断是否进入recovery模式，有**读取指定寄存器值**和**读取misc分区内容**（yocto2.4 uboot默认采取此方式，与Android8.0类似）这两种方式。

**1. 如果采用读取misc分区方式：**

需要在reboot前往misc分区写入特定数据，数据结构格式如下：

```c
/* keep same as bootable/recovery/bootloader.h */
struct bootloader_message {
	char command[32];    // such as "boot-recovery"
	char status[32];
	char recovery[768];  // such as "recovery"

	/* The 'recovery' field used to be 1024 bytes. It has only ever
	 been used to store the recovery command line, so 768 bytes
	 should be plenty.  We carve off the last 256 bytes to store the
	 stage string (for multistage packages) and possible future
	 expansion. */
	char stage[32];

	/* The 'reserved' field used to be 224 bytes when it was initially
	 carved off from the 1024-byte recovery field. Bump it up to
	 1184-byte so that the entire bootloader_message struct rounds up
	 to 2048-byte.
	 */
	char reserved[1184];
};
```

构造misc数据文件源代码：

```c
#include "stdlib.h"
#include "stdio.h"
#include "string.h"

#define file_name   "misc.bin"

struct bootloader_message {
	char command[32];
	char status[32];
	char recovery[768];
	char stage[32];
	char reserved[1184];
};

struct bootloader_message boot;

int main(int argc, char *argv[])
{
	FILE *file;

	if((file = fopen(file_name, "wb")) == NULL) {
		printf("open %s failed\n", file_name);
		return -1;
	}

	memset(&boot, 0, sizeof(struct bootloader_message));
	strcpy(boot.command, "boot-recovery");
	strcpy(boot.recovery, "recovery");

	fwrite(&boot, 1, sizeof(struct bootloader_message), file);

	fclose(file);

	printf("%s has been generated successfully\n", file_name);

	return 0;
}
```

执行如下命令编译源代码，生成可执行文件misc：

```bash
gcc misc.c -o misc
```

执行如下命令生成misc.bin数据文件：

```bash
./misc
```

执行如下命令将misc.bin数据文件写入misc分区：

```bash
dd if=misc.bin of=/dev/mmcblk0p3  # 假如misc分区文件为/dev/mmcblk0p3
sync
```

重启系统后，uboot便能从misc分区读取recovery需要的数据，并设置环境变量，进入recovery模式：

```
Fastboot: Got Recovery key pressing or recovery commands!
setup env for recovery..
Hit any key to stop autoboot:  0
boota mmc1 recovery
```



## Write misc data in Android8.0

在Android8.0中，执行reboot时（命令行执行reboot时调用reboot.c中的main函数，Android上层则是调用android_reboot），会设置sys.powerctl属性，使得init进程监听到属性变化，然后执行如下流程将command数据写入misc分区：

```c
android_reboot.c android_reboot或reboot.c main
    |-> init.cpp HandlePowerctlMessage
            |-> bootloader_message.cpp write_reboot_recovery
                    |-> // 往misc分区写入数据
```

