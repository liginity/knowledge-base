# QEMU

## 使用 qemu 在终端里安装和运行虚拟机

通过 ssh 连接到没有 GUI 的服务器时，需要在终端里安装和运行 qemu 虚拟机。

在虚拟机使用 UEFI 固件和使用 BIOS 固件这两种情况下，qemu 启动它们的情况会有不同，这影响需要使用的 qemu 参数。

下面的描述基于这些条件：

- 虚拟机使用 UEFI 固件的情况。
- 这是在 debian 12 物理机上执行的操作。

### 安装软件包

``` bash
sudo apt-get install qemu-system-x86 ovmf
```

### 准备虚拟机文件

``` bash
MACHINE_NAME="one-vm"
# create an image file, format is qcow2.
qemu-img create -f qcow2 "${MACHINE_NAME}.cow" 40G
# copy the OVMF VARS file for UEFI usage (it stores UEFI variables).
# this file is from package ovmf, debian.
cp "/usr/share/OVMF/OVMF_VARS_4M.ms.fd" ./
```

### 在终端内启动安装镜像

这一步的 qemu 命令会在终端中启动 `.iso` 中的系统，进入 grub 界面。

``` bash
qemu-system-x86_64 \
    -machine q35,smm=on \
    -drive file="${MACHINE_NAME}.cow",format=qcow2 \
    -global driver=cfi.pflash01,property=secure,value=on \
    -drive if=pflash,format=raw,unit=0,file=/usr/share/OVMF/OVMF_CODE_4M.ms.fd,readonly=on \
    -drive if=pflash,format=raw,unit=1,file=OVMF_VARS_4M.ms.fd \
    -cdrom debian-12.1.0-amd64-DVD-1.iso \
    -m 2G \
    -nic none \
    -nographic -serial mon:stdio -nodefaults
```

这里的 `-nographic -serial mon:stdio -nodefaults` 是关键的 qemu 选项，它们保证虚拟机能在终端里启动和显示文字信息，而不需要使用图形界面。

其它选项：

- `-nic none`：这个选项可以禁用虚拟机的网络，避免在系统安装过程中从软件源下载和安装更新（这可能比较慢）。

### 在 grub 界面中给内核添加启动参数

因为在 qemu 命令行中选择了 serial 设备作为虚拟机系统的显示界面，所以需要给内核添加 `console=ttyS0` 参数才可以让系统使用 serial 设备作为输入输出。

这里简要描述添加内核参数的步骤。

1. 在 grub 显示的菜单中，按 `E` 按键，进入编辑模式，编辑当前的菜单选项。（按方向键移动）
2. 在有 `linux` 的一行中，在 `vmlinuz` 后添加 `console=ttyS0` 参数。
3. 按 `Ctrl-X` 使用修改后的菜单选项启动系统。

例子：下面是修改 `debian-12.1.0-amd64-DVD-1.iso` 的 grub 菜单选项的效果。
grub 的 `Install` 菜单选项内容修改前：
```
setparams 'Install'

    set background_color=black
    linux    /install.amd/vmlinuz vga=788 --- quiet
    initrd   /install.amd/initrd.gz
```
删掉 `vga=788` 参数，添加 `console=ttyS0` 参数，如下：
```
setparams 'Install'

    set background_color=black
    linux    /install.amd/vmlinuz console=ttyS0 --- quiet
    initrd   /install.amd/initrd.gz
```