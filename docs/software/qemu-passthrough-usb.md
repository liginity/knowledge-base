## 给 qemu 虚拟机直通 usb 设备

给虚拟机直通宿主机的 usb 设备，可以让虚拟机直接访问 U 盘等物理设备。

下面的步骤描述了给虚拟机直通一个 U 盘的过程。

1. 获得访问 usb 设备的权限
    1. 把 usb 设备的所有者设置为当前用户
    2. 用 root 权限运行 qemu
2. 设置合适的 qemu 参数并启动虚拟机

对于步骤 1，下面选择第一种方法：找到 usb 设备的路径，并把文件的所有者设置为当前用户。

### 获得访问 usb 设备的权限

#### 找到 U 盘的设备路径

在 Linux 上插入一个 U 盘时，一般容易看到一个新的块设备，比如 `/dev/sda` 或者 `/dev/sdb`。但是这并不是 usb 设备本身。

通过 `lsusb` 命令，可以看到系统上的 usb 设备的信息。
```
$ lsusb
Bus 007 Device 003: ID 17ef:608c Lenovo USB Keyboard
Bus 001 Device 004: ID 17ef:608d Lenovo Optical Mouse
Bus 001 Device 005: ID 0930:6544 Toshiba Corp. TransMemory-Mini XXX 2.0 Stick
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```
由此可知，U 盘在 bus 001，device 005 的位置。
可以在 `/dev/bus/usb/` 目录中找到这个设备，就是 `/dev/bus/usb/001/005`。通过 `udevadm info /dev/bus/usb/001/005` 可以确认这个路径指代的设备是这个 U 盘。
另外，在 ID `0930:6544` 中，`0930` 是 U 盘的 vendor id，`6544` 是 product id。

#### 设置 U 盘设备的所有者

```
sudo chown userAAA /dev/bus/usb/001/005
```

### 设置 qemu 参数

为了直通前述 U 盘设备，在 qemu 命令中添加这些参数
```
-usb -device usb-host,vendorid=0x0930,productid=0x6544
```
- `-usb`：为虚拟机开启 USB 模拟功能。
- `vendorid=0x0930,productid=0x6544`：上面找到了 U 盘的 vendor id 和 product id，依次填入。

### 更多信息

这里探索 `/dev/sda` 和 usb 设备的关系。
通过 `udevadm info /dev/sda` 可以看到设备的信息。这里的 `/dev/sda` 是一个 U 盘。
```
$ udevadm info /dev/sda
P: /devices/pci0000:00/0000:00:08.1/0000:65:00.3/usb1/1-1/1-1:1.0/host0/target0:0:0/0:0:0:0/block/sda
M: sda
U: block
T: disk
D: b 8:0
N: sda
L: 0
S: disk/by-id/usb-TOSHIBA_TransMemory_8BCAA96A7EEBCE80907A4D62-0:0
S: disk/by-path/pci-0000:65:00.3-usb-0:1:1.0-scsi-0:0:0:0
S: disk/by-diskseq/21
Q: 21
E: DEVPATH=/devices/pci0000:00/0000:00:08.1/0000:65:00.3/usb1/1-1/1-1:1.0/host0/target0:0:0/0:0:0:0/block/sda
E: DEVNAME=/dev/sda
E: DEVTYPE=disk
E: DISKSEQ=21
E: MAJOR=8
E: MINOR=0
E: SUBSYSTEM=block
......
```
从设备的 path `/devices/pci0000:00/0000:00:08.1/0000:65:00.3/usb1/1-1/1-1:1.0/host0/target0:0:0/0:0:0:0/block/sda` 可以看到，它与 `usb1/1-1` 有关系。