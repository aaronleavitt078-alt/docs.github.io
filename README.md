This is the documentation for my Arch project: 

1. Downloaded the .iso file from the mirror
2. Downloaded the PGP signature and sha256sums.txt files. Also downloaded some other stuff but I didn’t end up using them
3. Used the command “ CertUtil -hashfile archlinux-2026.10.01-x86_64.iso SHA256 “ to get the hash: 684ded26c63240ff4a41e8c25ee84ea6da233f557364821f13d12c2b0a9059a5 (This was from a Linux Mint forum b.c. the wiki wasn’t very helpful for verifying on Windows.) 
4. Compared to given hash: 684ded26c63240ff4a41e8c25ee84ea6da233f557364821f13d12c2b0a9059a5
5. They look the same
6. Installed gpg4win
7. Used command “gpg --auto-key-locate clear,wkd -v --locate-external-key pierre@archlinux.org “ from the wiki to download the signing key
8. Used command “gpg --verify archlinux-2026.10.01-x86_64.iso.sig archlinux-2026.10.01-x86_64.iso”
9. Said it was a good signature but also gave a warning but it’s probably fine
10. Made a new VM in Oracle Virtualbox and loaded the ISO file into it. It says the guest OS will need to be installed manually which is I think what the project is about.
11. Selected “Use EFI” when specifying virtual hardware
12. Turned on VM
13. Pressed enter on “Arch Linux Install Medium”
14. Ran cat /sys/firmware/efi/fw_platform_size to check UEFI bitness: 64
15. Ran “ip link”, which listed “lo” and “emp0s3”
16. Ran “rfkill”: nothing printed to stdout
17. Ran “iwctl”, now in [iwd]
18. Ran “Device List”. No devices listed.
19. Ok so the internet just works without me doing anything. Checked this with ping
20. Used timedatectl to check if the time is synchronized. It says it’s synchronized but it’s in UTC so I don’t know what’s up with that
21. Used fdisk -l to view disks
22. Used: fdisk /dev/sda to begin partitioning that disk, began entering commands to fdisk
23. Used command g to delete old partition tables and make a new one
24. Used command n to make partition 1 with  2048 + 1G
25. n: partition 2  2099200 + 4G
26. n: partition 3 the rest
27. Changed type with t, 1, uefi: changed to EFI system
28. … : t, 2, swap: changed from linux filesystem to swap
29. …: t, 3, L (look up all), 23 (x86-64 root): changed from linux filesystem to linux root
30. Redo all of that because I forgot to write the the changes
31. p to check changes
32. w to write changes
33. fdisk -l confirms they made
34. Used command mkfs.fat -F 32 /dev/sda1 to format the EFI system
35. Used command mkswap /dev/sda2 to format swap partition
36. Used command mkfs.ext4 /dev/sda3 to format root filesystem
37. Mounted root filesystem to mount /dev/sda3 /mnt
38. Mounted EFI filesystem with mount --mkdir /dev/sda1 /mnt/boot
39. Turned on the swap partition with swapon /dev/sda2
40. Downloaded essential boot packages with pacstrap -K /mnt base linux //if I wasn’t using a vm I probably should have installed grub and vim at this point too b.c. there were steps for configuring the network within the chroot environment that needed a text editor
41. Omitted the firmware and microcode because I’m using a VM
42. Use command genfstab -U /mnt >> /mnt/etc/fstab to make the fstab file
43. Checked the made file for errors because it said so, looks right I guess idk
44. Used arch-chroot -S /mnt to enter the system
45. Ran ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtimeAccording to the internet, this is Tulsa’s time zone
46. Ran hwclock --systohc to make /etc/adjtime file; checked and it does exist
47. Installed vim with pacman -S vim to edit files
48. Used locale-gen to generate a locale file
49. Used vim to uncomment the eng_us utf8 line
50. Used vim to make file /etc/locale.conf with contents LANG=en_US.UTF-8 I didn’t write it italicized so hopefully that’s not a problem
51. Used vim to make file /etc/hostname with contents “archVM”
52. I’m skipping the network configuration stuff because it kind of just works I think in the VM Edit: I did in fact need to do the network config stuff.
53. It said the initramfs step is usually not required so I’m gonna skip it
54. Began installing GRUB bootloader
55. Used command "pacman -S grub efibootmgr" to install those packages
56. Made directory /boot/EFI/GRUB
57. Used command "grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB" to install the boot loader
58. Skipping the secure boot stuff because it said in the warning to only use the commands if I need them and I don’t know if I need them
59. Used grub-mkconfig -o /boot/grub/grub.cfg to generate the config file. There was a warning which said os-probe wasn’t installed so non-os entries might show up in the menu
60. Ok so os-probe is for detecting other systems which I don’t want
61. Ok I think grub is all good now
62. Typed exit then reboot
63. Was able to log in as root into the system yay
64. So network stuff isn’t working
65. So it seems the network device enp0s3 was configured in the chroot environment but not in the booted environment so I am going to have to set up the network configuration stuff
66. Arch already has systemd-networkd to manage the network. I tried configuring it using the commands ln -s /usr/lib/systemd/network/80-wifi-station.network.example  /etc/systemd/network/80-wifi-station.network and ln -s /usr/lib/systemd/network/89-ethernet.network.example /etc/systemd/network/89-ethernet.network  to use the provided example files as config files which initially didn’t work but I realized I got the path to the example files wrong.
67. After restarting the network manager I can ping ip addresses now but dns doesn’t work
68. I think I need to configure systemd-resolved now
69. I enabled the daemon with: systemctl start systemd-resolved, and dns is now working
70. I am going to check if it works on reboot.
71. On reboot, none of it’s working. The links still exist though.
72. After reboot, restarting the network manager with “systemctl restart systemd-networkd” makes network connection work, and gives the network interface an ip address
73. Similarly, restarting systemd-resolved makes dns work
74. So now I just need to make them start on boot without having to do it manually
75. Used “systemctl enable  - - now systemd-resolved” and the same for systemd-networkd which should make them start up at boot
76. I think I typed the second wrong because it wasn’t working on reboot but I typed it and it made the links.
77. On re-reboot internet works now
78. Installed budgie de with “pacman -S budgie”, did all the defaults
79. Installed the gnome display manager with “pacman -S gdm”
80. “Systemctl enable gdm” enabled the display manager
81. Was able to login as root through the gui
82. Now I’m in a gui and can’t open anything
83. I tried and failed to boot into the command line but then I saw a button in the gui for starting with the budgie de that I had installed
84. Upon opening the de it threw an error: said it was trying to read the config file, then just repeated “Invalid Action: Invalid argument for action INVALID: ‘command’”
85. Ok I went into the grub boot config parameters and appended “single” to the linux line and I’m back in the command line
86. Disabled gdm, boots back into command line now
87. Gonna try using gnome as the de and hope it works better with gdm
88. pacman -S gnome
89. Ran “systemctl enable –now gdm” to return to the gnome display manager
90. Set it to log into the gnome de and it works fine, I have a terminal yay
91. Ran “pacman -S man-db man-pages” to have man
92. Ran “useradd -mk /etc/skel Aaron” to make my user account
93. Used passwd to set a password
94. Used “pacman -S sudo” to install the sudo package
95. Used “usermod -aG wheel Aaron” to give myself root privileges
96. Rebooted and tried using sudo as Aaron and it said I wasn’t in the sudoers file
97. So I need to edit the sudoers file but visudo says there is no editor
98. Ran “export SUDO_EDITOR=’usr/bin/vim’” so it uses vim instead of vi which doesn’t exist
99. Looking at the sudoers file, it seems it might preserve the environment variable?
100. It does not
101. Added line “Defaults editor=/usr/bin/vim” which does work now.
102. Uncommented line 126 in file opened by visudo which gives members of the wheel group superuser privileges
103. I have sudo now
104. Repeated steps 93, 94, 96 but with Codi
105. Ran “sudo pacman -S openssh” to install the ssh packages. They aren’t enabled but I don’t plan on using so
106. Added alias with “alias lah=’ls -lah’”
107. Added “alias ping=’ping -c 10
108. Added “alias memshow=’free -mlt’”
109. Had to add them to the .bashrc file in home to make them permanent
110. The terminal is already color coded so I don’t know if I need to do anything…
111. Ran “sudo pacman -S firefox” to install the web browser.

