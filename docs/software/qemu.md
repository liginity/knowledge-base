# QEMU

## 使用 qemu 在终端里安装和运行虚拟机

通过 ssh 连接到没有 GUI 的服务器时，需要在终端里安装和运行 qemu 虚拟机。

在虚拟机使用 UEFI 固件和使用 BIOS 固件这两种情况下，qemu 启动它们的情况会有不同，这影响需要使用的 qemu 参数。

下面描述使用 UEFI 固件的情况。

这是在 debian 12 物理机上执行的操作。

### 安装软件包

``` bash
sudo apt-get install qemu-system-x86 ovmf
```

### 准备虚拟机文件

``` bash
MACHINE_NAME="one-vm"
# create an image file, format is qcow2.
qemu-img create -f qcow2 ${MACHINE_NAME}.cow 40G
# copy the OVMF VARS file for UEFI usage (it stores UEFI variables).
# this file is from package ovmf, debian.
cp "/usr/share/OVMF/OVMF_VARS_4M.ms.fd" ./
```

### 在终端内启动安装镜像

``` bash
qemu-system-x86_64 \
    -machine q35,smm=on \
    -drive file=${MACHINE_NAME}.cow,format=qcow2 \
    -global driver=cfi.pflash01,property=secure,value=on \
    -drive if=pflash,format=raw,unit=0,file=/usr/share/OVMF/OVMF_CODE_4M.ms.fd,readonly=on \
    -drive if=pflash,format=raw,unit=1,file=OVMF_VARS_4M.ms.fd \
    -cdrom debian-12.1.0-amd64-DVD-1.iso \
    -m 2G \
    -nographic -serial mon:stdio -nodefaults
```

这里的 `-nographic -serial mon:stdio -nodefaults` 是关键的 qemu 选项。