# Convert to a read-only file system 

The SD Card is definitely the weak part of a Raspberry Pi. Even with a lighweight Operating System files are constantly written to flash and this finally leads to bad blocks on the long run.

To avoid wearing out SD card containing the OS, is it preferable to convert a clean install to a read-only file system.

The following guide provides some relevant commands to 
- disable swap memory
- replace syslog manager (option)
- mount some partitions in RAM (e.g. /var/log)
- check and disable some unused services

> **Reference :** 
> - https://www.dzombak.com/blog/2021/11/Reducing-SD-Card-Wear-on-a-Raspberry-Pi-or-Armbian-Device.html#if-the-system-cant-use-a-read-only-filesystem
> - https://www.paulcourt.co.uk/post/pi-swap.html

## Check used memory especially for swap

Verify with `free -m` the size of the swap

```
pi@domopi:~ $ free -m
               total        used        free      shared  buff/cache   available
Mem:             922          42         800           0          79         830
Swap:             99           0          99
```

## Remove and deactivate swap file

To permanently disable swap use the following commands :

```
sudo dphys-swapfile swapoff
sudo dphys-swapfile uninstall
sudo update-rc.d dphys-swapfile disable
```
```
sudo service dphys-swapfile stop
sudo systemctl disable dphys-swapfile.service
```

To verify this change, reboot the Pi and run `free -m`, which should show all 0s in the swap row

## Replace syslog manager (**optional**)

This step is optional, disabling journaling on the filesystem could bring difficulties to recover a system in case of crash or power outage.

```
#sudo apt-get install busybox-syslogd; dpkg --purge rsyslog
```

## Mount some partitions in RAM instead of SD

We want the following partitions to be stored in RAM using tmpfs, not on the SD card:
`/tmp`, `/var/tmp` and `/var/log`

Run mount first, along with `cat /ets/fstab`, to check which folders are already in a tmpfs.

```
sudo mount -l | grep tmpfs
```

Edit `/etc/fstab` and add the following lines

```
tmpfs /tmp tmpfs defaults,noatime,nosuid,size=100m 0 0
tmpfs /var/tmp tmpfs defaults,noatime,nosuid,size=30m 0 0
tmpfs /var/log tmpfs defaults,noatime,nosuid,mode=0755,size=100m 0 0
```

Additionnaly `logrotate` can also be moved to tmpfs, it stores some state /var/lib/logrotate 

```
tmpfs  /var/lib/logrotate  tmpfs  defaults,noatime,nosuid,nodev,noexec,size=1m,mode=0755  0  0
```

If you make a new entry in fstab it will not be applied automatically. Therefore you must reload / refresh the entries. A reboot will do this but that is not a friendly way to do it. A quick way to reload new entries in /etc/fstab (fstab) is to use the mount command 

```
sudo mount -av
```

## Check SD card usage

Find the biggest offenders in /var/log 

```
sudo du -hs /var/log/* | sort -rh | head -n 5
```

The following command will print all files under the /var directory, whose modifications are less than 60 minutes old.

```
sudo find /var -mmin -60
```
```
/var/tmp
/var/lib/apt/daily_lock
/var/lib/systemd/timesync/clock
/var/lib/systemd/timers/stamp-e2scrub_all.timer
/var/lib/systemd/timers/stamp-logrotate.timer
/var/lib/systemd/timers/stamp-man-db.timer
/var/lib/systemd/timers/stamp-apt-daily-upgrade.timer
/var/lib/logrotate
/var/lib/logrotate/status
...
```

## Check running services 

First list the services running on the system and managed by **Systemd** :

```
sudo systemctl --type=service --state=running
  UNIT                      LOAD   ACTIVE SUB     DESCRIPTION
  avahi-daemon.service      loaded active running Avahi mDNS/DNS-SD Stack
  bluetooth.service         loaded active running Bluetooth service
  cron.service              loaded active running Regular background program processing daemon
  dbus.service              loaded active running D-Bus System Message Bus
  dhcpcd.service            loaded active running DHCP Client Daemon
  getty@tty1.service        loaded active running Getty on tty1
  hciuart.service           loaded active running Configure Bluetooth Modems connected by UART
  rng-tools-debian.service  loaded active running LSB: rng-tools (Debian variant)
  rsyslog.service           loaded active running System Logging Service
  ssh.service               loaded active running OpenBSD Secure Shell server
  systemd-journald.service  loaded active running Journal Service
  systemd-logind.service    loaded active running User Login Management
  systemd-timesyncd.service loaded active running Network Time Synchronization
  systemd-udevd.service     loaded active running Rule-based Manager for Device Events and Files
  triggerhappy.service      loaded active running triggerhappy global hotkey daemon
  user@1000.service         loaded active running User Manager for UID 1000
  wpa_supplicant.service    loaded active running WPA supplicant
```
Then disable running services not needed via `sudo systemctl disable name-of-service.service`


## Mask some dailyservices 

Some daily tasks/jobs can be also masked through `systemctl`

- Daily `man` database regeneration:
```
sudo systemctl mask man-db.timer
```

- Daily `apt` update
```
sudo systemctl mask apt-daily.timer
sudo systemctl mask apt-daily-upgrade.timer
```

> :memo: **Note:**
> A masked service cannot be started by any means, and must be unmasked to be usable.

## Cleanup unused software

Uninstall some other items that all or most read-only Raspberry Pi guides recommend removing (unless you’re using any of this software):

```
sudo apt remove --purge wolfram-engine triggerhappy xserver-common lightdm && sudo apt autoremove --purge
```