# WIN10 Virtual Machine in QEMU/KVM with Gvtg vGPU
![GVT-g_virt-manager](https://user-images.githubusercontent.com/58810777/115971485-092be780-a549-11eb-914d-6af5b5ff3fd8.png)
![Screenshot from 2021-04-21 20-48-07](https://user-images.githubusercontent.com/58810777/115971495-0df09b80-a549-11eb-9e4e-cd7bd3f3cc29.png)

Windows VM running on Qemu/KVM with cpu pinning, GVTg passthrought and hypervisor hydden from the guest.

# Host setup
Assuming the host has never been set up for running a VM in qemu/kvm.

0) sudo apt-get install qemu qemu-system-x86 qemu-kvm qemu-utils virt-manager libvirt-daemon-driver-qemu
sudo systemctl enable libvirtd
sudo systemctl enable virtlogd.socket

1) **Kernel modules**
    
    Add:
    -   kvmgt
    -   vfio-iommu-type1
    -   vfio-mdev
    
    to:
    -   /etc/modules

    Rebuild initramfs: 
    -   sudo update-initramfs -u

2) **Boot parameters**
    
    Modify GRUB_CMDLINE_LINUX line in:
    -   /etc/default/grub
    
    To match the one in "grub"

    update the bootloader: 
    -   sudo update-grub

3) **Libvirtd permissions**
    
    add kvm and libvirtd-qemu to the root group:
    -   usermod -a -G root kvm
    -   usermod -a -G root libvirtd-qemu
sudo usermod -a -G libvirt $USER
sudo usermod -a -G kvm $USER
sudo virsh net-start default
sudo virsh net-autostart default
# 配置Libvirt

**编辑libvirt.conf**<br>
`sudo nano /etc/libvirt/libvirtd.conf`

找到这两行，把前面的`#`删掉：
```
unix_sock_group = "libvirt"
unix_sock_rw_perms = "0770"
```
可以选择把这两行添加到文件的最后面（日志文件）:
```
log_filters="1:qemu"
log_outputs="1:file:/var/log/libvirt/libvirtd.log"
```
* * *
**编辑qemu.conf**<br>
`sudo nano /etc/libvirt/qemu.conf`<br>

把 `#user = "root"` 改成 `user = "yourusername"`,<br>
`#group = "root"` 改成 `group = "libvirt"`<br>

把自己添加到libvirt组里面：<br>
`sudo usermod -a -G libvirt 用户名`
* * *

**重启电脑或者重启libvirt**
```
sudo systemctl restart libvirtd.service
sudo systemctl restart virtlogd.socket
```
4) **Gvtg hook**
    
    copy the content of "qemu" in: 
    -   /etc/libvirtd/hooks/qemu
    
    make the hook executable: 
    -   sudo chmod +X /etc/libvirtd/hooks/qemu

6) **Create a virtual disk** 
    
    (40GB example):
    -   qemu-img create -f qcow2 name.qcow2 40G

7) **Define VM**
-  Create a .xml file and copy the content from "win10-virt.xml", or directly download the file instead
-   Open the xml file replace PATHTOHDD with the path to your virtual disk
-   Define VM:  
    -   sudo virsh define win10-virt.xml
-   Now the VM should be visible in Virt-manager

8) **For further performance optimization:**
    -   https://wiki.archlinux.org/index.php/Intel_GVT-g
    -   https://wiki.archlinux.org/index.php/QEMU
# Guest setup

1) First Boot, Windows install:
    -   Add a Windwos10 ISO as Cdrom and set it as the boot device (via virt-manager) 
    -   Download virtio drives for Windows https://github.com/virtio-win/virtio-win-pkg-scripts
    -   Add Virtio-driver ISO as Cdrom
    -   Boot
    -   Follow this guide: https://linuxhint.com/install_virtio_drivers_kvm_qemu_windows_vm/
