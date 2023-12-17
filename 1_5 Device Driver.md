Device Drivers
==============

See the kerneldoc for the struct device_driver.

请参阅kerneldoc中的struct device_driver

## 1.Allocation

Device drivers are statically allocated structures. Though there may
be multiple devices in a system that a driver supports, struct
device_driver represents the driver as a whole (not a particular
device instance).

设备驱动程序是静态分配的结构。尽管系统中可能有多个驱动程序支持的设备，但struct device_driver代表驱动程序作为整体（而不是特定的设备实例）。

## 2.Initialization

The driver must initialize at least the name and bus fields. It should
also initialize the devclass field (when it arrives), so it may obtain
the proper linkage internally. It should also initialize as many of
the callbacks as possible, though each is optional.

驱动程序必须至少初始化name和bus字段。它还应该初始化devclass字段（当它到达时），以便它可以在内部获取正确的链接。它还应该尽可能初始化尽可能多的回调，尽管每个回调都是可选的。

## 3.Declaration

As stated above, struct device_driver objects are statically
allocated. Below is an example declaration of the eepro100
driver. This declaration is hypothetical only; it relies on the driver
being converted completely to the new model:

如上所述，struct device_driver对象是静态分配的。以下是eepro100驱动程序的示例声明。此声明仅为假设，它依赖于驱动程序完全转换为新模型：

```c
  static struct device_driver eepro100_driver = {
         .name		= "eepro100",
         .bus		= &pci_bus_type,

         .probe		= eepro100_probe,
         .remove		= eepro100_remove,
         .suspend		= eepro100_suspend,
         .resume		= eepro100_resume,
  };
```

Most drivers will not be able to be converted completely to the new
model because the bus they belong to has a bus-specific structure with
bus-specific fields that cannot be generalized.

大多数驱动程序将无法完全转换为新模型，因为它们所属的总线具有具有特定于总线的结构和字段的总线特定结构，无法泛化。

The most common example of this are device ID structures. A driver
typically defines an array of device IDs that it supports. The format
of these structures and the semantics for comparing device IDs are
completely bus-specific. Defining them as bus-specific entities would
sacrifice type-safety, so we keep bus-specific structures around.

其中一个常见的例子是设备ID结构。驱动程序通常定义其支持的设备ID数组。这些结构的格式以及比较设备ID的语义完全取决于总线。将它们定义为总线特定的实体将牺牲类型安全性，因此我们保留了总线特定的结构。

Bus-specific drivers should include a generic struct device_driver in
the definition of the bus-specific driver. Like this:

总线特定的驱动程序应在总线特定驱动程序的定义中包含一个通用的struct device_driver。就像这样：

```c
  struct pci_driver {
         const struct pci_device_id *id_table;
         struct device_driver	  driver;
  };
```

A definition that included bus-specific fields would look like
(using the eepro100 driver again):

包含总线特定字段的定义将如下所示（再次使用eepro100驱动程序）：

```c
  static struct pci_driver eepro100_driver = {
         .id_table       = eepro100_pci_tbl,
         .driver	       = {
		.name		= "eepro100",
		.bus		= &pci_bus_type,
		.probe		= eepro100_probe,
		.remove		= eepro100_remove,
		.suspend	= eepro100_suspend,
		.resume		= eepro100_resume,
         },
  };
```

Some may find the syntax of embedded struct initialization awkward or
even a bit ugly. So far, it's the best way we've found to do what we want...

有些人可能会觉得嵌套结构初始化的语法有些奇怪甚至有点丑陋。到目前为止，这是我们找到的做我们想要的事情的最好方式……

## 4.Registration

```c
int driver_register(struct device_driver *drv);
```

The driver registers the structure on startup. For drivers that have
no bus-specific fields (i.e. don't have a bus-specific driver
structure), they would use driver_register and pass a pointer to their
struct device_driver object.

在启动时，驱动程序将结构注册。对于没有总线特定字段的驱动程序（即没有总线特定驱动程序结构的驱动程序），它们将使用driver_register并传递指向其struct device_driver对象的指针。

Most drivers, however, will have a bus-specific structure and will
need to register with the bus using something like pci_driver_register.

然而，大多数驱动程序将具有总线特定的结构，并且将需要使用诸如pci_driver_register之类的内容向总线注册。

It is important that drivers register their driver structure as early as
possible. Registration with the core initializes several fields in the
struct device_driver object, including the reference count and the
lock. These fields are assumed to be valid at all times and may be
used by the device model core or the bus driver.

对于驱动程序，很重要的是在尽可能早的时候注册其驱动程序结构。与核心的注册会初始化struct device_driver对象中的几个字段，包括引用计数和锁。这些字段被假定在任何时候都是有效的，并且可能由设备模型核心或总线驱动程序使用。

## 5.Transition Bus Drivers

## 5.过渡总线驱动程序

By defining wrapper functions, the transition to the new model can be
made easier. Drivers can ignore the generic structure altogether and
let the bus wrapper fill in the fields. For the callbacks, the bus can
define generic callbacks that forward the call to the bus-specific
callbacks of the drivers.

通过定义包装函数，可以更轻松地过渡到新模型。驱动程序可以完全忽略通用结构，并让总线包装器填充字段。对于回调，总线可以定义通用回调，将调用转发到驱动程序的总线特定回调。

This solution is intended to be only temporary. In order to get class
information in the driver, the drivers must be modified anyway. Since
converting drivers to the new model should reduce some infrastructural
complexity and code size, it is recommended that they are converted as
class information is added.

这个解决方案仅用于临时使用。为了在驱动程序中获取类信息，驱动程序必须进行修改。由于将驱动程序转换为新模型应该减少一些基础结构的复杂性和代码大小，建议在添加类信息时进行转换。

## 6.Access

Once the object has been registered, it may access the common fields of
the object, like the lock and the list of devices::

对象注册后，它可以访问对象的常见字段，如锁和设备列表：

```c
int driver_for_each_dev(struct device_driver *drv, void *data, int (*callback)(struct device *dev, void *data));
```

The devices field is a list of all the devices that have been bound to
the driver. The LDM core provides a helper function to operate on all
the devices a driver controls. This helper locks the driver on each
node access, and does proper reference counting on each device as it
accesses it.

devices字段是所有已绑定到驱动程序的设备的列表。LDM核心提供了一个辅助函数来处理驱动程序控制的所有设备。此助手在每次访问节点时锁定驱动程序，并在访问设备时对每个设备执行适当的引用计数。

## 7.sysfs

When a driver is registered, a sysfs directory is created in its
bus's directory. In this directory, the driver can export an interface
to userspace to control operation of the driver on a global basis;
e.g. toggling debugging output in the driver.

当驱动程序注册时，在其总线的目录中创建了一个sysfs目录。在此目录中，驱动程序可以向用户空间导出接口，以在全局范围内控制驱动程序的操作；例如，在驱动程序中切换调试输出。

A future feature of this directory will be a 'devices' directory. This
directory will contain symlinks to the directories of devices it
supports.

此目录的一个未来功能将是'devices'目录。此目录将包含到其支持的设备的目录的符号链接。

## 8.Callbacks

```c
int	(*probe)(struct device *dev);
```

The probe() entry is called in task context, with the bus's rwsem locked
and the driver partially bound to the device.  Drivers commonly use
container_of() to convert "dev" to a bus-specific type, both in probe()
and other routines.  That type often provides device resource data, such
as pci_dev.resource[] or platform_device.resources, which is used in
addition to dev->platform_data to initialize the driver.

The probe()函数在任务上下文中调用，总线的rwsem被锁定，驱动程序部分绑定到设备。驱动程序通常在probe()和其他例行程序中使用container_of()将"dev"转换为总线特定类型。该类型通常提供设备资源数据，例如pci_dev.resource[]或platform_device.resources，这些数据除了dev->platform_data外还用于初始化驱动程序。

This callback holds the driver-specific logic to bind the driver to a
given device.  That includes verifying that the device is present, that
it's a version the driver can handle, that driver data structures can
be allocated and initialized, and that any hardware can be initialized.
Drivers often store a pointer to their state with dev_set_drvdata().
When the driver has successfully bound itself to that device, then probe()
returns zero and the driver model code will finish its part of binding
the driver to that device.

此回调保存了将驱动程序绑定到给定设备的驱动程序特定逻辑。这包括验证设备是否存在，它是驱动程序可以处理的版本，可以分配和初始化驱动程序数据结构，以及可以初始化任何硬件。驱动程序通常使用dev_set_drvdata()将指针存储到其状态。当驱动程序成功绑定到该设备时，probe()返回零，驱动程序模型代码将完成其绑定到该设备的部分。

A driver's probe() may return a negative errno value to indicate that
the driver did not bind to this device, in which case it should have
released all resources it allocated.

驱动程序的probe()可能返回负的errno值，以指示驱动程序未绑定到此设备，在这种情况下，它应该释放所有分配的资源。

Optionally, probe() may return -EPROBE_DEFER if the driver depends on
resources that are not yet available (e.g., supplied by a driver that
hasn't initialized yet).  The driver core will put the device onto the
deferred probe list and will try to call it again later. If a driver
must defer, it should return -EPROBE_DEFER as early as possible to
reduce the amount of time spent on setup work that will need to be
unwound and reexecuted at a later time.

可选地，probe()可能返回-EPROBE_DEFER，如果驱动程序依赖尚未可用的资源（例如，由尚未初始化的驱动程序提供）。驱动程序核心将设备放入延迟探测列表，并稍后尝试再次调用它。如果驱动程序必须推迟，它应该尽早返回-EPROBE_DEFER，以减少在需要在稍后时间解开和重新执行的设置工作上花费的时间。

```
warning:
      -EPROBE_DEFER must not be returned if probe() has already created
      child devices, even if those child devices are removed again
      in a cleanup path. If -EPROBE_DEFER is returned after a child
      device has been registered, it may result in an infinite loop of
      .probe() calls to the same driver.

如果在probe()已经创建子设备的情况下返回-EPROBE_DEFER，即使这些子设备在清理路径中被再次移除，也可能导致对同一驱动程序的.probe()调用的无限循环。
```
```c
void (*sync_state)(struct device *dev);
```

sync_state is called only once for a device. It's called when all the consumer
devices of the device have successfully probed. The list of consumers of the
device is obtained by looking at the device links connecting that device to its
consumer devices.

sync_state仅对设备调用一次。当设备的所有使用者设备成功探测时调用。通过查看连接该设备与其使用者设备的设备链路，可以获取该设备的使用者列表。


The first attempt to call sync_state() is made during late_initcall_sync() to
give firmware and drivers time to link devices to each other. During the first
attempt at calling sync_state(), if all the consumers of the device at that
point in time have already probed successfully, sync_state() is called right
away. If there are no consumers of the device during the first attempt, that
too is considered as "all consumers of the device have probed" and sync_state()
is called right away.

第一次尝试调用sync_state()是在late_initcall_sync()期间进行的，以便固件和驱动程序有时间将设备链接在一起。在第一次尝试调用sync_state()期间，如果在该时刻该设备的所有使用者已成功探测，则立即调用sync_state()。如果在第一次尝试期间没有该设备的使用者，也将其视为“该设备的所有使用者已成功探测”，并立即调用sync_state()。

If during the first attempt at calling sync_state() for a device, there are
still consumers that haven't probed successfully, the sync_state() call is
postponed and reattempted in the future only when one or more consumers of the
device probe successfully. If during the reattempt, the driver core finds that
there are one or more consumers of the device that haven't probed yet, then
sync_state() call is postponed again.

如果在第一次尝试调用sync_state()时，仍有一些尚未成功探测的使用者，那么将推迟并在将来重新尝试，仅当该设备的一个或多个使用者成功探测时。如果在重新尝试期间，驱动程序核心发现该设备的一个或多个使用者尚未成功探测，那么再次推迟sync_state()调用。

A typical use case for sync_state() is to have the kernel cleanly take over
management of devices from the bootloader. For example, if a device is left on
and at a particular hardware configuration by the bootloader, the device's
driver might need to keep the device in the boot configuration until all the
consumers of the device have probed. Once all the consumers of the device have
probed, the device's driver can synchronize the hardware state of the device to
match the aggregated software state requested by all the consumers. Hence the
name sync_state().

sync_state()的典型用例是使内核干净地从引导加载程序接管设备的管理。例如，如果引导加载程序通过特定的硬件配置保留设备处于打开状态，那么设备的驱动程序可能需要保持设备处于引导配置，直到设备的所有使用者都已成功探测。一旦设备的所有使用者都已成功探测，设备的驱动程序可以同步设备的硬件状态，以匹配所有使用者请求的聚合软件状态。因此称为sync_state()。

While obvious examples of resources that can benefit from sync_state() include
resources such as regulator, sync_state() can also be useful for complex
resources like IOMMUs. For example, IOMMUs with multiple consumers (devices
whose addresses are remapped by the IOMMU) might need to keep their mappings
fixed at (or additive to) the boot configuration until all its consumers have
probed.

虽然sync_state()有益于像调节器这样的资源的明显例子，但对于像IOMMU这样的复杂资源，sync_state()也可能很有用。例如，具有多个使用者（通过IOMMU重新映射地址的设备）的IOMMU可能需要保持其映射保持在（或者增加到）引导配置，直到其所有使用者都已成功探测。

While the typical use case for sync_state() is to have the kernel cleanly take
over management of devices from the bootloader, the usage of sync_state() is
not restricted to that. Use it whenever it makes sense to take an action after
all the consumers of a device have probed:

虽然sync_state()的典型用例是使内核从引导加载程序中干净地接管设备的管理，但sync_state()的使用并不限于此。每当在所有设备的使用者成功探测后执行操作时，使用它是有意义的:

```c
int	(*remove)(struct device *dev);
```

remove is called to unbind a driver from a device. This may be
called if a device is physically removed from the system, if the
driver module is being unloaded, during a reboot sequence, or
in other cases.

remove用于从设备解绑驱动程序。可能在从系统中物理移除设备时调用，如果正在卸载驱动程序模块，或在重新启动序列期间调用，或在其他情况下调用。

It is up to the driver to determine if the device is present or
not. It should free any resources allocated specifically for the
device; i.e. anything in the device's driver_data field.

由驱动程序决定设备是否存在。它应该释放专为设备分配的任何资源，即设备的driver_data字段中的任何内容。

If the device is still present, it should quiesce the device and place
it into a supported low-power state.

如果设备仍然存在，则应将设备静止，并将其置于受支持的低功耗状态。

```c
int	(*suspend)(struct device *dev, pm_message_t state);
```

suspend is called to put the device in a low power state.

suspend用于将设备置于低功耗状态。

```c
int	(*resume)(struct device *dev);
```

Resume is used to bring a device back from a low power state.

Resume用于从低功耗状态唤醒设备。

## 9.Attributes

```c
struct driver_attribute {
          struct attribute        attr;
          ssize_t (*show)(struct device_driver *driver, char *buf);
          ssize_t (*store)(struct device_driver *, const char *buf, size_t count);
  };
```

Device drivers can export attributes via their sysfs directories.
Drivers can declare attributes using a DRIVER_ATTR_RW and DRIVER_ATTR_RO
macro that works identically to the DEVICE_ATTR_RW and DEVICE_ATTR_RO
macros.

设备驱动程序可以通过其 sysfs 目录导出属性。驱动程序可以使用 `DRIVER_ATTR_RW` 和 `DRIVER_ATTR_RO` 宏声明属性，其工作方式与 `DEVICE_ATTR_RW` 和 `DEVICE_ATTR_RO` 宏完全相同。

Example:

```c
DRIVER_ATTR_RW(debug);
```

This is equivalent to declaring:

这等效于声明：

```c
struct driver_attribute driver_attr_debug;
```

This can then be used to add and remove the attribute from the
driver's directory using:

然后，可以使用以下方法将属性添加和移除到驱动程序的目录中：

```c
int driver_create_file(struct device_driver *, const struct driver_attribute *);

void driver_remove_file(struct device_driver *, const struct driver_attribute *);
```