# Makefile for NetBSD/hpcsh image.

# set image file name
FILE=`date +%Y-%m-%d`-netbsd-hpcsh.img

# iso file information
#IMAGE=/usr/release/images/NetBSD-8.99.25-hpcsh-201810250900Z.iso
#IMAGE=/usr/release/images/NetBSD-7.1.2_PATCH-hpcsh.iso
IMAGE=/usr/release/images/NetBSD-8.0_STABLE-hpcsh.iso 

PKGSRC=../../pkgsrc/pkgsrc-2018-10-13.tar.gz

SETS=/mnt1/hpcsh/binary/sets
INSTALLATION=/mnt1/hpcsh/installation
KERNEL=/mnt1/hpcsh/binary/kernel

# extract sets.
LIST=base etc comp man modules tests xbase xcomp xetc xfont xserver 

# mount point
MNT=/mnt
VND=vnd0

# mount cdrom
MNT_CDROM=/mnt1
VND_CDROM=vnd1

all: file newfs cdmount extract bootarea cdunmount

# make image file. 512*3813376=2GB
file: 
	echo Create ${FILE}.
	dd if=/dev/zero of=${FILE} bs=512 count=3813376 progress=50000
	sleep 3

newfs:
	vnconfig ${VND} ${FILE}
	disklabel -R ${VND} DISKLABEL.proto
	newfs_msdos /dev/r${VND}e
	newfs -n 500000 -b 4096 /dev/r${VND}a
	fdisk -ufF -0 -s 6/32/122848 ${FILE} #DOS partition:DISKLABEL.proto
	vnconfig -u ${VND}

cdmount:
	echo mount cdrom image.
	vnconfig ${VND_CDROM} ${IMAGE}
	mount_cd9660 /dev/${VND_CDROM}a ${MNT_CDROM}

cdunmount:
	echo mount cdrom image.
	umount ${MNT_CDROM}
	vnconfig -u ${VND_CDROM}

extract:
	echo extract sets.
	vnconfig ${VND} ${FILE}
	fsck -fy /dev/r${VND}a
	mount /dev/${VND}a ${MNT}
	for i in ${LIST} ; do (tar xzpf ${SETS}/$${i}.tgz -C ${MNT};); done
	(cd ${MNT}/dev;./MAKEDEV all)
#	tar xzpf ${PKGSRC} -C ${MNT}/usr
	mkdir ${MNT}/msdos
	mkdir ${MNT}/kern
	mkdir ${MNT}/proc
	umount ${MNT}
	vnconfig -u ${VND}

bootarea:
	echo boot area.
	vnconfig ${VND} ${FILE}
	fsck_msdos -fy /dev/r${VND}e
	mount /dev/${VND}e ${MNT}
	
	cp ${INSTALLATION}/hpcboot-sh3.exe ${MNT}
	cp ${INSTALLATION}/hpcboot-sh4.exe ${MNT}
	cp ${KERNEL}/netbsd-GENERIC.gz     ${MNT}
	cp ${KERNEL}/netbsd-HPW650PA.gz    ${MNT}
#	cp msdos/hpcboot.cnf ${MNT}
	
	umount ${MNT}
	vnconfig -u ${VND}

# stop sshd for package install
pkg:
	vnconfig ${VND} ${FILE}
	fsck -fy /dev/r${VND}a
	mount /dev/${VND}a ${MNT}
	echo rc_configured=YES >> ${MNT}/etc/rc.conf
	echo sshd=NO           >> ${MNT}/etc/rc.conf
	echo syslogd=NO        >> ${MNT}/etc/rc.conf
	echo dhcpcd=NO         >> ${MNT}/etc/rc.conf
	cp NET ${MNT}/root
	umount ${MNT}
	vnconfig -u ${VND}

# clean logfiles
logclean:
	vnconfig ${VND} ${FILE}
	fsck /dev/r${VND}a
	mount /dev/${VND}a ${MNT}
	rm -f ${MNT}/var/run/dmesg.boot
	rm -f ${MNT}/var/log/lastlogx 
	rm -f ${MNT}/var/log/wtmpx
	rm -f ${MNT}/var/db/dhcpcd-wi0.lease
	rm -f ${MNT}/var/db/dhcpcd-wi0.lease6
	rm -f ${MNT}/root/Package/dmesg
	rm -f ${MNT}/root/Package/pkginfo
	rm -f ${MNT}/root/NET
	rm -rf ${MNT}/var/run/dhcpcd
	rm -rf ${MNT}/var/run/resolvconf
	umount ${MNT}
	vnconfig -u ${VND}

# copy default setupfile
release:
	echo copy default files and make device nodes.
	vnconfig ${VND} ${FILE}
	fsck /dev/r${VND}a
	mount /dev/${VND}a ${MNT}
	tar cvpf - etc root  |(cd /mnt;tar xvpf -)
	umount ${MNT}
	vnconfig -u ${VND}

# fetch newest isofileimage
fetch:
	./NewestISO |tee Fetch
	sh Fetch
	rm Fetch
