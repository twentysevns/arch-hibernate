# arch-hibernate
automatic hibernate with systemd timer
1. create file in `/usr/local/bin/hibernate.sh`
```
#!/bin/bash

/usr/bin/acpi -b | /usr/bin/awk -F'[,:%]' '{print $2, $3}' | (
        read -r status capacity
        if [ "$status" = Discharging ] && [ "$capacity" -lt 7 ]; then
                /usr/bin/systemctl hibernate
        fi
)
```
2. create file in `/etc/systemd/system/battery.service`
```
[Unit]
Description=Automatic hibernate

[Service]
Type=oneshot
ExecStart=/usr/local/bin/hibernate.sh
User=root
Group=systemd-journal
```
3. create file in `/etc/systemd/system/battery.timer`
```
[Unit]
Description=Periodical checking of battery status every two minutes

[Timer]
OnBootSec=2min
OnUnitActiveSec=2min

[Install]
WantedBy=battery.service
```
