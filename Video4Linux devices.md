Video4Linux devices
-------------------

- [2.1 Introduction](#21-introduction)
- [2.2 Structure of a V4L driver](#22-structure-of-a-v4l-driver)
- [2.3 Structure of the V4L2 framework](#23-structure-of-the-v4l2-framework)
- [2.4 Video device' s internal representation](#24-video-device-s-internal-representation)
	- [2.4.1 ioctls and locking](#241-ioctls-and-locking)
	- [2.4.2 Video device registration](#242-video-device-registration)
	- [2.4.3 video device debugging](#243-video-device-debugging)
	- [2.4.4 Video device cleanup](#244-video-device-cleanup)
	- [2.4.5 helper functions](#245-helper-functions)
	- [2.4.6 video\_device functions and data structures](#246-video_device-functions-and-data-structures)
- [2.5 V4L2 device instance](#25-v4l2-device-instance)
	- [2.5.1 v4l2\_device functions and data structures](#251-v4l2_device-functions-and-data-structures)
- [2.6 V4L2 File handlers](#26-v4l2-file-handlers)
	- [2.6.1 V4L2 fh functions and data structures](#261-v4l2-fh-functions-and-data-structures)
- [2.7 V4L2 sub-devices](#27-v4l2-sub-devices)
	- [2.7.1 Subdev registration](#271-subdev-registration)
		- [2.7.1.1 Registering synchronous sub-devices](#2711-registering-synchronous-sub-devices)
		- [2.7.1.2 Registering asynchronous sub-devices](#2712-registering-asynchronous-sub-devices)
		- [2.7.1.3 Asynchronous sub-device notifiers](#2713-asynchronous-sub-device-notifiers)
		- [2.7.1.4 Asynchronous sub-device notifier for sub-devices](#2714-asynchronous-sub-device-notifier-for-sub-devices)
		- [2.7.1.5 Asynchronous sub-device registration helper for camera sensor drivers](#2715-asynchronous-sub-device-registration-helper-for-camera-sensor-drivers)
		- [2.7.1.6 Asynchronous sub-device notifier example](#2716-asynchronous-sub-device-notifier-example)
		- [2.7.1.7 Asynchronous sub-device notifier callbacks](#2717-asynchronous-sub-device-notifier-callbacks)
	- [2.7.2 Calling subdev operations](#272-calling-subdev-operations)
- [2.8 V4L2 sub-device userspace API](#28-v4l2-sub-device-userspace-api)
- [2.9 Read-only sub-device userspace API](#29-read-only-sub-device-userspace-api)
- [2.10 I2C sub-device drivers](#210-i2c-sub-device-drivers)
- [2.11 Centrally managed subdev active state](#211-centrally-managed-subdev-active-state)
- [2.12 Streams, multiplexed media pads and internal routing](#212-streams-multiplexed-media-pads-and-internal-routing)
- [2.13 V4L2 sub-device functions and data structures](#213-v4l2-sub-device-functions-and-data-structures)
- [2.14 V4L2 events](#214-v4l2-events)
	- [2.14.1 Event subscription](#2141-event-subscription)
	- [2.14.2 Unsubscribing an event](#2142-unsubscribing-an-event)
	- [2.14.3 Check if there's a pending event](#2143-check-if-theres-a-pending-event)
	- [2.14.4 How events work](#2144-how-events-work)
		- [2.14.4.1 V4L2 event functions and data structures](#21441-v4l2-event-functions-and-data-structures)
- [2.15 V4L2 Controls](#215-v4l2-controls)
	- [2.15.1 Introduction](#2151-introduction)
	- [2.15.2 Objects in the framework](#2152-objects-in-the-framework)
	- [2.15.3 Basic usage for V4L2 and sub-device drivers](#2153-basic-usage-for-v4l2-and-sub-device-drivers)
	- [2.15.4 Inheriting Sub-device Controls](#2154-inheriting-sub-device-controls)
	- [2.15.5 Accessing Control Values](#2155-accessing-control-values)
	- [2.15.6 Menu Controls](#2156-menu-controls)
	- [2.15.7 Custom Controls](#2157-custom-controls)
	- [2.15.8 Active and Grabbed Controls](#2158-active-and-grabbed-controls)
	- [2.15.9 Control Clusters](#2159-control-clusters)
	- [2.15.10 Handling autogain/gain-type Controls with Auto Clusters](#21510-handling-autogaingain-type-controls-with-auto-clusters)
	- [2.15.11 VIDIOC\_LOG\_STATUS Support](#21511-vidioc_log_status-support)
	- [2.15.12 Different Handlers for Different Video Nodes](#21512-different-handlers-for-different-video-nodes)
	- [2.15.13 Finding Controls](#21513-finding-controls)
	- [2.15.14 Preventing Controls inheritance](#21514-preventing-controls-inheritance)
	- [2.15.15 V4L2\_CTRL\_TYPE\_CTRL\_CLASS Controls](#21515-v4l2_ctrl_type_ctrl_class-controls)
	- [2.15.16 Adding Notify Callbacks](#21516-adding-notify-callbacks)
	- [2.15.17 v4l2\_ctrl functions and data structures](#21517-v4l2_ctrl-functions-and-data-structures)
- [2.16 V4L2 videobuf2 functions and data structures](#216-v4l2-videobuf2-functions-and-data-structures)
- [2.17 V4L2 DV Timings functions](#217-v4l2-dv-timings-functions)
- [2.18 V4L2 Media Controller functions and data structures](#218-v4l2-media-controller-functions-and-data-structures)
- [2.19 V4L2 Media Controller functions and data structures](#219-v4l2-media-controller-functions-and-data-structures)
- [2.20 V4L2 Media Bus functions and data structures](#220-v4l2-media-bus-functions-and-data-structures)
- [2.21 V4L2 Memory to Memory functions and data structures](#221-v4l2-memory-to-memory-functions-and-data-structures)
- [2.22 V4L2 async kAPI](#222-v4l2-async-kapi)
- [2.23 V4L2 fwnode kAPI](#223-v4l2-fwnode-kapi)
- [2.24 V4L2 CCI kAPI](#224-v4l2-cci-kapi)
- [2.25 V4L2 rect helper functions](#225-v4l2-rect-helper-functions)
- [2.26 Tuner functions and data structures](#226-tuner-functions-and-data-structures)
- [2.27 V4L2 common functions and data structures](#227-v4l2-common-functions-and-data-structures)
- [2.28 Hauppauge TV EEPROM functions and data structures](#228-hauppauge-tv-eeprom-functions-and-data-structures)


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

- `video_device`->queue: a pointer to the struct vb2_queue associated with this device node. If queue is not ``NULL``, and queue->lock is not ``NULL``, then queue->lock is used for the queuing ioctls (``VIDIOC_REQBUFS``, ``CREATE_BUFS``, ``QBUF``, ``DQBUF``,  ``QUERYBUF``, ``PREPARE_BUF``, ``STREAMON`` and ``STREAMOFF``) instead of the lock above. That way the `vb2 <vb2_framework>` queuing framework does not have to wait for other ioctls.   This queue pointer is also used by the `vb2 <vb2_framework>` helper functions to check for queuing ownership (i.e. is the filehandle calling it allowed to do the operation).

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

V4L 核心提供了可选的锁服务。主要服务是 video_device 结构中的 lock 字段，它是指向互斥体的指针。如果设置了此指针，那么它将被 unlocked_ioctl 使用以串行化所有 ioctl。

If you are using the `videobuf2 framework` , then there is a second lock that you can set: `video_device`->queue->lock. If set, then this lock will be used instead of `video_device`->lock to serialize all queuing ioctls (see the previous section for the full list of those ioctls).

如果正在使用 videobuf2 框架 ，那么还有第二个锁，可以设置：video_device->queue->lock。如果设置，那么该锁将用于串行化所有队列 ioctl（请参见上一节以获取完整列表）而不是 video_device->lock。

The advantage of using a different lock for the queuing ioctls is that for some drivers (particularly USB drivers) certain commands such as setting controls can take a long time, so you want to use a separate lock for the buffer queuing ioctls. That way your ``VIDIOC_DQBUF`` doesn't stall because the driver is busy changing the e.g. exposure of the webcam.

使用不同的锁查询 ioctl 的优势在于，对于某些驱动程序（特别是 USB 驱动程序），某些命令，如设置控件，可能需要很长时间，因此您希望为缓冲区 queuing ioctl 使用单独的锁。这样，如果驱动程序正在忙于更改网络摄像头的曝光时间，您的 VIDIOC_DQBUF 也不会停滞。

Of course, you can always do all the locking yourself by leaving both lock pointers at ``NULL``.

当然，您始终可以通过将两个锁指针保留为空来自行完成所有的锁。

In the case of `videobuf2 framework>` you will need to implement the ``wait_prepare()`` and ``wait_finish()`` callbacks to unlock/lock if applicable. If you use the ``queue->lock`` pointer, then you can use the helper functions `vb2_ops_wait_prepare` and `vb2_ops_wait_finish`.

在 videobuf2 框架的情况下，如果适用，您将需要实现 wait_prepare() 和 wait_finish() 回调以进行解锁/锁定。如果使用 queue->lock 指针，则可以使用辅助函数 vb2_ops_wait_prepare 和 vb2_ops_wait_finish。

The implementation of a hotplug disconnect should also take the lock from `video_device` before calling v4l2_device_disconnect. If you are also using `video_device`->queue->lock, then you have to first lock `video_device`->queue->lock followed by `video_device`->lock. That way you can be sure no ioctl is running when you call `v4l2_device_disconnect`.

在执行热插拔断开连接时，还应该从 `video_device` 获取锁定之前调用 `v4l2_device_disconnect` 。如果还使用  `video_device->queue->lock` ，则必须先锁定 `video_device->queue->lock`，然后锁定 `video_device->lock`。这样，您可以确保在调用 `v4l2_device_disconnect` 时没有 ioctl 正在运行。

## 2.4.2 Video device registration

Next you register the video device with `video_register_device`. This will create the character device for you.

接下来，您使用 `video_register_device` 注册Video设备。这将为您创建字符设备。

```c
err = video_register_device(vdev, VFL_TYPE_VIDEO, -1);
if (err) {
    video_device_release(vdev); /* or kfree(my_vdev); */
	return err;
}
```

If the `v4l2_device` parent device has a not ``NULL`` mdev field, the video device entity will be automatically registered with the media device.

如果 `v4l2_device` 父设备具有非 `NULL` 的 `mdev` 字段，则视频设备实体将自动与媒体设备注册。

Which device is registered depends on the type argument. The following types exist:

注册哪个设备取决于类型参数。存在以下类型：

| `vfl_devnode_type` | Device name | Usage |
| ------------------ | ----------- | ----- |
| ``VFL_TYPE_VIDEO`` | ``/dev/videoX`` | for video input/output devices |
| ``VFL_TYPE_VBI`` | ``/dev/vbiX`` | for vertical blank data (i.e. closed captions, teletext) |
| ``VFL_TYPE_RADIO`` | ``/dev/radioX`` | for radio tuners |
| ``VFL_TYPE_SUBDEV`` | ``/dev/v4l-subdevX`` | for V4L2 subdevices |
| ``VFL_TYPE_SDR`` | ``/dev/swradioX`` | for Software Defined Radio(SDR) tuners |
| ``VFL_TYPE_TOUCH`` | ``/dev/v4l-touchX`` | for touch sensors |

The last argument gives you a certain amount of control over the device node number used (i.e. the X in ``videoX``). Normally you will pass -1 to let the v4l2 framework pick the first free number. But sometimes users want to select a specific node number. It is common that drivers allow the user to select a specific device node number through a driver module option. That number is then passed to this function and video_register_device will attempt to select that device node number. If that number was already in use, then the next free device node number will be selected and it will send a warning to the kernel log.

最后一个参数使您对使用的设备节点号（即 `videoX` 中的 X）有一定的控制。通常，您将传递 -1，以让 v4l2 框架选择第一个可用的编号。但有时用户希望选择特定的节点编号。通常，驱动程序允许用户通过驱动程序模块选项选择特定的设备节点号。然后将该数字传递给此函数，`video_register_device` 将尝试选择该设备节点号。如果该数值已在使用中，则将选择下一个可用的设备节点号，并向内核日志发送警告。

Another use-case is if a driver creates many devices. In that case it can be useful to place different video devices in separate ranges. For example, video capture devices start at 0, video output devices start at 16. So you can use the last argument to specify a minimum device node number and the v4l2 framework will try to pick the first free number that is equal or higher to what you passed. If that fails, then it will just pick the first free number.

另一种用例是如果驱动程序创建了许多设备。在这种情况下，将不同的视频设备放在不同的范围内可能很有用。例如，视频捕获设备从 0 开始，视频输出设备从 16 开始。因此，您可以使用最后一个参数指定最小设备节点号，并且 v4l2 框架将尝试选择第一个等于或高于您传递的空闲号。如果失败，则它将只选择第一个可用的号码。

Since in this case you do not care about a warning about not being able to select the specified device node number, you can call the function `video_register_device_no_warn` instead.

由于在这种情况下，您不关心无法选择指定的设备节点号的警告，因此可以调用 `video_register_device_no_warn` 函数，而不是 `video_register_device` 函数。

Whenever a device node is created some attributes are also created for you. If you look in ``/sys/class/video4linux`` you see the devices. Go into e.g. ``video0`` and you will see 'name', 'dev_debug' and 'index' attributes. The 'name' attribute is the 'name' field of the video_device struct. The 'dev_debug' attribute can be used to enable core debugging. See the next section for more detailed information on this.

在 `/sys/class/video4linux` 中创建设备节点时，还将为您创建一些属性。如果进入例如 `video0`，则会看到 'name'、'dev_debug' 和 'index' 属性。'name' 属性是 `video_device` 结构的 'name' 字段。'dev_debug' 属性可用于启用核心调试。有关此信息的更详细信息，请参见下一节。

The 'index' attribute is the index of the device node: for each call to `video_register_device()` the index is just increased by 1. The first video device node you register always starts with index 0.

'index' 属性是设备节点的索引：对于对 `video_register_device()` 的每次调用，索引仅增加 1。您始终注册的第一个视频设备节点始终从索引 0 开始。

Users can setup udev rules that utilize the index attribute to make fancy device names (e.g. '``mpegX``' for MPEG video capture device nodes).

用户可以设置 udev 规则，利用索引属性制作花哨的设备名称（例如 MPEG 视频捕获设备节点的 'mpegX'）。

After the device was successfully registered, then you can use these fields:

一旦成功注册设备节点，那么您就可以使用这些字段：

- `video_device`->vfl_type: the device type passed to `video_register_device`.
- `video_device`->minor: the assigned device minor number.
- `video_device`->num: the device node number (i.e. the X in ``videoX``).
- `video_device`->index: the device index number.

If the registration failed, then you need to call `video_device_release` to free the allocated `video_device` struct, or free your own struct if the  `video_device` was embedded in it. The ``vdev->release()`` callback will never be called if the registration failed, nor should you ever attempt to unregister the device if the registration failed.

如果注册失败，则需要调用 `video_device_release` 以释放分配的 `video_device` 结构，或者如果 `video_device` 嵌入在其中，则释放您自己的结构。如果注册失败，`vdev->release()` 回调将永远不会被调用，也不应尝试取消注册设备。

## 2.4.3 video device debugging

The 'dev_debug' attribute that is created for each video, vbi, radio or swradio device in ``/sys/class/video4linux/<devX>/`` allows you to enable logging of file operations.

在 `/sys/class/video4linux/<devX>/` 中为每个视频、vbi、收音机或软收音机设备创建的 'dev_debug' 属性允许您启用文件操作的日志记录。

It is a bitmask and the following bits can be set:

它是一个位掩码，可以设置以下位：

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

当需要删除视频设备节点时，无论是在卸载驱动程序期间还是因为 USB 设备断开连接，都应该使用 `video_unregister_device` 注销它们：

	video_unregister_device() (vdev);

This will remove the device nodes from sysfs (causing udev to remove them from ``/dev``).

这将从 `sysfs` 中删除设备节点（导致 udev 从 /dev 中删除它们）。

After `video_unregister_device` returns no new opens can be done. However, in the case of USB devices some application might still have one of these device nodes open. So after the unregister all file operations (except release, of course) will return an error as well.

在 `video_unregister_device` 返回后，不再可以进行新的打开。但是，在 USB 设备的情况下，一些应用程序可能仍然打开其中一个这些设备节点。因此，在注销后，所有文件操作（当然除了释放）都将返回错误。

When the last user of the video device node exits, then the ``vdev->release()`` callback is called and you can do the final cleanup there.

当视频设备节点的最后一个用户退出时，将调用 vdev->release() 回调，您可以在其中进行最终清理。

Don't forget to cleanup the media entity associated with the video device if it has been initialized:

如果已经初始化, 不要忘记清理与视频设备关联的媒体实体：

	 media_entity_cleanup(&vdev->entity);

This can be done from the release callback.

可以在释放回调中执行此操作。

## 2.4.5 helper functions

There are a few useful helper functions:

有一些有用的辅助函数：

- file and `video_device` private data

You can set/get driver private data in the video_device struct using:

您可以使用以下函数在 `video_device` 结构中设置/获取驱动程序私有数据：

    video_get_drvdata(vdev);
	video_set_drvdata(vdev);

Note that you can safely call   `video_set_drvdata` before calling `video_register_device`.

请注意，您可以在调用 `video_register_device` 之前安全地调用 video_set_drvdata。

And this function:

还有这个函数：

	video_devdata(struct file *file);

returns the video_device belonging to the file struct.

返回文件结构所属的 `video_device`。

The `video_devdata` function combines `video_get_drvdata` with `video_devdata`:

    video_drvdata (struct file *file);

You can go from a `video_device` struct to the v4l2_device struct using:

您可以通过以下方式从 `video_device` 结构转到 `v4l2_device` 结构：

```c
struct v4l2_device *v4l2_dev = vdev->v4l2_dev;
```

- Device node name

The `video_device` node kernel name can be retrieved using:

可以使用以下函数检索 `video_device` 节点内核名称：

    video_device_node_name(vdev);

The name is used as a hint by userspace tools such as udev. The functionshould be used where possible instead of accessing the video_device::num and video_device::minor fields.

该名称用作用户空间工具（例如 udev）的提示。应在可能的情况下使用该函数，而不是访问 `video_device::num` 和 `video_device::minor` 字段。

## 2.4.6 video_device functions and data structures

reference:

    include/media/v4l2-dev.h



# 2.5 V4L2 device instance

Each device instance is represented by a struct v4l2_device. Very simple devices can just allocate this struct, but most of the time you would embed this struct inside a larger struct.

每个设备实例都由 `struct v4l2_device` 表示。非常简单的设备可以直接分配此结构，但大多数情况下，您会将此结构嵌套在一个较大的结构中。

You must register the device instance by calling:

您必须通过调用以下方法注册设备实例：

    v4l2_device_register(dev, v4l2_dev).

Registration will initialize the `v4l2_device` struct. If the dev->driver_data field is ``NULL``, it will be linked to `v4l2_dev <v4l2_device>` argument.

注册将初始化 `struct v4l2_device`。如果 `dev->driver_data` 字段为 `NULL`，它将链接到 `v4l2_dev` 参数。

Drivers that want integration with the media device framework need to set dev->driver_data manually to point to the driver-specific device structure that embed the struct v4l2_device instance. This is achieved by a ``dev_set_drvdata()`` call before registering the V4L2 device instance. They must also set the struct v4l2_device mdev field to point to a properly initialized and registered  `media_device` instance.

希望与媒体设备框架集成的驱动程序需要手动设置 `dev->driver_data`，将其指向嵌套 `struct v4l2_device` 实例的特定于驱动程序的设备结构。这通过在注册 V4L2 设备实例之前调用 `dev_set_drvdata()` 来实现。他们还必须将 `struct v4l2_device mdev` 字段设置为指向已正确初始化和注册的 `struct media_device` 实例。

If  `v4l2_dev <v4l2_device>`->name is empty then it will be set to a value derived from dev (driver name followed by the bus_id, to be precise). If you set it up before  calling `v4l2_device_register` then it will be untouched. If dev is ``NULL``, then you **must** setup `v4l2_dev <v4l2_device>` ->name before calling `v4l2_device_register`.

如果 `v4l2_dev -> name` 为空，则将其设置为从 `dev` 派生的值（确切地说，是驱动程序名称后跟 bus_id）。如果在调用 `v4l2_device_register` 之前设置了它，则不会更改。如果 `dev` 为 `NULL`，则在调用 `v4l2_device_register` 之前**必须**设置 `v4l2_dev -> name`。

You can use `v4l2_device_set_name` to set the name based on a driver name and a driver-global atomic_t instance. This will generate names like ``ivtv0``, ``ivtv1``, etc. If the name ends with a digit, then it will insert a dash: ``cx18-0``, ``cx18-1``, etc. This function returns the instance number.

您可以使用 `v4l2_device_set_name` 根据驱动程序名称和驱动程序全局 `atomic_t` 实例设置名称。这将生成类似于 `ivtv0`、`ivtv1` 等的名称。如果名称以数字结尾，则会插入破折号：`cx18-0`、`cx18-1` 等。此函数返回实例编号。

The first ``dev`` argument is normally the ``struct device`` pointer of a ``pci_dev``, ``usb_interface`` or ``platform_device``. It is rare for dev to be ``NULL``, but it happens with ISA devices or when one device creates multiple PCI devices, thus making it impossible to associate `v4l2_dev <v4l2_device>` with a particular parent.

第一个 `dev` 参数通常是 `pci_dev`、`usb_interface` 或 `platform_device` 的 `struct device` 指针。`dev` 为 `NULL` 的情况很少见，但在 ISA 设备或当一个设备创建多个 PCI 设备时发生，因此无法将 `struct v4l2_device` 与特定的父设备关联。

You can also supply a ``notify()`` callback that can be called by sub-devices to notify you of events. Whether you need to set this depends on the sub-device. Any notifications a sub-device supports must be defined in a header in ``include/media/subdevice.h``.

您还可以提供一个 `notify()` 回调，可以由子设备调用以通知您事件。是否需要设置这取决于子设备。子设备支持的任何通知都必须在 `include/media/subdevice.h` 中的头文件中定义。

V4L2 devices are unregistered by calling:

通过调用以下方法取消注册 V4L2 设备：

    v4l2_device_unregister(v4l2_dev).

If the dev->driver_data field points to  `v4l2_dev <v4l2_device>`, it will be reset to ``NULL``. Unregistering will also automatically unregister all subdevs from the device.

如果 `dev->driver_data` 字段指向 `v4l2_dev`，它将被重置为 `NULL`。注销还将自动注销设备的所有子设备。

If you have a hotpluggable device (e.g. a USB device), then when a disconnect happens the parent device becomes invalid. Since  `v4l2_device` has a pointer to that parent device it has to be cleared as well to mark that the parent is gone. To do this call:

如果您有一个可热插拔的设备（例如 USB 设备），那么在发生断开连接时，父设备将变得无效。由于 `struct v4l2_device` 具有指向父设备的指针，因此也必须将其清除，以标记父设备已经不存在。为此，请调用：

    v4l2_device_disconnect(v4l2_dev).

This does *not* unregister the subdevs, so you still need to call the `v4l2_device_unregister` function for that. If your driver is not hotpluggable, then there is no need to call `v4l2_device_disconnect`.

这*不会*注销子设备，因此您仍然需要调用 `v4l2_device_unregister` 函数。如果您的驱动程序不可热插拔，则不需要调用 `v4l2_device_disconnect`。

Sometimes you need to iterate over all devices registered by a specific driver. This is usually the case if multiple device drivers use the same hardware. E.g. the ivtvfb driver is a framebuffer driver that uses the ivtv hardware. The same is true for alsa drivers for example.

有时您需要迭代由特定驱动程序注册的所有设备。如果多个设备驱动程序使用相同的硬件，则通常会出现这种情况。例如，ivtvfb 驱动程序是一个使用 ivtv 硬件的帧缓冲驱动程序。对于 ALSA 驱动程序也是如此。

You can iterate over all registered devices as follows:

您可以按照以下方式迭代所有已注册的设备：

```c
static int callback(struct device *dev, void *p)
{
    struct v4l2_device *v4l2_dev = dev_get_drvdata(dev);

    /* test if this device was inited */
    if (v4l2_dev == NULL)
    	return 0;
    		...
    	return 0;
    }

int iterate(void *p)
{
    struct device_driver *drv;
    int err;

    /* Find driver 'ivtv' on the PCI bus.
    pci_bus_type is a global. For USB buses use usb_bus_type. */
    drv = driver_find("ivtv", &pci_bus_type);
    /* iterate over all ivtv device instances */
    err = driver_for_each_device(drv, NULL, p, callback);
    put_driver(drv);
    return err;
}
```

Sometimes you need to keep a running counter of the device instance. This is commonly used to map a device instance to an index of a module option array.

有时您需要保持设备实例的运行计数。这通常用于将设备实例映射到模块选项数组的索引。

The recommended approach is as follows:

推荐的方法如下：

```c
static atomic_t drv_instance = ATOMIC_INIT(0);

static int drv_probe(struct pci_dev *pdev, const struct pci_device_id *pci_id)
{
    ...
    state->instance = atomic_inc_return(&drv_instance) - 1;
}
```

If you have multiple device nodes then it can be difficult to know when it is safe to unregister  `v4l2_device` for hotpluggable devices. For this purpose  `v4l2_device` has refcounting support. The refcount is increased whenever `video_register_device` is called and it is decreased whenever that device node is released. When the refcount reaches zero, then the `v4l2_device` release() callback is called. You can do your final cleanup there.

如果有多个设备节点，则很难知道何时安全地注销热插拔设备的 `struct v4l2_device`。为此，`struct v4l2_device` 具有引用计数支持。每次调用 `video_register_device` 时，引用计数都会增加，而在释放该设备节点时，它会减少。当引用计数达到零时，将调用 `struct v4l2_device release()` 回调。您可以在其中进行最终清理。

If other device nodes (e.g. ALSA) are created, then you can increase and decrease the refcount manually as well by calling:

如果还创建了其他设备节点（例如 ALSA），则还可以通过手动调用以下方法来增加和减少引用计数：

    v4l2_device_get(v4l2_dev).

    or:

    v4l2_device_put(v4l2_dev).

Since the initial refcount is 1 you also need to call `v4l2_device_put` in the ``disconnect()`` callback (for USB devices) or in the ``remove()`` callback (for e.g. PCI devices), otherwise the refcount will never reach 0.

由于初始引用计数为 1，因此您还需要在“disconnect”回调（对于 USB 设备）或“remove”回调（例如 PCI 设备）中调用 `v4l2_device_put`，否则引用计数将永远不会达到 0。

## 2.5.1 v4l2_device functions and data structures

    include/media/v4l2-device.h


# 2.6 V4L2 File handlers

struct v4l2_fh provides a way to easily keep file handle specific data that is used by the V4L2 framework.

`struct v4l2_fh` 提供了一种轻松保留文件句柄特定数据的方法，该数据由 V4L2 框架使用。

**Attention**

	New drivers must use struct v4l2_fh since it is also used to implement priority handling (`VIDIOC_G_PRIORITY`, `VIDIOC_S_PRIORITY`).

    新驱动程序必须使用 `struct v4l2_fh`，因为它还用于实现优先级处理（`VIDIOC_G_PRIORITY`, `VIDIOC_S_PRIORITY`）。


The users of `v4l2_fh` (in the V4L2 framework, not the driver) know
whether a driver uses `v4l2_fh` as its ``file->private_data`` pointer
by testing the ``V4L2_FL_USES_V4L2_FH`` bit in `video_device`->flags.
This bit is set whenever `v4l2_fh_init` is called.

在 V4L2 框架中使用 `v4l2_fh`（不是驱动程序中的）的用户通过测试 `video_device`->flags 中的 `V4L2_FL_USES_V4L2_FH` 位来知道驱动程序是否使用 `v4l2_fh` 作为其 `file->private_data` 指针。每当调用 `v4l2_fh_init` 时，此位都会被设置。

struct v4l2_fh is allocated as a part of the driver's own file handle
structure and ``file->private_data`` is set to it in the driver's ``open()`` function by the driver.

`struct v4l2_fh` 作为驱动程序自己文件句柄结构的一部分分配，并且在驱动程序的 `open()` 函数中将 `file->private_data` 设置为它。

In many cases the struct v4l2_fh will be embedded in a larger
structure. In that case you should call:

在许多情况下，`struct v4l2_fh` 将嵌入在一个较大的结构中。在这种情况下，您应该调用：

1.  `v4l2_fh_init` and `v4l2_fh_add` in ``open()``
2.  `v4l2_fh_del` and `v4l2_fh_exit` in ``release()``

Drivers can extract their own file handle structure by using the container_of
macro.

驱动程序可以使用 `container_of` 宏提取其自己的文件句柄结构。

Example:

```c
struct my_fh {
	int blah;
	struct v4l2_fh fh;
};

	...

int my_open(struct file *file)
{
	struct my_fh *my_fh;
	struct video_device *vfd;
	int ret;

	...

	my_fh = kzalloc(sizeof(*my_fh), GFP_KERNEL);

    ...

	v4l2_fh_init(&my_fh->fh, vfd);

	...

	file->private_data = &my_fh->fh;
	v4l2_fh_add(&my_fh->fh);
	return 0;
}

int my_release(struct file *file)
{
	struct v4l2_fh *fh = file->private_data;
	struct my_fh *my_fh = container_of(fh, struct my_fh, fh);

	...
	v4l2_fh_del(&my_fh->fh);
	v4l2_fh_exit(&my_fh->fh);
	kfree(my_fh);
	return 0;
}
```

Below is a short description of the `v4l2_fh` functions used:

以下是使用的 `v4l2_fh` 函数的简短描述：

v4l2_fh_init(fh, vdev)

- Initialise the file handle. This **MUST** be performed in the driver's `v4l2_file_operations`->open() handler.
- 初始化文件句柄。必须在驱动程序的 `v4l2_file_operations`->open() 处理程序中执行。


v4l2_fh_add(fh)

- Add a `v4l2_fh` to `video_device` file handle list. Must be called once the file handle is completely initialized.
- 将 `v4l2_fh` 添加到 `video_device` 文件句柄列表中。一旦文件句柄完全初始化，必须调用此函数。

v4l2_fh_del(fh)

- Unassociate the file handle from `video_device`. The file handle
  exit function may now be called.
- 取消将文件句柄与 `video_device` 关联。现在可以调用文件句柄的退出函数。

v4l2_fh_exit(fh)

- Uninitialise the file handle. After uninitialisation the `v4l2_fh`
  memory can be freed.
- 取消初始化文件句柄。取消初始化后，可以释放 `v4l2_fh` 内存。

If struct v4l2_fh is not embedded, then you can use these helper functions:

如果 `struct v4l2_fh` 没有嵌入，那么您可以使用以下这些辅助函数：

v4l2_fh_open(struct file *filp)

- This allocates a struct v4l2_fh, initializes it and adds it to
  the struct video_device associated with the file struct.
- 分配 `struct v4l2_fh`，初始化它并将它添加到与文件结构关联的 `struct video_device` 中。

v4l2_fh_release(struct file *filp)

- This deletes it from the struct video_device associated with the
  file struct, uninitialised the `v4l2_fh` and frees it.
- 从与文件结构关联的 `struct video_device` 中删除它，取消初始化 `v4l2_fh` 并释放它。

These two functions can be plugged into the v4l2_file_operation's ``open()`` and ``release()`` ops.

这两个函数可以插入到 `v4l2_file_operation` 的 `open()` 和 `release()` 操作中。

Several drivers need to do something when the first file handle is opened and when the last file handle closes. Two helper functions were added to check whether the `v4l2_fh` struct is the only open filehandle of the associated device node:

一些驱动程序在打开第一个文件句柄时需要执行某些操作，并在关闭最后一个文件句柄时执行某些操作。两个辅助函数被添加以检查 `v4l2_fh` 结构是否是关联设备节点的唯一打开文件句柄：

v4l2_fh_is_singular(fh)

- Returns 1 if the file handle is the only open file handle, else 0.
- 如果文件句柄是唯一的打开文件句柄，则返回 1，否则返回 0。

v4l2_fh_is_singular_file(struct file *filp)

- Same, but it calls v4l2_fh_is_singular with filp->private_data.
- 相同，但它调用 `v4l2_fh_is_singular` 并带有 `filp->private_data`。

## 2.6.1 V4L2 fh functions and data structures

    include/media/v4l2-fh.h



# 2.7 V4L2 sub-devices

Many drivers need to communicate with sub-devices. These devices can do all
sort of tasks, but most commonly they handle audio and/or video muxing,
encoding or decoding. For webcams common sub-devices are sensors and camera
controllers.

许多驱动程序需要与子设备进行通信。这些设备可以执行各种任务，但最常见的是处理音频和/或
视频的多路复用、编码或解码。对于网络摄像头，常见的子设备是传感器和摄像头控制器。

Usually these are I2C devices, but not necessarily. In order to provide the driver with a consistent interface to these sub-devices the `v4l2_subdev` struct (v4l2-subdev.h) was created.

通常这些是I2C设备，但不一定如此。为了为驱动程序提供与这些子设备的一致接口，创建
了`v4l2_subdev`结构体（v4l2-subdev.h）。

Each sub-device driver must have a `v4l2_subdev` struct. This struct can be stand-alone for simple sub-devices or it might be embedded in a larger struct if more state information needs to be stored. Usually there is a low-level device struct (e.g. ``i2c_client``) that contains the device data as setup by the kernel. It is recommended to store that pointer in the private data of `v4l2_subdev` using `v4l2_set_subdevdata`. That makes it easy to go from a `v4l2_subdev` to the actual low-level bus-specific
device data.

每个子设备驱动程序必须有一个`v4l2_subdev`结构体。这个结构体可以是一个简单子设备的独立
结构体，或者如果需要存储更多状态信息，可以嵌入在较大的结构体中。通常有一个低级别设备结
构体（例如“i2c_client”），其中包含内核设置的设备数据。建议使用`v4l2_set_subdevdata`
将该指针存储在`v4l2_subdev`的私有数据中。这样可以轻松地从`v4l2_subdev`转到实际的总线
特定的设备数据。

You also need a way to go from the low-level struct to `v4l2_subdev`. For the common i2c_client struct the i2c_set_clientdata() call is used to store a `v4l2_subdev` pointer, for other buses you may have to use other methods.

还需要一种方法从低级别结构转到`v4l2_subdev`。对于常见的i2c_client结构体，使
用`i2c_set_clientdata()`调用来存储`v4l2_subdev`指针，对于其他总线，可能需要使用其他
方法。

Bridges might also need to store per-subdev private data, such as a pointer to bridge-specific per-subdev private data. The `v4l2_subdev` structure provides host private data for that purpose that can be accessed with `v4l2_get_subdev_hostdata` and `v4l2_set_subdev_hostdata`.

桥接可能还需要存储每个子设备的私有数据，例如桥接特定的每个子设备私有数据的指针。
`v4l2_subdev`结构提供了用于此目的的主机私有数据，可以使
用`v4l2_get_subdev_hostdata`和`v4l2_set_subdev_hostdata`访问。

From the bridge driver perspective, you load the sub-device module and somehow obtain the `v4l2_subdev` pointer. For i2c devices this is easy: you call ``i2c_get_clientdata()``. For other buses something similar needs to be done. Helper functions exist for sub-devices on an I2C bus that do most of this tricky work for you.

从桥接驱动程序的角度来看，您加载子设备模块并以某种方式获取`v4l2_subdev`指针。对于i2c
设备，这很容易：调用`i2c_get_clientdata()`即可。对于其他总线，可能需要执行类似的操作。
对于在I2C总线上的子设备的帮助函数已经为您执行了大部分棘手的工作。

Each `v4l2_subdev` contains function pointers that sub-device drivers can implement (or leave ``NULL`` if it is not applicable). Since sub-devices can do so many different things and you do not want to end up with a huge ops struct of which only a handful of ops are commonly implemented, the function pointers are sorted according to category and each category has its own ops struct.

每个`v4l2_subdev`包含子设备驱动程序可以实现（如果不适用，可以将其保留为`NULL`）的函数
指针。由于子设备可以执行许多不同的操作，我们不希望最终得到一个仅有少数ops通常被实现的
巨大ops结构，因此函数指针根据类别进行排序，每个类别都有自己的ops结构。

The top-level ops struct contains pointers to the category ops structs, which may be NULL if the subdev driver does not support anything from that category.

顶层的ops结构包含对类别ops结构的指针，如果子设备驱动程序不支持该类别的任何内容，则该指针可能为NULL。

It looks like this:

它看起来像这样：

```c
	struct v4l2_subdev_core_ops {
		int (*log_status)(struct v4l2_subdev *sd);
		int (*init)(struct v4l2_subdev *sd, u32 val);
		...
	};

	struct v4l2_subdev_tuner_ops {
		...
	};

	struct v4l2_subdev_audio_ops {
		...
	};

	struct v4l2_subdev_video_ops {
		...
	};

	struct v4l2_subdev_pad_ops {
		...
	};

	struct v4l2_subdev_ops {
		const struct v4l2_subdev_core_ops  *core;
		const struct v4l2_subdev_tuner_ops *tuner;
		const struct v4l2_subdev_audio_ops *audio;
		const struct v4l2_subdev_video_ops *video;
		const struct v4l2_subdev_pad_ops *video;
	};
```

The core ops are common to all subdevs, the other categories are implemented depending on the sub-device. E.g. a video device is unlikely to support the audio ops and vice versa.

核心ops适用于所有子设备，其他类别根据子设备而实现。例如，视频设备不太可能支持音频ops，反之亦然。

This setup limits the number of function pointers while still making it easy to add new ops and categories.

这种设置限制了函数指针的数量，同时使得很容易添加新的ops和类别。

A sub-device driver initializes the `v4l2_subdev` struct using:

子设备驱动程序使用以下方式初始化`v4l2_subdev`结构：

	v4l2_subdev_init(sd, &ops).

Afterwards you need to initialize `sd`->name with a unique name and set the module owner. This is done for you if you use the i2c helper functions.

之后，您需要使用唯一的名称初始化`sd`->name，并设置模块所有者。如果使用i2c帮助函数，则会为您执行这些操作。

If integration with the media framework is needed, you must initialize the `media_entity` struct embedded in the `v4l2_subdev` struct (entity field) by calling `media_entity_pads_init`, if the entity has pads:

如果需要与媒体框架集成，必须通过调用`media_entity_pads_init`初始化嵌入在`v4l2_subdev`结构中的`media_entity`结构体。如果不这样做，您将无法创建链路并将子设备与其他设备连接在一起。

```c
	struct media_pad *pads = &my_sd->pads;
	int err;

	err = media_entity_pads_init(&sd->entity, npads, pads);
```

The pads array must have been previously initialized. There is no need to manually set the struct media_entity function and name fields, but the revision field must be initialized if needed.

pads数组必须事先完成初始化。不必手动设置media_entity结构体的函数和名称字段，但如有需要，修订字段必须进行初始化。

A reference to the entity will be automatically acquired/released when the subdev device node (if any) is opened/closed.

在打开/关闭子设备节点（如果有的话），将自动获取/释放对该实体的引用。

Don't forget to cleanup the media entity before the sub-device is destroyed:

在销毁子设备之前，不要忘记清理媒体实体。
```c
	media_entity_cleanup(&sd->entity);
```

If a sub-device driver implements sink pads, the subdev driver may set the link_validate field in `v4l2_subdev_pad_ops` to provide its own link validation function. For every link in the pipeline, the link_validate pad operation of the sink end of the link is called. In both cases the driver is still responsible for validating the correctness of the format configuration between sub-devices and video nodes.

如果子设备驱动程序实现了sink pads，子设备驱动程序可以在`v4l2_subdev_pad_ops`中设置link_validate字段，以提供自己的链路验证函数。对于管道中的每个链接，将调用链接末端（sink端）的link_validate pad操作。在这两种情况下，驱动程序仍然负责验证子设备和视频节点之间格式配置的正确性。

If link_validate op is not set, the default function `v4l2_subdev_link_validate_default` is used instead. This function ensures that width, height and the media bus pixel code are equal on both source and sink of the link. Subdev drivers are also free to use this function to perform the checks mentioned above in addition to their own checks.

如果未设置link_validate操作，则将使用默认函数`v4l2_subdev_link_validate_default`。此函数确保链接的源和汇端的宽度、高度和媒体总线像素代码相等。子设备驱动程序还可以自由使用此函数来执行上述检查以及它们自己的检查。

## 2.7.1 Subdev registration

There are currently two ways to register subdevices with the V4L2 core. The first (traditional) possibility is to have subdevices registered by bridge drivers. This can be done when the bridge driver has the complete information about subdevices connected to it and knows exactly when to register them. This is typically the case for internal subdevices, like video data processing units within SoCs or complex PCI(e) boards, camera sensors in USB cameras or connected to SoCs, which pass information about them to bridge drivers, usually in their platform data.

目前有两种向V4L2核心注册子设备的方法。第一种（传统的）可能性是由桥接驱动程序注册子设备。当桥接驱动程序具有与其连接的子设备的完整信息并且确切知道何时注册它们时，可以执行此操作。这通常适用于内部子设备，例如SoC内的视频数据处理单元或复杂的PCI(e)板卡，USB摄像头中的摄像头传感器或连接到SoC的摄像头，它们将有关信息传递给桥接驱动程序，通常在其平台数据中。

There are however also situations where subdevices have to be registered asynchronously to bridge devices. An example of such a configuration is a Device Tree based system where information about subdevices is made available to the system independently from the bridge devices, e.g. when subdevices are defined in DT as I2C device nodes. The API used in this second case is described further
below.

然而，也存在一些情况下，子设备必须以异步方式向桥接设备注册。这样的配置示例是基于设备树的系统，其中有关子设备的信息独立于桥接设备而提供给系统，例如，当子设备在设备树中定义为I2C设备节点时。在这种第二种情况下使用的API将进一步描述。

Using one or the other registration method only affects the probing process, the run-time bridge-subdevice interaction is in both cases the same.

使用其中一种注册方法仅影响探测过程，在运行时桥接-子设备交互在两种情况下都是相同的。

### 2.7.1.1 Registering synchronous sub-devices

In the **synchronous** case a device (bridge) driver needs to register the
`v4l2_subdev` with the v4l2_device:

在**同步**情况下，设备（桥接）驱动程序需要向v4l2_device注册`v4l2_subdev`：

	v4l2_device_register_subdev(v4l2_dev, sd).

This can fail if the subdev module disappeared before it could be registered. After this function was called successfully the subdev->dev field points to the `v4l2_device`.

如果在子设备模块注册之前该模块消失，这可能导致注册失败。在此函数成功调用后，subdev->dev字段将指向`v4l2_device`。

If the v4l2_device parent device has a non-NULL mdev field, the sub-device entity will be automatically registered with the media device.

如果v4l2_device的父设备具有非NULL的mdev字段，子设备实体将自动注册到媒体设备。

You can unregister a sub-device using:

您可以使用以下方法注销子设备：

	v4l2_device_unregister_subdev(sd).

Afterwards the subdev module can be unloaded and `sd`->dev == ``NULL``.

之后，可以卸载子设备模块，`sd`->dev == `NULL`。

### 2.7.1.2 Registering asynchronous sub-devices

In the **asynchronous** case subdevice probing can be invoked independently of the bridge driver availability. The subdevice driver then has to verify whether all the requirements for a successful probing are satisfied. This can include a check for a master clock availability. If any of the conditions aren't satisfied the driver might decide to return ``-EPROBE_DEFER`` to request further reprobing attempts. Once all conditions are met the subdevice shall be registered using the `v4l2_async_register_subdev` function. Unregistration is performed using the `v4l2_async_unregister_subdev` call. Subdevices registered this way are stored in a global list of subdevices, ready to be picked up by bridge drivers.

在**异步**情况下，子设备的探测可以独立于桥接驱动程序的可用性而被调用。然后，子设备驱动程序必须验证是否满足了成功探测的所有要求。这可能包括检查主时钟的可用性。如果其中任何条件不满足，驱动程序可能决定返回`-EPROBE_DEFER`以请求进一步的重新探测尝试。一旦满足所有条件，就可以使用`v4l2_async_register_subdev`函数注册子设备。注销则使用`v4l2_async_unregister_subdev`调用完成。以这种方式注册的子设备存储在全局子设备列表中，可以由桥接驱动程序使用。

### 2.7.1.3 Asynchronous sub-device notifiers

Bridge drivers in turn have to register a notifier object. This is performed
using the `v4l2_async_nf_register` call. To unregister the notifier the
driver has to call `v4l2_async_nf_unregister`. Before releasing memory
of an unregister notifier, it must be cleaned up by calling
`v4l2_async_nf_cleanup`.

桥接驱动程序反过来必须注册一个通知对象。这通过调用`v4l2_async_nf_register`来执行。要注销通知程序，驱动程序必须调用`v4l2_async_nf_unregister`。在释放注销通知程序的内存之前，必须通过调用`v4l2_async_nf_cleanup`进行清理。

Before registering the notifier, bridge drivers must do two things: first, the
notifier must be initialized using the `v4l2_async_nf_init`.  Second,
bridge drivers can then begin to form a list of async connection descriptors
that the bridge device needs for its
operation. `v4l2_async_nf_add_fwnode`,
`v4l2_async_nf_add_fwnode_remote` and `v4l2_async_nf_add_i2c`

在注册通知程序之前，桥接驱动程序必须执行两个操作：首先，必须使用`v4l2_async_nf_init`对通知程序进行初始化。其次，桥接驱动程序可以开始形成异步连接描述符的列表，这是桥接设备在其操作中所需的。使用`v4l2_async_nf_add_fwnode`、`v4l2_async_nf_add_fwnode_remote`和`v4l2_async_nf_add_i2c`。

Async connection descriptors describe connections to external sub-devices the
drivers for which are not yet probed. Based on an async connection, a media data
or ancillary link may be created when the related sub-device becomes
available. There may be one or more async connections to a given sub-device but
this is not known at the time of adding the connections to the notifier. Async
connections are bound as matching async sub-devices are found, one by one.

异步连接描述符描述了与尚未进行驱动程序探测的外部子设备的连接。基于异步连接，当相关子设备变得可用时，可能会创建媒体数据或辅助链接。对于给定的子设备，可能存在一个或多个异步连接，但在将连接添加到通知程序时无法确定。一旦找到匹配的异步子设备，异步连接将被绑定，逐个连接。

### 2.7.1.4 Asynchronous sub-device notifier for sub-devices

A driver that registers an asynchronous sub-device may also register an
asynchronous notifier. This is called an asynchronous sub-device notifier andthe
process is similar to that of a bridge driver apart from that the notifier is
initialised using `v4l2_async_subdev_nf_init` instead. A sub-device
notifier may complete only after the V4L2 device becomes available, i.e. there's
a path via async sub-devices and notifiers to a notifier that is not an
asynchronous sub-device notifier.

注册异步子设备的驱动程序也可以注册一个异步通知程序。这被称为异步子设备通知程序，其过程与桥接驱动程序类似，除了通知程序是使用`v4l2_async_subdev_nf_init`初始化的。异步子设备通知程序可能仅在V4L2设备可用后完成，即存在通过异步子设备和通知程序到不是异步子设备通知程序的通路。

### 2.7.1.5 Asynchronous sub-device registration helper for camera sensor drivers

`v4l2_async_register_subdev_sensor` is a helper function for sensor
drivers registering their own async connection, but it also registers a notifier
and further registers async connections for lens and flash devices found in
firmware. The notifier for the sub-device is unregistered and cleaned up with
the async sub-device, using `v4l2_async_unregister_subdev`.

`v4l2_async_register_subdev_sensor`是传感器驱动程序注册其自己的异步连接的辅助函数，但它还注册了一个通知程序，并为在固件中找到的镜头和闪光灯设备注册了进一步的异步连接。子设备的通知程序会使用`v4l2_async_unregister_subdev`一同注销并清理。

### 2.7.1.6 Asynchronous sub-device notifier example

These functions allocate an async connection descriptor which is of type struct
`v4l2_async_connection` embedded in a driver-specific struct. The &struct
`v4l2_async_connection` shall be the first member of this struct:

这些函数分配一个异步连接描述符，其类型为嵌入在驱动程序特定结构中的`struct v4l2_async_connection`。`&struct v4l2_async_connection`应该是此结构的第一个成员：

```c
	struct my_async_connection {
		struct v4l2_async_connection asc;
		...
	};

	struct my_async_connection *my_asc;
	struct fwnode_handle *ep;

	...

	my_asc = v4l2_async_nf_add_fwnode_remote(&notifier, ep,
						 struct my_async_connection);
	fwnode_handle_put(ep);

	if (IS_ERR(my_asc))
		return PTR_ERR(my_asc);
```

### 2.7.1.7 Asynchronous sub-device notifier callbacks

The V4L2 core will then use these connection descriptors to match asynchronously
registered subdevices to them. If a match is detected the ``.bound()`` notifier
callback is called. After all connections have been bound the .complete()
callback is called. When a connection is removed from the system the
``.unbind()`` method is called. All three callbacks are optional.

V4L2核心将使用这些连接描述符将异步注册的子设备与它们匹配。如果检测到匹配，将调用`.bound()`通知程序回调。在所有连接都已绑定后，将调用`.complete()`回调。当从系统中移除连接时，将调用`.unbind()`方法。这三个回调都是可选的。

Drivers can store any type of custom data in their driver-specific
`v4l2_async_connection` wrapper. If any of that data requires special
handling when the structure is freed, drivers must implement the ``.destroy()``
notifier callback. The framework will call it right before freeing the
`v4l2_async_connection`.

驱动程序可以在其驱动程序特定的`v4l2_async_connection`包装器中存储任何类型的自定义数据。如果在释放结构时需要对该数据进行特殊处理，驱动程序必须实现`.destroy()`通知程序回调。框架将在释放`v4l2_async_connection`之前立即调用它。

## 2.7.2 Calling subdev operations

The advantage of using `v4l2_subdev` is that it is a generic struct and
does not contain any knowledge about the underlying hardware. So a driver might
contain several subdevs that use an I2C bus, but also a subdev that is
controlled through GPIO pins. This distinction is only relevant when setting
up the device, but once the subdev is registered it is completely transparent.

使用`v4l2_subdev`的优势在于它是一个通用的结构，不包含有关底层硬件的任何信息。因此，驱动程序可能包含使用I2C总线的多个子设备，还可能包含通过GPIO引脚控制的子设备。这种区别仅在设置设备时才相关，但一旦子设备被注册，它就是完全透明的。

Once the subdev has been registered you can call an ops function either
directly:

一旦子设备已注册，您可以直接调用ops函数：

```c
	err = sd->ops->core->g_std(sd, &norm);
```

but it is better and easier to use this macro:

但最好且更容易使用这个宏：

```c
	err = v4l2_subdev_call(sd, core, g_std, &norm);
```

The macro will do the right ``NULL`` pointer checks and returns ``-ENODEV``
if `sd <v4l2_subdev>` is ``NULL``, ``-ENOIOCTLCMD`` if either
`sd <v4l2_subdev>`->core or `sd <v4l2_subdev>`->core->g_std is ``NULL``, or the actual result of the
`sd <v4l2_subdev>`->ops->core->g_std ops.

该宏将执行正确的`NULL`指针检查，如果`sd <v4l2_subdev>`为`NULL`，则返回`-ENODEV`，如果`sd <v4l2_subdev>`->core或`sd <v4l2_subdev>`->core->g_std为`NULL`，则返回`-ENOIOCTLCMD`，或者返回`sd <v4l2_subdev>`->ops->core->g_std ops的实际结果。

It is also possible to call all or a subset of the sub-devices:

也可以调用所有或子设备的子集：

```c
	v4l2_device_call_all(v4l2_dev, 0, core, g_std, &norm);
```

Any subdev that does not support this ops is skipped and error results are
ignored. If you want to check for errors use this:

不支持此ops的任何子设备都会被跳过，错误结果将被忽略。如果想要检查错误，可以使用以下方法：

```c
	err = v4l2_device_call_until_err(v4l2_dev, 0, core, g_std, &norm);
```

Any error except ``-ENOIOCTLCMD`` will exit the loop with that error. If no
errors (except ``-ENOIOCTLCMD``) occurred, then 0 is returned.

除了`-ENOIOCTLCMD`之外的任何错误都将以该错误退出循环。如果没有发生错误（除了`-ENOIOCTLCMD`），则返回0。

The second argument to both calls is a group ID. If 0, then all subdevs are
called. If non-zero, then only those whose group ID match that value will
be called. Before a bridge driver registers a subdev it can set
`sd <v4l2_subdev>`->grp_id to whatever value it wants (it's 0 by
default). This value is owned by the bridge driver and the sub-device driver
will never modify or use it.

这两个调用的第二个参数是一个组ID。如果为0，则调用所有子设备。如果非零，则只调用其组ID与该值匹配的子设备。在桥接驱动程序注册子设备之前，它可以将`sd <v4l2_subdev>`->grp_id设置为任何值（默认为0）。这个值由桥接驱动程序拥有，子设备驱动程序永远不会修改或使用它。

The group ID gives the bridge driver more control how callbacks are called.
For example, there may be multiple audio chips on a board, each capable of
changing the volume. But usually only one will actually be used when the
user want to change the volume. You can set the group ID for that subdev to
e.g. AUDIO_CONTROLLER and specify that as the group ID value when calling
``v4l2_device_call_all()``. That ensures that it will only go to the subdev
that needs it.

组ID为桥接驱动程序提供了更多控制回调如何被调用的方式。例如，一个板上可能有多个音频芯片，每个都能够更改音量。但通常，当用户要更改音量时，只有一个会实际使用。您可以为该子设备设置组ID，例如`AUDIO_CONTROLLER`，并在调用`v4l2_device_call_all()`时将其指定为组ID值。这确保它只会传递给需要它的子设备。

If the sub-device needs to notify its v4l2_device parent of an event, then
it can call ``v4l2_subdev_notify(sd, notification, arg)``. This macro checks
whether there is a ``notify()`` callback defined and returns ``-ENODEV`` if not.
Otherwise the result of the ``notify()`` call is returned.

如果子设备需要通知其v4l2_device父设备发生的事件，那么它可以调用`v4l2_subdev_notify(sd, notification, arg)`。此宏检查是否定义了`notify()`回调，如果没有，则返回`-ENODEV`。否则，返回`notify()`调用的结果。

# 2.8 V4L2 sub-device userspace API

Bridge drivers traditionally expose one or multiple video nodes to userspace,
and control subdevices through the `v4l2_subdev_ops` operations in
response to video node operations. This hides the complexity of the underlying
hardware from applications. For complex devices, finer-grained control of the
device than what the video nodes offer may be required. In those cases, bridge
drivers that implement `the media controller API <media_controller>` may
opt for making the subdevice operations directly accessible from userspace.

传统上，桥接驱动程序向用户空间公开一个或多个视频节点，并通过对视频节点操作的响应中的`v4l2_subdev_ops`操作来控制子设备。这将底层硬件的复杂性隐藏在应用程序中。对于复杂设备，可能需要比视频节点提供的更精细的设备控制。在这些情况下，实现`媒体控制器API <media_controller>`的桥接驱动程序可能选择使子设备操作直接从用户空间访问。

Device nodes named ``v4l-subdev``\ *X* can be created in ``/dev`` to access
sub-devices directly. If a sub-device supports direct userspace configuration
it must set the ``V4L2_SUBDEV_FL_HAS_DEVNODE`` flag before being registered.

可以在`/dev`中创建命名为`v4l-subdev` *X*的设备节点，以直接访问子设备。如果子设备支持直接的用户空间配置，则必须在注册之前设置`V4L2_SUBDEV_FL_HAS_DEVNODE`标志。

After registering sub-devices, the `v4l2_device` driver can create
device nodes for all registered sub-devices marked with
``V4L2_SUBDEV_FL_HAS_DEVNODE`` by calling
`v4l2_device_register_subdev_nodes`. Those device nodes will be
automatically removed when sub-devices are unregistered.

在注册子设备之后，`v4l2_device`驱动程序可以通过调用`v4l2_device_register_subdev_nodes`为所有标记有`V4L2_SUBDEV_FL_HAS_DEVNODE`的已注册子设备创建设备节点。这些设备节点将在取消注册子设备时自动删除。

The device node handles a subset of the V4L2 API.

设备节点处理V4L2 API的子集。

``VIDIOC_QUERYCTRL``,
``VIDIOC_QUERYMENU``,
``VIDIOC_G_CTRL``,
``VIDIOC_S_CTRL``,
``VIDIOC_G_EXT_CTRLS``,
``VIDIOC_S_EXT_CTRLS`` and
``VIDIOC_TRY_EXT_CTRLS``:

    The controls ioctls are identical to the ones defined in V4L2. They
	behave identically, with the only exception that they deal only with
	controls implemented in the sub-device. Depending on the driver, those
	controls can be also be accessed through one (or several) V4L2 device
	nodes.

    控制的IOCTL与V4L2中定义的相同。它们的行为相同，唯一的异常是它们仅处理在子设备中实现的控制。根据驱动程序，这些控制也可以通过一个（或多个）V4L2设备节点访问。

``VIDIOC_DQEVENT``,
``VIDIOC_SUBSCRIBE_EVENT`` and
``VIDIOC_UNSUBSCRIBE_EVENT``

	The events ioctls are identical to the ones defined in V4L2. They
	behave identically, with the only exception that they deal only with
	events generated by the sub-device. Depending on the driver, those
	events can also be reported by one (or several) V4L2 device nodes.

    事件的IOCTL与V4L2中定义的相同。它们的行为相同，唯一的例外是它们仅处理由子设备生成的事件。根据驱动程序，这些事件也可以由一个（或多个）V4L2设备节点报告。

	Sub-device drivers that want to use events need to set the
	``V4L2_SUBDEV_FL_HAS_EVENTS`` `v4l2_subdev`.flags before registering
	the sub-device. After registration events can be queued as usual on the
	`v4l2_subdev`.devnode device node.

    想要使用事件的子设备驱动程序需要在注册子设备之前设置`v4l2_subdev`的`V4L2_SUBDEV_FL_HAS_EVENTS`标志。注册后，事件可以像往常一样排队在`v4l2_subdev`.devnode设备节点上。

	To properly support events, the ``poll()`` file operation is also
	implemented.

    为了正确支持事件，还实现了`poll()`文件操作。

Private ioctls

	All ioctls not in the above list are passed directly to the sub-device
	driver through the core::ioctl operation.

    所有不在上述列表中的IOCTL将通过`core::ioctl`操作直接传递给子设备驱动程序。

# 2.9 Read-only sub-device userspace API

Bridge drivers that control their connected subdevices through direct calls to
the kernel API realized by `v4l2_subdev_ops` structure do not usually
want userspace to be able to change the same parameters through the subdevice
device node and thus do not usually register any.

通过对`v4l2_subdev_ops`结构实现的内核API直接调用来控制其连接的子设备的桥接驱动程序通常不希望用户空间通过子设备设备节点更改相同的参数，因此通常不注册任何设备节点。

It is sometimes useful to report to userspace the current subdevice
configuration through a read-only API, that does not permit applications to
change to the device parameters but allows interfacing to the subdevice device
node to inspect them.

有时候，通过只读API向用户空间报告当前子设备配置是有用的，这不允许应用程序更改设备参数，但允许通过子设备设备节点对其进行检查。

For instance, to implement cameras based on computational photography, userspace
needs to know the detailed camera sensor configuration (in terms of skipping,
binning, cropping and scaling) for each supported output resolution. To support
such use cases, bridge drivers may expose the subdevice operations to userspace
through a read-only API.

例如，为了实现基于计算摄影的摄像头，用户空间需要了解每个支持的输出分辨率的详细摄像头传感器配置（跳帧、分bin、裁剪和缩放）。为了支持这样的用例，桥接驱动程序可能会通过只读API向用户空间公开子设备操作。

To create a read-only device node for all the subdevices registered with the
``V4L2_SUBDEV_FL_HAS_DEVNODE`` set, the `v4l2_device` driver should call
`v4l2_device_register_ro_subdev_nodes`.

为了为所有注册了`V4L2_SUBDEV_FL_HAS_DEVNODE`标志的子设备创建只读设备节点，`v4l2_device`驱动程序应调用`v4l2_device_register_ro_subdev_nodes`。

Access to the following ioctls for userspace applications is restricted on
sub-device device nodes registered with
`v4l2_device_register_ro_subdev_nodes`.

对于使用`v4l2_device_register_ro_subdev_nodes`注册的子设备设备节点，用户空间应用程序对以下IOCTL的访问受到限制。

``VIDIOC_SUBDEV_S_FMT``,
``VIDIOC_SUBDEV_S_CROP``,
``VIDIOC_SUBDEV_S_SELECTION``:

	These ioctls are only allowed on a read-only subdevice device node
	for the `V4L2_SUBDEV_FORMAT_TRY <v4l2-subdev-format-whence>`
	formats and selection rectangles.

    这些IOCTL仅允许在只读子设备设备节点上对`V4L2_SUBDEV_FORMAT_TRY <v4l2-subdev-format-whence>`格式和选择矩形进行操作。

``VIDIOC_SUBDEV_S_FRAME_INTERVAL``,
``VIDIOC_SUBDEV_S_DV_TIMINGS``,
``VIDIOC_SUBDEV_S_STD``:

	These ioctls are not allowed on a read-only subdevice node.

    在只读子设备节点上不允许执行这些IOCTL。

In case the ioctl is not allowed, or the format to modify is set to
``V4L2_SUBDEV_FORMAT_ACTIVE``, the core returns a negative error code and
the errno variable is set to ``-EPERM``.

如果不允许执行ioctl，或者要修改的格式设置为`V4L2_SUBDEV_FORMAT_ACTIVE`，则核心返回负错误代码，并将errno变量设置为`-EPERM`。

# 2.10 I2C sub-device drivers

Since these drivers are so common, special helper functions are available to
ease the use of these drivers (``v4l2-common.h``).

由于这些驱动程序非常常见，因此提供了特殊的辅助函数来简化对这些驱动程序的使用（`v4l2-common.h`）。

The recommended method of adding `v4l2_subdev` support to an I2C driver
is to embed the `v4l2_subdev` struct into the state struct that is
created for each I2C device instance. Very simple devices have no state
struct and in that case you can just create a `v4l2_subdev` directly.

向I2C驱动程序添加`v4l2_subdev`支持的推荐方法是将`v4l2_subdev`结构嵌入到为每个I2C设备实例创建的状态结构中。非常简单的设备没有状态结构，在这种情况下，您可以直接创建一个`v4l2_subdev`。

A typical state struct would look like this (where 'chipname' is replaced by
the name of the chip):

典型的状态结构可能如下所示（其中'chipname'被替换为芯片的名称）：

```c
	struct chipname_state {
		struct v4l2_subdev sd;
		...  /* additional state fields */
	};
```

Initialize the `v4l2_subdev` struct as follows:

初始化`v4l2_subdev`结构如下：

```c
	v4l2_i2c_subdev_init(&state->sd, client, subdev_ops);
```

This function will fill in all the fields of `v4l2_subdev` ensure that
the `v4l2_subdev` and i2c_client both point to one another.

此函数将填充`v4l2_subdev`的所有字段，确保`v4l2_subdev`和i2c_client相互引用。

You should also add a helper inline function to go from a `v4l2_subdev`
pointer to a chipname_state struct:

您还应该添加一个辅助的内联函数，以从`v4l2_subdev`指针转到`chipname_state`结构：

```c
	static inline struct chipname_state *to_state(struct v4l2_subdev *sd)
	{
		return container_of(sd, struct chipname_state, sd);
	}
```

Use this to go from the `v4l2_subdev` struct to the ``i2c_client``
struct:

使用这个函数从`v4l2_subdev`结构转到`i2c_client`结构：

```c
	struct i2c_client *client = v4l2_get_subdevdata(sd);
```

And this to go from an ``i2c_client`` to a `v4l2_subdev` struct:

使用这个函数从`i2c_client`转到`v4l2_subdev`结构：

```c
	struct v4l2_subdev *sd = i2c_get_clientdata(client);
```

Make sure to call `v4l2_device_unregister_subdev`
when the ``remove()`` callback is called. This will unregister the sub-device
from the bridge driver. It is safe to call this even if the sub-device was
never registered.

确保在调用`remove()`回调时调用`v4l2_device_unregister_subdev`。这将从桥接驱动程序中注销子设备。即使子设备从未注册，调用这个函数也是安全的。

You need to do this because when the bridge driver destroys the i2c adapter
the ``remove()`` callbacks are called of the i2c devices on that adapter.
After that the corresponding v4l2_subdev structures are invalid, so they
have to be unregistered first. Calling `v4l2_device_unregister_subdev`
from the ``remove()`` callback ensures that this is always done correctly.

需要这样做是因为当桥接驱动程序销毁i2c适配器时，会调用该适配器上的i2c设备的`remove()`回调。之后，相应的`v4l2_subdev`结构将无效，因此必须首先注销它们。从`remove()`回调中调用`v4l2_device_unregister_subdev`确保这总是正确完成的。

The bridge driver also has some helper functions it can use:

桥接驱动程序还有一些可用的辅助函数：

```c
	struct v4l2_subdev *sd = v4l2_i2c_new_subdev(v4l2_dev, adapter,
					"module_foo", "chipid", 0x36, NULL);
```

This loads the given module (can be ``NULL`` if no module needs to be loaded)
and calls `i2c_new_client_device` with the given ``i2c_adapter`` and
chip/address arguments. If all goes well, then it registers the subdev with
the v4l2_device.

这会加载给定的模块（如果不需要加载模块，则可以为`NULL`），并使用给定的`i2c_adapter`和芯片/地址参数调用`i2c_new_client_device`。如果一切顺利，它将在v4l2_device中注册子设备。

You can also use the last argument of `v4l2_i2c_new_subdev` to pass
an array of possible I2C addresses that it should probe. These probe addresses
are only used if the previous argument is 0. A non-zero argument means that you
know the exact i2c address so in that case no probing will take place.

您还可以使用`v4l2_i2c_new_subdev`的最后一个参数传递一个可能的I2C地址数组，它应该探测这些地址。只有在前一个参数为0时，才使用这些探测地址。非零参数表示您知道确切的i2c地址，因此在这种情况下将不进行探测。

Both functions return ``NULL`` if something went wrong.

两个函数如果出现错误，都会返回`NULL`。

Note that the chipid you pass to `v4l2_i2c_new_subdev` is usually
the same as the module name. It allows you to specify a chip variant, e.g.
"saa7114" or "saa7115". In general though the i2c driver autodetects this.
The use of chipid is something that needs to be looked at more closely at a
later date. It differs between i2c drivers and as such can be confusing.
To see which chip variants are supported you can look in the i2c driver code
for the i2c_device_id table. This lists all the possibilities.

请注意，传递给`v4l2_i2c_new_subdev`的chipid通常与模块名相同。它允许您指定芯片的变体，例如"saa7114"或"saa7115"。然而，通常i2c驱动程序会自动检测这个。对于chipid的使用需要在以后更仔细地研究一下。它在i2c驱动程序之间有所不同，因此可能会令人困惑。要查看支持哪些芯片变体，可以查看i2c驱动程序代码中的i2c_device_id表。这列出了所有可能性。

There are one more helper function:

还有一个辅助函数：

`v4l2_i2c_new_subdev_board` uses an `i2c_board_info` struct
which is passed to the i2c driver and replaces the irq, platform_data and addr
arguments.

`v4l2_i2c_new_subdev_board` 使用一个`i2c_board_info`结构，该结构传递给i2c驱动程序，替换irq、platform_data和addr参数。

If the subdev supports the s_config core ops, then that op is called with
the irq and platform_data arguments after the subdev was setup.

如果子设备支持`s_config`核心操作，那么在设置子设备之后，该操作将使用irq和platform_data参数调用。

The `v4l2_i2c_new_subdev` function will call
`v4l2_i2c_new_subdev_board`, internally filling a
`i2c_board_info` structure using the ``client_type`` and the
``addr`` to fill it.

`v4l2_i2c_new_subdev`函数将调用`v4l2_i2c_new_subdev_board`，内部使用`client_type`和`addr`来填充一个`i2c_board_info`结构。

# 2.11 Centrally managed subdev active state

Traditionally V4L2 subdev drivers maintained internal state for the active
device configuration. This is often implemented as e.g. an array of struct
v4l2_mbus_framefmt, one entry for each pad, and similarly for crop and compose
rectangles.

传统上，V4L2子设备驱动程序为活动设备配置维护了内部状态。通常，这被实现为例如`struct v4l2_mbus_framefmt`的数组，每个pad对应一个条目，crop和compose矩形也是如此。

In addition to the active configuration, each subdev file handle has an array of
struct v4l2_subdev_pad_config, managed by the V4L2 core, which contains the try
configuration.

除了活动配置之外，每个子设备文件句柄都有一个由V4L2核心管理的`struct v4l2_subdev_pad_config`数组，其中包含try配置。

To simplify the subdev drivers the V4L2 subdev API now optionally supports a
centrally managed active configuration represented by
`v4l2_subdev_state`. One instance of state, which contains the active
device configuration, is stored in the sub-device itself as part of
the `v4l2_subdev` structure, while the core associates a try state to
each open file handle, to store the try configuration related to that file
handle.

为了简化子设备驱动程序，V4L2子设备API现在可选地支持由`v4l2_subdev_state`表示的集中管理的活动配置。该状态的一个实例，其中包含活动设备配置，存储在子设备本身中，作为`v4l2_subdev`结构的一部分，而核心则将try状态与每个打开的文件句柄关联起来，以存储与该文件句柄相关的try配置。

Sub-device drivers can opt-in and use state to manage their active configuration
by initializing the subdevice state with a call to v4l2_subdev_init_finalize()
before registering the sub-device. They must also call v4l2_subdev_cleanup()
to release all the allocated resources before unregistering the sub-device.
The core automatically allocates and initializes a state for each open file
handle to store the try configurations and frees it when closing the file
handle.

子设备驱动程序可以选择使用状态来管理其活动配置，通过在注册子设备之前调用`v4l2_subdev_init_finalize()`初始化子设备状态。在取消注册子设备之前，它们还必须调用`v4l2_subdev_cleanup()`释放所有分配的资源。核心会为每个打开的文件句柄自动分配和初始化一个状态，以存储try配置，并在关闭文件句柄时释放它。

V4L2 sub-device operations that use both the `ACTIVE and TRY formats`
receive the correct state to operate on through
the 'state' parameter. The state must be locked and unlocked by the
caller by calling `v4l2_subdev_lock_state()` and
`v4l2_subdev_unlock_state()`. The caller can do so by calling the subdev
operation through the `v4l2_subdev_call_state_active()` macro.

对于同时使用`ACTIVE和TRY格式`的V4L2子设备操作，通过'state'参数接收正确的状态进行操作。调用者必须通过调用`v4l2_subdev_lock_state()`和`v4l2_subdev_unlock_state()`来锁定和解锁状态。调用者可以通过通过`v4l2_subdev_call_state_active()`宏调用子设备操作来实现这一点。

Operations that do not receive a state parameter implicitly operate on the
subdevice active state, which drivers can exclusively access by
calling `v4l2_subdev_lock_and_get_active_state()`. The sub-device active
state must equally be released by calling `v4l2_subdev_unlock_state()`.

不接收状态参数的操作隐式地操作于子设备活动状态上，驱动程序可以通过调用`v4l2_subdev_lock_and_get_active_state()`独占性地访问子设备活动状态。子设备活动状态同样必须通过调用`v4l2_subdev_unlock_state()`释放。

Drivers must never manually access the state stored in the `v4l2_subdev`
or in the file handle without going through the designated helpers.

驱动程序绝不能在没有经过指定的辅助程序的情况下直接访问存储在`v4l2_subdev`或文件句柄中的状态。

While the V4L2 core passes the correct try or active state to the subdevice
operations, many existing device drivers pass a NULL state when calling
operations with `v4l2_subdev_call()`. This legacy construct causes
issues with subdevice drivers that let the V4L2 core manage the active state,
as they expect to receive the appropriate state as a parameter. To help the
conversion of subdevice drivers to a managed active state without having to
convert all callers at the same time, an additional wrapper layer has been
added to v4l2_subdev_call(), which handles the NULL case by getting and locking
the callee's active state with `v4l2_subdev_lock_and_get_active_state()`,
and unlocking the state after the call.

虽然V4L2核心将正确的try或活动状态传递给子设备操作，但许多现有的设备驱动程序在调用具有`v4l2_subdev_call()`的操作时传递了空状态。这种传统的构造对让V4L2核心管理活动状态的子设备驱动程序造成了问题，因为它们期望接收适当的状态作为参数。为了帮助子设备驱动程序将其转换为受管理的活动状态，而无需同时转换所有调用者，`v4l2_subdev_call()`添加了一个额外的包装层，通过使用`v4l2_subdev_lock_and_get_active_state()`获取和锁定被调用者的活动状态，并在调用后解锁该状态，来处理空状态的情况。

The whole subdev state is in reality split into three parts: the
v4l2_subdev_state, subdev controls and subdev driver's internal state. In the
future these parts should be combined into a single state. For the time being
we need a way to handle the locking for these parts. This can be accomplished
by sharing a lock. The v4l2_ctrl_handler already supports this via its 'lock'
pointer and the same model is used with states. The driver can do the following
before calling v4l2_subdev_init_finalize():

整个子设备状态实际上分为三个部分：`v4l2_subdev_state`、子设备控制和子设备驱动程序的内部状态。在将来，这些部分应该合并为一个单一的状态。目前我们需要一种方式来处理这些部分的锁定。这可以通过共享锁来实现。`v4l2_ctrl_handler`已经通过其'lock'指针支持了这一点，状态也使用相同的模型。在调用`v4l2_subdev_init_finalize()`之前，驱动程序可以执行以下操作：

```c
	sd->ctrl_handler->lock = &priv->mutex;
	sd->state_lock = &priv->mutex;
```

This shares the driver's private mutex between the controls and the states.

这将在控件和状态之间共享驱动程序

# 2.12 Streams, multiplexed media pads and internal routing

A subdevice driver can implement support for multiplexed streams by setting
the V4L2_SUBDEV_FL_STREAMS subdev flag and implementing support for
centrally managed subdev active state, routing and stream based
configuration.

子设备驱动程序可以通过设置V4L2_SUBDEV_FL_STREAMS子设备标志并实现对集中管理的子设备活动状态、路由和基于流的配置的支持来实现对复用流的支持。

# 2.13 V4L2 sub-device functions and data structures

    include/media/v4l2-subdev.h

# 2.14 V4L2 events

The V4L2 events provide a generic way to pass events to user space.
The driver must use `v4l2_fh` to be able to support V4L2 events.

V4L2 事件提供了一种通用的方式将事件传递到用户空间。驱动程序必须使用 `v4l2_fh` 来支持 V4L2 事件。

Events are subscribed per-filehandle. An event specification consists of a
``type`` and is optionally associated with an object identified through the
``id`` field. If unused, then the ``id`` is 0. So an event is uniquely
identified by the ``(type, id)`` tuple.

事件是按文件句柄订阅的。事件规范由 `type` 构成，并可选地与 `id` 字段关联，如果未使用，则 `id` 为 0。因此，事件由 `(type, id)` 元组唯一标识。

The `v4l2_fh` struct has a list of subscribed events on its
``subscribed`` field.

`v4l2_fh` 结构具有其 `subscribed` 字段中的已订阅事件列表。

When the user subscribes to an event, a `v4l2_subscribed_event`
struct is added to `v4l2_fh.subscribed`, one for every
subscribed event.

当用户订阅事件时，对于每个订阅的事件，都会向 `v4l2_fh` 的 `subscribed` 字段添加一个 `v4l2_subscribed_event` 结构。

Each `v4l2_subscribed_event` struct ends with a
`v4l2_kevent` ringbuffer, with the size given by the caller
of `v4l2_event_subscribe`. This ringbuffer is used to store any events
raised by the driver.

每个 `v4l2_subscribed_event` 结构都以一个 `v4l2_kevent` 环形缓冲区结束，其大小由 `v4l2_event_subscribe` 的调用者给出。此环形缓冲区用于存储驱动程序生成的任何事件。

So every ``(type, ID)`` event tuple will have its own
`v4l2_kevent` ringbuffer. This guarantees that if a driver is
generating lots of events of one type in a short time, then that will
not overwrite events of another type.

因此，每个 `(type, ID)` 事件元组都有其自己的 `v4l2_kevent` 环形缓冲区。这保证了如果一个驱动程序在短时间内生成大量相同类型的事件，那么不会覆盖另一类型的事件。

But if you get more events of one type than the size of the
`v4l2_kevent` ringbuffer, then the oldest event will be dropped
and the new one added.

但是，如果某一类型的事件比 `v4l2_kevent` 环形缓冲区的大小多，那么最旧的事件将被丢弃，新事件将被添加。

The `v4l2_kevent` struct links into the ``available``
list of the `v4l2_fh` struct so `VIDIOC_DQEVENT` will
know which event to dequeue first.

`v4l2_kevent` 结构链接到 `v4l2_fh` 结构的 ``available`` 列表中，这样 `VIDIOC_DQEVENT` 就知道要首先出列哪个事件。

Finally, if the event subscription is associated with a particular object
such as a V4L2 control, then that object needs to know about that as well
so that an event can be raised by that object. So the ``node`` field can
be used to link the `v4l2_subscribed_event` struct into a list of
such objects.

最后，如果事件订阅与特定对象相关联，例如 V4L2 控制，则该对象也需要知道这一点，以便可以由该对象引发事件。因此，``node`` 字段可以用于将 `v4l2_subscribed_event` 结构链接到这些对象的列表中。

So to summarize:

因此，总结一下：

- struct v4l2_fh has two lists: one of the ``subscribed`` events,
  and one of the ``available`` events.
- `struct v4l2_fh` 有两个列表：一个是 ``subscribed`` 事件列表，
  另一个是 ``available`` 事件列表。

- struct v4l2_subscribed_event has a ringbuffer of raised
  (pending) events of that particular type.
- `struct v4l2_subscribed_event` 有一个提升
  (pending) 的特定类型事件的环形缓冲区。

- If struct v4l2_subscribed_event is associated with a specific
  object, then that object will have an internal list of
  struct v4l2_subscribed_event so it knows who subscribed an
  event to that object.
- 如果 `struct v4l2_subscribed_event` 与特定
  对象相关联，那么该对象将具有一个内部列表，
  其中包含订阅该对象的 `struct v4l2_subscribed_event`。

Furthermore, the internal struct v4l2_subscribed_event has
``merge()`` and ``replace()`` callbacks which drivers can set. These
callbacks are called when a new event is raised and there is no more room.

此外，内部的 `struct v4l2_subscribed_event` 具有
``merge()`` 和 ``replace()`` 回调，驱动程序可以设置这些
回调，在生成新事件并没有足够空间时将调用这些回调。

The ``replace()`` callback allows you to replace the payload of the old event
with that of the new event, merging any relevant data from the old payload
into the new payload that replaces it. It is called when this event type has
a ringbuffer with size is one, i.e. only one event can be stored in the
ringbuffer.

`replace()`回调允许您使用新事件的数据替换旧事件的数据，合并从旧数据中提取的任何相关数据到替换它的新数据。当此事件类型的环形缓冲区大小为一时，将调用此回调，即只能存储一个事件。

The ``merge()`` callback allows you to merge the oldest event payload into
that of the second-oldest event payload. It is called when
the ringbuffer has size is greater than one.

`merge()` 回调允许将最旧事件的数据合并到第二旧事件的数据中。当环形缓冲区的大小大于一时，将调用此回调。

This way no status information is lost, just the intermediate steps leading
up to that state.

这样就不会丢失任何状态信息，只是将导致该状态的中间步骤合并到该状态的新事件中。

A good example of these ``replace``/``merge`` callbacks is in v4l2-event.c:
``ctrls_replace()`` and ``ctrls_merge()`` callbacks for the control event.

**Note**

	these callbacks can be called from interrupt context, so they must be fast.

	这里需要注意的是，这些回调可能会从中断上下文中调用，因此必须执行得很快。

In order to queue events to video device, drivers should call:

为了将事件排队到视频设备，驱动程序应调用：

	v4l2_event_queue(vdev, ev);

The driver's only responsibility is to fill in the type and the data fields.
The other fields will be filled in by V4L2.

驱动程序的唯一责任是填充类型和数据字段。其他字段将由 V4L2 填充。

## 2.14.1 Event subscription

Subscribing to an event is via:

订阅事件使用：

	v4l2_event_subscribe(fh, sub, elems, ops)

This function is used to implement `video_device`->
`ioctl_ops <v4l2_ioctl_ops>`-> ``vidioc_subscribe_event``,
but the driver must check first if the driver is able to produce events
with specified event id, and then should call
`v4l2_event_subscribe` to subscribe the event.

此函数用于实现 `video_device` -> `ioctl_ops` -> `vidioc_subscribe_event`，但驱动程序必须首先检查驱动程序是否能够生成具有指定事件 id 的事件，然后应调用 `v4l2_event_subscribe` 来订阅该事件。

The elems argument is the size of the event queue for this event. If it is 0,
then the framework will fill in a default value (this depends on the event
type).

`elems` 参数是此事件的事件队列的大小。如果它为 0，则框架将填充默认值（这取决于事件类型）。

The ops argument allows the driver to specify a number of callbacks:

`ops` 参数允许驱动程序指定一些回调：

| Callback | Description |
| -------- | ----------- |
| add | called when a new listener gets added (subscribing to the same event twice will only cause this callback to get called once) |
| del | called when a listener stops listening |
| replace | replace event 'old' with event 'new'. |
| merge | merge event 'old' into event 'new'. |


All 4 callbacks are optional, if you don't want to specify any callbacks
the ops argument itself maybe ``NULL``.

这 4 个回调都是可选的，如果不想指定任何回调，`ops` 参数本身可能为 `NULL`。

## 2.14.2 Unsubscribing an event

Unsubscribing to an event is via:

取消订阅事件使用：

	v4l2_event_unsubscribe(fh, sub)

This function is used to implement `video_device`->
`ioctl_ops <v4l2_ioctl_ops>`-> ``vidioc_unsubscribe_event``.
A driver may call `v4l2_event_unsubscribe` directly unless it
wants to be involved in unsubscription process.

此函数用于实现 `video_device` -> `ioctl_ops` -> `vidioc_unsubscribe_event`。驱动程序可以直接调用 `v4l2_event_unsubscribe`，除非它想要参与取消订阅过程。

The special type ``V4L2_EVENT_ALL`` may be used to unsubscribe all events. The
drivers may want to handle this in a special way.

特殊的类型 `V4L2_EVENT_ALL` 可以用于取消订阅所有事件。驱动程序可能希望以特殊方式处理这种情况。

## 2.14.3 Check if there's a pending event

Checking if there's a pending event is via:

检查是否有待处理的事件使用：

	v4l2_event_pending(fh)

This function returns the number of pending events. Useful when implementing
poll.

此函数返回待处理事件的数量。在实现 `poll` 时非常有用。

## 2.14.4 How events work

Events are delivered to user space through the poll system call. The driver
can use `v4l2_fh`->wait (a wait_queue_head_t) as the argument for
``poll_wait()``.

通过 `poll` 系统调用将事件传递到用户空间。驱动程序可以使用 `v4l2_fh`->`wait`（一个 `wait_queue_head_t`）作为 `poll_wait()` 的参数。

There are standard and private events. New standard events must use the
smallest available event type. The drivers must allocate their events from
their own class starting from class base. Class base is
``V4L2_EVENT_PRIVATE_START`` + n * 1000 where n is the lowest available number.
The first event type in the class is reserved for future use, so the first
available event type is 'class base + 1'.

有标准事件和私有事件。新的标准事件必须使用最小的可用事件类型。驱动程序必须从其自己的类中分配其事件，从类基数开始。类基数为 `V4L2_EVENT_PRIVATE_START` + n * 1000，其中 n 是最低可用数字。类中的第一个事件类型保留供将来使用，因此第一个可用事件类型为 '类基数 + 1'。

An example on how the V4L2 events may be used can be found in the OMAP
3 ISP driver (``drivers/media/platform/ti/omap3isp``).

在 OMAP 3 ISP 驱动程序（`drivers/media/platform/ti/omap3isp`）中可以找到如何使用 V4L2 事件的示例。

A subdev can directly send an event to the `v4l2_device` notify
function with ``V4L2_DEVICE_NOTIFY_EVENT``. This allows the bridge to map
the subdev that sends the event to the video node(s) associated with the
subdev that need to be informed about such an event.

子设备可以直接通过 `V4L2_DEVICE_NOTIFY_EVENT` 向 `v4l2_device` 通知函数发送事件。这使得桥能够将发送事件的子设备映射到与需要了解此类事件的子设备关联的视频节点。

### 2.14.4.1 V4L2 event functions and data structures

	include/media/v4l2-event.h

# 2.15 V4L2 Controls

## 2.15.1 Introduction

The V4L2 control API seems simple enough, but quickly becomes very hard to
implement correctly in drivers. But much of the code needed to handle controls
is actually not driver specific and can be moved to the V4L core framework.

V4L2 控制 API 看起来足够简单，但在驱动程序中正确实现它很快变得非常困难。但是，处理控制所需的许多代码实际上并不是驱动程序特定的，可以移到 V4L 核心框架中。

After all, the only part that a driver developer is interested in is:

毕竟，驱动程序开发人员感兴趣的唯一部分是：

1) How do I add a control?
2) How do I set the control's value? (i.e. s_ctrl)

1) 如何添加一个控制？
2) 如何设置控制的值？(即 s_ctrl)

And occasionally:

3) How do I get the control's value? (i.e. g_volatile_ctrl)
4) How do I validate the user's proposed control value? (i.e. try_ctrl)

3) 如何获取控制的值？(即 g_volatile_ctrl)
4) 如何验证用户提出的控制值？(即 try_ctrl)

All the rest is something that can be done centrally.

其余的事情都可以集中完成。

The control framework was created in order to implement all the rules of the
V4L2 specification with respect to controls in a central place. And to make
life as easy as possible for the driver developer.

控制框架是为了在中心地方实现关于 V4L2 规范控制的所有规则而创建的。并且尽量使驱动程序开发变得尽可能简单。

Note that the control framework relies on the presence of a struct
`v4l2_device` for V4L2 drivers and struct v4l2_subdev for
sub-device drivers.

请注意，控制框架依赖于 V4L2 驱动程序的 struct v4l2_device 和子设备驱动程序的 struct v4l2_subdev的存在。

## 2.15.2 Objects in the framework

There are two main objects:

有两个主要对象：

The `v4l2_ctrl` object describes the control properties and keeps
track of the control's value (both the current value and the proposed new
value).

struct v4l2_ctrl 对象描述了控制属性并跟踪控制的值（当前值和建议的新值）。

`v4l2_ctrl_handler` is the object that keeps track of controls. It
maintains a list of v4l2_ctrl objects that it owns and another list of
references to controls, possibly to controls owned by other handlers.

struct v4l2_ctrl_handler 是保持控制的对象。它维护一个它拥有的 v4l2_ctrl 对象的列表和另一个可能是其他处理程序拥有的控制的引用列表。

## 2.15.3 Basic usage for V4L2 and sub-device drivers

1. Prepare the driver:

	准备驱动程序：


	```c
	#include <media/v4l2-ctrls.h>
	```

- 1.1) Add the handler to your driver's top-level struct:
	
	将处理程序添加到驱动程序的顶层结构：

	For V4L2 drivers:

	```c
	struct foo_dev {
		...
		struct v4l2_device v4l2_dev;
		...
		struct v4l2_ctrl_handler ctrl_handler;
		...
	};
	```

	For sub-device drivers:

	```c
	struct foo_dev {
		...
		struct v4l2_subdev sd;
		...
		struct v4l2_ctrl_handler ctrl_handler;
		...
	};
	```

- 1.2) Initialize the handler:

	```c
	v4l2_ctrl_handler_init(&foo->ctrl_handler, nr_of_controls);
	```

	The second argument is a hint telling the function how many controls this
	handler is expected to handle. It will allocate a hashtable based on this
	information. It is a hint only.

	第二个参数是一个提示，告诉函数此处理程序预计将处理多少个控件。它将根据此信息分配一个基于哈希表的数据结构。这只是一个提示。

- 1.3) Hook the control handler into the driver:

	For V4L2 drivers:

	```c
	foo->v4l2_dev.ctrl_handler = &foo->ctrl_handler;
	```

	For sub-device drivers:

	```c
	foo->sd.ctrl_handler = &foo->ctrl_handler;
	```

- 1.4) Clean up the handler at the end:

	```c
	v4l2_ctrl_handler_free(&foo->ctrl_handler);
	```

2. Add controls:

	You add non-menu controls by calling `v4l2_ctrl_new_std`:

	通过调用 v4l2_ctrl_new_std 来添加非菜单控制：

	```c
	struct v4l2_ctrl *v4l2_ctrl_new_std(struct v4l2_ctrl_handler *hdl,
			const struct v4l2_ctrl_ops *ops,
			u32 id, s32 min, s32 max, u32 step, s32 def);
	```

	Menu and integer menu controls are added by calling
	`v4l2_ctrl_new_std_menu`:

	通过调用 v4l2_ctrl_new_std_menu 来添加菜单和整数菜单控制：

	```c
	struct v4l2_ctrl *v4l2_ctrl_new_std_menu(struct v4l2_ctrl_handler *hdl,
			const struct v4l2_ctrl_ops *ops,
			u32 id, s32 max, s32 skip_mask, s32 def);
	```

	Menu controls with a driver specific menu are added by calling
	`v4l2_ctrl_new_std_menu_items`:

	通过调用 v4l2_ctrl_new_std_menu_items 来添加具有驱动程序特定菜单的菜单控制：

	```c
       struct v4l2_ctrl *v4l2_ctrl_new_std_menu_items(
                       struct v4l2_ctrl_handler *hdl,
                       const struct v4l2_ctrl_ops *ops, u32 id, s32 max,
                       s32 skip_mask, s32 def, const char * const *qmenu);
	```

	Standard compound controls can be added by calling
	`v4l2_ctrl_new_std_compound`:

	可以通过调用 `v4l2_ctrl_new_std_compound` 函数添加标准复合控件：

	```c
       struct v4l2_ctrl *v4l2_ctrl_new_std_compound(struct v4l2_ctrl_handler *hdl,
                       const struct v4l2_ctrl_ops *ops, u32 id,
                       const union v4l2_ctrl_ptr p_def);
	```

	Integer menu controls with a driver specific menu can be added by calling
	`v4l2_ctrl_new_int_menu`:

	可以通过调用 `v4l2_ctrl_new_int_menu` 函数添加具有驱动程序特定菜单的整数菜单控件：

	```c
	struct v4l2_ctrl *v4l2_ctrl_new_int_menu(struct v4l2_ctrl_handler *hdl,
			const struct v4l2_ctrl_ops *ops,
			u32 id, s32 max, s32 def, const s64 *qmenu_int);
	```

	These functions are typically called right after the
	`v4l2_ctrl_handler_init`:

	这些函数通常在 `v4l2_ctrl_handler_init` 之后立即调用：

	```c
	static const s64 exp_bias_qmenu[] = {
	       -2, -1, 0, 1, 2
	};
	static const char * const test_pattern[] = {
		"Disabled",
		"Vertical Bars",
		"Solid Black",
		"Solid White",
	};

	v4l2_ctrl_handler_init(&foo->ctrl_handler, nr_of_controls);
	v4l2_ctrl_new_std(&foo->ctrl_handler, &foo_ctrl_ops,
			V4L2_CID_BRIGHTNESS, 0, 255, 1, 128);
	v4l2_ctrl_new_std(&foo->ctrl_handler, &foo_ctrl_ops,
			V4L2_CID_CONTRAST, 0, 255, 1, 128);
	v4l2_ctrl_new_std_menu(&foo->ctrl_handler, &foo_ctrl_ops,
			V4L2_CID_POWER_LINE_FREQUENCY,
			V4L2_CID_POWER_LINE_FREQUENCY_60HZ, 0,
			V4L2_CID_POWER_LINE_FREQUENCY_DISABLED);
	v4l2_ctrl_new_int_menu(&foo->ctrl_handler, &foo_ctrl_ops,
			V4L2_CID_EXPOSURE_BIAS,
			ARRAY_SIZE(exp_bias_qmenu) - 1,
			ARRAY_SIZE(exp_bias_qmenu) / 2 - 1,
			exp_bias_qmenu);
	v4l2_ctrl_new_std_menu_items(&foo->ctrl_handler, &foo_ctrl_ops,
			V4L2_CID_TEST_PATTERN, ARRAY_SIZE(test_pattern) - 1, 0,
			0, test_pattern);
	...
	if (foo->ctrl_handler.error) {
		int err = foo->ctrl_handler.error;

		v4l2_ctrl_handler_free(&foo->ctrl_handler);
		return err;
	}
	```

	The `v4l2_ctrl_new_std` function returns the v4l2_ctrl pointer to
	the new control, but if you do not need to access the pointer outside the
	control ops, then there is no need to store it.

	`v4l2_ctrl_new_std` 函数返回指向新控件的 `v4l2_ctrl` 指针，但如果您不需要在控件操作之外访问指针，则无需存储它。

	The `v4l2_ctrl_new_std` function will fill in most fields based on
	the control ID except for the min, max, step and default values. These are
	passed in the last four arguments. These values are driver specific while
	control attributes like type, name, flags are all global. The control's
	current value will be set to the default value.

	`v4l2_ctrl_new_std` 函数将基于控件 ID 填充大多数字段，除了最小、最大、步长和默认值。这些值是驱动程序特定的，而控件属性如类型、名称、标志等则是全局的。控件的当前值将设置为默认值。

	The `v4l2_ctrl_new_std_menu` function is very similar but it is
	used for menu controls. There is no min argument since that is always 0 for
	menu controls, and instead of a step there is a skip_mask argument: if bit
	X is 1, then menu item X is skipped.

	`v4l2_ctrl_new_std_menu` 函数非常相似，但用于菜单控件。由于菜单控件的最小值始终为 0，因此没有最小值参数，而替代步长的是 `skip_mask` 参数：如果第 X 位为 1，则跳过菜单项 X。

	The `v4l2_ctrl_new_int_menu` function creates a new standard
	integer menu control with driver-specific items in the menu. It differs
	from v4l2_ctrl_new_std_menu in that it doesn't have the mask argument and
	takes as the last argument an array of signed 64-bit integers that form an
	exact menu item list.

	`v4l2_ctrl_new_int_menu` 函数创建一个具有驱动程序特定菜单项的新标准整数菜单控件。它与 `v4l2_ctrl_new_std_menu` 的区别在于它没有掩码参数，而是将最后一个参数作为由带符号的 64 位整数组成的确切菜单项列表。

	The `v4l2_ctrl_new_std_menu_items` function is very similar to
	v4l2_ctrl_new_std_menu but takes an extra parameter qmenu, which is the
	driver specific menu for an otherwise standard menu control. A good example
	for this control is the test pattern control for capture/display/sensors
	devices that have the capability to generate test patterns. These test
	patterns are hardware specific, so the contents of the menu will vary from
	device to device.

	`v4l2_ctrl_new_std_menu_items` 函数与 `v4l2_ctrl_new_std_menu` 非常相似，但接受额外的参数 `qmenu`，它是用于标准菜单控件的驱动程序特定菜单。对于此控件的一个很好的例子是捕获/显示/传感器设备的测试图案控件，这些设备具有生成测试图案的能力。这些测试图案是硬件特定的，因此菜单的内容会因设备而异。

	Note that if something fails, the function will return NULL or an error and
	set ctrl_handler->error to the error code. If ctrl_handler->error was already
	set, then it will just return and do nothing. This is also true for
	v4l2_ctrl_handler_init if it cannot allocate the internal data structure.

	请注意，如果出现错误，函数将返回 `NULL` 或错误，并将 `ctrl_handler->error` 设置为错误代码。如果 `ctrl_handler->error` 已经设置，则它将仅返回而不执行任何操作。对于无法分配内部数据结构的 `v4l2_ctrl_handler_init` 也是如此。

	This makes it easy to init the handler and just add all controls and only check
	the error code at the end. Saves a lot of repetitive error checking.

	这使得很容易初始化处理程序并添加所有控件，仅在最后检查错误代码。这样可以节省很多重复的错误检查。

	It is recommended to add controls in ascending control ID order: it will be
	a bit faster that way.

	建议按升序控件 ID 顺序添加控件：这样做会更快一些。

3. Optionally force initial control setup:

	```c
	v4l2_ctrl_handler_setup(&foo->ctrl_handler);
	```

	This will call s_ctrl for all controls unconditionally. Effectively this
	initializes the hardware to the default control values. It is recommended
	that you do this as this ensures that both the internal data structures and
	the hardware are in sync.

	这将无条件地为所有控件调用 `s_ctrl`。实际上，这将使用默认控件值初始化硬件。建议您执行此操作，以确保内部数据结构和硬件保持同步。

4. Finally: implement the `v4l2_ctrl_ops`

	```c
	static const struct v4l2_ctrl_ops foo_ctrl_ops = {
		.s_ctrl = foo_s_ctrl,
	};
	```

	Usually all you need is s_ctrl:

	通常，您只需要实现 `s_ctrl`：

	```c
	static int foo_s_ctrl(struct v4l2_ctrl *ctrl)
	{
		struct foo *state = container_of(ctrl->handler, struct foo, ctrl_handler);

		switch (ctrl->id) {
		case V4L2_CID_BRIGHTNESS:
			write_reg(0x123, ctrl->val);
			break;
		case V4L2_CID_CONTRAST:
			write_reg(0x456, ctrl->val);
			break;
		}
		return 0;
	}
	```

	The control ops are called with the v4l2_ctrl pointer as argument.
	The new control value has already been validated, so all you need to do is
	to actually update the hardware registers.

	控件操作使用 `v4l2_ctrl` 指针作为参数调用。新的控件值已经经过验证，因此您唯一需要做的就是实际更新硬件寄存器。


	You're done! And this is sufficient for most of the drivers we have. No need
	to do any validation of control values, or implement QUERYCTRL, QUERY_EXT_CTRL
	and QUERYMENU. And G/S_CTRL as well as G/TRY/S_EXT_CTRLS are automatically supported.

	您已经完成了！这对于我们大多数的驱动程序来说已经足够了。无需验证控制值，也无需实现 `QUERYCTRL`、`QUERY_EXT_CTRL` 和 `QUERYMENU`。`G/S_CTRL` 以及 `G/TRY/S_EXT_CTRLS` 也会被自动支持。

**Note**

	The remainder sections deal with more advanced controls topics and scenarios.
   	In practice the basic usage as described above is sufficient for most drivers.

	剩余的部分涉及更高级的控件主题和场景。实际上，如上所述的基本用法对于大多数驱动程序已经足够。

## 2.15.4 Inheriting Sub-device Controls

When a sub-device is registered with a V4L2 driver by calling
v4l2_device_register_subdev() and the ctrl_handler fields of both v4l2_subdev
and v4l2_device are set, then the controls of the subdev will become
automatically available in the V4L2 driver as well. If the subdev driver
contains controls that already exist in the V4L2 driver, then those will be
skipped (so a V4L2 driver can always override a subdev control).

当通过调用 `v4l2_device_register_subdev()` 向 V4L2 驱动注册子设备，并设置了 `v4l2_subdev` 和 `v4l2_device` 的 `ctrl_handler` 字段时，子设备的控件将自动在 V4L2 驱动中变为可用。如果子设备驱动程序包含在 V4L2 驱动中已存在的控件，则会跳过这些控件（因此 V4L2 驱动始终可以覆盖子设备控件）。

What happens here is that v4l2_device_register_subdev() calls
v4l2_ctrl_add_handler() adding the controls of the subdev to the controls
of v4l2_device.

在这里发生的事情是，`v4l2_device_register_subdev()` 调用 `v4l2_ctrl_add_handler()`，将子设备的控件添加到 `v4l2_device` 的控件中。

## 2.15.5 Accessing Control Values

The following union is used inside the control framework to access
control values:

以下联合体在控件框架内用于访问控件值：

```c
	union v4l2_ctrl_ptr {
		s32 *p_s32;
		s64 *p_s64;
		char *p_char;
		void *p;
	};
```

The v4l2_ctrl struct contains these fields that can be used to access both
current and new values:

`v4l2_ctrl` 结构包含以下字段，可用于访问当前值和新值：

```c
	s32 val;
	struct {
		s32 val;
	} cur;


	union v4l2_ctrl_ptr p_new;
	union v4l2_ctrl_ptr p_cur;
```

If the control has a simple s32 type, then:

如果控件具有简单的 `s32` 类型，那么：

```c
	&ctrl->val == ctrl->p_new.p_s32
	&ctrl->cur.val == ctrl->p_cur.p_s32
```

For all other types use ctrl->p_cur.p<something>. Basically the val
and cur.val fields can be considered an alias since these are used so often.

对于所有其他类型，请使用 `ctrl->p_cur.p<something>`。基本上，`val` 和 `cur.val` 字段可以视为别名，因为它们经常被使用。

Within the control ops you can freely use these. The val and cur.val speak for
themselves. The p_char pointers point to character buffers of length
ctrl->maximum + 1, and are always 0-terminated.

在控件操作中，您可以自由使用这些字段。`val` 和 `cur.val` 是直观的。`p_char` 指针指向长度为 `ctrl->maximum + 1` 的字符缓冲区，并且始终以 0 结尾。

Unless the control is marked volatile the p_cur field points to the
current cached control value. When you create a new control this value is made
identical to the default value. After calling v4l2_ctrl_handler_setup() this
value is passed to the hardware. It is generally a good idea to call this
function.

除非控件标记为 `volatile`，否则 `p_cur` 字段指向当前缓存的控件值。当创建新控件时，此值与默认值相同。在调用 `v4l2_ctrl_handler_setup()` 之后，此值传递给硬件。通常最好调用此函数。

Whenever a new value is set that new value is automatically cached. This means
that most drivers do not need to implement the g_volatile_ctrl() op. The
exception is for controls that return a volatile register such as a signal
strength read-out that changes continuously. In that case you will need to
implement g_volatile_ctrl like this:

每当设置新值时，新值将自动缓存。这意味着大多数驱动程序不需要实现 `g_volatile_ctrl()` 操作。例外情况是对于返回连续更改的易失性寄存器（例如信号强度读数）的控件。在这种情况下，您需要像这样实现 `g_volatile_ctrl`：

```c
	static int foo_g_volatile_ctrl(struct v4l2_ctrl *ctrl)
	{
		switch (ctrl->id) {
		case V4L2_CID_BRIGHTNESS:
			ctrl->val = read_reg(0x123);
			break;
		}
	}
```

Note that you use the 'new value' union as well in g_volatile_ctrl. In general
controls that need to implement g_volatile_ctrl are read-only controls. If they
are not, a V4L2_EVENT_CTRL_CH_VALUE will not be generated when the control
changes.

请注意，在 `g_volatile_ctrl` 中也要使用 'new value' 联合体。通常，需要实现 `g_volatile_ctrl` 的控件是只读控件。如果它们不是，当控件更改时将不会生成 `V4L2_EVENT_CTRL_CH_VALUE` 事件。

To mark a control as volatile you have to set V4L2_CTRL_FLAG_VOLATILE:

要将控件标记为易失性，必须设置 `V4L2_CTRL_FLAG_VOLATILE`：

```c
	ctrl = v4l2_ctrl_new_std(&sd->ctrl_handler, ...);
	if (ctrl)
		ctrl->flags |= V4L2_CTRL_FLAG_VOLATILE;
```

For try/s_ctrl the new values (i.e. as passed by the user) are filled in and
you can modify them in try_ctrl or set them in s_ctrl. The 'cur' union
contains the current value, which you can use (but not change!) as well.

对于 try/s_ctrl，新值（即由用户传递的值）将被填充，您可以在 `try_ctrl` 中修改它们或在 `s_ctrl` 中设置它们。'cur' 联合体包含当前值，您也可以使用（但不能更改）它。

If s_ctrl returns 0 (OK), then the control framework will copy the new final
values to the 'cur' union.

如果 `s_ctrl` 返回 0（OK），则控件框架将将新的最终值复制到 'cur' 联合体中。

While in g_volatile/s/try_ctrl you can access the value of all controls owned
by the same handler since the handler's lock is held. If you need to access
the value of controls owned by other handlers, then you have to be very careful
not to introduce deadlocks.

在 `g_volatile/s/try_ctrl` 中，您可以访问同一处理程序拥有的所有控件的值，因为处理程序的锁已被保持。如果需要访问其他处理程序拥有的控件的值，则必须非常小心，以防止引入死锁。

Outside of the control ops you have to go through to helper functions to get
or set a single control value safely in your driver:

在控件操作之外，您必须通过辅助函数在驱动程序中安全地获取或设置单个控件值：

```c
	s32 v4l2_ctrl_g_ctrl(struct v4l2_ctrl *ctrl);
	int v4l2_ctrl_s_ctrl(struct v4l2_ctrl *ctrl, s32 val);
```

These functions go through the control framework just as VIDIOC_G/S_CTRL ioctls
do. Don't use these inside the control ops g_volatile/s/try_ctrl, though, that
will result in a deadlock since these helpers lock the handler as well.

这些函数通过控件框架执行，就像 `VIDIOC_G/S_CTRL` ioctls 一样。但是，请勿在控件操作 `g_volatile/s/try_ctrl` 中使用它们，这将导致死锁，因为这些帮助程序也会锁定处理程序。

You can also take the handler lock yourself:

您还可以自己获取处理程序锁：

```c
	mutex_lock(&state->ctrl_handler.lock);
	pr_info("String value is '%s'\n", ctrl1->p_cur.p_char);
	pr_info("Integer value is '%s'\n", ctrl2->cur.val);
	mutex_unlock(&state->ctrl_handler.lock);
```

## 2.15.6 Menu Controls

The v4l2_ctrl struct contains this union:

`v4l2_ctrl` 结构包含以下联合体：

```c
	union {
		u32 step;
		u32 menu_skip_mask;
	};
```

For menu controls menu_skip_mask is used. What it does is that it allows you
to easily exclude certain menu items. This is used in the VIDIOC_QUERYMENU
implementation where you can return -EINVAL if a certain menu item is not
present. Note that VIDIOC_QUERYCTRL always returns a step value of 1 for
menu controls.

对于菜单控件，使用 `menu_skip_mask`。其作用是允许您轻松排除特定的菜单项。这在 `VIDIOC_QUERYMENU` 实现中使用，您可以在某个菜单项不存在时返回 -EINVAL。请注意，对于菜单控件，`VIDIOC_QUERYCTRL` 总是返回步长值 1。

A good example is the MPEG Audio Layer II Bitrate menu control where the
menu is a list of standardized possible bitrates. But in practice hardware
implementations will only support a subset of those. By setting the skip
mask you can tell the framework which menu items should be skipped. Setting
it to 0 means that all menu items are supported.

一个很好的例子是 MPEG 音频层 II 比特率菜单控件，其中菜单是标准化的可能比特率的列表。但实际上，硬件实现只会支持其中的一部分。通过设置跳过掩码，您可以告诉框架应该跳过哪些菜单项。将其设置为 0 意味着支持所有菜单项。

You set this mask either through the v4l2_ctrl_config struct for a custom
control, or by calling v4l2_ctrl_new_std_menu().

您可以通过 `v4l2_ctrl_config` 结构为自定义控件设置此掩码，或通过调用 `v4l2_ctrl_new_std_menu()` 设置。


## 2.15.7 Custom Controls

Driver specific controls can be created using v4l2_ctrl_new_custom():

可以使用 `v4l2_ctrl_new_custom()` 创建特定于驱动程序的控件：

```c
	static const struct v4l2_ctrl_config ctrl_filter = {
		.ops = &ctrl_custom_ops,
		.id = V4L2_CID_MPEG_CX2341X_VIDEO_SPATIAL_FILTER,
		.name = "Spatial Filter",
		.type = V4L2_CTRL_TYPE_INTEGER,
		.flags = V4L2_CTRL_FLAG_SLIDER,
		.max = 15,
		.step = 1,
	};

	ctrl = v4l2_ctrl_new_custom(&foo->ctrl_handler, &ctrl_filter, NULL);
```

The last argument is the priv pointer which can be set to driver-specific
private data.

最后一个参数是 `priv` 指针，可以设置为特定于驱动程序的私有数据。

The v4l2_ctrl_config struct also has a field to set the is_private flag.

`v4l2_ctrl_config` 结构还有一个字段用于设置 `is_private` 标志。

If the name field is not set, then the framework will assume this is a standard
control and will fill in the name, type and flags fields accordingly.

如果未设置 `name` 字段，则框架将假定这是一个标准控件，并相应地填充 `name`、`type` 和 `flags` 字段。


## 2.15.8 Active and Grabbed Controls

If you get more complex relationships between controls, then you may have to
activate and deactivate controls. For example, if the Chroma AGC control is
on, then the Chroma Gain control is inactive. That is, you may set it, but
the value will not be used by the hardware as long as the automatic gain
control is on. Typically user interfaces can disable such input fields.

如果在控件之间存在更复杂的关系，那么您可能需要激活和停用控件。例如，如果色度AGC控制打开，则色度增益控制无效。也就是说，您可以设置它，但只要自动增益控制打开，硬件就不会使用该值。通常，用户界面可以禁用这些输入字段。

You can set the 'active' status using v4l2_ctrl_activate(). By default all
controls are active. Note that the framework does not check for this flag.
It is meant purely for GUIs. The function is typically called from within
s_ctrl.

您可以使用 `v4l2_ctrl_activate()` 设置 'active' 状态。默认情况下，所有控件都是活动的。请注意，框架不会检查此标志。它纯粹是为了GUI。通常从 `s_ctrl` 中调用此函数。

The other flag is the 'grabbed' flag. A grabbed control means that you cannot
change it because it is in use by some resource. Typical examples are MPEG
bitrate controls that cannot be changed while capturing is in progress.

另一个标志是 'grabbed' 标志。被占用的控件意味着您不能更改它，因为它正在某些资源中使用。典型的例子是在捕获正在进行时不能更改的 MPEG 比特率控制。

If a control is set to 'grabbed' using v4l2_ctrl_grab(), then the framework
will return -EBUSY if an attempt is made to set this control. The
v4l2_ctrl_grab() function is typically called from the driver when it
starts or stops streaming.

如果使用 `v4l2_ctrl_grab()` 将控件设置为 'grabbed'，则如果尝试设置此控件，框架将返回 -EBUSY。通常，在驱动程序启动或停止流时，会从驱动程序中调用 `v4l2_ctrl_grab()` 函数。

## 2.15.9 Control Clusters

By default all controls are independent from the others. But in more
complex scenarios you can get dependencies from one control to another.
In that case you need to 'cluster' them:

默认情况下，所有控件都是独立的。但在更复杂的场景中，您可能会从一个控件到另一个控件得到依赖关系。在这种情况下，您需要将它们“集群”起来：

```c
	struct foo {
		struct v4l2_ctrl_handler ctrl_handler;
	#define AUDIO_CL_VOLUME (0)
	#define AUDIO_CL_MUTE   (1)
		struct v4l2_ctrl *audio_cluster[2];
		...
	};

	state->audio_cluster[AUDIO_CL_VOLUME] =
		v4l2_ctrl_new_std(&state->ctrl_handler, ...);
	state->audio_cluster[AUDIO_CL_MUTE] =
		v4l2_ctrl_new_std(&state->ctrl_handler, ...);
	v4l2_ctrl_cluster(ARRAY_SIZE(state->audio_cluster), state->audio_cluster);
```

From now on whenever one or more of the controls belonging to the same
cluster is set (or 'gotten', or 'tried'), only the control ops of the first
control ('volume' in this example) is called. You effectively create a new
composite control. Similar to how a 'struct' works in C.

从现在开始，只要设置（或'获取'或'尝试'）同一集群中的一个或多个控件，只会调用第一个控件的控件操作（在此示例中为 'volume'）。实际上，您创建了一个新的复合控件。类似于在 C 中 'struct' 的工作方式。

So when s_ctrl is called with V4L2_CID_AUDIO_VOLUME as argument, you should set
all two controls belonging to the audio_cluster:

因此，当使用 V4L2_CID_AUDIO_VOLUME 作为参数调用 s_ctrl 时，您应该设置属于 audio_cluster 的所有两个控件：

```c
	static int foo_s_ctrl(struct v4l2_ctrl *ctrl)
	{
		struct foo *state = container_of(ctrl->handler, struct foo, ctrl_handler);

		switch (ctrl->id) {
		case V4L2_CID_AUDIO_VOLUME: {
			struct v4l2_ctrl *mute = ctrl->cluster[AUDIO_CL_MUTE];

			write_reg(0x123, mute->val ? 0 : ctrl->val);
			break;
		}
		case V4L2_CID_CONTRAST:
			write_reg(0x456, ctrl->val);
			break;
		}
		return 0;
	}
```

In the example above the following are equivalent for the VOLUME case:

在上面的示例中，对于 VOLUME 情况，以下是等效的：

```c
	ctrl == ctrl->cluster[AUDIO_CL_VOLUME] == state->audio_cluster[AUDIO_CL_VOLUME]
	ctrl->cluster[AUDIO_CL_MUTE] == state->audio_cluster[AUDIO_CL_MUTE]
```

In practice using cluster arrays like this becomes very tiresome. So instead
the following equivalent method is used:

在实践中，像这样使用集群数组变得非常繁琐。因此，使用以下等效方法：

```c
	struct {
		/* audio cluster */
		struct v4l2_ctrl *volume;
		struct v4l2_ctrl *mute;
	};
```

The anonymous struct is used to clearly 'cluster' these two control pointers,
but it serves no other purpose. The effect is the same as creating an
array with two control pointers. So you can just do:

匿名结构用于清晰地“集群”这两个控件指针，但它没有其他用途。效果与创建带有两个控件指针的数组相同。因此，您可以直接执行以下操作：

```c
	state->volume = v4l2_ctrl_new_std(&state->ctrl_handler, ...);
	state->mute = v4l2_ctrl_new_std(&state->ctrl_handler, ...);
	v4l2_ctrl_cluster(2, &state->volume);
```

And in foo_s_ctrl you can use these pointers directly: state->mute->val.

在 foo_s_ctrl 中，您可以直接使用这些指针：state->mute->val。

Note that controls in a cluster may be NULL. For example, if for some
reason mute was never added (because the hardware doesn't support that
particular feature), then mute will be NULL. So in that case we have a
cluster of 2 controls, of which only 1 is actually instantiated. The
only restriction is that the first control of the cluster must always be
present, since that is the 'master' control of the cluster. The master
control is the one that identifies the cluster and that provides the
pointer to the v4l2_ctrl_ops struct that is used for that cluster.

请注意，集群中的控件可能为 NULL。例如，如果由于某种原因未添加静音（因为硬件不支持该特定功能），则静音将为 NULL。因此，在这种情况下，我们有一个包含 2 个控件的集群，其中只有 1 个实际实例化。唯一的限制是集群的第一个控件必须始终存在，因为它是集群的“主”控件。主控件是标识集群并提供用于该集群的 v4l2_ctrl_ops 结构指针的控件。

Obviously, all controls in the cluster array must be initialized to either
a valid control or to NULL.

显然，集群数组中的所有控件都必须初始化为有效的控件或为 NULL。

In rare cases you might want to know which controls of a cluster actually
were set explicitly by the user. For this you can check the 'is_new' flag of
each control. For example, in the case of a volume/mute cluster the 'is_new'
flag of the mute control would be set if the user called VIDIOC_S_CTRL for
mute only. If the user would call VIDIOC_S_EXT_CTRLS for both mute and volume
controls, then the 'is_new' flag would be 1 for both controls.

在罕见的情况下，您可能希望知道集群中的哪些控件实际上是由用户显式设置的。为此，您可以检查每个控件的 'is_new' 标志。例如，在音量/静音集群的情况下，如果用户仅为静音调用 VIDIOC_S_CTRL，则静音控件的 'is_new' 标志将被设置。如果用户为静音和音量

The 'is_new' flag is always 1 when called from v4l2_ctrl_handler_setup().

控件同时调用 VIDIOC_S_EXT_CTRLS，则两个控件的 'is_new' 标志都将为 1。


## 2.15.10 Handling autogain/gain-type Controls with Auto Clusters

A common type of control cluster is one that handles 'auto-foo/foo'-type
controls. Typical examples are autogain/gain, autoexposure/exposure,
autowhitebalance/red balance/blue balance. In all cases you have one control
that determines whether another control is handled automatically by the hardware,
or whether it is under manual control from the user.

一种常见的控件集群类型是处理 'auto-foo/foo' 类型控件。典型的例子包括自动增益/增益、自动曝光/曝光、自动白平衡/红平衡/蓝平衡。在所有情况下，您有一个控件确定另一个控件是由硬件自动处理，还是由用户手动控制。

If the cluster is in automatic mode, then the manual controls should be
marked inactive and volatile. When the volatile controls are read the
g_volatile_ctrl operation should return the value that the hardware's automatic
mode set up automatically.

如果集群处于自动模式，则手动控件应标记为非活动且易失效。当读取易失性控件时，`g_volatile_ctrl` 操作应返回硬件的自动模式自动设置的值。

If the cluster is put in manual mode, then the manual controls should become
active again and the volatile flag is cleared (so g_volatile_ctrl is no longer
called while in manual mode). In addition just before switching to manual mode
the current values as determined by the auto mode are copied as the new manual
values.

如果集群处于手动模式，则手动控件应再次变为活动状态，并且清除易失性标志（因此在手动模式下不再调用 `g_volatile_ctrl`）。此外，在切换到手动模式之前，由自动模式确定的当前值将被复制为新的手动值。

Finally the V4L2_CTRL_FLAG_UPDATE should be set for the auto control since
changing that control affects the control flags of the manual controls.

最后，对于自动控件，应设置 `V4L2_CTRL_FLAG_UPDATE`，因为更改该控件会影响手动控件的控制标志。

In order to simplify this a special variation of v4l2_ctrl_cluster was
introduced:

为了简化这一点，引入了 v4l2_ctrl_cluster 的一种特殊变体：

```c
	void v4l2_ctrl_auto_cluster(unsigned ncontrols, struct v4l2_ctrl **controls,
				    u8 manual_val, bool set_volatile);
```

The first two arguments are identical to v4l2_ctrl_cluster. The third argument
tells the framework which value switches the cluster into manual mode. The
last argument will optionally set V4L2_CTRL_FLAG_VOLATILE for the non-auto controls.
If it is false, then the manual controls are never volatile. You would typically
use that if the hardware does not give you the option to read back to values as
determined by the auto mode (e.g. if autogain is on, the hardware doesn't allow
you to obtain the current gain value).

前两个参数与 `v4l2_ctrl_cluster` 相同。第三个参数告诉框架哪个值将集群切换到手动模式。最后一个参数将可选地为非自动控件设置 `V4L2_CTRL_FLAG_VOLATILE`。如果为 false，则手动控件永远不是易失性的。如果硬件不允许您按照自动模式确定的方式读回值（例如，如果自动增益打开，则硬件不允许您获取当前增益值），则通常会使用该选项。

The first control of the cluster is assumed to be the 'auto' control.

集群的第一个控件被假定为 'auto' 控件。

Using this function will ensure that you don't need to handle all the complex
flag and volatile handling.

使用此函数将确保您不需要处理所有复杂的标志和易失性处理。

## 2.15.11 VIDIOC_LOG_STATUS Support

This ioctl allow you to dump the current status of a driver to the kernel log.
The v4l2_ctrl_handler_log_status(ctrl_handler, prefix) can be used to dump the
value of the controls owned by the given handler to the log. You can supply a
prefix as well. If the prefix didn't end with a space, then ': ' will be added
for you.

此 ioctl 允许您将驱动程序的当前状态转储到内核日志中。`v4l2_ctrl_handler_log_status(ctrl_handler, prefix)` 可以用于将给定处理程序拥有的控件的值转储到日志中。您还可以提供一个前缀。如果前缀不以空格结尾，那么将为您添加 ': '。


## 2.15.12 Different Handlers for Different Video Nodes

Usually the V4L2 driver has just one control handler that is global for
all video nodes. But you can also specify different control handlers for
different video nodes. You can do that by manually setting the ctrl_handler
field of struct video_device.

通常，V4L2 驱动程序只有一个控件处理程序，对所有视频节点都是全局的。但是您也可以为不同的视频节点指定不同的控件处理程序。您可以通过手动设置 struct video_device 的 ctrl_handler 字段来实现这一点。

That is no problem if there are no subdevs involved but if there are, then
you need to block the automatic merging of subdev controls to the global
control handler. You do that by simply setting the ctrl_handler field in
struct v4l2_device to NULL. Now v4l2_device_register_subdev() will no longer
merge subdev controls.

如果没有涉及到子设备，这不是问题，但如果涉及到子设备，那么您需要阻止子设备控件自动合并到全局控件处理程序。您可以通过在 struct v4l2_device 中的 ctrl_handler 字段设置为 NULL 来实现。现在，v4l2_device_register_subdev() 将不再合并子设备控件。

After each subdev was added, you will then have to call v4l2_ctrl_add_handler
manually to add the subdev's control handler (sd->ctrl_handler) to the desired
control handler. This control handler may be specific to the video_device or
for a subset of video_device's. For example: the radio device nodes only have
audio controls, while the video and vbi device nodes share the same control
handler for the audio and video controls.

在添加每个子设备之后，您将需要手动调用 v4l2_ctrl_add_handler 来将子设备的控制处理程序（sd->ctrl_handler）添加到所需的控制处理程序中。该控制处理程序可能特定于 video_device 或 video_device 的子集。例如：广播设备节点仅具有音频控件，而视频和 vbi 设备节点共享用于音频和视频控件的相同控件处理程序。

If you want to have one handler (e.g. for a radio device node) have a subset
of another handler (e.g. for a video device node), then you should first add
the controls to the first handler, add the other controls to the second
handler and finally add the first handler to the second. For example:

如果要使一个处理程序（例如广播设备节点）具有另一个处理程序（例如视频设备节点）的子集，那么您应该首先将控件添加到第一个处理程序，将其他控件添加到第二个处理程序，最后将第一个处理程序添加到第二个处理程序。例如：

```c
	v4l2_ctrl_new_std(&radio_ctrl_handler, &radio_ops, V4L2_CID_AUDIO_VOLUME, ...);
	v4l2_ctrl_new_std(&radio_ctrl_handler, &radio_ops, V4L2_CID_AUDIO_MUTE, ...);
	v4l2_ctrl_new_std(&video_ctrl_handler, &video_ops, V4L2_CID_BRIGHTNESS, ...);
	v4l2_ctrl_new_std(&video_ctrl_handler, &video_ops, V4L2_CID_CONTRAST, ...);
	v4l2_ctrl_add_handler(&video_ctrl_handler, &radio_ctrl_handler, NULL);
```

The last argument to v4l2_ctrl_add_handler() is a filter function that allows
you to filter which controls will be added. Set it to NULL if you want to add
all controls.

v4l2_ctrl_add_handler() 的最后一个参数是一个过滤函数，允许您过滤要添加的控件。如果要添加所有控件，将其设置为 NULL。

Or you can add specific controls to a handler:

或者您可以向处理程序添加特定的控件：

```c
	volume = v4l2_ctrl_new_std(&video_ctrl_handler, &ops, V4L2_CID_AUDIO_VOLUME, ...);
	v4l2_ctrl_new_std(&video_ctrl_handler, &ops, V4L2_CID_BRIGHTNESS, ...);
	v4l2_ctrl_new_std(&video_ctrl_handler, &ops, V4L2_CID_CONTRAST, ...);
```

What you should not do is make two identical controls for two handlers.
For example:

您不应该做的是为两个处理程序创建两个相同的控件。例如：

```c
	v4l2_ctrl_new_std(&radio_ctrl_handler, &radio_ops, V4L2_CID_AUDIO_MUTE, ...);
	v4l2_ctrl_new_std(&video_ctrl_handler, &video_ops, V4L2_CID_AUDIO_MUTE, ...);
```

This would be bad since muting the radio would not change the video mute
control. The rule is to have one control for each hardware 'knob' that you
can twiddle.

这是不好的，因为静音收音机不会更改视频静音控制。规则是为您可以调整的每个硬件 '旋钮' 拥有一个控件。

## 2.15.13 Finding Controls

Normally you have created the controls yourself and you can store the struct
v4l2_ctrl pointer into your own struct.

通常，您自己创建控件，并且可以将 struct v4l2_ctrl 指针存储到您自己的结构中。

But sometimes you need to find a control from another handler that you do
not own. For example, if you have to find a volume control from a subdev.

但有时您需要从您不拥有的另一个处理程序中查找控件，例如，如果您需要从子设备中查找音量控件。

You can do that by calling v4l2_ctrl_find:

您可以通过调用 v4l2_ctrl_find 来实现：

```c
	struct v4l2_ctrl *volume;

	volume = v4l2_ctrl_find(sd->ctrl_handler, V4L2_CID_AUDIO_VOLUME);
```

Since v4l2_ctrl_find will lock the handler you have to be careful where you
use it. For example, this is not a good idea:

由于 v4l2_ctrl_find 会锁定处理程序，因此您必须小心在何处使用它。例如，以下不是一个好主意：

```c
	struct v4l2_ctrl_handler ctrl_handler;

	v4l2_ctrl_new_std(&ctrl_handler, &video_ops, V4L2_CID_BRIGHTNESS, ...);
	v4l2_ctrl_new_std(&ctrl_handler, &video_ops, V4L2_CID_CONTRAST, ...);
```

...and in video_ops.s_ctrl:

...在 video_ops.s_ctrl 中：

```c
	case V4L2_CID_BRIGHTNESS:
		contrast = v4l2_find_ctrl(&ctrl_handler, V4L2_CID_CONTRAST);
		...
```

When s_ctrl is called by the framework the ctrl_handler.lock is already taken, so
attempting to find another control from the same handler will deadlock.

当框架调用 s_ctrl 时，ctrl_handler.lock 已被获取，因此尝试从相同处理程序中查找另一个控件将导致死锁。

It is recommended not to use this function from inside the control ops.

建议不要在控制操作中使用此函数。

## 2.15.14 Preventing Controls inheritance

When one control handler is added to another using v4l2_ctrl_add_handler, then
by default all controls from one are merged to the other. But a subdev might
have low-level controls that make sense for some advanced embedded system, but
not when it is used in consumer-level hardware. In that case you want to keep
those low-level controls local to the subdev. You can do this by simply
setting the 'is_private' flag of the control to 1:

当使用v4l2_ctrl_add_handler将一个控件处理程序添加到另一个控件处理程序时，默认情况下，一个的所有控件会合并到另一个中。但是一个子设备可能具有在一些先进的嵌入式系统中有意义的低级控件，但在用于消费级硬件时则没有意义。在这种情况下，您希望将这些低级控件保留在子设备本地。您可以通过简单地将控件的'is_private'标志设置为1来实现此目的：

```c
	static const struct v4l2_ctrl_config ctrl_private = {
		.ops = &ctrl_custom_ops,
		.id = V4L2_CID_...,
		.name = "Some Private Control",
		.type = V4L2_CTRL_TYPE_INTEGER,
		.max = 15,
		.step = 1,
		.is_private = 1,
	};

	ctrl = v4l2_ctrl_new_custom(&foo->ctrl_handler, &ctrl_private, NULL);
```

These controls will now be skipped when v4l2_ctrl_add_handler is called.

当调用v4l2_ctrl_add_handler时，这些控件现在将被跳过。

## 2.15.15 V4L2_CTRL_TYPE_CTRL_CLASS Controls

Controls of this type can be used by GUIs to get the name of the control class.
A fully featured GUI can make a dialog with multiple tabs with each tab
containing the controls belonging to a particular control class. The name of
each tab can be found by querying a special control with ID <control class | 1>.

此类型的控件可用于供图形用户界面(GUI)使用，以获取控件类别的名称。一个功能完备的GUI可以创建一个对话框，其中包含多个选项卡，每个选项卡包含属于特定控件类别的控件。每个选项卡的名称可以通过查询具有 ID `<control class | 1>` 的特殊控件来获得。

Drivers do not have to care about this. The framework will automatically add
a control of this type whenever the first control belonging to a new control
class is added.

驱动程序不需要关心这一点。每当添加属于新控件类别的第一个控件时，框架将自动添加一个此类型的控件。

## 2.15.16 Adding Notify Callbacks

Sometimes the platform or bridge driver needs to be notified when a control
from a sub-device driver changes. You can set a notify callback by calling
this function:

有时，平台或桥接驱动程序需要在子设备驱动程序的控件更改时得到通知。您可以通过调用此函数来设置通知回调：

```c
	void v4l2_ctrl_notify(struct v4l2_ctrl *ctrl,
		void (*notify)(struct v4l2_ctrl *ctrl, void *priv), void *priv);
```

Whenever the give control changes value the notify callback will be called
with a pointer to the control and the priv pointer that was passed with
v4l2_ctrl_notify. Note that the control's handler lock is held when the
notify function is called.

每当给定控件更改值时，将调用通知回调，其中包含指向控件的指针和通过 `v4l2_ctrl_notify` 传递的 priv 指针。请注意，在调用通知函数时，保持控制处理程序锁。

There can be only one notify function per control handler. Any attempt
to set another notify function will cause a WARN_ON.

每个控制处理程序只能有一个通知函数。任何尝试设置另一个通知函数都将导致 `WARN_ON`。

## 2.15.17 v4l2_ctrl functions and data structures

	include/media/v4l2-ctrls.h

# 2.16 V4L2 videobuf2 functions and data structures

	include/media/videobuf2-core.h
	include/media/videobuf2-v4l2.h
	include/media/videobuf2-memops.h

# 2.17 V4L2 DV Timings functions

	include/media/v4l2-dv-timings.h

# 2.18 V4L2 Media Controller functions and data structures

	include/media/v4l2-mc.h

# 2.19 V4L2 Media Controller functions and data structures

	include/media/v4l2-mc.h

# 2.20 V4L2 Media Bus functions and data structures

	include/media/v4l2-mediabus.h

# 2.21 V4L2 Memory to Memory functions and data structures

	include/media/v4l2-mem2mem.h

# 2.22 V4L2 async kAPI

	include/media/v4l2-async.h

# 2.23 V4L2 fwnode kAPI

	include/media/v4l2-fwnode.h

# 2.24 V4L2 CCI kAPI

	include/media/v4l2-cci.h

# 2.25 V4L2 rect helper functions

	include/media/v4l2-rect.h

# 2.26 Tuner functions and data structures

	include/media/tuner.h
	include/media/tuner-types.h

# 2.27 V4L2 common functions and data structures

	include/media/v4l2-common.h
	include/media/v4l2-ioctl.h

# 2.28 Hauppauge TV EEPROM functions and data structures

	include/media/tveeprom.h