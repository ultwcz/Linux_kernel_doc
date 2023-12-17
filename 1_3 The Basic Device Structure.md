The Basic Device Structure
==========================

See the kerneldoc for the struct device.

请参阅结构体device的内核文档。

**Programming Interface**

The bus driver that discovers the device uses this to register the
device with the core

发现设备的总线驱动程序使用以下方式将设备注册到核心：

```c
  int device_register(struct device * dev);
```

The bus should initialize the following fields:

总线应该初始化以下字段：


    - parent
    - name
    - bus_id
    - bus

A device is removed from the core when its reference count goes to
0. The reference count can be adjusted using::

设备从核心中移除时，其引用计数变为0。可以使用以下方式调整引用计数：

```c
  struct device * get_device(struct device * dev);
  void put_device(struct device * dev);
```

get_device() will return a pointer to the struct device passed to it
if the reference is not already 0 (if it's in the process of being
removed already).

如果引用计数尚未为0（如果正在删除该引用），`get_device()`将返回传递给它的`struct device`的指针。

A driver can access the lock in the device structure using::

驱动程序可以使用以下方式访问设备结构中的锁：

```c
  void lock_device(struct device * dev);
  void unlock_device(struct device * dev);
```

**Attributes**

**属性**

```c
  struct device_attribute {
	struct attribute	attr;
	ssize_t (*show)(struct device *dev, struct device_attribute *attr,
			char *buf);
	ssize_t (*store)(struct device *dev, struct device_attribute *attr,
			 const char *buf, size_t count);
  };
```

Attributes of devices can be exported by a device driver through sysfs.

设备的属性可以通过设备驱动程序通过sysfs导出。

Please see Documentation/filesystems/sysfs.rst for more information
on how sysfs works.

请参阅 Documentation/filesystems/sysfs.rst 以获取有关 sysfs 如何工作的更多信息。

As explained in Documentation/core-api/kobject.rst, device attributes must be
created before the KOBJ_ADD uevent is generated. The only way to realize
that is by defining an attribute group.

如 Documentation/core-api/kobject.rst 中所述，设备属性必须在生成 KOBJ_ADD uevent 之前创建。唯一实现这一点的方法是通过定义属性组。


Attributes are declared using a macro called DEVICE_ATTR

属性使用名为 DEVICE_ATTR 的宏声明：

```c
  #define DEVICE_ATTR(name,mode,show,store)
```

Example

示例:

```c
  static DEVICE_ATTR(type, 0444, type_show, NULL);
  static DEVICE_ATTR(power, 0644, power_show, power_store);
```

Helper macros are available for common values of mode, so the above examples can be simplified to

对于 mode 的常见值，提供了辅助宏，因此上述示例可以简化为：

```c
  static DEVICE_ATTR_RO(type);
  static DEVICE_ATTR_RW(power);
```

This declares two structures of type struct device_attribute with respective
names 'dev_attr_type' and 'dev_attr_power'. These two attributes can be
organized as follows into a group

这将声明两个类型为`struct device_attribute`的结构体，分别命名为 'dev_attr_type' 和 'dev_attr_power'。这两个属性可以组织如下到一个组中：

```c
  static struct attribute *dev_attrs[] = {
	&dev_attr_type.attr,
	&dev_attr_power.attr,
	NULL,
  };

  static struct attribute_group dev_group = {
	.attrs = dev_attrs,
  };

  static const struct attribute_group *dev_groups[] = {
	&dev_group,
	NULL,
  };
```

A helper macro is available for the common case of a single group, so the
above two structures can be declared using

对于单个组的常见情况，提供了一个辅助宏，因此上述两个结构可以使用以下方式声明：

```c
ATTRIBUTE_GROUPS(dev);
```

This array of groups can then be associated with a device by setting the
group pointer in struct device before device_register() is invoked

然后，可以通过在调用 device_register() 之前设置 struct device 中的 group 指针来将该数组与设备关联起来：

```c
dev->groups = dev_groups;
device_register(dev);
```

The device_register() function will use the 'groups' pointer to create the
device attributes and the device_unregister() function will use this pointer
to remove the device attributes.

device_register() 函数将使用 'groups' 指针创建设备属性，而 device_unregister() 函数将使用此指针删除设备属性。

Word of warning:  While the kernel allows device_create_file() and device_remove_file() to be called on a device at any time, userspace has
strict expectations on when attributes get created.  When a new device is
registered in the kernel, a uevent is generated to notify userspace (like
udev) that a new device is available.  If attributes are added after the
device is registered, then userspace won't get notified and userspace will
not know about the new attributes.

警告：虽然内核允许在任何时候调用 device_create_file() 和 device_remove_file() 来操作设备，但用户空间对属性创建时间有严格的期望。当在内核中注册新设备时，将生成一个 uevent 以通知用户空间（如 udev）有新设备可用。如果在设备注册后添加属性，那么用户空间将不会收到通知，用户空间将不会知道新属性的存在。

This is important for device driver that need to publish additional
attributes for a device at driver probe time.  If the device driver simply
calls device_create_file() on the device structure passed to it, then
userspace will never be notified of the new attributes.

对于需要在驱动程序探测时为设备发布附加属性的设备驱动程序而言，这一点非常重要。如果设备驱动程序仅在其传递的设备结构上调用 device_create_file()，则用户空间永远不会收到新属性的通知。
