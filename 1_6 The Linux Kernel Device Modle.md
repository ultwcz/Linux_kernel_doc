The Linux Kernel Device Model
=============================

Patrick Mochel	<mochel@digitalimplant.org>

Drafted 26 August 2002

Updated 31 January 2006

起草日期：2002年8月26日

更新日期：2006年1月31日

# 1.Overview

The Linux Kernel Driver Model is a unification of all the disparate driver
models that were previously used in the kernel. It is intended to augment the
bus-specific drivers for bridges and devices by consolidating a set of data
and operations into globally accessible data structures.

Linux内核驱动程序模型是对先前在内核中使用的所有不同的驱动程序模型的统一。它旨在通过将一系列数据和操作整合到全局可访问的数据结构中来增强针对桥接器和设备的特定总线驱动程序。

Traditional driver models implemented some sort of tree-like structure
(sometimes just a list) for the devices they control. There wasn't any
uniformity across the different bus types.

传统的驱动程序模型对其控制的设备实施了某种类似树状结构（有时只是一个列表）。在不同的总线类型之间没有统一性。

The current driver model provides a common, uniform data model for describing
a bus and the devices that can appear under the bus. The unified bus
model includes a set of common attributes which all busses carry, and a set
of common callbacks, such as device discovery during bus probing, bus
shutdown, bus power management, etc.

当前的驱动程序模型提供了一种通用的、统一的数据模型，用于描述总线和可能出现在总线下的设备。统一的总线模型包括所有总线都携带的一组公共属性，以及一组公共的回调，比如总线探测期间的设备发现、总线关闭、总线电源管理等。

The common device and bridge interface reflects the goals of the modern
computer: namely the ability to do seamless device "plug and play", power
management, and hot plug. In particular, the model dictated by Intel and
Microsoft (namely ACPI) ensures that almost every device on almost any bus
on an x86-compatible system can work within this paradigm.  Of course,
not every bus is able to support all such operations, although most
buses support most of those operations.

公共设备和桥接接口反映了现代计算机的目标，即能够实现无缝的设备“即插即用”、电源管理和热插拔。特别是由Intel和Microsoft（即ACPI）规定的模型确保几乎在x86兼容系统上的任何总线上的几乎每个设备都可以在这个范式内工作。当然，并非每个总线都能支持所有这些操作，尽管大多数总线都支持其中的大多数操作。

# 2.Downstream Access

Common data fields have been moved out of individual bus layers into a common
data structure. These fields must still be accessed by the bus layers,
and sometimes by the device-specific drivers.

公共数据字段已从各个总线层中移出到一个通用数据结构。这些字段仍然必须由总线层和有时由设备特定的驱动程序访问。

Other bus layers are encouraged to do what has been done for the PCI layer.
struct pci_dev now looks like this:

鼓励其他总线层效仿PCI层的做法。现在，struct pci_dev看起来像这样：

```c
  struct pci_dev {
	...

	struct device dev;     /* Generic device interface */
	...
  };
```

Note first that the struct device dev within the struct pci_dev is
statically allocated. This means only one allocation on device discovery.

首先注意，在struct pci_dev内的struct device dev是静态分配的。这意味着在设备发现时只有一次分配。

Note also that that struct device dev is not necessarily defined at the
front of the pci_dev structure.  This is to make people think about what
they're doing when switching between the bus driver and the global driver,
and to discourage meaningless and incorrect casts between the two.

还要注意，struct device dev不一定在pci_dev结构的前面定义。这是为了让人们在总线驱动程序和全局驱动程序之间切换时考虑正在进行的操作，并阻止无意义和不正确的两者之间的强制转换。

The PCI bus layer freely accesses the fields of struct device. It knows about
the structure of struct pci_dev, and it should know the structure of struct
device. Individual PCI device drivers that have been converted to the current
driver model generally do not and should not touch the fields of struct device,
unless there is a compelling reason to do so.

PCI总线层自由访问struct device的字段。它了解struct pci_dev的结构，应该了解struct device的结构。通常已转换为当前驱动程序模型的个别PCI设备驱动程序通常不应该触及struct device的字段，除非有充分的理由这样做。

The above abstraction prevents unnecessary pain during transitional phases.
If it were not done this way, then when a field was renamed or removed, every
downstream driver would break.  On the other hand, if only the bus layer
(and not the device layer) accesses the struct device, it is only the bus
layer that needs to change.

上述抽象在过渡阶段防止了不必要的痛苦。如果不是这样做，那么当字段被重命名或删除时，每个下游驱动程序都将中断。另一方面，如果只有总线层（而不是设备层）访问struct device，那么只有总线层需要更改。

# 3.User Interface

By virtue of having a complete hierarchical view of all the devices in the
system, exporting a complete hierarchical view to userspace becomes relatively
easy. This has been accomplished by implementing a special purpose virtual
file system named sysfs.

通过拥有对系统中所有设备的完整分层视图，将完整的分层视图导出到用户空间相对容易。这是通过实现一个名为sysfs的专用虚拟文件系统来完成的。

Almost all mainstream Linux distros mount this filesystem automatically; you
can see some variation of the following in the output of the "mount" command:

几乎所有主流的Linux发行版都会自动挂载此文件系统；你可以在“mount”命令的输出中看到以下变化的某种形式：

```
  $ mount
  ...
  none on /sys type sysfs (rw,noexec,nosuid,nodev)
  ...
  $
```

The auto-mounting of sysfs is typically accomplished by an entry similar to
the following in the /etc/fstab file:

自动挂载sysfs通常是通过/etc/fstab文件中类似以下条目来完成的：

```
  none     	/sys	sysfs    defaults	  	0 0
```
or something similar in the /lib/init/fstab file on Debian-based systems:

或者在Debian系系统上的/lib/init/fstab文件中的类似内容：

```
  none            /sys    sysfs    nodev,noexec,nosuid    0 0
```

If sysfs is not automatically mounted, you can always do it manually with:

如果sysfs没有自动挂载，你总是可以使用以下命令手动挂载：

```
# mount -t sysfs sysfs /sys
```

Whenever a device is inserted into the tree, a directory is created for it.
This directory may be populated at each layer of discovery - the global layer,
the bus layer, or the device layer.

每当设备插入树中时，都会为其创建一个目录。此目录可以在每个发现层（全局层、总线层或设备层）中进行填充。

The global layer currently creates two files - 'name' and 'power'. The
former only reports the name of the device. The latter reports the
current power state of the device. It will also be used to set the current
power state.

全局层目前创建了两个文件 - 'name' 和 'power'。前者仅报告设备的名称。后者报告设备的当前电源状态。它还将用于设置当前电源状态。

The bus layer may also create files for the devices it finds while probing the
bus. For example, the PCI layer currently creates 'irq' and 'resource' files
for each PCI device.

总线层还可以为它在总线探测期间找到的设备创建文件。例如，PCI层当前为每个PCI设备创建'irq'和'resource'文件。

A device-specific driver may also export files in its directory to expose
device-specific data or tunable interfaces.

设备特定的驱动程序还可以在其目录中导出文件，以公开设备特定的数据或可调整的接口。

More information about the sysfs directory layout can be found in
the other documents in this directory and in the file
Documentation/filesystems/sysfs.rst.

关于sysfs目录布局的更多信息可以在此目录中的其他文档和文件Documentation/filesystems/sysfs.rst中找到。