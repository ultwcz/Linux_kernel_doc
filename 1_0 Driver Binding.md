Driver Binding
============

Driver binding is the process of associating a device with a device driver that can control it. Bus drivers have typically handled this because there have been bus-specific structures to represent the devices and the drivers. With generic device and device driver structures, most of the binding can take place using common code.

驱动绑定是将设备与能够控制它的设备驱动程序关联起来的过程。通常由总线驱动程序处理这个过程，因为总线驱动程序有特定于总线的结构来表示设备和驱动程序。通过通用的设备和设备驱动程序结构，大部分绑定可以使用通用代码完成。

# 1 Bus
# 2 总线

The bus type structure contains a list of all devices that are on that bus type in the system. When device_register is called for a device, it is inserted into the end of this list. The bus object also contains a list of all drivers of that bus type. When driver_register is called for a driver, it is inserted at the end of this list. These are the two events which trigger driver binding.

总线类型结构包含系统中该总线类型上的所有设备列表。当对设备调用 `device_register` 时，它被插入到此列表的末尾。总线对象还包含该总线类型的所有驱动程序列表。当对驱动程序调用 `driver_register` 时，它被插入到此列表的末尾。这两个事件触发驱动绑定。

# 2 device_register

When a new device is added, the bus's list of drivers is iterated over to find one that supports it. In order to determine that, the device ID of the device must match one of the device IDs that the driver supports. The format and semantics for comparing IDs are bus-specific. Instead of trying to derive a complex state machine and matching algorithm, it is up to the bus driver to provide a callback to compare a device against the IDs of a driver. The bus returns 1 if a match was found; 0 otherwise.

当添加新设备时，总线的驱动程序列表被迭代，以找到支持该设备的驱动程序。为了确定这一点，设备的设备 ID 必须与驱动程序支持的设备 ID 之一匹配。用于比较 ID 的格式和语义是特定于总线的。不是尝试推导复杂的状态机和匹配算法，而是由总线驱动程序提供回调函数来比较设备与驱动程序的 ID。如果找到匹配，则返回1；否则返回0。

```c
int match(struct device * dev, struct device_driver * drv);
```

If a match is found, the device's driver field is set to the driver and the driver's probe callback is called. This gives the driver a chance to verify that it really does support the hardware, and that it's in a working state.

如果找到了匹配，设备的驱动字段将被设置为相应的驱动程序，并且会调用驱动程序的探测回调函数（probe callback）。这为驱动程序提供了验证其是否真的支持硬件并且处于工作状态的机会。

# 3 Device Class

Upon the successful completion of probe, the device is registered with the class to which it belongs. Device drivers belong to one and only one class, and that is set in the driver's devclass field. devclass_add_device is called to enumerate the device within the class and actually register it with the class, which happens with the class's register_dev callback.

在成功完成探测后，设备将被注册到其所属的类别中。设备驱动程序属于一个且仅一个类别，这在驱动程序的 `devclass` 字段中设置。调用 `devclass_add_device` 枚举类别中的设备并实际将其注册到类别中，这是通过类别的 `register_dev` 回调完成的。

# 4 Driver

When a driver is attached to a device, the device is inserted into the driver's list of devices.

当驱动程序附加到设备时，该设备被插入到驱动程序的设备列表中。

# 5 sysfs

A symlink is created in the bus's 'devices' directory that points to the device's directory in the physical hierarchy.

在总线的 'devices' 目录中创建一个符号链接，指向物理层次结构中设备的目录。

A symlink is created in the driver's 'devices' directory that points to the device's directory in the physical hierarchy.

在驱动程序的 'devices' 目录中创建一个符号链接，指向物理层次结构中设备的目录。

A directory for the device is created in the class's directory. A symlink is created in that directory that points to the device's physical location in the sysfs tree.

在类别的目录中为设备创建一个目录。在该目录中创建一个符号链接，指向 sysfs 树中设备的物理位置。

A symlink can be created (though this isn't done yet) in the device's physical directory to either its class directory, or the class's top-level directory. One can also be created to point to its driver's directory also.

可以创建一个符号链接（尽管目前还没有执行），将设备的物理目录指向其类别目录，或类别的顶级目录。还可以创建一个符号链接，指向其驱动程序的目录。

# 6 driver_register

The process is almost identical for when a new driver is added. The bus's list of devices is iterated over to find a match. Devices that already have a driver are skipped. All the devices are iterated over, to bind as many devices as possible to the driver.

当添加新驱动程序时，过程几乎与添加新设备相同。总线的设备列表被迭代，以找到匹配项。已经有驱动程序的设备将被跳过。迭代所有设备，以将尽可能多的设备绑定到驱动程序。

# 7 Removal

When a device is removed, the reference count for it will eventually go to 0. When it does, the remove callback of the driver is called. It is removed from the driver's list of devices and the reference count of the driver is decremented. All symlinks between the two are removed.

当一个设备被移除时，与其相关的引用计数最终会降至0。一旦降至0，就会调用驱动程序的移除回调函数。该设备会从驱动程序的设备列表中移除，并且减少驱动程序的引用计数。两者之间的所有符号链接也会被移除。

When a driver is removed, the list of devices that it supports is iterated over, and the driver's remove callback is called for each one. The device is removed from that list and the symlinks removed.

当一个驱动程序被移除时，会遍历它所支持的设备列表，并为每个设备调用驱动程序的移除回调函数。设备会从列表中移除，并且相关的符号链接也会被删除。
