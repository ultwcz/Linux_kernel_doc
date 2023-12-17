Porting Drivers to the New Driver Model
=======================================

Patrick Mochel

7 January 2003


# 1.Overview

Please refer to `Documentation/driver-api/driver-model/*.rst` for definitions of
various driver types and concepts.

请参考 Documentation/driver-api/driver-model/*.rst 以获取各种驱动程序类型和概念的定义。

Most of the work of porting devices drivers to the new model happens
at the bus driver layer. This was intentional, to minimize the
negative effect on kernel drivers, and to allow a gradual transition
of bus drivers.

大多数将设备驱动程序迁移到新模型的工作发生在总线驱动程序层。这是有意为之的，以最小化对内核驱动程序的负面影响，并允许逐步过渡总线驱动程序。

In a nutshell, the driver model consists of a set of objects that can
be embedded in larger, bus-specific objects. Fields in these generic
objects can replace fields in the bus-specific objects.

简而言之，驱动程序模型由一组可以嵌入到更大、特定于总线的对象中的对象组成。这些通用对象中的字段可以替代总线特定对象中的字段。

The generic objects must be registered with the driver model core. By
doing so, they will exported via the sysfs filesystem. sysfs can be
mounted by doing:

通用对象必须向驱动程序模型核心注册。通过这样做，它们将通过 sysfs 文件系统导出。可以通过执行以下命令挂载 sysfs 文件系统：

    # mount -t sysfs sysfs /sys


# 2.The Process

## Step 0: Read include/linux/device.h for object and function definitions.
## 第0步： 阅读 include/linux/device.h 以获取对象和函数定义。

## Step 1: Registering the bus driver.
## 第1步： 注册总线驱动程序

- Define a struct bus_type for the bus driver:
- 定义总线驱动程序的结构体 bus_type：


  ```c
  struct bus_type pci_bus_type = {
        .name           = "pci",
  };
  ```

- Register the bus type.
- 注册总线类型

  This should be done in the initialization function for the bus type,
  which is usually the module_init(), or equivalent, function:

  这应该在总线类型的初始化函数中完成，通常是 module_init() 或等效函数：


  ```c
  static int __init pci_driver_init(void)
  {
        return bus_register(&pci_bus_type);
  }

  subsys_initcall(pci_driver_init);
  ```
  The bus type may be unregistered (if the bus driver may be compiled as a module) by doing:

  如果总线驱动程序可能作为模块编译，可以注销总线类型：


  ```c
  bus_unregister(&pci_bus_type);
  ```

- Export the bus type for others to use.
- 导出总线类型以供其他使用

  Other code may wish to reference the bus type, so declare it in a
  shared header file and export the symbol.

  其他代码可能希望引用总线类型，因此在共享的头文件中声明并导出该符号。

  From include/linux/pci.h:
  ```c
    extern struct bus_type pci_bus_type;
  ```
  From file the above code appears in:
  ```c
    EXPORT_SYMBOL(pci_bus_type);
  ```
- This will cause the bus to show up in /sys/bus/pci/ with two
  subdirectories: 'devices' and 'drivers':
- 这将导致总线显示在 /sys/bus/pci/ 中，具有两个子目录：'devices' 和 'drivers':

  ```
  # tree -d /sys/bus/pci/
  /sys/bus/pci/
  |-- devices
  `-- drivers
  ```

## Step 2: Registering Devices.
## 第2步：注册设备
struct device represents a single device. It mainly contains metadata
describing the relationship the device has to other entities.

struct device 代表单个设备。主要包含描述设备与其他实体关系的元数据。

- Embed a struct device in the bus-specific device type:
- 在总线特定的设备类型中嵌入 struct device：

  ```c
  struct pci_dev {
         ...
         struct  device  dev;            /* Generic device interface */
         ...
  };
  ```
  It is recommended that the generic device not be the first item in
  the struct to discourage programmers from doing mindless casts
  between the object types. Instead macros, or inline functions,
  should be created to convert from the generic object type:

  建议通用设备不是结构体中的第一个项，以防止程序员进行无意识的类型转换。应该创建宏或内联函数来转换为通用对象类型：
  
  ```c
    #define to_pci_dev(n) container_of(n, struct pci_dev, dev)

    or

    static inline struct pci_dev * to_pci_dev(struct kobject * kobj)
    {
	return container_of(n, struct pci_dev, dev);
    }
    ```

  This allows the compiler to verify type-safety of the operations
  that are performed (which is Good).

  这允许编译器验证执行的操作的类型安全性（这是好的）。

- Initialize the device on registration.
- 在注册时初始化设备。

  When devices are discovered or registered with the bus type, the
  bus driver should initialize the generic device. The most important
  things to initialize are the bus_id, parent, and bus fields.

  当设备被发现或与总线类型注册时，总线驱动程序应初始化通用设备。初始化最重要的是初始化 bus_id、parent 和 bus 字段。

  The bus_id is an ASCII string that contains the device's address on
  the bus. The format of this string is bus-specific. This is
  necessary for representing devices in sysfs.

  bus_id 是包含设备在总线上地址的 ASCII 字符串。该字符串的格式是特定于总线的。这是在 sysfs 中表示设备的必要条件。

  parent is the physical parent of the device. It is important that
  the bus driver sets this field correctly.

  parent 是设备的物理父级。总线驱动程序设置此字段是很重要的。

  The driver model maintains an ordered list of devices that it uses
  for power management. This list must be in order to guarantee that
  devices are shutdown before their physical parents, and vice versa.
  The order of this list is determined by the parent of registered
  devices.

  驱动程序模型维护一个用于电源管理的设备有序列表。为了保证在物理父级之前关闭设备，以及反之亦然，此列表必须有序。已注册设备的顺序由其父级确定。

  Also, the location of the device's sysfs directory depends on a
  device's parent. sysfs exports a directory structure that mirrors
  the device hierarchy. Accurately setting the parent guarantees that
  sysfs will accurately represent the hierarchy.

  此外，设备的 sysfs 目录的位置取决于设备的父级。sysfs 导出一个反映设备层次结构的目录结构。准确设置父级可以保证 sysfs 准确表示层次结构。

  The device's bus field is a pointer to the bus type the device
  belongs to. This should be set to the bus_type that was declared
  and initialized before.

  设备的 bus 字段是指向设备所属总线类型的指针。应将其设置为之前声明和初始化的 bus_type。

  Optionally, the bus driver may set the device's name and release
  fields.

  可选地，总线驱动程序可以设置设备的名称和释放字段。

  The name field is an ASCII string describing the device, like

  名称字段是描述设备的 ASCII 字符串，例如：

     "ATI Technologies Inc Radeon QD"

  The release field is a callback that the driver model core calls
  when the device has been removed, and all references to it have
  been released. More on this in a moment.

  释放字段是在设备已被移除并且对其的所有引用都已释放时驱动程序模型核心调用的回调。稍后详细说明。

- Register the device.
- 注册设备


  Once the generic device has been initialized, it can be registered
  with the driver model core by doing:
  一旦通用设备已初始化，可以通过以下方式向驱动程序模型核心注册它：


  ```c
    device_register(&dev->dev);
  ```
  It can later be unregistered by doing:
  可以通过以下方式稍后注销它：


  ```c
    device_unregister(&dev->dev);
  ```
  This should happen on buses that support hotpluggable devices.
  If a bus driver unregisters a device, it should not immediately free
  it. It should instead wait for the driver model core to call the
  device's release method, then free the bus-specific object.
  (There may be other code that is currently referencing the device
  structure, and it would be rude to free the device while that is
  happening).
  这应该发生在支持热插拔设备的总线上。如果总线驱动程序注销设备，它不应立即释放设备。相反，应等待驱动程序模型核心调用设备的释放方法，然后释放总线特定对象。此时可能有其他代码正在引用设备结构，而在发生这种情况时释放设备是不礼貌的.


  When the device is registered, a directory in sysfs is created.
  The PCI tree in sysfs looks like:
  当设备注册时，在 sysfs 中将创建一个目录。sysfs 中的 PCI 树如下所示：


  ```
    /sys/devices/pci0/
    |-- 00:00.0
    |-- 00:01.0
    |   `-- 01:00.0
    |-- 00:02.0
    |   `-- 02:1f.0
    |       `-- 03:00.0
    |-- 00:1e.0
    |   `-- 04:04.0
    |-- 00:1f.0
    |-- 00:1f.1
    |   |-- ide0
    |   |   |-- 0.0
    |   |   `-- 0.1
    |   `-- ide1
    |       `-- 1.0
    |-- 00:1f.2
    |-- 00:1f.3
    `-- 00:1f.5
  ```
  Also, symlinks are created in the bus's 'devices' directory
  that point to the device's directory in the physical hierarchy:
  此外，在总线的 'devices' 目录中将创建指向设备的物理层次结构中设备目录的符号链接：


  ```
    /sys/bus/pci/devices/
    |-- 00:00.0 -> ../../../devices/pci0/00:00.0
    |-- 00:01.0 -> ../../../devices/pci0/00:01.0
    |-- 00:02.0 -> ../../../devices/pci0/00:02.0
    |-- 00:1e.0 -> ../../../devices/pci0/00:1e.0
    |-- 00:1f.0 -> ../../../devices/pci0/00:1f.0
    |-- 00:1f.1 -> ../../../devices/pci0/00:1f.1
    |-- 00:1f.2 -> ../../../devices/pci0/00:1f.2
    |-- 00:1f.3 -> ../../../devices/pci0/00:1f.3
    |-- 00:1f.5 -> ../../../devices/pci0/00:1f.5
    |-- 01:00.0 -> ../../../devices/pci0/00:01.0/01:00.0
    |-- 02:1f.0 -> ../../../devices/pci0/00:02.0/02:1f.0
    |-- 03:00.0 -> ../../../devices/pci0/00:02.0/02:1f.0/03:00.0
    `-- 04:04.0 -> ../../../devices/pci0/00:1e.0/04:04.0
  ```
## 
## 第3步： 注册驱动程序
Step 3: Registering Drivers.

struct device_driver is a simple driver structure that contains a set
of operations that the driver model core may call.

struct device_driver 是一个简单的驱动程序结构，包含驱动程序模型核心可能调用的一组操作。

- Embed a struct device_driver in the bus-specific driver.
- 在总线特定的驱动程序中嵌入 struct device_driver:

  Just like with devices, do something like:

  ```c
    struct pci_driver {
           ...
           struct device_driver    driver;
    };
    ```

- Initialize the generic driver structure.
- 初始化通用驱动程序结构


  When the driver registers with the bus (e.g. doing pci_register_driver()),
  initialize the necessary fields of the driver: the name and bus
  fields.
  
  当驱动程序向总线注册时（例如执行 pci_register_driver()），初始化驱动程序的必要字段：name 和 bus 字段

- Register the driver.
- 注册驱动程序

  After the generic driver has been initialized, call:

  通用驱动程序初始化后，调用：

  ```c
    driver_register(&drv->driver);
  ```

  to register the driver with the core.

  When the driver is unregistered from the bus, unregister it from the
  core by doing:

  通过以下方式从总线注销驱动程序：

  ```c
    driver_unregister(&drv->driver);
  ```

  Note that this will block until all references to the driver have
  gone away. Normally, there will not be any.

  请注意，这将阻塞直到所有对驱动程序的引用都消失。通常情况下，不会有任何引用。

- Sysfs representation.
- Sysfs 表示

  Drivers are exported via sysfs in their bus's 'driver's directory.
  For example:

  驱动程序通过 sysfs 在其总线的 'driver' 目录中导出。例如：

  ```
    /sys/bus/pci/drivers/
    |-- 3c59x
    |-- Ensoniq AudioPCI
    |-- agpgart-amdk7
    |-- e100
    `-- serial
  ```

## Step 4: Define Generic Methods for Drivers.
## 第4步： 为驱动程序定义通用方法。

struct device_driver defines a set of operations that the driver model
core calls. Most of these operations are probably similar to
operations the bus already defines for drivers, but taking different
parameters.

struct device_driver 定义了驱动程序模型核心调用的一组操作。这些操作中的大多数可能与总线已经为驱动程序定义的操作相似，但采用不同的参数。

It would be difficult and tedious to force every driver on a bus to
simultaneously convert their drivers to generic format. Instead, the
bus driver should define single instances of the generic methods that
forward call to the bus-specific drivers. For instance:

强迫每个总线上的驱动程序同时将其驱动程序转换为通用格式将是困难且繁琐的。相反，总线驱动程序应该定义通用方法的单个实例，将调用转发到总线特定的驱动程序。例如：

```c
  static int pci_device_remove(struct device * dev)
  {
    struct pci_dev * pci_dev = to_pci_dev(dev);
    struct pci_driver * drv = pci_dev->driver;

    if (drv) {
            if (drv->remove)
                    drv->remove(pci_dev);
            pci_dev->driver = NULL;
    }
    return 0;
  }
```

The generic driver should be initialized with these methods before it is registered:

通用驱动程序在注册之前应该使用这些方法初始化：

```c
     /* initialize common driver fields */
     drv->driver.name = drv->name;
     drv->driver.bus = &pci_bus_type;
     drv->driver.probe = pci_device_probe;
     drv->driver.resume = pci_device_resume;
     drv->driver.suspend = pci_device_suspend;
     drv->driver.remove = pci_device_remove;

     /* register with core */
     driver_register(&drv->driver);
```

Ideally, the bus should only initialize the fields if they are not
already set. This allows the drivers to implement their own generic
methods.

理想情况下，总线只应在字段尚未设置时初始化这些字段。这允许驱动程序实现自己的通用方法。

## Step 5: Support generic driver binding.
## 第5步： 支持通用驱动程序绑定

The model assumes that a device or driver can be dynamically
registered with the bus at any time. When registration happens,
devices must be bound to a driver, or drivers must be bound to all
devices that it supports.

该模型假设设备或驱动程序可以在任何时候动态注册到总线。当注册发生时，设备必须绑定到驱动程序，或者驱动程序必须绑定到它支持的所有设备。

A driver typically contains a list of device IDs that it supports. The
bus driver compares these IDs to the IDs of devices registered with it.
The format of the device IDs, and the semantics for comparing them are
bus-specific, so the generic model does attempt to generalize them.

驱动程序通常包含其支持的设备的设备 ID 列表。总线驱动程序将这些 ID 与其注册的设备的 ID 进行比较。设备 ID 的格式和比较它们的语义是特定于总线的，因此通用模型不尝试将其概括。

Instead, a bus may supply a method in struct bus_type that does the
comparison:

相反，总线可以在 struct bus_type 中提供一种方法进行比较：

```c
int (*match)(struct device * dev, struct device_driver * drv);
```

match should return positive value if the driver supports the device,
and zero otherwise. It may also return error code (for example
-EPROBE_DEFER) if determining that given driver supports the device is
not possible.

如果驱动程序支持设备，则 match 应返回正值，否则返回零。如果确定给定驱动程序是否支持设备不可能，则可能还会返回错误代码（例如 -EPROBE_DEFER）。

When a device is registered, the bus's list of drivers is iterated
over. bus->match() is called for each one until a match is found.

当设备注册时，总线的驱动程序列表将被迭代。对于每个驱动程序，将为每个设备调用 bus->match()，直到找到匹配。

When a driver is registered, the bus's list of devices is iterated
over. bus->match() is called for each device that is not already
claimed by a driver.

当驱动程序注册时，总线的设备列表将被迭代。对于尚未被驱动程序声明的每个设备，将为每个设备调用 bus->match()。

When a device is successfully bound to a driver, device->driver is
set, the device is added to a per-driver list of devices, and a
symlink is created in the driver's sysfs directory that points to the
device's physical directory:

当设备成功绑定到驱动程序时，device->driver 被设置，该设备被添加到每个驱动程序的设备列表中，并在驱动程序的 sysfs 目录中创建一个符号链接，该符号链接指向设备的物理目录。

```
/sys/bus/pci/drivers/
|-- 3c59x
|   `-- 00:0b.0 -> ../../../../devices/pci0/00:0b.0
|-- Ensoniq AudioPCI
|-- agpgart-amdk7
|   `-- 00:00.0 -> ../../../../devices/pci0/00:00.0
|-- e100
|   `-- 00:0c.0 -> ../../../../devices/pci0/00:0c.0
`-- serial
```

This driver binding should replace the existing driver binding
mechanism the bus currently uses.

这个驱动绑定机制应该取代总线当前使用的现有驱动绑定机制。

## Step 6: Supply a hotplug callback.
## Step 6: 提供热插拔回调函数

Whenever a device is registered with the driver model core, the
userspace program /sbin/hotplug is called to notify userspace.
Users can define actions to perform when a device is inserted or
removed.

每当设备向驱动模型核心注册时，会调用用户空间程序 /sbin/hotplug 以通知用户空间。用户可以定义在插入或移除设备时执行的操作。

The driver model core passes several arguments to userspace via
environment variables, including

驱动模型核心通过环境变量向用户空间传递多个参数，其中包括：

- ACTION: set to 'add' or 'remove'
- DEVPATH: set to the device's physical path in sysfs.

- ACTION: 设置为 'add' 或 'remove'
- DEVPATH: 设置为设备在 sysfs 中的物理路径。


A bus driver may also supply additional parameters for userspace to
consume. To do this, a bus must implement the 'hotplug' method in
struct bus_type:

总线驱动程序还可以提供其他参数供用户空间使用。要做到这一点，总线必须在 struct bus_type 中实现 'hotplug' 方法：

```c
int (*hotplug) (struct device *dev, char **envp,
                     int num_envp, char *buffer, int buffer_size);
```

This is called immediately before /sbin/hotplug is executed.

这在执行 /sbin/hotplug 之前被立即调用。

## Step 7: Cleaning up the bus driver.
## Step 7: 清理总线驱动程序

The generic bus, device, and driver structures provide several fields
that can replace those defined privately to the bus driver.

通用的总线、设备和驱动程序结构提供了几个字段，可以替代总线驱动程序私有定义的那些。

- Device list.
- 设备列表

  struct bus_type contains a list of all devices registered with the bus type. This includes all devices on all instances of that bus type. An internal list that the bus uses may be removed, in favor of using this one.

  struct bus_type 包含一个注册到总线类型的所有设备的列表。这包括该总线类型的所有实例上的所有设备。总线使用的内部列表可以被移除，而使用这个列表。

  The core provides an iterator to access these devices:

  核心提供了一个迭代器来访问这些设备：

  ```c
  int bus_for_each_dev(struct bus_type * bus, struct device * start,
                       void * data, int (*fn)(struct device *, void *));
  ```

- Driver list.
- 驱动程序列表

  struct bus_type also contains a list of all drivers registered with it. An internal list of drivers that the bus driver maintains may be removed in favor of using the generic one.

  struct bus_type 还包含一个注册到它的所有驱动程序的列表。总线驱动程序维护的驱动程序的内部列表可以被移除，而使用通用的列表。

  The drivers may be iterated over, like devices:

  驱动程序可以像设备一样进行迭代：

  ```c
  int bus_for_each_drv(struct bus_type * bus, struct device_driver * start,
                       void * data, int (*fn)(struct device_driver *, void *));
  ```

  Please see drivers/base/bus.c for more information.

  请参阅 drivers/base/bus.c 以获取更多信息。


- rwsem

  struct bus_type contains an rwsem that protects all core accesses to the device and driver lists. This can be used by the bus driver internally, and should be used when accessing the device or driver lists the bus maintains.

  struct bus_type 包含一个 rwsem，用于保护对设备和驱动程序列表的所有核心访问。总线驱动程序可以在内部使用它，并且在访问总线维护的设备或驱动程序列表时应该使用它。

- Device and driver fields.
- 设备和驱动程序字段

  Some of the fields in struct device and struct device_driver duplicate fields in the bus-specific representations of these objects. Feel free to remove the bus-specific ones and favor the generic ones. Note though, that this will likely mean fixing up all the drivers that reference the bus-specific fields (though those should all be 1-line changes).

  struct device 和 struct device_driver 中的一些字段复制了这些对象的特定于总线的表示中的字段。可以删除特定于总线的字段，使用通用的字段。但请注意，这可能意味着修复所有引用特定于总线字段的驱动程序（尽管这些应该都是一行更改）。