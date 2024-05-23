# Arch Linux Guide
## Initial packages install
```bash
sudo pacman -S firefox nano git zip nvidia nvidia-utils lib32-nvidia-utils nvidia-settings
```
Reboot to complete the driver package installation.
## DRM kernel mode setting
Edit the default ```grub``` config:
```bash
sudo nano /etc/default/grub
```
Append this line into the ```GRUB_CMDLINE_LINUX_DEFAULT``` section:
```bash
nvidia_drm.modeset=1
```
Regenerate the ```grub``` config
```bash
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
Reboot to apply the new config.


> To verify nvidia_drm.modeset=1 was correctly applied after a reboot, execute the following:
```bash
sudo cat /sys/module/nvidia_drm/parameters/modeset
```
make sure it returns ```Y```.
## Early loading
Add the ```nvidia``` modules to ```mkinitcpio``` config:
```bash
sudo nano /etc/mkinitcpio.conf
```
Paste this line:
```bash
nvidia nvidia_modeset nvidia_uvm nvidia_drm
```
into ```MODULES```.
Regenerate the initramfs:
```bash
sudo mkinitcpio -P
```
Reboot to apply changes.
## Pacman hook
To avoid the possibility of forgetting to update initramfs after an NVIDIA driver upgrade, you may want to use a pacman hook:
```bash
sudo mkdir /etc/pacman.d/hooks/
```
```bash
sudo nano /etc/pacman.d/hooks/nvidia.hook
```
Paste this:
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
# Preserve video memory after suspend
To save and restore all video memory contents, use the ```NVreg_PreserveVideoMemoryAllocations=1``` for the nvidia kernel module.


Set module options:
```bash
sudo nano /etc/modprobe.d/nvidia.conf
```
Paste this:
```bash
options nvidia NVreg_PreserveVideoMemoryAllocations=1
```
Regenerate the initramfs:
```bash
sudo mkinitcpio -P
```
# Enable ```systemctl``` ```nvidia``` related services.
```bash
sudo systemctl enable nvidia-suspend.service
```
```bash
sudo systemctl enable nvidia-hibernate.service
```
```bash
sudo systemctl enable nvidia-resume.service
```
Reboot to apply changes.
# Force-Enable Wayland
To force-enable Wayland, override these rules by creating the following symlink:
```bash
sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules
```
Set this environment variable:
```bash
sudo nano /etc/environment
```
Add this to the end of the file:
```bash
MUTTER_DEBUG_KMS_THREAD_TYPE=user
```
# Failure on logout
Edit the GDM config:
```bash
sudo nano /etc/gdm/custom.conf
```
Add ```GdmXserverTimeout=60``` to the end of the ```[daemon]``` section.


Reboot to apply changes.
