制作文件系统：

	make menuconfig依赖库libncurses-dev
	make menuconfig 配置编译选项：Symbol: STATIC [=y] 
	make && make install
	
	cd _install
	mkdir -p etc dev mnt proc sys tmp etc/init.d/
	vim etc/fstab
		proc	/proc	proc	defaults	0	0
		tmpfs	/tmp	tmpfs	defaults	0	0
		sysfs	/sys	sysfs	defaults	0	0
	vim etc/init.d/rcS
		echo -e "Welcome to tinyLinux"
		/bin/mount -a
		echo -e "Remounting the root filesystem"
		mount  -o  remount,rw  /
		mkdir -p /dev/pts
		mount -t devpts devpts /dev/pts
		echo /sbin/mdev > /proc/sys/kernel/hotplug
		mdev -s
	chmod 755 etc/init.d/rcS
	vim etc/inittab
		::sysinit:/etc/init.d/rcS
		::respawn:-/bin/sh
		::askfirst:-/bin/sh
		::ctrlaltdel:/bin/umount -a -r
	chmod 755 etc/inittab
	cd dev
	sudo mknod console c 5 1
	sudo mknod null c 1 3
	sudo mknod tty1 c 4 1
	
	vim make_image.sh
		#!/bin/bash
		rm -rf rootfs.ext3
		rm -rf fs
		dd if=/dev/zero of=./rootfs.ext3 bs=1M count=32
		mkfs.ext3 rootfs.ext3
		mkdir fs
		mount -o loop rootfs.ext3 ./fs
		cp -rf ./_install/* ./fs
		umount ./fs
		gzip --best -c rootfs.ext3 > rootfs.img.gz
	chmod a+x make_image.sh
	
	
编译内核

	export ARCH=x86
	make x86_64_defconfig
	make menuconfig 配置编译选项：
		BLK_DEV_RAM_SIZE [=65536] 
		DEBUG_INFO_DWARF_TOOLCHAIN_DEFAULT [=y]
	make
	
qemu

	qemu-system-x86_64 \
	  -kernel ./linux/arch/x86_64/boot/bzImage  \
	  -initrd ./busybox/rootfs.img.gz   \
	  -append "root=/dev/ram init=/linuxrc"  \
	  -serial file:output.txt
	
