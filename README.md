# linux-surface-ubuntu
How to create your own live usb for surface pro 4 with touchscreen support + drivers
World's First Linux Live Usb for Microsoft Surface Pro 4 with all drivers.
 
Guide has been repurposed frmo the following two sources:
https://help.ubuntu.com/community/LiveCDCustomization
https://github.com/jakeday/linux-surface
 
This guide works best from an ubuntu-18.04.1-desktop-amd64 system, have not tested support for other systems.
 
How to (Follow every single step and do not skip anything):
 
Step 1: Preparing environment
 
mkdir linuxusb/
cd linuxusb/
wget -c http://releases.ubuntu.com/18.04/ubuntu-18.04.1-desktop-amd64.iso
sudo apt install squashfs-tools genisoimage qemu qemu-kvm
mkdir mnt
sudo mount -o loop ubuntu-18.04.1-desktop-amd64.iso mnt
mkdir extract-cd
sudo rsync --exclude=/casper/filesystem.squashfs -a mnt/ extract-cd
sudo unsquashfs mnt/casper/filesystem.squashfs
sudo mv squashfs-root edit
sudo cp /etc/resolv.conf edit/etc/
sudo mount -o bind /run/ edit/run
sudo mount --bind /dev/ edit/dev
 
 
Step 2: Chrooting and Preparing
 
sudo chroot edit #This will place you into the chroot
mount -t proc none /proc
mount -t sysfs none /sys
mount -t devpts none /dev/pts
export HOME=/root
export LC_ALL=C
nano /etc/apt/sources.list # delete all lines then paste below
    deb http://archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse
    deb http://archive.ubuntu.com/ubuntu/ bionic main restricted
    deb http://archive.ubuntu.com/ubuntu/ bionic multiverse
    deb http://archive.ubuntu.com/ubuntu/ bionic universe
    deb http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted
    deb http://archive.ubuntu.com/ubuntu/ bionic-updates multiverse
    deb http://archive.ubuntu.com/ubuntu/ bionic-updates universe
    deb http://security.ubuntu.com/ubuntu bionic-security main restricted
    deb http://security.ubuntu.com/ubuntu bionic-security multiverse
    deb http://security.ubuntu.com/ubuntu bionic-security universe
apt update
apt install git curl wget sed
git clone https://github.com/jakeday/linux-surface.git ~/linux-surface
cd ~/linux-surface
 
Step 2.5: Optional installation of packages for your new system, add whatever packages you want
apt install nmap build-essential binutils-dev libncurses5-dev libssl-dev ccache bison flex libelf-dev tmux htop python
 
Step 3: Modifying setup.sh
nano setup.sh
    Modify line 25:
    SUR_MODEL="$(dmidecode | grep "Product Name" -m 1 | xargs | sed -e 's/Product Name: //g')"
     to
    SUR_MODEL="Surface Pro 4"
 
Step 4: Installing kernel
sh setup.sh
    If given the option of 'yes/no' always type yes
 
Step 5: Cleaning up
apt clean
rm -rf /tmp/* ~/.bash_history
umount /proc || umount -lf /proc
umount /sys
umount /dev/pts
exit
 
Step 6: Copying newly created vmlinuz and initrd
sudo umount edit/dev
sudo cp edit/boot/vmlinuz-4.17.3-surface-linux-surface extract-cd/casper/vmlinuz
sudo cp edit/boot/initrd.img-4.17.3-surface-linux-surface extract-cd/casper/initrd.lz
 
Step 7: Preparing the iso
sudo chmod +w extract-cd/casper/filesystem.manifest
sudo su
chroot edit dpkg-query -W --showformat='${Package} ${Version}\n' > extract-cd/casper/filesystem.manifest
exit # exit su
sudo cp extract-cd/casper/filesystem.manifest extract-cd/casper/filesystem.manifest-desktop
sudo sed -i '/ubiquity/d' extract-cd/casper/filesystem.manifest-desktop
sudo sed -i '/casper/d' extract-cd/casper/filesystem.manifest-desktop
sudo rm extract-cd/casper/filesystem.squashfs
sudo mksquashfs edit extract-cd/casper/filesystem.squashfs
sudo su
printf $(du -sx --block-size=1 edit | cut -f1) > extract-cd/casper/filesystem.size
exit # exit su
 
Step 8: Customization of Image name
sudo nano extract-cd/README.diskdefines # modify the image name as you like
 
Step 9: Updating md5sum
cd extract-cd
sudo rm md5sum.txt
find -type f -print0 | sudo xargs -0 md5sum | grep -v isolinux/boot.cat | sudo tee md5sum.txt
 
Step 10: Creating the Iso
sudo mkisofs -D -r -V "$IMAGE_NAME" -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o ../surface-pro-4-live-usb.iso
cd ..
 
Step 11: Testing the iso in qemu
sudo kvm -cdrom surface-pro-4-live-usb.iso -boot d -m 3000
    #Once it boots open up terminal
    uname -a # check if the kernel is correct (ie: if it has the word surface in it)
    exit
 
Step 12: Writing to usb stick
#Do not use unetbootin or dd to write this image to a usb stick.
#Copy the iso onto a windows computer and use http://download.tuxfamily.org/lilicreator/stable/LinuxLive%20USB%20Creator%202.9.4.exe to create your usb.
#Once installed onto a usb stick power down your surface pro 4
#Hold the physical Volume + button and press the power button
#Do not let go of the volume button until you see the bios menu
#Disable secure boot
#Change boot device order and drag the usb option to the top
#Power down computer
#Power on computer
#Enjoy
