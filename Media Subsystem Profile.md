Media Subsystem Profile
=======================

Overview
--------

The media subsystem covers support for a variety of devices: stream
capture, analog and digital TV streams, cameras, remote controllers, HDMI CEC
and media pipeline control.

媒体子系统涵盖对各种设备的支持：流捕获、模拟和数字电视流、摄像头、遥控器、HDMI CEC 和媒体管道控制。

It covers, mainly, the contents of those directories:

主要包括以下目录的内容：

  - drivers/media
  - drivers/staging/media
  - Documentation/admin-guide/media
  - Documentation/driver-api/media
  - Documentation/userspace-api/media
  - Documentation/devicetree/bindings/media/\ [1]_
  - include/media

[1] Device tree bindings are maintained by the
       OPEN FIRMWARE AND FLATTENED DEVICE TREE BINDINGS maintainers
       (see the MAINTAINERS file). So, changes there must be reviewed
       by them before being merged via the media subsystem's development
       tree.

设备树绑定由OPEN FIRMWARE AND FLATTENED DEVICE TREE BINDINGS维护者负责（请参阅 MAINTAINERS 文件）。因此，在通过媒体子系统的development tree合并之前，那里的更改必须由他们审核。

Both media userspace and Kernel APIs are documented and the documentation
must be kept in sync with the API changes. It means that all patches that
add new features to the subsystem must also bring changes to the
corresponding API files.

媒体用户空间和内核 API 都有文档，文档必须与 API 更改保持同步。这意味着向子系统添加新功能的所有补丁也必须对应 API 文件进行更改。

Due to the size and wide scope of the media subsystem, media's
maintainership model is to have sub-maintainers that have a broad
knowledge of a specific aspect of the subsystem. It is the sub-maintainers'
task to review the patches, providing feedback to users if the patches are
following the subsystem rules and are properly using the media kernel and
userspace APIs.

由于媒体子系统的规模庞大且范围广泛，Media的维护模型是设立子维护者，他们对子系统的特定方面有广泛的了解。子维护者的任务是审查补丁，向用户提供反馈，确保补丁遵循子系统规则，并正确使用媒体内核和用户空间 API。

Patches for the media subsystem must be sent to the media mailing list
at linux-media@vger.kernel.org as plain text only e-mail. Emails with
HTML will be automatically rejected by the mail server. It could be wise
to also copy the sub-maintainer(s).

媒体子系统的补丁必须以纯文本的形式发送到 linux-media@vger.kernel.org 邮件列表。带有 HTML 的电子邮件将被邮件服务器自动拒绝。复制给子维护者也可能是一个明智的选择。

Media's workflow is heavily based on Patchwork, meaning that, once a patch
is submitted, the e-mail will first be accepted by the mailing list
server, and, after a while, it should appear at:

Media的工作流程主要基于 Patchwork，也就是说，一旦提交了一个补丁，电子邮件将首先被邮件列表服务器接受，一段时间后，它应该会出现在：

   - https://patchwork.linuxtv.org/project/linux-media/list/

If it doesn't automatically appear there after a few minutes, then
probably something went wrong on your submission. Please check if the
email is in plain text\ [2]_ only and if your emailer is not mangling
whitespaces before complaining or submitting them again.

如果在几分钟内它没有自动出现，那么可能是你提交的时候出了问题。在抱怨或重新提交之前，请检查电子邮件是否只是纯文本，以及你的电子邮件程序是否没有损坏空格。

You can check if the mailing list server accepted your patch, by looking at:

你可以通过查看以下链接来检查邮件列表服务器是否接受了你的补丁：

   - https://lore.kernel.org/linux-media/

.. [2] If your email contains HTML, the mailing list server will simply
       drop it, without any further notice.

如果你的电子邮件包含 HTML，邮件列表服务器将会直接丢弃它，而不会有进一步的通知。


Media maintainers
-----------------

At the media subsystem, we have a group of senior developers that
are responsible for doing the code reviews at the drivers (also known as
sub-maintainers), and another senior developer responsible for the
subsystem as a whole. For core changes, whenever possible, multiple
media maintainers do the review.

在媒体子系统中，我们有一组负责在驱动程序中进行代码审查的资深开发人员（也称为子维护者），以及负责整个子系统的另一位资深开发人员。对于核心更改，如果可能的话，多个媒体维护者会进行审查。

The media maintainers that work on specific areas of the subsystem are:

负责子系统特定区域的媒体维护者包括：

- Remote Controllers (infrared):
    Sean Young <sean@mess.org>

- HDMI CEC:
    Hans Verkuil <hverkuil@xs4all.nl>

- Media controller drivers:
    Laurent Pinchart <laurent.pinchart@ideasonboard.com>

- ISP, v4l2-async, v4l2-fwnode, v4l2-flash-led-class and Sensor drivers:
    Sakari Ailus <sakari.ailus@linux.intel.com>

- V4L2 drivers and core V4L2 frameworks:
    Hans Verkuil <hverkuil@xs4all.nl>

The subsystem maintainer is:

子系统维护者是：

- Mauro Carvalho Chehab <mchehab@kernel.org>

Media maintainers may delegate a patch to other media maintainers as needed.
On such case, checkpatch's ``delegate`` field indicates who's currently
responsible for reviewing a patch.

媒体维护者可以根据需要将补丁委派给其他媒体维护者。在这种情况下，checkpatch 的 "delegate" 字段会指示当前负责审查补丁的人。

Submit Checklist Addendum
-------------------------

Patches that change the Open Firmware/Device Tree bindings must be
reviewed by the Device Tree maintainers. So, DT maintainers should be
Cc:ed when those are submitted via devicetree@vger.kernel.org mailing
list.

更改 Open Firmware/Device Tree 绑定的补丁必须由设备树维护者审查。因此，在通过 devicetree@vger.kernel.org 邮件列表提交这些补丁时，应将设备树维护者抄送给他们。

There is a set of compliance tools at https://git.linuxtv.org/v4l-utils.git/
that should be used in order to check if the drivers are properly
implementing the media APIs:

在 https://git.linuxtv.org/v4l-utils.git/ 中有一组遵守规范的工具，用于检查驱动程序是否正确实现了媒体 API：

| Type | Tool |
| ---- | ---- |
| V4L2 drivers\ [3]_	| `v4l2-compliance`` |
| V4L2 virtual drivers |``contrib/test/test-media`` |
| CEC drivers	|	``cec-compliance`` |


.. [3] The ``v4l2-compliance`` also covers the media controller usage inside
       V4L2 drivers.

[3] v4l2-compliance 也涵盖了 V4L2 驱动程序中的媒控制器的使用。

Other compilance tools are under development to check other parts of the
subsystem.

其他合规性工具正在开发中，用于检查子系统的其他部分。

Those tests need to pass before the patches go upstream.

这些测试在补丁上游之前必须通过。

Also, please notice that we build the Kernel with::

此外，请注意，我们使用以下命令构建内核：

    make CF=-D__CHECK_ENDIAN__ CONFIG_DEBUG_SECTION_MISMATCH=y C=1 W=1 CHECK=check_script

Where the check script is:

其中检查脚本为：

    #!/bin/bash
	/devel/smatch/smatch -p=kernel $@ >&2
	/devel/sparse/sparse $@ >&2

Be sure to not introduce new warnings on your patches without a
very good reason.

在没有很好的理由的情况下，请确保在你的补丁中不引入新的警告。

Style Cleanup Patches
---------------------

Style cleanups are welcome when they come together with other changes
at the files where the style changes will affect.

样式清理在文件的样式变更会受到其他更改的影响时是受欢迎的。

We may accept pure standalone style cleanups, but they should ideally
be one patch for the whole subsystem (if the cleanup is low volume),
or at least be grouped per directory. So, for example, if you're doing a
big cleanup change set at drivers under drivers/media, please send a single
patch for all drivers under drivers/media/pci, another one for
drivers/media/usb and so on.

我们可能会接受纯粹的独立样式清理，但最好是整个子系统的一个补丁（如果清理的内容很少），或者至少按目录分组。因此，例如，如果你正在对 drivers/media 目录下的所有驱动程序进行大型清理更改集，请发送一个用于 drivers/media/pci 下的所有驱动程序的单个补丁，另一个用于 drivers/media/usb，依此类推。

Coding Style Addendum
---------------------

Media development uses ``checkpatch.pl`` on strict mode to verify the code
style, e.g.:

媒体开发使用 checkpatch.pl 以严格模式验证代码样式，例如：

	$ ./scripts/checkpatch.pl --strict --max-line-length=80

In principle, patches should follow the coding style rules, but exceptions
are allowed if there are good reasons. On such case, maintainers and reviewers
may question about the rationale for not addressing the ``checkpatch.pl``.

原则上，补丁应该遵循编码风格规则，但如果有充分的理由，可以允许例外。在这种情况下，维护者和审阅者可能会问为什么不解决 checkpatch.pl 的问题。

Please notice that the goal here is to improve code readability. On
a few cases, ``checkpatch.pl`` may actually point to something that would
look worse. So, you should use good sense.

请注意，这里的目标是提高代码可读性。在少数情况下，checkpatch.pl 实际上可能指向看起来更糟糕的东西。因此，你应该用良好的判断力。

Note that addressing one ``checkpatch.pl`` issue (of any kind) alone may lead
to having longer lines than 80 characters per line. While this is not
strictly prohibited, efforts should be made towards staying within 80
characters per line. This could include using re-factoring code that leads
to less indentation, shorter variable or function names and last but not
least, simply wrapping the lines.

请注意，单独解决 checkpatch.pl 的任何问题（任何类型）可能导致每行超过 80 个字符。虽然这不是严格禁止的，但应该努力保持在每行 80 个字符以内。这可能包括使用重构代码以减少缩进、缩短变量或函数名称以及最后但并非最不重要的是简单地包装行。

In particular, we accept lines with more than 80 columns:

    - on strings, as they shouldn't be broken due to line length limits;
    - when a function or variable name need to have a big identifier name,
      which keeps hard to honor the 80 columns limit;
    - on arithmetic expressions, when breaking lines makes them harder to
      read;
    - when they avoid a line to end with an open parenthesis or an open
      bracket.

Key Cycle Dates
---------------

New submissions can be sent at any time, but if they intend to hit the
next merge window they should be sent before -rc5, and ideally stabilized
in the linux-media branch by -rc6.

新提交可以随时发送，但如果它们打算在下一个合并窗口中生效，它们应该在 -rc5 之前发送，并在 -rc6 之前理想地在 linux-media 分支中稳定下来。

Review Cadence
--------------

Provided that your patch is at https://patchwork.linuxtv.org, it should
be sooner or later handled, so you don't need to re-submit a patch.

只要你的补丁在 https://patchwork.linuxtv.org，它迟早会被处理，因此你不需要重新提交补丁。

Except for bug fixes, we don't usually add new patches to the development
tree between -rc6 and the next -rc1.

除了修复错误之外，在 -rc6 和下一个 -rc1 之间，我们通常不会向开发树中添加新的补丁。

Please notice that the media subsystem is a high traffic one, so it
could take a while for us to be able to review your patches. Feel free
to ping if you don't get a feedback in a couple of weeks or to ask
other developers to publicly add Reviewed-by and, more importantly,
``Tested-by:`` tags.

请注意，媒体子系统是一个高流量的子系统，因此我们可能需要一段时间才能审查你的补丁。如果几周内没有得到反馈，请随时催促，或要求其他开发人员公开添加 Reviewed-by 和更重要的 Tested-by: 标签。

Please note that we expect a detailed description for ``Tested-by:``,
identifying what boards were used at the test and what it was tested.

请注意，对于 Tested-by:，我们期望有详细的说明，指明测试时使用的板子以及测试内容。
