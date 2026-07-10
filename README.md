# openSUSE Kernel Compiled with Apple T2 Support
[![build result](https://build.opensuse.org/projects/home:zruoyou/packages/kernel-source-t2/badge.svg?type=default)](https://build.opensuse.org/package/show/home:zruoyou/kernel-source-t2)
[![Build T2 openSUSE Kernel](https://github.com/ruoyouz/opensuse-kernel-t2/actions/workflows/build-kernel.yml/badge.svg?event=schedule)](https://github.com/ruoyouz/opensuse-kernel-t2/actions/workflows/build-kernel.yml)

__NOT TESTED (yet)__

This repo is only used for build triggering and release archiving. The actual building process is hosted on [Open Build Service](https://build.opensuse.org/package/show/home:zruoyou/kernel-source-t2).

T2 patches are pull from [t2linux/linux-t2-patches](https://github.com/t2linux/linux-t2-patches) developed by the [t2linux project](https://t2linux.org).

## [Post-install](https://wiki.t2linux.org/guides/postinstall/) ##
0. Use [script](https://wiki.t2linux.org/tools/firmware.sh) to export the wifi driver to Linux
1. Add `intel_iommu=on iommu=pt pm_async=off` to `/etc/kernel/cmdline`
2. `echo apple-bce | sudo tee /etc/modules-load.d/t2.conf`
3. `echo "force_drivers+=\" apple-bce \"" | sudo tee /etc/dracut.conf.d/t2linux-modules.conf`
4. `sudo dracut --force`
5. (Optional) Install [tiny-dfr](https://github.com/AsahiLinux/tiny-dfr) for touch-bar and [t2fanrd](https://github.com/GnomedDev/T2FanRD) for fan control.

## Common issues ##
### Fix NM notification ###
```
cat <<EOF | sudo tee /etc/udev/rules.d/99-network-t2-ncm.rules
SUBSYSTEM=="net", ACTION=="add", ATTR{address}=="ac:de:48:00:11:22", NAME="t2_ncm"
EOF
```
```
cat <<EOF | sudo tee /etc/NetworkManager/conf.d/99-network-t2-ncm.conf
[main]
no-auto-default=t2_ncm
EOF
```
### Fix suspend ###
Create and **enable** `/etc/systemd/system/suspend-fix-t2.service`:
```
[Unit]
Description=Disable and Re-Enable Apple BCE Module (and Wi-Fi)
Before=sleep.target
StopWhenUnneeded=yes

[Service]
User=root
Type=oneshot
RemainAfterExit=yes

#ExecStart=/usr/bin/modprobe -r brcmfmac_wcc
#ExecStart=/usr/bin/modprobe -r brcmfmac
ExecStart=/usr/bin/rmmod -f apple-bce

ExecStop=/usr/bin/modprobe apple-bce
#ExecStop=/usr/bin/modprobe brcmfmac
#ExecStop=/usr/bin/modprobe brcmfmac_wcc

[Install]
WantedBy=sleep.target
```
