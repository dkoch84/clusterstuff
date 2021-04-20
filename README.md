## Provision sdcards
1. `dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=32`
2. `fdisk /dev/mmcblk0` -> o, p, n, p, 1, 32768, enter, w
3. `mkfs.ext4 /dev/mmcblk0p1`
4. `mount /dev/mmcblk0p1 root/`
5. `bsdtar -xpf ArchLinuxARM-aarch64-latest.tar.gz -C root`
6. `wget http://os.archlinuxarm.org/os/rockchip/boot/rock64/boot.scr -O root/boot/boot.scr`
7. `umount root`
8. `dd if=rksd_loader.img of=/dev/mmcblk0 seek=64 conv=notrunc`
9. `dd if=u-boot.itb of=/dev/mmcblk0 seek=16384 conv=notrunc`

## Insert, boot and install minimal requirements for ansible and emergency needs.

pass: alarm

``` bash
su root
pacman-key --init
pacman-key --populate archlinuxarm
rm /boot/boot.scr
pacman -Sy uboot-rock64 sudo rxvt-unicode-terminfo python
```
Do wheel stuff
```
# still as root
visudo /etc/sudoers

# uncomment this line
%wheel ALL=(ALL) ALL   
``` 
then `usermod -aG wheel alarm`. You should be able to use sudo now as 'alarm' user.      

Set a sane hostname
`sudo echo 'kubelet-01' > /etc/hostname`

Make MAC addresses unique
```bash
vi /etc/systemd/network/00-default.link
```
Match the default MAC showing up on the network, supply a "unique" one instead.
```ini
[Match]
MACAddress=da:19:c8:7a:6d:f4 

[Link]
MACAddress=da:19:c8:7a:6d:f5 
NamePolicy=kernel database onboard slot path
```

## Ansible

install [yay](https://galaxy.ansible.com/jonsible/yay) on the ansible host

```
ansible-galaxy install jonsible.yay
```
`vim ~/.ansible/roles/jonsible.yay/tasks/main.yml`

```
#change line 38 to:
state: present
```

Configure ansible hosts
```
# /etc/ansible/hosts

[control]                                
localhost ansible_connection=local

[admin]
kubeadm ansible_user=alarm

[worker]
kubelet-[01:03] ansible_user=alarm
```

Apply some playbooks...