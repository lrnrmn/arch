# Arch Linux Guide
```bash
sudo pacman -S firefox nano git zip nvidia nvidia-utils lib32-nvidia-utils nvidia-settings
```
```bash
reboot
```
```bash
sudo nano /etc/default/grub
```
Append this line:
```bash
nvidia_drm.modeset=1
```
into the ```GRUB_CMDLINE_LINUX_DEFAULT``` section.
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
```bash
reboot
```
Check that:
```bash
sudo cat /sys/module/nvidia_drm/parameters/modeset
```
return ```Y```
```bash
sudo nano /etc/mkinitcpio.conf
```
Append this line:
```bash
nvidia nvidia_modeset nvidia_uvm nvidia_drm
```
into ```MODULES```.
```bash
sudo mkinitcpio -P
```
```bash
reboot
```
```bash
sudo mkdir /etc/pacman.d/hooks/
```
```bash
sudo nano /etc/pacman.d/hooks/nvidia.hook
```
Write this:
```bash
[Trigger]
Operation=Install
Operation=Upgrade
Operation=Remove
Type=Package
# Uncomment the installed NVIDIA package
Target=nvidia
#Target=nvidia-open
#Target=nvidia-lts
# If running a different kernel, modify below to match
Target=linux

[Action]
Description=Updating NVIDIA module in initcpio
Depends=mkinitcpio
When=PostTransaction
NeedsTargets
Exec=/bin/sh -c 'while read -r trg; do case $trg in linux*) exit 0; esac; done; /usr/bin/mkinitcpio -P'
```
```bash
sudo nano /etc/modprobe.d/nvidia.conf
```
Write this:
```bash
options nvidia NVreg_PreserveVideoMemoryAllocations=1
```
```bash
sudo mkinitcpio -P
```
```bash
sudo systemctl enable nvidia-suspend.service
```
```bash
sudo systemctl enable nvidia-hibernate.service
```
```bash
sudo systemctl enable nvidia-resume.service
```
```bash
reboot
```
```bash
sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules
```
```bash
sudo nano /etc/environment
```
Add this to the end of the file:
```bash
MUTTER_DEBUG_KMS_THREAD_TYPE=user
```
```bash
sudo nano /etc/gdm/custom.conf
```
Add ```GdmXserverTimeout=60``` to the end of the ```[daemon]``` section.
```bash
reboot
```
