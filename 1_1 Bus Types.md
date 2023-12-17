Bus Types
=========

# 1 Definition
# 1 定义
See the kerneldoc for the struct bus_type.

查看结构体`bus_type`的内核文档。

```c
int bus_register(struct bus_type *bus);
```

# 2 Declaration
# 2 声明

Each bus type in the kernel (PCI, USB, etc) should declare one static object of this type. They must initialize the name field, and may optionally initialize the `match` callback

内核中的每种总线类型（如PCI、USB等）应该声明此类型的一个静态对象。它们必须初始化`name`字段，并可以选择初始化`match`回调函数：

```c
struct bus_type pci_bus_type = {
       .name	= "pci",
       .match	= pci_bus_match,
};
```
The structure should be exported to drivers in a header file:

这个结构体应该在头文件中向驱动程序导出：

```c
extern struct bus_type pci_bus_type;
```

# 3 Registration
# 3 注册

When a bus driver is initialized, it calls `bus_register`.
This initializes the rest of the fields in the bus object and inserts it into a global list of bus types. Once the bus object is registered,
the fields in it are usable by the bus driver.

当总线驱动程序初始化时，它调用`bus_register`。这将初始化总线对象中的其余字段并将其插入总线类型的全局列表中。一旦总线对象被注册，其中的字段就可以被总线驱动程序使用。

# 4 Callbacks
# 4 回调

	match(): Attaching Drivers to Devices

The format of device ID structures and the semantics for comparing
them are inherently bus-specific. Drivers typically declare an array
of device IDs of devices they support that reside in a bus-specific
driver structure.

设备ID结构的格式和比较它们的语义在本质上是与总线相关的。驱动程序通常在总线特定的驱动程序结构中声明它们支持的设备的设备ID数组。

The purpose of the match callback is to give the bus an opportunity to
determine if a particular driver supports a particular device by
comparing the device IDs the driver supports with the device ID of a
particular device, without sacrificing bus-specific functionality or
type-safety.

`match`回调函数的目的是让总线有机会确定特定驱动程序是否支持特定设备，通过比较驱动程序支持的设备ID与特定设备的设备ID，而不会牺牲总线特定的功能或类型安全性。

When a driver is registered with the bus, the bus's list of devices is
iterated over, and the match callback is called for each device that
does not have a driver associated with it.

当驱动程序与总线注册时，总线的设备列表被迭代，对于每个没有与之关联驱动程序的设备，都会调用`match`回调函数。

# 5 Device and Driver Lists

The lists of devices and drivers are intended to replace the local
lists that many buses keep. They are lists of struct devices and
struct device_drivers, respectively. Bus drivers are free to use the
lists as they please, but conversion to the bus-specific type may be
necessary.

设备和驱动程序列表旨在取代许多总线保留的本地列表。它们分别是`struct devices`和`struct device_drivers`的列表。总线驱动程序可以根据需要自由使用这些列表，但可能需要将其转换为总线特定的类型。

The LDM core provides helper functions for iterating over each list:

LDM核心提供了用于在每个列表上进行迭代的辅助函数：

```c
int bus_for_each_dev(struct bus_type *bus, struct device *start,
                   void *data, int (*fn)(struct device *, void *));

int bus_for_each_drv(struct bus_type *bus, struct device_driver *start,
                   void *data, int (*fn)(struct device_driver *, void *));
```

These helpers iterate over the respective list, and call the callback
for each device or driver in the list. All list accesses are
synchronized by taking the bus's lock (read currently). The reference
count on each object in the list is incremented before the callback is
called; it is decremented after the next object has been obtained. The
lock is not held when calling the callback.

这些辅助函数在各自的列表上进行迭代，并为列表中的每个设备或驱动程序调用回调函数。所有列表访问都通过获取总线的锁（目前为读锁）同步进行。在调用回调函数之前，列表中每个对象的引用计数都会增加；在获取下一个对象之后，它会减少。在调用回调函数时，不会持有锁。

# 6 sysfs

There is a top-level directory named 'bus'.

Each bus gets a directory in the bus directory, along with two default

每个总线在总线目录中都有一个目录，还有两个默认目录

	/sys/bus/pci/
	|-- devices
	`-- drivers

Drivers registered with the bus get a directory in the bus's drivers

已在总线注册的驱动程序在总线的drivers目录中获得一个目录

	/sys/bus/pci/
	|-- devices
	`-- drivers
	    |-- Intel ICH
	    |-- Intel ICH Joystick
	    |-- agpgart
	    `-- e100

Each device that is discovered on a bus of that type gets a symlink in
the bus's devices directory to the device's directory in the physical hierarchy

在该类型总线上发现的每个设备都会在总线的devices目录中得到一个符号链接，指向物理层次结构中设备的目录.

	/sys/bus/pci/
	|-- devices
	|   |-- 00:00.0 -> ../../../root/pci0/00:00.0
	|   |-- 00:01.0 -> ../../../root/pci0/00:01.0
	|   `-- 00:02.0 -> ../../../root/pci0/00:02.0
	`-- drivers

# 7 Exporting Attributes

```c
struct bus_attribute {
    struct attribute attr;
    ssize_t (*show)(const struct bus_type *, char *buf);
    ssize_t (*store)(const struct bus_type *, const char *buf, size_t count);
};
```

Bus drivers can export attributes using the BUS_ATTR_RW macro that works similarly to the DEVICE_ATTR_RW macro for devices. For example, a definition like this

总线驱动程序可以使用与设备的`DEVICE_ATTR_RW`宏类似的`BUS_ATTR_RW`宏导出属性。例如，像这样的定义

```c
static BUS_ATTR_RW(debug);
```
is equivalent to declaring

等效于声明
```c
static bus_attribute bus_attr_debug;
```
This can then be used to add and remove the attribute from the bus's sysfs directory using

然后可以使用这个来添加和移除总线的sysfs目录中的属性

```c
int bus_create_file(struct bus_type *, struct bus_attribute *);
void bus_remove_file(struct bus_type *, struct bus_attribute *);
```