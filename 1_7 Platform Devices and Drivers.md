Platform Devices and Drivers
============================

See <linux/platform_device.h> for the driver model interface to the
platform bus:  platform_device, and platform_driver.  This pseudo-bus
is used to connect devices on busses with minimal infrastructure,
like those used to integrate peripherals on many system-on-chip
processors, or some "legacy" PC interconnects; as opposed to large
formally specified ones like PCI or USB.

请参阅<linux/platform_device.h>以获取与平台总线的驱动程序模型接口的详细信息：platform_device 和 platform_driver。这个伪总线用于连接在具有最小基础结构的总线上的设备，比如用于集成许多系统芯片处理器上的外围设备，或者一些“传统”PC互连；与PCI或USB等大型正式规范总线相对。

# 1.Platform devices

Platform devices are devices that typically appear as autonomous
entities in the system. This includes legacy port-based devices and
host bridges to peripheral buses, and most controllers integrated
into system-on-chip platforms.  What they usually have in common
is direct addressing from a CPU bus.  Rarely, a platform_device will
be connected through a segment of some other kind of bus; but its
registers will still be directly addressable.

平台设备是通常作为系统中独立实体出现的设备。这包括传统的基于端口的设备、主机桥接到外围总线的设备，以及集成到系统芯片平台的大多数控制器。它们通常具有来自CPU总线的直接寻址。很少情况下，平台设备将通过某种其他类型的总线段连接；但其寄存器仍然是可以直接寻址的。

Platform devices are given a name, used in driver binding, and a
list of resources such as addresses and IRQs:

平台设备被赋予一个名称，用于驱动程序绑定，以及一组资源列表，如地址和IRQs：

```c
  struct platform_device {
	const char	*name;
	u32		id;
	struct device	dev;
	u32		num_resources;
	struct resource	*resource;
  };
```

# 2.Platform drivers

Platform drivers follow the standard driver model convention, where
discovery/enumeration is handled outside the drivers, and drivers
provide probe() and remove() methods.  They support power management
and shutdown notifications using the standard conventions:

平台驱动程序遵循标准的驱动程序模型约定，其中发现/枚举是在驱动程序之外处理的，而驱动程序提供 probe() 和 remove() 方法。它们支持使用标准约定进行电源管理和关闭通知：

```c
  struct platform_driver {
	int (*probe)(struct platform_device *);
	int (*remove)(struct platform_device *);
	void (*shutdown)(struct platform_device *);
	int (*suspend)(struct platform_device *, pm_message_t state);
	int (*suspend_late)(struct platform_device *, pm_message_t state);
	int (*resume_early)(struct platform_device *);
	int (*resume)(struct platform_device *);
	struct device_driver driver;
  };
```

Note that probe() should in general verify that the specified device hardware
actually exists; sometimes platform setup code can't be sure.  The probing
can use device resources, including clocks, and device platform_data.

请注意，probe() 通常应该验证指定的设备硬件是否实际存在；有时平台设置代码不能确定。探测可以使用设备资源，包括时钟和设备 platform_data。

Platform drivers register themselves the normal way:

平台驱动程序通过以下常规方式注册自己：

```c
int platform_driver_register(struct platform_driver *drv);
```

Or, in common situations where the device is known not to be hot-pluggable,
the probe() routine can live in an init section to reduce the driver's
runtime memory footprint:

或者，在设备被知道不是热插拔的常见情况下，probe() 例程可以存在于 init 部分，以减少驱动程序的运行时内存占用：

```c
int platform_driver_probe(struct platform_driver *drv,
			  int (*probe)(struct platform_device *))
```

Kernel modules can be composed of several platform drivers. The platform core
provides helpers to register and unregister an array of drivers:

内核模块可以由多个平台驱动程序组成。平台核心提供了注册和注销驱动程序数组的辅助函数：

```
	int __platform_register_drivers(struct platform_driver * const *drivers,
				      unsigned int count, struct module *owner);
	void platform_unregister_drivers(struct platform_driver * const *drivers,
					 unsigned int count);
```

If one of the drivers fails to register, all drivers registered up to that
point will be unregistered in reverse order. Note that there is a convenience
macro that passes THIS_MODULE as owner parameter:

如果其中一个驱动程序未能注册，直到该点注册的所有驱动程序都将以相反的顺序注销。请注意，有一个方便的宏，它将 THIS_MODULE 作为 owner 参数传递：

```
#define platform_register_drivers(drivers, count)
```

# 3.Device Enumeration

As a rule, platform specific (and often board-specific) setup code will
register platform devices:

一般规则是，特定于平台（通常也是特定于板级）的设置代码将注册平台设备：

```c
int platform_device_register(struct platform_device *pdev);

	int platform_add_devices(struct platform_device **pdevs, int ndev);
```

The general rule is to register only those devices that actually exist,
but in some cases extra devices might be registered.  For example, a kernel
might be configured to work with an external network adapter that might not
be populated on all boards, or likewise to work with an integrated controller
that some boards might not hook up to any peripherals.

一般规则是仅注册实际存在的设备，但在某些情况下，可能会注册额外的设备。例如，内核可能被配置为与可能不在所有板上填充的外部网络适配器一起工作，或者与某些板可能不连接到任何外围设备的集成控制器一起工作。

In some cases, boot firmware will export tables describing the devices
that are populated on a given board.   Without such tables, often the
only way for system setup code to set up the correct devices is to build
a kernel for a specific target board.  Such board-specific kernels are
common with embedded and custom systems development.

在某些情况下，引导固件将导出描述在给定板上填充的设备的表。如果没有这些表，通常系统设置代码设置正确设备的唯一方法是为特定的目标板构建内核。这种特定于板的内核在嵌入式和自定义系统开发中很常见。

In many cases, the memory and IRQ resources associated with the platform
device are not enough to let the device's driver work.  Board setup code
will often provide additional information using the device's platform_data
field to hold additional information.

在许多情况下，与平台设备相关联的内存和IRQ资源不足以让设备的驱动程序工作。板级设置代码通常会使用设备的 platform_data 字段提供附加信息。

Embedded systems frequently need one or more clocks for platform devices,
which are normally kept off until they're actively needed (to save power).
System setup also associates those clocks with the device, so that
calls to clk_get(&pdev->dev, clock_name) return them as needed.

嵌入式系统通常需要一个或多个时钟用于平台设备，通常在需要时保持关闭以节省电源。系统设置还将这些时钟与设备关联起来，以便调用 clk_get(&pdev->dev, clock_name) 在需要时返回它们。

# 4.Legacy Drivers:  Device Probing

Some drivers are not fully converted to the driver model, because they take
on a non-driver role:  the driver registers its platform device, rather than
leaving that for system infrastructure.  Such drivers can't be hotplugged
or coldplugged, since those mechanisms require device creation to be in a
different system component than the driver.

一些驱动程序没有完全转换为驱动程序模型，因为它们承担了非驱动程序角色：驱动程序注册其平台设备，而不是将其留给系统基础设施。这样的驱动程序无法进行热插拔或冷插拔，因为这些机制要求设备创建在与驱动程序不同的系统组件中。

The only "good" reason for this is to handle older system designs which, like
original IBM PCs, rely on error-prone "probe-the-hardware" models for hardware
configuration.  Newer systems have largely abandoned that model, in favor of
bus-level support for dynamic configuration (PCI, USB), or device tables
provided by the boot firmware (e.g. PNPACPI on x86).  There are too many
conflicting options about what might be where, and even educated guesses by
an operating system will be wrong often enough to make trouble.

这样做的唯一“好”原因是处理旧的系统设计，例如原始的IBM PC，依赖于容易出错的“探测硬件”模型进行硬件配置。较新的系统大多已经放弃了这个模型，而转而支持动态配置的总线级支持（PCI、USB），或者由引导固件提供的设备表（例如x86上的PNPACPI）。有关可能在哪里以及即使是由操作系统的有知识的

This style of driver is discouraged.  If you're updating such a driver,
please try to move the device enumeration to a more appropriate location,
outside the driver.  This will usually be cleanup, since such drivers
tend to already have "normal" modes, such as ones using device nodes that
were created by PNP or by platform device setup.

不建议使用这种风格的驱动程序。如果您正在更新此类驱动程序，请尝试将设备枚举移到更合适的位置，即驱动程序之外。这通常是清理工作，因为这些驱动程序通常已经有“正常”模式，例如使用由PNP或平台设备设置创建的设备节点。

None the less, there are some APIs to support such legacy drivers.  Avoid
using these calls except with such hotplug-deficient drivers:

尽管如此，还有一些支持此类传统驱动程序的 API。除非使用这些不支持热插拔的驱动程序，否则请避免使用这些调用：

```c
struct platform_device *platform_device_alloc(
			const char *name, int id);
```

You can use platform_device_alloc() to dynamically allocate a device, which
you will then initialize with resources and platform_device_register().
A better solution is usually:

您可以使用 platform_device_alloc() 动态分配设备，然后使用资源和 platform_device_register() 进行初始化。更好的解决方案通常是：

```c
	struct platform_device *platform_device_register_simple(
			const char *name, int id,
			struct resource *res, unsigned int nres);
```

You can use platform_device_register_simple() as a one-step call to allocate
and register a device.

您可以使用 platform_device_register_simple() 作为一步调用，以分配和注册设备。

# 5.Device Naming and Driver Binding

The platform_device.dev.bus_id is the canonical name for the devices.
It's built from two components:

platform_device.dev.bus_id 是设备的规范名称。它由两个组件构建：

    * platform_device.name ... which is also used to for driver matching.
    * platform_device.name ... 也用于驱动程序匹配。

    * platform_device.id ... the device instance number, or else "-1"
      to indicate there's only one.
    * platform_device.id ... 设备实例编号，否则为“-1”表示只有一个。


These are concatenated, so name/id "serial"/0 indicates bus_id "serial.0", and
"serial/3" indicates bus_id "serial.3"; both would use the platform_driver
named "serial".  While "my_rtc"/-1 would be bus_id "my_rtc" (no instance id)
and use the platform_driver called "my_rtc".

这些被连接在一起，因此名称/id "serial"/0 表示 bus_id "serial.0"，而 "serial/3" 表示 bus_id "serial.3"；两者都将使用名为 "serial" 的 platform_driver。而 "my_rtc"/-1 将是 bus_id "my_rtc"（无实例 id），并使用名为 "my_rtc" 的 platform_driver。

Driver binding is performed automatically by the driver core, invoking
driver probe() after finding a match between device and driver.  If the
probe() succeeds, the driver and device are bound as usual.  There are
three different ways to find such a match:

驱动程序绑定是由驱动程序核心自动执行的，它在设备和驱动程序之间找到匹配后调用 driver probe()。如果 probe() 成功，驱动程序和设备将像通常一样绑定。有三种不同的方法可以找到这样的匹配：

    - Whenever a device is registered, the drivers for that bus are
      checked for matches.  Platform devices should be registered very
      early during system boot.
    - 每当注册设备时，都会检查该总线的驱动程序以进行匹配。平台设备应该在系统引导期间非常早期注册。

    - When a driver is registered using platform_driver_register(), all
      unbound devices on that bus are checked for matches.  Drivers
      usually register later during booting, or by module loading.
    - 当使用 platform_driver_register() 注册驱动程序时，将检查该总线上所有未绑定的设备是否匹配。驱动程序通常在引导过程中注册，或通过模块加载。

    - Registering a driver using platform_driver_probe() works just like
      using platform_driver_register(), except that the driver won't
      be probed later if another device registers.  (Which is OK, since
      this interface is only for use with non-hotpluggable devices.)
     - 使用 platform_driver_probe() 注册驱动程序的方式与使用 platform_driver_register() 相同，只是如果另一个设备注册，该驱动程序将不会在以后进行探测（这是可以的，因为此接口仅用于不可热插拔的设备）。


# 6.Early Platform Devices and Drivers

The early platform interfaces provide platform data to platform device
drivers early on during the system boot. The code is built on top of the
early_param() command line parsing and can be executed very early on.

早期平台接口在系统引导期间早期提供平台设备驱动程序的平台数据。该代码建立在 early_param() 命令行解析之上，并可以在引导期间非常早期执行。

Example: "earlyprintk" class early serial console in 6 steps

示例：“earlyprintk”类早期串行控制台的 6 个步骤

## 1.Registering early platform device data
## 1.早期注册平台设备数据

The architecture code registers platform device data using the function
early_platform_add_devices(). In the case of early serial console this
should be hardware configuration for the serial port. Devices registered
at this point will later on be matched against early platform drivers.

架构代码使用函数 early_platform_add_devices() 注册平台设备数据。在这种情况下，这应该是串行端口的硬件配置。在稍后将匹配到早期平台驱动程序的平台设备。

## 2.Parsing kernel command line
## 2.解析内核命令行
The architecture code calls parse_early_param() to parse the kernel
command line. This will execute all matching early_param() callbacks.
User specified early platform devices will be registered at this point.
For the early serial console case the user can specify port on the
kernel command line as "earlyprintk=serial.0" where "earlyprintk" is
the class string, "serial" is the name of the platform driver and
0 is the platform device id. If the id is -1 then the dot and the
id can be omitted.

架构代码调用 parse_early_param() 来解析内核命令行。这将执行所有匹配的 early_param() 回调。在此时，将注册用户指定的早期平台设备。对于早期串行控制台的情况，用户可以在内核命令行中指定端口为“earlyprintk=serial.0”，其中“earlyprintk”是类字符串，“serial”是平台驱动程序的名称，0 是平台设备的 id。如果 id 是 -1，那么点和 id 可以省略。

## 3.Installing early platform drivers belonging to a certain class
## 3.安装属于某个类的早期平台驱动程序

The architecture code may optionally force registration of all early
platform drivers belonging to a certain class using the function
early_platform_driver_register_all(). User specified devices from
step 2 have priority over these. This step is omitted by the serial
driver example since the early serial driver code should be disabled
unless the user has specified port on the kernel command line.

架构代码可以选择使用函数 early_platform_driver_register_all() 强制注册属于某个类的所有早期平台驱动程序。步骤 2 中的用户指定设备优先于这些设备。对于早期串行驱动程序示例，此步骤被省略，因为除非用户在内核命令行上指定端口，否则应禁用早期串行驱动程序代码。

## 4.Early platform driver registration
## 4.早期平台驱动程序注册

Compiled-in platform drivers making use of early_platform_init() are
automatically registered during step 2 or 3. The serial driver example
should use early_platform_init("earlyprintk", &platform_driver).

使用 early_platform_init() 编译的平台驱动程序将在步骤 2 或 3 中自动注册。早期串行驱动程序示例应使用 early_platform_init("earlyprintk", &platform_driver)。

## 5.Probing of early platform drivers belonging to a certain class
## 5.对属于某个类的早期平台驱动程序进行探测

The architecture code calls early_platform_driver_probe() to match
registered early platform devices associated with a certain class with
registered early platform drivers. Matched devices will get probed().
This step can be executed at any point during the early boot. As soon
as possible may be good for the serial port case.

架构代码调用 early_platform_driver_probe() 来匹配与已注册的属于某个类的早期平台驱动程序相关联的早期平台设备。匹配的设备将被探测()。此步骤可以在早期引导期间的任何时候执行。在早期引导期间尽早执行可能是不错的选择，特别是对于串行端口的情况。

## 6.Inside the early platform driver probe()
## 6.在早期平台驱动程序探测() 内部

The driver code needs to take special care during early boot, especially
when it comes to memory allocation and interrupt registration. The code
in the probe() function can use is_early_platform_device() to check if
it is called at early platform device or at the regular platform device
time. The early serial driver performs register_console() at this point.

当涉及到内存分配和中断注册时，驱动程序代码在早期引导阶段需要特别小心。probe() 函数中的代码可以使用 is_early_platform_device() 来检查它是在早期平台设备初始化时调用还是在常规平台设备初始化时调用。早期串行驱动在此时执行 register_console()。

For further information, see <linux/platform_device.h>.