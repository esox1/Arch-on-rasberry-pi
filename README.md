# Arch-on-rasberry-pi
**How to install Arch linux on the rasp. pi without a screen or a keyboard using only a secure shell client.** 

In this guide I am using a 8G Sdcard, pi3 & my archlinux latptop 

1- first thing first, you want to connect your Sdcard to your pc via a sdcard adapter or a usb.

2- Run command `lsblk` to find the parititions of your Sdcard with their assigned names. it will probably be memk. or sdx  -- 'x'  
   can be letters a or b

3- Once you know your Sdcard name, run the command `sudo fdisk -l /dev/sdb`. Where sdb is your SDcard parition name. it was sdb   
   for my case.

4- Type `o` to clear any existing partitions on the Sdcard, then `p` to list partitions, there should be no partitions left. Then               
   type `n` for new partition, then `p` for primary, `1` to assign the first partition on the drive, then press `ENTER` to accept 
   the default first sector, then type `+200M` for the last sector. We just dedicated a 200M for the first partition, which will 
   be used as the boot partition.

5- Type `t`, for partition type, then type `c` to set the first partition to type W95 FAT32(LBA).

6- Type `n` one more time, then `p` for primary, `2` to assign the second partition on the drive, then press `ENTER` twice to 
   accept the default first and last sector.

7- Write the partition table and exit by typing `w`

8- Type `lsblk`, you should have two partitions. **sdb1** with size **200M** & **sdb2** with the rest of the Sdcard size in my  
   case it was **7.4G**

9- Now, we need to create and mount a filesystem on the Sdcard partitions. type `mkfs.vfat /dev/sdb1`. This will create a FAT32 
   filesystem on our first partition (200M). We are going to have our boot files on that partition

10- Type `mkdir -p ~/Desktop/pi3/{boot,root}` to create a boot and a root subfolders in pi3 in your Desktop. Or whichever 
    directory you prefer.

11- Type `mount /dev/sdb1 ~/Desktop/pi3/boot`. Type `mkfs.ext4 /dev/sdb2` to format your second partition. Type `mount /dev/sdb2/   
~/Desktop/pi3/root/`

12- Now that we have our two partitions mounted in root and boot subfolders under pi3. lets download the image of arch linux. 
    Type `cd ~/Downloads && wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz` to download the image. Type `tar zxvf ~/Downloads/ArchLinuxARM-rpi-2-latest.tar.gz -C ~/Desktop/pi3/root/` to extract the contents of the image into your root 
     folder

13- Next we are going to move the boot folder that is in the ~/Desktop/pi3/root/ to our boot partition. Type `mv ~/Desktop/pi3/root/boot/* ~/Desktop/pi3/boot/`

14- That's it. Let's unmount the root and boot paritions. Type `umount ~/Desktop/pi3/`

15- In this step we need to connect ot our pi3 to configure arch. By the way our default username is alarm - password is alarm. The root password is root. To connect to pi3 board, use the hdmi and your usb keyboard and type root as the username and the password is root. By the way, I dont have a usb keyboard so I am going to ssh to the board and use my laptops keyboard to configure arch on pi3. To do that, first connect your pi board to your router using ethernet. Next we need to figure out the asssinged ip address by the router to the pi3. Type 'sudo nmap -sP 192.168.1.0/24' { to find your ip address type on linux type 'ifconfig' it should be next to inet}. Nmap outputs a lit of all the connected devices along with my pi3 assigned ip address

16- Type 'ssh -o StrickHostKeyChecking=no alarm@192.168.1.237' then type 'alarm'. Type ENTER to connect. We need to go root. Type 'su -l root' and Type 'root' then ENTER. Now we are root.

17- IF you are connected with ethernet, you should have an ip-address. If you want to configure the wifi.The first step to connect to an encrypted wireless network is having wpa_supplicant obtain authentication from a WPA authenticator. In order to do this, wpa_supplicant must be configured so that it will be able to submit the correct credentials to the authenticator. If you been following everything, you should be in root. Type using your favorite text editor, for me its vim, 'vim /etc/wpa_supplicant/wifi.conf' then save.


18- In your terminal, type 'wpa_passphrase MYSSID passphrase  > /etc/wpa_supplicant/example.conf' where MYSSID your ssid name and the password. This will only create a network section.

19- First start wpa_supplicant command, whose most commonly used arguments are:

    -B - Fork into background.
    -c filename - Path to configuration file.
    -i interface - Interface to listen on.
Type 'wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wifi.conf'. You should see "Succesfully initialized wpa_supplicant" on the screen. To check your interfaces type 'ip link'. To get the name of wireless interface tpye 'iw dev, the name of the interface will be output after the word "Interface". I get wlan0

20- Once we have network running (REVIEW THIS STEP)


21- Type 'pacman-key --init'to create a trustdb and generate pacman keyring master key. Type 'pacman -Syu' to download & install the latest repos of packages

22- installing sudo. Type 'pacman -S sudo'. Now to give user alarm to super privilges. Type 'visudo'. Scroll to "# User privilege specification" under root ALL=(ALL) All. Type "alarm ALL=(ALL) All. Now exit out of root by type 'exit' and try to update pacman database as user alarm. Type 'sudo pacman -Syu'. Now exit out of root by type 'exit' and try to update pacman database as user alarm. Type 'sudo pacman -Syu'. It should work  ( REVIEW this step to add user with a passwd)

23- lets set a timezone. Type 'sudo ln -S /usr/share/zoneinfo/America/New_York /etc/localtime'. Set the hardware clock. Type 'hwclock --systohc --utc'


24- lets create a new user. type 'useradd -m -g users -G audio,lp,optical,storage,video,games,power,scanner,wheel -s /bin/bash neyu'. Type 'passwd neyu' to set the passwd for new user neyu. Next, input the passwd then Press Enter then enter passwd again tap ENTER.


25- We need to edit /etc/sudoers file to allow use of the wheel group for our normal users. type 'visudo' again. Uncomment the "#%wheel ALL=(ALL) ALL" line by deleting the 'hashtag #' infront of %wheel ALL=(ALL) ALL


26- let's install a desktop manger I choose  XFCE since it's light on resources. if It wasnt for the limited memory on pi3 I would have installed GNOME. I do use window manger more than a desktop enviroment. But for the sake of this guide let's install XFCE. Type 'sudo pacman -S xfce4 xorg-server xorg-xrefresh xf86-video-fbdev  xarchiver'


27- Next lets install a login screen prompt manager. I will use sddm since its smaller in size compared to what I would use which is gdm from gnome DE. Type 'sudo pacman -S sddm'. To enable sddm. Type 'sudo systemctl enable sddm'. It will create a symbol link in /etc/systemd/system/display-manager.servie


28- Type 'sudo systemctl start sddm'. Congrats, now you should be on login screen. type youur suer password to login. Once you are logged in. Iwould install the network-manager package which will detect and configure system to automatically connect to network.Type 'sudo pacman -S network-manager-applet networkmanager'.By the way I forgot to tell you to install 'bash-completion' package which autocompletes strings in bash by tabbing the TAB key.


29- Now lets enable NetworkManger. Type 'sudo systemctl enable NetworkManager.service'. To start NetworkManger type 'sudo systemctl start NetworkManager'. We have to disable dhcpcd and sysemd-networkd since you are going to be using NetworkManger. Type 'sudo systemctl stop dhcpcd.service'. Next Type 'reboot'

30- once you are rebooted, you should see a drop down icon in the corner, that should be our NetworkManger applet, click on a new wifi connection and  type in your ssid and password. Now you should be connected to your wifi.

31- Next lets intall yaourt which is a package manger to install AUR (ARCH USER REPOSITORY) packages. Install git package to git clone the yaourt repo and package query. Type 'sudo pacman -S base-devel git && git clone https://aur.archlinux.org/package-query.git' then Type 'cd package-query && makepkg -si'. If you get a prompt to install any package press ENTER. Once it installs package-query package. Type 'cd .. && git clone https://aur.archlinux.org/yaourt.git'. Type 'cd yaourt && makepkg -si. Press ENTER when you ge tthe prompt to intall yaourt. Type 'cd ..' and You are ready to install our audio and bluetooth packages.

32- Type yaourt -S pi-bluetooth. If you get any prompts to edit a package. Tap the 'n' key on your keyboard. If you get any prompts to continue to install a package, tap the 'y' key on your keyboard. lets enable our bluetooth. Type 'sudo systemctl enable bluetooth.service' and 'sudo systemctl start bluetooth.service'

33- Next installing audio and a bdowrser (firefox is my choice).Type 'sudo pacman -S pulseaudio-alsa pulseaudio-bluetooth bluez-libs pavucontrol firefox bluez-utils bluez-firmware'. To enable audio on pi. Type 'sudo vim /boot/config.txt' and add "dtparam=audio=on"

34- I wil install some themes, Type ' yaourt gtk-theme' and 'choose 55, 58, 53, 41, 38' (gtk-theme-arc-git, gtk-engine-murrine, gtk-theme-aurora-nuevo, gtk-theme-elementary).


35- configure pacman to be a little colorful. Type 'sudo vim /etc/pacman.conf' under "#Misc options" uncomment "Color, TotalDownload, VerbosePkgLists". To something coll with the progressbar. Type 'ILoveCandy'in the file anywhere. Save and exit.

36- Finally to create all your default directories in $HOME like Documents Desktop Music. Type 'sudo pacman -S xdg-user-dirs' and 'xdg-user-dirs-update'
