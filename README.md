# 在我的路由器里安装ipxe用来启动网络winpe系统
## 路由器是k3，系统为openwrt

## 使用[ipxe](https://ipxe.org)方案编译，配置。
## 说明：
undionly.kpxe与ipxe.efi都是新编译的。两种都是加载boot.ipxe，再由boot.ipxe判断是传统启动，还是efi启动。再加载menu.ipxe或uefi.ipxe

路由器插上U盘。分区fdisk /dev/sda。格式化 mkfs.ext4 /dev/sda1

我用的是ext4文件系统。其他文件系统也可以。

挂载mount /dev/sda1 /mnt/sda1

通常openwrt可以自动挂载

将ipxe文件夹拷贝到/mnt/sda1/目录下。

开启samba共享，更容易使用。

编辑模板，注释 invalid users = root

添加用户：
```
smbpasswd -a root
```
重启samba服务：
```
service samba4 restart
```

### 路由器dnsmasq的config不能自定义。曲线救国。

vim /usr/share/dnsmasq/dhcpbogushostname.conf
### 粘贴以下内容：
```
dhcp-match=set:bios,option:client-arch,0
dhcp-match=set:ipxe,175
enable-tftp
tftp-root=/mnt/sda1/ipxe/
dhcp-boot=tag:!ipxe,tag:bios,undionly.kpxe
dhcp-boot=tag:!ipxe,tag:!bios,ipxe.efi
dhcp-boot=tag:ipxe,ipxe.efi
```

我用的PE是 [灰大师](http://www.sadchina.com/) 
====================================================================================


====================================================================================
## 以下为其他参考配置。
```
dnsmasq版本2.76以上
dnsmasq.conf配置

enable-tftp
tftp-lowercase
dhcp-no-override
tftp-root=/mnt/sda1/pxeboot

dhcp-match=set:iPXE,175

dhcp-vendorclass=set:flag,PXEClient:Arch:00000
dhcp-vendorclass=set:flag,PXEClient:Arch:00006
dhcp-vendorclass=set:flag,PXEClient:Arch:00007
dhcp-vendorclass=set:flag,PXEClient:Arch:00009

tag-if=set:load,tag:!iPXE,tag:flag

pxe-prompt="Press F8 or Enter key for PXE menu.", 5
#BIOS MENU
pxe-service=tag:load,X86PC, "BIOS ipxe undionly", undionly.kpxe
pxe-service=tag:load,X86PC, "BIOS ipxe.pxe", ipxe.pxe
pxe-service=tag:load,X86PC, "BIOS Microsoft PXE", pxeboot.n12
pxe-service=tag:load,X86PC, "boot from local", 0
#UEFI MENU
pxe-service=tag:load,IA32_EFI, "Microsoft UEFI (IA32_EFI)", bootia32.efi
pxe-service=tag:load,X86-64_EFI, "Microsoft UEFI (X86-64_EFI)", bootx64.efi
pxe-service=tag:load,BC_EFI, "Microsoft UEFI(BC-EFI)", bootx64.efi
pxe-service=tag:load,6, "iPXE snponly UEFI32(6)", snponly32.efi
pxe-service=tag:load,7, "iPXE snponly UEFI(7)", snponly.efi
pxe-service=tag:load,9, "iPXE snponly UEFI(9)", snponly.efi
pxe-service=tag:load,06,  "iPXE UEFI32(06)", ipxe32.efi
pxe-service=tag:load,07,  "iPXE UEFI(07)", ipxe.efi
pxe-service=tag:load,09,  "iPXE UEFI(09)", ipxe.efi

dhcp-boot=tag:iPXE,ipxemenu.txt
```