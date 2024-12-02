# arch-hibernate
Automatic hibernating when lowbatt with systemd service & timer, swapfile also.

0. Switch to root
```
sudo su
````
1. Create swap, size must be higher than your memories (RAM)
```
fallocate -l 8G /swap
```
2. Set secure permission
```
chmod 600 /swap
```
3. Create & Activated
```
mkswap /swap ; swapon /swap
```
4. Add `/etc/fstab` line
```
# swap in /swap
/swap          none  swap  defaults     0 0
```

5. Type `filefrag -v /swap | awk '$1=="0:" {print substr($4, 1, length($4)-2)}'`
```
6391808
```
6. Put the result to `/boot/loader/entries/[your_configuration_file].conf` in `options` line with `resume_offset=`, for example:
```
title	Arch
linux	/vmlinuz-linux
initrd	/amd-ucode.img
initrd	/initramfs-linux.img
options	root=UUID=f6b173de-c4ce-4ef4-8fa0-87d788dcd991 rw resume_offset=6391808
```
7. Create file in `/usr/local/bin/hibernate.sh`
```
#!/bin/bash

/usr/bin/acpi -b | /usr/bin/awk -F'[,:%]' '{print $2, $3}' | (
        read -r status capacity
        if [ "$status" = Discharging ] && [ "$capacity" -lt 10 ]; then
                /usr/bin/systemctl hibernate
        fi
)
```
8. Create file in `/etc/systemd/system/battery.service`
```
[Unit]
Description=Automatic hibernate

[Service]
Type=oneshot
ExecStart=/usr/local/bin/hibernate.sh
User=root
Group=systemd-journal
```
9. Create file in `/etc/systemd/system/battery.timer`
```
[Unit]
Description=Periodical checking of battery status every two minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=2min

[Install]
WantedBy=battery.service
```
10. Add hook `resume` in `/etc/mkinitcpio.conf`
```
HOOKS=(base systemd resume autodetect modconf block filesystems keyboard fsck)
```
11. Then type in terminal
```
mkinitcpio -P
```
12. Give executable permission
```
chmod +x /usr/local/bin/hibernate.sh
```
13. Finally, enable the service.
```
systemctl enable --now battery.{service,timer}
```
