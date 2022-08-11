# arch-hibernate
Automatic hibernate with systemd service & timer, with swapfile

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
5. Create file in `/usr/local/bin/hibernate.sh`
```
#!/bin/bash

/usr/bin/acpi -b | /usr/bin/awk -F'[,:%]' '{print $2, $3}' | (
        read -r status capacity
        if [ "$status" = Discharging ] && [ "$capacity" -lt 7 ]; then
                /usr/bin/systemctl hibernate
        fi
)
```
6. Create file in `/etc/systemd/system/battery.service`
```
[Unit]
Description=Automatic hibernate

[Service]
Type=oneshot
ExecStart=/usr/local/bin/hibernate.sh
User=root
Group=systemd-journal
```
7. Create file in `/etc/systemd/system/battery.timer`
```
[Unit]
Description=Periodical checking of battery status every two minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=2min

[Install]
WantedBy=battery.service
```
8. Add hook `resume` before `autodetect`
```
HOOKS=(base systemd resume autodetect modconf block filesystems keyboard fsck)
```
9. Then type in terminal
```
mkinitcpio -P
```
11. Last, type this too.
```
systemctl enable --now battery.{service,timer}
```
