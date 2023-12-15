Video4Linux devices
-------------------

2.1. Introduction

2.2. Structure of a V4L driver

2.3. Structure of the V4L2 framework

2.4. Video device's internal representation

2.5. V4L2 device instance

2.6. V4L2 File handlers

2.7. V4L2 sub-devices

2.8. V4L2 sub-device userspace API

2.9. Read-only sub-device userspace API

2.10. I2C sub-device drivers

2.11. Centrally managed subdev active state

2.12. Streams, multiplexed media pads and internal routing

2.13. V4L2 sub-device functions and data structures

2.14. V4L2 events

2.15. V4L2 Controls

2.16. V4L2 videobuf2 functions and data structures

2.17. V4L2 DV Timings functions

2.18. V4L2 flash functions and data structures

2.19. V4L2 Media Controller
functions and data structures

2.20. V4L2 Media Bus functions and data structures

2.21. V4L2 Memory to Memory functions and data structures

2.22. V4L2 async kAPI

2.23. V4L2 fwnode kAPI

2.24. V4L2 CCI kAPI

2.25. V4L2 rect helper functions

2.26. Tuner functions and data structures

2.27. V4L2 common functions and data structures
s

2.28. Hauppauge TV EEPROM functions and data structures


# 2.1 Introduction

The V4L2 drivers tend to be very complex due to the complexity of the
hardware: most devices have multiple ICs, export multiple device nodes in
/dev, and create also non-V4L2 devices such as DVB, ALSA, FB, I2C and input
(IR) devices.

由于硬件的复杂性，V4L2（Video4Linux2）驱动往往非常复杂：大多数设备拥有多个IC，在/dev中导出多个设备节点，还会创建非V4L2设备，如DVB、ALSA、FB、I2C和输入（IR）设备。

Especially the fact that V4L2 drivers have to setup supporting ICs to
do audio/video muxing/encoding/decoding makes it more complex than most.
Usually these ICs are connected to the main bridge driver through one or
more I2C buses, but other buses can also be used. Such devices are
called 'sub-devices'.

特别是V4L2驱动必须设置支持的IC以执行音频/视频多路复用/编码/解码，使其比大多数驱动更为复杂。通常，这些IC通过一个或多个I2C总线连接到主桥接驱动程序，但也可以使用其他总线。这些设备被称为“子设备”。

For a long time the framework was limited to the video_device struct for
creating V4L device nodes and video_buf for handling the video buffers
(note that this document does not discuss the video_buf framework).

长时间以来，该框架仅限于使用video_device结构创建V4L设备节点和使用video_buf处理视频缓冲区（请注意，本文档不讨论video_buf框架）。

This meant that all drivers had to do the setup of device instances and
connecting to sub-devices themselves. Some of this is quite complicated
to do right and many drivers never did do it correctly.

这意味着所有驱动程序都必须自行设置设备实例并连接到子设备。某些操作确实相当复杂，许多驱动程序从未能正确执行。

There is also a lot of common code that could never be refactored due to
the lack of a framework.

由于缺乏框架，许多共同的代码也无法进行重构。

So this framework sets up the basic building blocks that all drivers
need and this same framework should make it much easier to refactor
common code into utility functions shared by all drivers.

因此，该框架设置了所有驱动程序都需要的基本构建块，而且同一框架应该能够更轻松地将共同的代码重构为所有驱动程序共享的实用函数。

A good example to look at as a reference is the v4l2-pci-skeleton.c
source that is available in samples/v4l/. It is a skeleton driver for
a PCI capture card, and demonstrates how to use the V4L2 driver
framework. It can be used as a template for real PCI video capture driver.

一个很好的参考示例是samples/v4l/目录中提供的v4l2-pci-skeleton.c源文件。它是一个PCI捕获卡的骨架驱动程序，演示了如何使用V4L2驱动程序框架。它可以用作实际PCI视频捕获驱动程序的模板。

# 2.2 Structure of a V4L driver

All drivers have the following structure:

所有驱动程序都具有以下结构：

- 1) A struct for each device instance containing the device state.

- 1) 每个设备实例都有一个结构，包含设备状态。


- 2) A way of initializing and commanding sub-devices (if any).

- 2) 初始化和控制子设备（如果有）的方法。

- 3) Creating V4L2 device nodes (/dev/videoX, /dev/vbiX and /dev/radioX)
   and keeping track of device-node specific data.

- 3) 创建V4L2设备节点（/dev/videoX、/dev/vbiX和/dev/radioX），并跟踪设备节点特定的数据。

- 4) Filehandle-specific structs containing per-filehandle data;

- 4) 包含每个文件句柄数据的文件句柄特定结构；

- 5) video buffer handling.

- 5) 处理视频缓冲区。

This is a rough schematic of how it all relates:

这是一个大致的关系图：

    device instances
      |
      +-sub-device instances
      |
      \-V4L2 device nodes
	       |
	       \-filehandle instances


# 2.3 Structure of the V4L2 framework

The framework closely resembles the driver structure: it has a v4l2_device
struct for the device instance data, a v4l2_subdev struct to refer to
sub-device instances, the video_device struct stores V4L2 device node data
and the v4l2_fh struct keeps track of filehandle instances.

该框架与驱动程序结构非常相似：它有一个v4l2_device结构，用于存储设备实例数据，一个v4l2_subdev结构用于引用子设备实例，video_device结构存储V4L2设备节点数据，v4l2_fh结构用于跟踪文件句柄实例。

The V4L2 framework also optionally integrates with the media framework. If a
driver sets the struct v4l2_device mdev field, sub-devices and video nodes
will automatically appear in the media framework as entities.

V4L2框架还可以选择与媒体框架集成。如果驱动程序设置了struct v4l2_device mdev字段，子设备和视频节点将自动出现在媒体框架中作为实体。


# 2.4 Video device' s internal representation

The actual device nodes in the ``/dev`` directory are created using the `video_device` struct (``v4l2-dev.h``). This struct can either be allocated dynamically or embedded in a larger struct.

在 `/dev` 目录中，实际设备节点是使用 `video_device` 结构（`v4l2-dev.h`）创建的。该结构可以动态分配，也可以嵌入到较大的结构中。

To allocate it dynamically use `video_device_alloc`:

要动态分配它，请使用 `video_device_alloc`：

```c
struct video_device *vdev = video_device_alloc();

if (vdev == NULL)
    return -ENOMEM;

vdev->release = video_device_release;
```

If you embed it in a larger struct, then you must set the ``release()`` callback to your own function:

如果将其嵌入到较大的结构中，那么您必须设置 `release()` 到自己的回调函数：

```c
struct video_device *vdev = &my_vdev->vdev;

vdev->release = my_vdev_release;
```

The ``release()`` callback must be set and it is called when the last user of the video device exits.

`release()` 回调必须被设置，当视频设备的最后一个用户退出时，将调用它。

The default `video_device_release` callback currently just calls ``kfree`` to free the allocated memory.

默认的 video_device_release 回调目前只是调用 kfree 以释放分配的内存。

There is also a `video_device_release_empty` function that does nothing (is empty) and should be used if the struct is embedded and there is nothing to do when it is released.

还有一个名为 video_device_release_empty 的函数，什么也不做（为空），如果结构是嵌入的且在释放时不需要进行任何操作，则应使用它。

You should also set these fields of `video_device`:

还应设置 video_device 的以下字段：

- `video_device`->v4l2_dev: must be set to the `v4l2_device` parent device.

- `video_device`->v4l2_dev: 必须设置为 v4l2_device 的父设备。

- `video_device`->name: set to something descriptive and unique.

- `video_device`->name: 设置为描述性和唯一的名称。

- `video_device`->vfl_dir: set this to ``VFL_DIR_RX`` for capture devices (``VFL_DIR_RX`` has value 0, so this is normally already the default), set to ``VFL_DIR_TX`` for output devices and ``VFL_DIR_M2M`` for mem2mem (codec) devices.

- `video_device`->vfl_dir: 为 捕获设备设置为 VFL_DIR_RX（VFL_DIR_RX 的值为 0，所以这通常已经是默认值），为输出设备设置为 VFL_DIR_TX，对于 mem2mem（编解码器）设备设置为 VFL_DIR_M2M。

- `video_device`->fops: set to the `v4l2_file_operations` struct.

- `video_device`->fops: 设置为 v4l2_file_operations 结构。

- `video_device`->ioctl_ops: if you use the `v4l2_ioctl_ops` to simplify ioctl maintenance (highly recommended to use this and it might become compulsory in the future!), then set this to your `v4l2_ioctl_ops` struct. The `video_device`->vfl_type and `video_device`->vfl_dir fields are used to disable ops that do not match the type/dir combination. E.g. VBI ops are disabled for non-VBI nodes, and output ops  are disabled for a capture device. This makes it possible to provide just one `v4l2_ioctl_ops` struct for both vbi and video nodes.

- `video_device`->ioctl_ops: 如果使用 v4l2_ioctl_ops 来简化 ioctl 的维护（强烈建议使用它，未来可能成为强制要求），则设置为您的 v4l2_ioctl_ops 结构。video_device->vfl_type 和 video_device->vfl_dir 字段用于禁用与类型/方向组合不匹配的操作。例如，对于非 VBI 节点，会禁用 VBI 操作，对于捕获设备会禁用输出操作。这使得可以为 vbi 和视频节点提供一个 v4l2_ioctl_ops 结构。

- `video_device`->lock: leave to ``NULL`` if you want to do all the locking  in the driver. Otherwise you give it a pointer to a struct ``mutex_lock`` and before the `video_device`->unlocked_ioctl file operation is called this lock will be taken by the core and released afterwards. See the next section for more details.

- `video_device`->lock: 如果要在驱动程序中执行所有锁定操作，则将其保留为空。否则，将其指针指定为指向 mutex_lock 结构的指针，在调用 video_device->unlocked_ioctl 文件操作之前，核心将获取此锁定并在之后释放。有关更多详细信息，请参阅下一节。

- `video_device`->queue: a pointer to the struct vb2_queue associated with this device node. If queue is not ``NULL``, and queue->lock is not ``NULL``, then queue->lock is used for the queuing ioctls (``VIDIOC_REQBUFS``, ``CREATE_BUFS``, ``QBUF``, ``DQBUF``,  ``QUERYBUF``, ``PREPARE_BUF``, ``STREAMON`` and ``STREAMOFF``) instead of the lock above. That way the :ref:`vb2 <vb2_framework>` queuing framework does not have to wait for other ioctls.   This queue pointer is also used by the :ref:`vb2 <vb2_framework>` helper functions to check for queuing ownership (i.e. is the filehandle calling it allowed to do the operation).

- `video_device`->queue: 指向与此设备节点关联的 struct vb2_queue 的指针。如果 queue 不是 NULL，并且 queue->lock 不是 NULL，则 queue->lock 用于排队 ioctl（VIDIOC_REQBUFS，CREATE_BUFS，QBUF，DQBUF，QUERYBUF，PREPARE_BUF，STREAMON 和 STREAMOFF）而不是上面的锁。这样一来，vb2 排队框架不必等待其他 ioctl。这个队列指针也被 vb2 辅助函数用于检查排队所有权（即调用它的文件句柄是否允许执行该操作）。

- `video_device`->prio: keeps track of the priorities. Used to implement ``VIDIOC_G_PRIORITY`` and ``VIDIOC_S_PRIORITY``. If left to ``NULL``, then it will use the struct v4l2_prio_state in `v4l2_device`. If you want to have a separate priority state per (group of) device node(s),   then you can point it to your own struct `v4l2_prio_state`.

- `video_device`->prio: 跟踪优先级。用于实现 VIDIOC_G_PRIORITY 和 VIDIOC_S_PRIORITY。如果留为空，则将使用 v4l2_device 中的 struct v4l2_prio_state。如果要为每个（组）设备节点使用单独的优先级状态，则可以将其指向自己的 struct v4l2_prio_state 结构。

- `video_device`->dev_parent: you only set this if v4l2_device was registered with ``NULL`` as the parent ``device`` struct. This only happens in cases where one hardware device has multiple PCI devices that all share the same `v4l2_device` core.

- `video_device`->dev_parent: 只有在 v4l2_device 使用 NULL 作为父 device 结构注册时，才会设置这个。这仅发生在一个硬件设备有多个 PCI 设备，所有这些设备都共享相同 v4l2_device 核心的情况下。

The cx88 driver is an example of this: one core `v4l2_device` struct, but it is used by both a raw video PCI device (cx8800) and a MPEG PCI device (cx8802). Since the `v4l2_device` cannot be associated with two PCI devices at the same time it is setup without a parent device. But when the struct video_device is initialized you **do** know which parent PCI device to use and so you set ``dev_device`` to the correct PCI device.

cx88 驱动程序就是这种情况的例子：一个核心的 v4l2_device 结构，但它同时由原始视频 PCI 设备（cx8800）和 MPEG PCI 设备（cx8802）使用。由于 v4l2_device 不能同时关联两个 PCI 设备，因此它是在没有父设备的情况下设置的。但是，当初始化 struct video_device 时，确实知道要使用哪个父 PCI 设备，因此将 dev_device 设置为正确的 PCI 设备。

If you use `v4l2_ioctl_ops`, then you should set `video_device`->unlocked_ioctl to `video_ioctl2` in your `v4l2_file_operations` struct.

如果使用 v4l2_ioctl_ops，则应将 video_device->unlocked_ioctl 设置为 video_ioctl2 在您的 v4l2_file_operations 结构中。

In some cases you want to tell the core that a function you had specified in your `v4l2_ioctl_ops` should be ignored. You can mark such ioctls by calling this function before `video_register_device` is called:

在某些情况下，您可能希望告诉核心在 v4l2_ioctl_ops 中指定的功能应被忽略。您可以在调用 video_register_device 之前通过调用此函数标记这样的 ioctl：

    v4l2_disable_ioctl(vdev, cmd)

This tends to be needed if based on external factors (e.g. which card is being used) you want to turns off certain features in `v4l2_ioctl_ops` without having to make a new struct.

如果基于外部因素（例如使用的卡）您想要关闭 v4l2_ioctl_ops 中的某些功能而无需制作新的结构，这可能是必需的。

The `v4l2_file_operations` struct is a subset of file_operations. The main difference is that the inode argument is omitted since it is never used.

v4l2_file_operations 结构是 file_operations 的一个子集。主要区别在于省略了 inode 参数，因为从未使用。

If integration with the media framework is needed, you must initialize the `media_entity` struct embedded in the `video_device` struct (entity field) by calling `media_entity_pads_init`:

如果需要与媒体框架集成，必须通过调用 media_entity_pads_init 初始化嵌入在 video_device 结构中的 media_entity 结构（entity 字段）：

```c
struct media_pad *pad = &my_vdev->pad;
int err;

err = media_entity_pads_init(&vdev->entity, 1, pad);
```

The pads array must have been previously initialized. There is no need to manually set the struct media_entity type and name fields.

pads 数组必须先前初始化。无需手动设置 struct media_entity 的类型和名称字段。

A reference to the entity will be automatically acquired/released when the video device is opened/closed.

打开视频设备时，将自动获取/释放对实体的引用。

## 2.4.1 ioctls and locking

The V4L core provides optional locking services. The main service is the lock field in struct video_device, which is a pointer to a mutex. If you set this pointer, then that will be used by unlocked_ioctl to serialize all ioctls.

V4L 核心提供了可选的锁定服务。主要服务是 video_device 结构中的 lock 字段，它是指向互斥体的指针。如果设置了此指针，那么它将被 unlocked_ioctl 使用以串行化所有 ioctl。

If you are using the `videobuf2 framework` , then there is a second lock that you can set: `video_device`->queue->lock. If set, then this lock will be used instead of `video_device`->lock to serialize all queuing ioctls (see the previous section for the full list of those ioctls).

如果正在使用 videobuf2 框架 ，那么还有第二个锁，可以设置：video_device->queue->lock。如果设置，那么该锁将用于串行化所有排队 ioctl（请参见上一节以获取完整列表）而不是 video_device->lock。

The advantage of using a different lock for the queuing ioctls is that for some drivers (particularly USB drivers) certain commands such as setting controls can take a long time, so you want to use a separate lock for the buffer queuing ioctls. That way your ``VIDIOC_DQBUF`` doesn't stall because the driver is busy changing the e.g. exposure of the webcam.

使用不同的锁查询 ioctl 的优势在于，对于某些驱动程序（特别是 USB 驱动程序），某些命令，如设置控件，可能需要很长时间，因此您希望为缓冲区 queuing ioctl 使用单独的锁。这样，如果驱动程序正在忙于更改网络摄像头的曝光时间，您的 VIDIOC_DQBUF 就不会停滞。

Of course, you can always do all the locking yourself by leaving both lock pointers at ``NULL``.

当然，您始终可以通过将两个锁指针保留为空来自行完成所有锁定。

In the case of `videobuf2 framework>` you will need to implement the ``wait_prepare()`` and ``wait_finish()`` callbacks to unlock/lock if applicable. If you use the ``queue->lock`` pointer, then you can use the helper functions `vb2_ops_wait_prepare` and `vb2_ops_wait_finish`.

在 videobuf2 框架的情况下，如果适用，您将需要实现 wait_prepare() 和 wait_finish() 回调以进行解锁/锁定。如果使用 queue->lock 指针，则可以使用帮助函数 vb2_ops_wait_prepare 和 vb2_ops_wait_finish。

The implementation of a hotplug disconnect should also take the lock from `video_device` before calling v4l2_device_disconnect. If you are also using `video_device`->queue->lock, then you have to first lock `video_device`->queue->lock followed by `video_device`->lock. That way you can be sure no ioctl is running when you call `v4l2_device_disconnect`.

## 2.4.2 Video device registration

Next you register the video device with `video_register_device`. This will create the character device for you.

```c
err = video_register_device(vdev, VFL_TYPE_VIDEO, -1);
if (err) {
    video_device_release(vdev); /* or kfree(my_vdev); */
	return err;
}
```

If the `v4l2_device` parent device has a not ``NULL`` mdev field, the video device entity will be automatically registered with the media device.

Which device is registered depends on the type argument. The following types exist:

| `vfl_devnode_type` | Device name | Usage |
| ------------------ | ----------- | ----- |
| ``VFL_TYPE_VIDEO`` | ``/dev/videoX`` | for video input/output devices |
| ``VFL_TYPE_VBI`` | ``/dev/vbiX`` | for vertical blank data (i.e. closed captions, teletext) |
| ``VFL_TYPE_RADIO`` | ``/dev/radioX`` | for radio tuners |
| ``VFL_TYPE_SUBDEV`` | ``/dev/v4l-subdevX`` | for V4L2 subdevices |
| ``VFL_TYPE_SDR`` | ``/dev/swradioX`` | for Software Defined Radio(SDR) tuners |
| ``VFL_TYPE_TOUCH`` | ``/dev/v4l-touchX`` | for touch sensors |

The last argument gives you a certain amount of control over the device node number used (i.e. the X in ``videoX``). Normally you will pass -1 to let the v4l2 framework pick the first free number. But sometimes users want to select a specific node number. It is common that drivers allow the user to select a specific device node number through a driver module option. That number is then passed to this function and video_register_device will attempt to select that device node number. If that number was already in use, then the next free device node number will be selected and it will send a warning to the kernel log.

Another use-case is if a driver creates many devices. In that case it can be useful to place different video devices in separate ranges. For example, video capture devices start at 0, video output devices start at 16. So you can use the last argument to specify a minimum device node number and the v4l2 framework will try to pick the first free number that is equal or higher to what you passed. If that fails, then it will just pick the first free number.

Since in this case you do not care about a warning about not being able to select the specified device node number, you can call the function `video_register_device_no_warn` instead.

Whenever a device node is created some attributes are also created for you. If you look in ``/sys/class/video4linux`` you see the devices. Go into e.g. ``video0`` and you will see 'name', 'dev_debug' and 'index' attributes. The 'name' attribute is the 'name' field of the video_device struct. The 'dev_debug' attribute can be used to enable core debugging. See the next section for more detailed information on this.

The 'index' attribute is the index of the device node: for each call to `video_register_device()` the index is just increased by 1. The first video device node you register always starts with index 0.

Users can setup udev rules that utilize the index attribute to make fancy device names (e.g. '``mpegX``' for MPEG video capture device nodes).

After the device was successfully registered, then you can use these fields:

- `video_device`->vfl_type: the device type passed to `video_register_device`.
- `video_device`->minor: the assigned device minor number.
- `video_device`->num: the device node number (i.e. the X in ``videoX``).
- `video_device`->index: the device index number.

If the registration failed, then you need to call `video_device_release` to free the allocated `video_device`
struct, or free your own struct if the  `video_device` was embedded in it. The ``vdev->release()`` callback will never be called if the registration failed, nor should you ever attempt to unregister the device if the registration failed.

## 2.4.3 video device debugging

The 'dev_debug' attribute that is created for each video, vbi, radio or swradio device in ``/sys/class/video4linux/<devX>/`` allows you to enable logging of file operations.

It is a bitmask and the following bits can be set:

| Mask | Description |
| ---- | ----------- |
| 0x01 | Log the ioctl name and error code. VIDIOC_(D)QBUF ioctls are only logged if bit 0x08 is also set. |
| 0x02 | Log the ioctl name arguments and error code. VIDIOC_(D)QBUF ioctls are only logged if bit 0x08 is also set. |
| 0x04 | Log the file operations open, release, read, write, mmap and get_unmapped_area. The read and write operations are only logged if bit 0x08 is also set. |
| 0x08 | Log the read and write file operations and the VIDIOC_QBUF and VIDIOC_DQBUF ioctls. |
| 0x10 | Log the poll file operation. |
| 0x20 | Log error and messages in the control operations. |

## 2.4.4 Video device cleanup

When the video device nodes have to be removed, either during the unload of the driver or because the USB device was disconnected, then you should
unregister them with:

	video_unregister_device() (vdev);

This will remove the device nodes from sysfs (causing udev to remove them from ``/dev``).

After `video_unregister_device` returns no new opens can be done. However, in the case of USB devices some application might still have one of these device nodes open. So after the unregister all file operations (except release, of course) will return an error as well.

When the last user of the video device node exits, then the ``vdev->release()`` callback is called and you can do the final cleanup there.

Don't forget to cleanup the media entity associated with the video device if it has been initialized:

	 media_entity_cleanup(&vdev->entity);

This can be done from the release callback.


## 2.4.5 helper functions

There are a few useful helper functions:

- file and `video_device` private data

You can set/get driver private data in the video_device struct using:

    video_get_drvdata(vdev);
	video_set_drvdata(vdev);

Note that you can safely call   `video_set_drvdata` before calling `video_register_device`.

And this function:

	video_devdata(struct file *file);

returns the video_device belonging to the file struct.

The `video_devdata` function combines `video_get_drvdata` with `video_devdata`:

    video_drvdata (struct file *file);

You can go from a `video_device` struct to the v4l2_device struct using:

```c
struct v4l2_device *v4l2_dev = vdev->v4l2_dev;
```

- Device node name

The `video_device` node kernel name can be retrieved using:

    video_device_node_name(vdev);

The name is used as a hint by userspace tools such as udev. The functionshould be used where possible instead of accessing the video_device::num and video_device::minor fields.

## 2.4.6 video_device functions and data structures

reference:

    include/media/v4l2-dev.h
