#!/bin/bash


################
#Debootstrap
################
echo "Debootstrapping the chroot based of vivid"

sudo apt-get install debootstrap
sudo apt-get install syslinux squashfs-tools genisoimage

mkdir /home/spherox-os
cd spherox-os

sudo sudo debootstrap --arch=amd64 trusty chroot

sudo mount --bind /dev chroot/dev

#Copying network 
sudo cp /etc/hosts chroot/etc/hosts
sudo cp /etc/resolv.conf chroot/etc/resolv.conf
sudo cp /etc/apt/sources.list chroot/etc/apt/sources.list

###############
#Chroot
###############
sudo chroot chroot

mount none -t proc /proc
mount none -t sysfs /sys
mount none -t devpts /dev/pts
export HOME=/root
export LC_ALL=C
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 12345678  #Substitute "12345678" with the PPA's OpenPGP ID.
apt-get update
apt-get install --yes dbus
dbus-uuidgen > /var/lib/dbus/machine-id
dpkg-divert --local --rename --add /sbin/initctl

#############
#Software
#############
cd chroot
sudo su
apt-get --yes upgrade
apt-get install --yes grub2 plymouth-x11
apt-get install --no-install-recommends network-manager
apt-get install --yes skype wine steam firefox xubuntu-desktop
apt-get install --yes casper lupin-casper virtualbox libreoffice file-roller gnome-calculator gnome-system monitor
apt-get install --yes gimp gedit xserver-xorg-video-all nano
apt-get install --yes ubiquity user-setup os-prober live-initramfs


apt-get install ubiquity-frontend-gtk

rm /var/lib/dbus/machine-id

rm /sbin/initctl
dpkg-divert --rename --remove /sbin/initctl

sudo update-initramfs -u

#Image
cd 
cd /home/spherox-os

mkdir -p image/{casper,isolinux,install}
cp chroot/boot/vmlinuz-2.6.**-**-generic image/casper/vmlinuz
cp chroot/boot/initrd.img-2.6.**-**-generic image/casper/initrd.lz

cp /usr/lib/syslinux/isolinux.bin image/isolinux/
cp /boot/memtest86+.bin image/install/memtest

printf "\x18" >emptyfile

echo "DEFAULT live
LABEL live
  menu label ^Start or install Ubuntu Remix
  kernel /casper/vmlinuz
  append  file=/cdrom/preseed/ubuntu.seed boot=casper initrd=/casper/initrd.lz quiet splash --
LABEL check
  menu label ^Check CD for defects
  kernel /casper/vmlinuz
  append  boot=casper integrity-check initrd=/casper/initrd.lz quiet splash --
LABEL memtest
  menu label ^Memory test
  kernel /install/memtest
  append -
LABEL hd
  menu label ^Boot from first hard disk
  localboot 0x80
  append -
DISPLAY isolinux.txt
TIMEOUT 300
PROMPT 1 

#prompt flag_val
# 
# If flag_val is 0, display the "boot:" prompt 
# only if the Shift or Alt key is pressed,
# or Caps Lock or Scroll lock is set (this is the default).
# If  flag_val is 1, always display the "boot:" prompt.
#  http://linux.die.net/man/1/syslinux   syslinux manpage " > image/isolinux/isolinux.cfg

sudo chroot chroot dpkg-query -W --showformat='${Package} ${Version}\n' | sudo tee image/casper/filesystem.manifest
sudo cp -v image/casper/filesystem.manifest image/casper/filesystem.manifest-desktop
REMOVE='ubiquity ubiquity-frontend-gtk ubiquity-frontend-kde casper lupin-casper live-initramfs user-setup discover1 xresprobe os-prober libdebian-installer4'
for i in $REMOVE 
do
        sudo sed -i "/${i}/d" image/casper/filesystem.manifest-desktop
done

printf $(sudo du -sx --block-size=1 chroot | cut -f1) > image/casper/filesystem.size
sudo mksquashfs chroot image/casper/filesystem.squashfs -e boot
echo "#define DISKNAME  Ubuntu Remix
#define TYPE  binary
#define TYPEbinary  1
#define ARCH  i386
#define ARCHi386  1
#define DISKNUM  1
#define DISKNUM1  1
#define TOTALNUM  0
#define TOTALNUM0  1" > image/README.diskdefines

touch image/ubuntu

mkdir image/.disk
cd image/.disk
touch base_installable
echo "full_cd/single" > cd_type
echo "Ubuntu Remix 14.04" > info  # Update version number to match your OS version
cd ../..

sudo -s
(cd image && find . -type f -print0 | xargs -0 md5sum | grep -v "\./md5sum.txt" > md5sum.txt)
exit

cd image
sudo mkisofs -r -V "spherox-v1" -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o ../ubuntu-remix.iso .
cd ..

