# linux-surface-ubuntu<br />
How to create your own live usb for surface pro 4 with touchscreen support + drivers<br />
 <br />
Guide has been repurposed from the following two sources:<br />
https://help.ubuntu.com/community/LiveCDCustomization<br />
https://github.com/jakeday/linux-surface<br />
 <br />
This guide works best from an ubuntu-18.04.1-desktop-amd64 system, have not tested support for other systems.<br />
 <br />
How to (Follow every single step and do not skip anything):<br />
 <br />
Step 1: Preparing environment<br />
 <br />
mkdir linuxusb/<br />
cd linuxusb/<br />
wget -c http://releases.ubuntu.com/18.04/ubuntu-18.04.1-desktop-amd64.iso<br />
sudo apt install squashfs-tools genisoimage qemu qemu-kvm<br />
mkdir mnt<br />
sudo mount -o loop ubuntu-18.04.1-desktop-amd64.iso mnt<br />
mkdir extract-cd<br />
sudo rsync --exclude=/casper/filesystem.squashfs -a mnt/ extract-cd<br />
sudo unsquashfs mnt/casper/filesystem.squashfs<br />
sudo mv squashfs-root edit<br />
sudo cp /etc/resolv.conf edit/etc/<br />
sudo mount -o bind /run/ edit/run<br />
sudo mount --bind /dev/ edit/dev<br />
 <br />
 <br />
Step 2: Chrooting and Preparing<br />
 <br />
sudo chroot edit #This will place you into the chroot<br />
mount -t proc none /proc<br />
mount -t sysfs none /sys<br />
mount -t devpts none /dev/pts<br />
export HOME=/root<br />
export LC_ALL=C<br />
nano /etc/apt/sources.list # delete all lines then paste below<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://archive.ubuntu.com/ubuntu/ bionic-backports main restricted universe multiverse<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://archive.ubuntu.com/ubuntu/ bionic main restricted<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://archive.ubuntu.com/ubuntu/ bionic multiverse<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://archive.ubuntu.com/ubuntu/ bionic universe<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://archive.ubuntu.com/ubuntu/ bionic-updates main restricted<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://archive.ubuntu.com/ubuntu/ bionic-updates multiverse<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://archive.ubuntu.com/ubuntu/ bionic-updates universe<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://security.ubuntu.com/ubuntu bionic-security main restricted<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://security.ubuntu.com/ubuntu bionic-security multiverse<br />
&nbsp;&nbsp;&nbsp;&nbsp;deb http://security.ubuntu.com/ubuntu bionic-security universe<br />
apt update<br />
apt install git curl wget sed<br />
git clone https://github.com/jakeday/linux-surface.git ~/linux-surface<br />
cd ~/linux-surface<br />
 <br />
Step 2.5: Optional installation of packages for your new system, add whatever packages you want<br />
apt install nmap build-essential binutils-dev libncurses5-dev libssl-dev ccache bison flex libelf-dev tmux htop python<br />
Install katoolin, and choose whatever packages you like if you want a pentesting focused usb<br />
 <br />
Step 3: Modifying setup.sh<br />
nano setup.sh<br />
&nbsp;&nbsp;&nbsp;&nbsp;Modify line 25:<br />
&nbsp;&nbsp;&nbsp;&nbsp;SUR_MODEL="$(dmidecode | grep "Product Name" -m 1 | xargs | sed -e 's/Product Name: //g')"<br />
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;to<br />
&nbsp;&nbsp;&nbsp;&nbsp;SUR_MODEL="Surface Pro 4"<br />
 <br />
Step 4: Installing kernel<br />
sh setup.sh<br />
&nbsp;&nbsp;&nbsp;&nbsp;If given the option of 'yes/no' always type yes<br />
 <br />
Step 5: Cleaning up<br />
apt clean<br />
rm -rf /tmp/* ~/.bash_history<br />
umount /proc || umount -lf /proc<br />
umount /sys<br />
umount /dev/pts<br />
exit<br />
 <br />
Step 6: Copying newly created vmlinuz and initrd<br />
sudo umount edit/dev<br />
sudo cp edit/boot/vmlinuz-4.17.3-surface-linux-surface extract-cd/casper/vmlinuz<br />
sudo cp edit/boot/initrd.img-4.17.3-surface-linux-surface extract-cd/casper/initrd.lz<br />
 <br />
Step 7: Preparing the iso<br />
sudo chmod +w extract-cd/casper/filesystem.manifest<br />
sudo su<br />
chroot edit dpkg-query -W --showformat='${Package} ${Version}\n' > extract-cd/casper/filesystem.manifest<br />
exit # exit su<br />
sudo cp extract-cd/casper/filesystem.manifest extract-cd/casper/filesystem.manifest-desktop<br />
sudo sed -i '/ubiquity/d' extract-cd/casper/filesystem.manifest-desktop<br />
sudo sed -i '/casper/d' extract-cd/casper/filesystem.manifest-desktop<br />
sudo rm extract-cd/casper/filesystem.squashfs<br />
sudo mksquashfs edit extract-cd/casper/filesystem.squashfs<br />
sudo su<br />
printf $(du -sx --block-size=1 edit | cut -f1) > extract-cd/casper/filesystem.size<br />
exit # exit su<br />
 <br />
Step 8: Customization of Image name<br />
sudo nano extract-cd/README.diskdefines # modify the image name as you like<br />
 <br />
Step 9: Updating md5sum<br />
cd extract-cd<br />
sudo rm md5sum.txt<br />
find -type f -print0 | sudo xargs -0 md5sum | grep -v isolinux/boot.cat | sudo tee md5sum.txt<br />
 <br />
Step 10: Creating the Iso<br />
sudo mkisofs -D -r -V "$IMAGE_NAME" -cache-inodes -J -l -b isolinux/isolinux.bin -c isolinux/boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table -o ../surface-pro-4-live-usb.iso<br />
cd ..<br />
 <br />
Step 11: Testing the iso in qemu<br />
sudo kvm -cdrom surface-pro-4-live-usb.iso -boot d -m 3000<br />
&nbsp;&nbsp;&nbsp;&nbsp;#Once it boots open up terminal<br />
&nbsp;&nbsp;&nbsp;&nbsp;uname -a # check if the kernel is correct (ie: if it has the word surface in it)<br />
&nbsp;&nbsp;&nbsp;&nbsp;exit<br />
 <br />
Step 12: Writing to usb stick<br />
#Do not use unetbootin or dd to write this image to a usb stick.<br />
#Copy the iso onto a windows computer and use http://download.tuxfamily.org/lilicreator/stable/LinuxLive%20USB%20Creator%202.9.4.exe to create your usb.<br />
#Once installed onto a usb stick power down your surface pro 4<br />
#Hold the physical Volume + button and press the power button<br />
#Do not let go of the volume button until you see the bios menu<br />
#Disable secure boot<br />
#Change boot device order and drag the usb option to the top<br />
#Power down computer<br />
#Power on computer<br />
#Enjoy<br />
