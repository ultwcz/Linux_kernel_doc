Devres - Managed Device Resource
================================

Tejun Heo	<teheo@suse.de>

First draft	10 January 2007
>
   1. Intro			: Huh? Devres?
   2. Devres			: Devres in a nutshell
   3. Devres Group		: Group devres'es and release them together
   4. Details			: Life time rules, calling context, ...
   5. Overhead			: How much do we have to pay for this?
   6. List of managed interfaces: Currently implemented managed interfaces

> 
  1. 介绍 : 什么是Devres？
  2. Devres : 简要了解Devres
  3. Devres组 : 对组的Devres进行分组并一起释放
  4. 详细信息 : 生命周期规则，调用上下文，...
  5. 开销 : 我们需要为此付出多少？
  6. 托管接口列表 : 当前实现的托管接口


## 1. Intro
--------

devres came up while trying to convert libata to use iomap.  Each
iomapped address should be kept and unmapped on driver detach.  For
example, a plain SFF ATA controller (that is, good old PCI IDE) in
native mode makes use of 5 PCI BARs and all of them should be
maintained.

在尝试将libata转换为使用iomap时，出现了Devres。每个iomapped地址应在驱动程序分离时保持并取消映射。例如，本机模式下的普通SFF ATA控制器（即老式PCI IDE）使用5个PCI BARs，所有这些BAR都应该被维护。

As with many other device drivers, libata low level drivers have
sufficient bugs in ->remove and ->probe failure path.  Well, yes,
that's probably because libata low level driver developers are lazy
bunch, but aren't all low level driver developers?  After spending a
day fiddling with braindamaged hardware with no document or
braindamaged document, if it's finally working, well, it's working.

与许多其他设备驱动程序一样，libata低级别驱动程序在->remove和->probe失败路径上存在足够的错误。好吧，是的，这可能是因为libata低级别驱动程序开发人员是一群懒惰的家伙，但所有低级别驱动程序开发人员都是如此吗？在花费一天的时间摆弄没有文档或有缺陷的文档的脑残硬件后，如果最终工作了，那就是工作了。

For one reason or another, low level drivers don't receive as much
attention or testing as core code, and bugs on driver detach or
initialization failure don't happen often enough to be noticeable.
Init failure path is worse because it's much less travelled while
needs to handle multiple entry points.

由于某种原因，低级别驱动程序没有得到与核心代码一样多的关注或测试，而驱动程序分离或初始化失败时的错误不太容易被注意到。初始化失败路径更糟糕，因为它很少被访问，但需要处理多个入口点。

So, many low level drivers end up leaking resources on driver detach
and having half broken failure path implementation in ->probe() which
would leak resources or even cause oops when failure occurs.  iomap
adds more to this mix.  So do msi and msix.

因此，许多低级别驱动程序最终在驱动程序分离时泄漏资源，并在->probe()中具有半破碎的失败路径实现，这将泄漏资源甚至在发生故障时导致oops。iomap为这个混合物增加了更多内容。msi和msix也是如此。

## 2. Devres
---------

devres is basically linked list of arbitrarily sized memory areas
associated with a struct device.  Each devres entry is associated with
a release function.  A devres can be released in several ways.  No
matter what, all devres entries are released on driver detach.  On
release, the associated release function is invoked and then the
devres entry is freed.

Devres基本上是与struct device关联的任意大小内存区域的链表。每个devres条目都与一个释放函数相关联。Devres可以通过几种方式释放。无论如何，所有devres条目都在驱动程序分离时释放。在释放时，调用相关的释放函数，然后释放devres条目。

Managed interface is created for resources commonly used by device
drivers using devres.  For example, coherent DMA memory is acquired
using dma_alloc_coherent().  The managed version is called
dmam_alloc_coherent().  It is identical to dma_alloc_coherent() except
for the DMA memory allocated using it is managed and will be
automatically released on driver detach.  Implementation looks like
the following

使用devres，为设备驱动程序常用的资源创建了托管接口。例如，使用dma_alloc_coherent()获取协调DMA内存。托管版本称为dmam_alloc_coherent()。它与dma_alloc_coherent()相同，只是使用它分配的DMA内存是受管理的，并将在驱动程序分离时自动释放。实现如下：

```c
  struct dma_devres {
	size_t		size;
	void		*vaddr;
	dma_addr_t	dma_handle;
  };

  static void dmam_coherent_release(struct device *dev, void *res)
  {
	struct dma_devres *this = res;

	dma_free_coherent(dev, this->size, this->vaddr, this->dma_handle);
  }

  dmam_alloc_coherent(dev, size, dma_handle, gfp)
  {
	struct dma_devres *dr;
	void *vaddr;

	dr = devres_alloc(dmam_coherent_release, sizeof(*dr), gfp);
	...

	/* alloc DMA memory as usual */
	vaddr = dma_alloc_coherent(...);
	...

	/* record size, vaddr, dma_handle in dr */
	dr->vaddr = vaddr;
	...

	devres_add(dev, dr);

	return vaddr;
  }
```

If a driver uses dmam_alloc_coherent(), the area is guaranteed to be
freed whether initialization fails half-way or the device gets
detached.  If most resources are acquired using managed interface, a
driver can have much simpler init and exit code.  Init path basically
looks like the following::

如果驱动程序使用dmam_alloc_coherent()，则该区域将在初始化失败或设备分离时得到保证释放。如果大多数资源使用托管接口获取，驱动程序的初始化和退出代码可以更简单。初始化路径基本上如下所示：

```c
  my_init_one()
  {
	struct mydev *d;

	d = devm_kzalloc(dev, sizeof(*d), GFP_KERNEL);
	if (!d)
		return -ENOMEM;

	d->ring = dmam_alloc_coherent(...);
	if (!d->ring)
		return -ENOMEM;

	if (check something)
		return -EINVAL;
	...

	return register_to_upper_layer(d);
  }
```

And exit path

退出路径如下：

```c
  my_remove_one()
  {
	unregister_from_upper_layer(d);
	shutdown_my_hardware();
  }
```

As shown above, low level drivers can be simplified a lot by using
devres.  Complexity is shifted from less maintained low level drivers
to better maintained higher layer.  Also, as init failure path is
shared with exit path, both can get more testing.

如上所示，通过使用devres，可以大大简化低级别驱动程序。复杂性从维护较差的低级别驱动程序转移到维护较好的高层。而且，由于初始化失败路径与退出路径共享，两者都可以得到更多的测试。

Note though that when converting current calls or assignments to
managed devm_* versions it is up to you to check if internal operations
like allocating memory, have failed. Managed resources pertains to the
freeing of these resources *only* - all other checks needed are still
on you. In some cases this may mean introducing checks that were not
necessary before moving to the managed devm_* calls.

请注意，尽管在将当前调用或分配转换为托管的devm_版本时，您需要检查内部操作（如内存分配）是否失败。托管资源仅涉及释放这些资源 - 所有其他所需的检查仍由您完成。在某些情况下，这可能意味着引入以前在移至托管的devm_调用之前不必要的检查。

## 3. Devres group
---------------

Devres entries can be grouped using devres group.  When a group is
released, all contained normal devres entries and properly nested
groups are released.  One usage is to rollback series of acquired
resources on failure.  For example:

Devres条目可以使用Devres组进行分组。当释放组时，所有包含的正常Devres条目和正确嵌套的组都将被释放。一种用法是在失败时回滚一系列获取的资源。例如：

```c
  if (!devres_open_group(dev, NULL, GFP_KERNEL))
	return -ENOMEM;

  acquire A;
  if (failed)
	goto err;

  acquire B;
  if (failed)
	goto err;
  ...

  devres_remove_group(dev, NULL);
  return 0;

 err:
  devres_release_group(dev, NULL);
  return err_code;
```

As resource acquisition failure usually means probe failure, constructs
like above are usually useful in midlayer driver (e.g. libata core
layer) where interface function shouldn't have side effect on failure.
For LLDs, just returning error code suffices in most cases.

由于资源获取失败通常意味着探测失败，因此像上面的结构通常在中间层驱动程序（例如libata核心层）中很有用，其中接口函数不应在失败时具有副作用。对于LLDs，只返回错误代码通常就足够了。

Each group is identified by `void *id`.  It can either be explicitly
specified by @id argument to devres_open_group() or automatically
created by passing NULL as @id as in the above example.  In both
cases, devres_open_group() returns the group's id.  The returned id
can be passed to other devres functions to select the target group.
If NULL is given to those functions, the latest open group is
selected.

每个组由void *id标识。它可以通过在devres_open_group()的@id参数中显式指定，也可以通过将@id传递为NULL而自动创建，就像上面的示例一样。在这两种情况下，devres_open_group()都会返回组的id。返回的id可以传递给其他devres函数，以选择目标组。如果将NULL传递给这些函数，将选择最新打开的组。

For example, you can do something like the following:

例如，可以执行以下操作：

```c
  int my_midlayer_create_something()
  {
	if (!devres_open_group(dev, my_midlayer_create_something, GFP_KERNEL))
		return -ENOMEM;

	...

	devres_close_group(dev, my_midlayer_create_something);
	return 0;
  }

  void my_midlayer_destroy_something()
  {
	devres_release_group(dev, my_midlayer_create_something);
  }
```

## 4. Details
----------

Lifetime of a devres entry begins on devres allocation and finishes
when it is released or destroyed (removed and freed) - no reference
counting.

Devres条目的生命周期始于devres分配并在释放或销毁（删除并释放）时结束 - 没有引用计数。

devres core guarantees atomicity to all basic devres operations and
has support for single-instance devres types (atomic
lookup-and-add-if-not-found).  Other than that, synchronizing
concurrent accesses to allocated devres data is caller's
responsibility.  This is usually non-issue because bus ops and
resource allocations already do the job.

Devres核心对所有基本devres操作都保证原子性，并支持单实例devres类型（原子查找并在未找到时添加）。除此之外，同步对分配的devres数据的并发访问是调用者的责任。这通常不是问题，因为总线操作和资源分配已经完成了这项工作。

For an example of single-instance devres type, read pcim_iomap_table()
in lib/devres.c.

有关单实例devres类型的示例，请参阅lib/devres.c中的pcim_iomap_table()。

All devres interface functions can be called without context if the
right gfp mask is given.

所有devres接口函数都可以在没有上下文的情况下调用，如果给定了正确的gfp掩码。

## 5. Overhead
-----------

Each devres bookkeeping info is allocated together with requested data
area.  With debug option turned off, bookkeeping info occupies 16
bytes on 32bit machines and 24 bytes on 64bit (three pointers rounded
up to ull alignment).  If singly linked list is used, it can be
reduced to two pointers (8 bytes on 32bit, 16 bytes on 64bit).

每个devres簿记信息与请求的数据区一起分配。在关闭调试选项的情况下，簿记信息在32位机器上占用16字节，在64位机器上占用24字节（三个指针向上取整到ull对齐）。如果使用单向链接列表，可以减少到两个指针（32位机器上为8字节，64位机器上为16字节）。

Each devres group occupies 8 pointers.  It can be reduced to 6 if
singly linked list is used.

每个devres组占用8个指针。如果使用单向链接列表，可以减少到6个。

Memory space overhead on ahci controller with two ports is between 300
and 400 bytes on 32bit machine after naive conversion (we can
certainly invest a bit more effort into libata core layer).

在具有两个端口的ahci控制器上的内存空间开销在天真转换后在32位机器上为300到400字节（我们肯定可以对libata核心层进行更多努力）。

## 6. List of managed interfaces
-----------------------------
```
CLOCK
  devm_clk_get()
  devm_clk_get_optional()
  devm_clk_put()
  devm_clk_bulk_get()
  devm_clk_bulk_get_all()
  devm_clk_bulk_get_optional()
  devm_get_clk_from_child()
  devm_clk_hw_register()
  devm_of_clk_add_hw_provider()
  devm_clk_hw_register_clkdev()

DMA
  dmaenginem_async_device_register()
  dmam_alloc_coherent()
  dmam_alloc_attrs()
  dmam_free_coherent()
  dmam_pool_create()
  dmam_pool_destroy()

DRM
  devm_drm_dev_alloc()

GPIO
  devm_gpiod_get()
  devm_gpiod_get_array()
  devm_gpiod_get_array_optional()
  devm_gpiod_get_index()
  devm_gpiod_get_index_optional()
  devm_gpiod_get_optional()
  devm_gpiod_put()
  devm_gpiod_unhinge()
  devm_gpiochip_add_data()
  devm_gpio_request()
  devm_gpio_request_one()

I2C
  devm_i2c_add_adapter()
  devm_i2c_new_dummy_device()

IIO
  devm_iio_device_alloc()
  devm_iio_device_register()
  devm_iio_dmaengine_buffer_setup()
  devm_iio_kfifo_buffer_setup()
  devm_iio_kfifo_buffer_setup_ext()
  devm_iio_map_array_register()
  devm_iio_triggered_buffer_setup()
  devm_iio_triggered_buffer_setup_ext()
  devm_iio_trigger_alloc()
  devm_iio_trigger_register()
  devm_iio_channel_get()
  devm_iio_channel_get_all()
  devm_iio_hw_consumer_alloc()
  devm_fwnode_iio_channel_get_by_name()

INPUT
  devm_input_allocate_device()

IO region
  devm_release_mem_region()
  devm_release_region()
  devm_release_resource()
  devm_request_mem_region()
  devm_request_free_mem_region()
  devm_request_region()
  devm_request_resource()

IOMAP
  devm_ioport_map()
  devm_ioport_unmap()
  devm_ioremap()
  devm_ioremap_uc()
  devm_ioremap_wc()
  devm_ioremap_resource() : checks resource, requests memory region, ioremaps
  devm_ioremap_resource_wc()
  devm_platform_ioremap_resource() : calls devm_ioremap_resource() for platform device
  devm_platform_ioremap_resource_byname()
  devm_platform_get_and_ioremap_resource()
  devm_iounmap()

  Note: For the PCI devices the specific pcim_*() functions may be used, see below.

IRQ
  devm_free_irq()
  devm_request_any_context_irq()
  devm_request_irq()
  devm_request_threaded_irq()
  devm_irq_alloc_descs()
  devm_irq_alloc_desc()
  devm_irq_alloc_desc_at()
  devm_irq_alloc_desc_from()
  devm_irq_alloc_descs_from()
  devm_irq_alloc_generic_chip()
  devm_irq_setup_generic_chip()
  devm_irq_domain_create_sim()

LED
  devm_led_classdev_register()
  devm_led_classdev_register_ext()
  devm_led_classdev_unregister()
  devm_led_trigger_register()
  devm_of_led_get()

MDIO
  devm_mdiobus_alloc()
  devm_mdiobus_alloc_size()
  devm_mdiobus_register()
  devm_of_mdiobus_register()

MEM
  devm_free_pages()
  devm_get_free_pages()
  devm_kasprintf()
  devm_kcalloc()
  devm_kfree()
  devm_kmalloc()
  devm_kmalloc_array()
  devm_kmemdup()
  devm_krealloc()
  devm_krealloc_array()
  devm_kstrdup()
  devm_kstrdup_const()
  devm_kvasprintf()
  devm_kzalloc()

MFD
  devm_mfd_add_devices()

MUX
  devm_mux_chip_alloc()
  devm_mux_chip_register()
  devm_mux_control_get()
  devm_mux_state_get()

NET
  devm_alloc_etherdev()
  devm_alloc_etherdev_mqs()
  devm_register_netdev()

PER-CPU MEM
  devm_alloc_percpu()
  devm_free_percpu()

PCI
  devm_pci_alloc_host_bridge()  : managed PCI host bridge allocation
  devm_pci_remap_cfgspace()	: ioremap PCI configuration space
  devm_pci_remap_cfg_resource()	: ioremap PCI configuration space resource

  pcim_enable_device()		: after success, all PCI ops become managed
  pcim_iomap()			: do iomap() on a single BAR
  pcim_iomap_regions()		: do request_region() and iomap() on multiple BARs
  pcim_iomap_regions_request_all() : do request_region() on all and iomap() on multiple BARs
  pcim_iomap_table()		: array of mapped addresses indexed by BAR
  pcim_iounmap()		: do iounmap() on a single BAR
  pcim_iounmap_regions()	: do iounmap() and release_region() on multiple BARs
  pcim_pin_device()		: keep PCI device enabled after release
  pcim_set_mwi()		: enable Memory-Write-Invalidate PCI transaction

PHY
  devm_usb_get_phy()
  devm_usb_get_phy_by_node()
  devm_usb_get_phy_by_phandle()
  devm_usb_put_phy()

PINCTRL
  devm_pinctrl_get()
  devm_pinctrl_put()
  devm_pinctrl_get_select()
  devm_pinctrl_register()
  devm_pinctrl_register_and_init()
  devm_pinctrl_unregister()

POWER
  devm_reboot_mode_register()
  devm_reboot_mode_unregister()

PWM
  devm_pwmchip_add()
  devm_pwm_get()
  devm_fwnode_pwm_get()

REGULATOR
  devm_regulator_bulk_register_supply_alias()
  devm_regulator_bulk_get()
  devm_regulator_bulk_get_const()
  devm_regulator_bulk_get_enable()
  devm_regulator_bulk_put()
  devm_regulator_get()
  devm_regulator_get_enable()
  devm_regulator_get_enable_optional()
  devm_regulator_get_exclusive()
  devm_regulator_get_optional()
  devm_regulator_irq_helper()
  devm_regulator_put()
  devm_regulator_register()
  devm_regulator_register_notifier()
  devm_regulator_register_supply_alias()
  devm_regulator_unregister_notifier()

RESET
  devm_reset_control_get()
  devm_reset_controller_register()

RTC
  devm_rtc_device_register()
  devm_rtc_allocate_device()
  devm_rtc_register_device()
  devm_rtc_nvmem_register()

SERDEV
  devm_serdev_device_open()

SLAVE DMA ENGINE
  devm_acpi_dma_controller_register()
  devm_acpi_dma_controller_free()

SPI
  devm_spi_alloc_master()
  devm_spi_alloc_slave()
  devm_spi_register_master()

WATCHDOG
  devm_watchdog_register_device()
```