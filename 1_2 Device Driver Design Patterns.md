Device Driver Design Patterns
=============================

This document describes a few common design patterns found in device drivers.
It is likely that subsystem maintainers will ask driver developers to
conform to these design patterns.

这份文档描述了设备驱动程序中常见的几种设计模式。子系统的维护者可能会要求驱动程序开发人员遵循这些设计模式。

1. State Container
2. container_of()

**1. State Container**

While the kernel contains a few device drivers that assume that they will
only be probed() once on a certain system (singletons), it is custom to assume
that the device the driver binds to will appear in several instances. This
means that the probe() function and all callbacks need to be reentrant.

The most common way to achieve this is to use the state container design
pattern. It usually has this form

尽管内核中包含一些设备驱动程序，假设它们在某个系统上只会被探测（probed）一次（单例模式），但通常假设驱动程序绑定的设备会以多个实例出现。这意味着`probe()`函数和所有回调函数都需要是可重入的。

实现这一点最常见的方式是使用状态容器设计模式。它通常具有以下形式：

```c

  struct foo {
      spinlock_t lock; /* Example member */
      (...)
  };

  static int foo_probe(...)
  {
      struct foo *foo;

      foo = devm_kzalloc(dev, sizeof(*foo), GFP_KERNEL);
      if (!foo)
          return -ENOMEM;
      spin_lock_init(&foo->lock);
      (...)
  }
```

This will create an instance of struct foo in memory every time probe() is
called. This is our state container for this instance of the device driver.
Of course it is then necessary to always pass this instance of the
state around to all functions that need access to the state and its members.

For example, if the driver is registering an interrupt handler, you would
pass around a pointer to struct foo like this

每次调用`probe()`时，这将在内存中创建`struct foo`的一个实例。这是该设备驱动程序实例的状态容器。当然，随后需要始终将该状态的实例传递给所有需要访问该状态及其成员的函数。

例如，如果驱动程序正在注册中断处理程序，您将以以下方式传递指向`struct foo`的指针：

```c
  static irqreturn_t foo_handler(int irq, void *arg)
  {
      struct foo *foo = arg;
      (...)
  }

  static int foo_probe(...)
  {
      struct foo *foo;

      (...)
      ret = request_irq(irq, foo_handler, 0, "foo", foo);
  }
```

This way you always get a pointer back to the correct instance of foo in
your interrupt handler.

这样，您始终可以在中断处理程序中获得指向正确`foo`实例的指针。

**2. container_of()**

Continuing on the above example we add an offloaded work

在上面的例子中，我们添加一个外部的工作：

```c
  struct foo {
      spinlock_t lock;
      struct workqueue_struct *wq;
      struct work_struct offload;
      (...)
  };

  static void foo_work(struct work_struct *work)
  {
      struct foo *foo = container_of(work, struct foo, offload);

      (...)
  }

  static irqreturn_t foo_handler(int irq, void *arg)
  {
      struct foo *foo = arg;

      queue_work(foo->wq, &foo->offload);
      (...)
  }

  static int foo_probe(...)
  {
      struct foo *foo;

      foo->wq = create_singlethread_workqueue("foo-wq");
      INIT_WORK(&foo->offload, foo_work);
      (...)
  }
```

The design pattern is the same for an hrtimer or something similar that will
return a single argument which is a pointer to a struct member in the callback.

对于将返回一个指向回调中`struct`成员的指针的 hrtimer 或类似物，设计模式是相同的。


`container_of()` is a macro defined in <linux/kernel.h>

`container_of()` 是在 `<linux/kernel.h>` 中定义的一个宏。

What `container_of()` does is to obtain a pointer to the containing struct from
a pointer to a member by a simple subtraction using the offsetof() macro from
standard C, which allows something similar to object oriented behaviours.
Notice that the contained member must not be a pointer, but an actual member
for this to work.

`container_of()` 所做的是通过使用标准 C 中的 `offsetof()` 宏进行简单的减法，从一个成员指针获取到包含结构体的指针，这允许类似面向对象的行为。请注意，被包含的成员不能是指针，而必须是一个实际的成员，才能使此功能正常工作。

We can see here that we avoid having global pointers to our struct foo *
instance this way, while still keeping the number of parameters passed to the
work function to a single pointer.

在这里，我们避免以这种方式拥有对`struct foo *`实例的全局指针，同时仍将传递给工作函数的参数数量保持为单一指针。
