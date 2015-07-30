#nvidia_graphics

####Table of Contents

1. [Overview](#overview)
2. [Module Description - What the module does and why it is useful](#module-description)
3. [Setup - The basics of getting started with nvidia_graphics](#setup)
    * [What nvidia_graphics affects](#what-nvidia_graphics-affects)
    * [Setup requirements](#setup-requirements)
    * [Beginning with nvidia_graphics](#beginning-with-nvidia_graphics)
4. [Usage - Configuration options and additional functionality](#usage)
5. [Reference - An under-the-hood peek at what the module is doing and how](#reference)
5. [Limitations - OS compatibility, etc.](#limitations)
6. [Development - Guide for contributing to the module](#development)

##Overview

This module installs and maintains NVIDIA's proprietary graphics
driver for their video cards, using driver installers downloaded from
the NVIDIA website.

It should work on any Red Hattish distro, and any Puppet from 2.7 on.

##Module Description

NVIDIA makes graphics chipsets. Open-source drivers for this hardware
exist, such as Nouveau, which can do 2D and some 3D graphics with
these chipsets, but to unlock the full potential of the hardware one
must use the proprietary driver.

The driver is structured as a huge gob of proprietary code, with a
small adapter that is compiled as a module for the Linux kernel. For
every kernel you use, the adapter code must be built. When you install
a kernel upgrade, the adapter code must be rebuilt if the driver is to
continue working.

Some distributions have integrated the driver into their packaging
systems and automated this rebuilding process. RHEL6 hasn't.

So, this module contrives to insert a check at boot time to make sure
the driver will work when the X server starts. With this in place, you
can upgrade your kernel, X server, or Mesa without having to remember
to reinstall the video driver.

Legacy NVIDIA hardware is dealt with automatically.


##Setup

###What nvidia_graphics affects

* It changes your grub configuration to disable the nouveau video
  driver and graphical boot.
* It installs a new init script, which runs at boot time.
* It installs the NVIDIA proprietary graphics driver, which
  overwrites (Mesa) OpenGL libraries and OpenGL-related X server
  extensions.
* It configures the X server to use the proprietary driver.


###Setup Requirements

You need pluginsync on, because this module installs custom facts.

	
###Beginning with nvidia_graphics

First, download the NVIDIA drivers into a directory. If you are
administering a group of workstations, make this a directory they can
all get to over the network, via NFS or what-have-you. Let us call
this directory the "installer dir." In this directory make at least
one symlink, called latest-x86_64, pointing at the latest NVIDIA
driver installer you have.

Include the `nvidia_graphics::proprietary` class; pass the installer
dir as a parameter to it. It is harmless to include this class on
nodes with no NVIDIA graphics card: it will do nothing in that
case.

After the node is configured, reboot it, so the changed grub settings
will take. The `nvidia-rebuild` service will start at boot time, and
attempt to install the driver if it is not in a functional state.


##Usage

```
    class { 'nvidia_graphics::proprietary':
      installer_dir => '/net/my/cool/driver/place',
    }
```

With the `nvidia-rebuild` service in place, you can trigger a driver
reinstall yourself by becoming root and restarting the service, like:

```
    service nvidia-rebuild restart
```

You can't do this while there is an X server running; but it's likely
that if you are issuing this command it will be because the X server
is not running properly.

The complete set of driver installers that may be expected in the
installer dir is:

* latest-x86_64
* legacy-340-x86_64
* legacy-304-x86_64
* legacy-17314-x86_64
* latest-i386
* legacy-340-i386
* legacy-304-i386
* legacy-17314-i386

(The i386 and x86_64 are architecture names which occur in the output
of `uname -m`.) The `legacy-*` correspond to [legacy driver
versions](http://www.nvidia.com/object/IO_32667.html) put out by
NVIDIA; you only need to obtain these older drivers if you have some
older NVIDIA graphics cards.


##Reference

###Classes

####`nvidia_graphics::proprietary`
The class to include to make all the cool stuff happen.
#####Parameters
######`installer_dir`

Specifies a directory which contains NVIDIA graphics driver installers
with certain peculiar filenames, as listed [above](#usage). A string.


###Facts

All booleans.

* `has_nvidia_graphics_card`: Whether this host has any NVIDIA
  graphics hardware.

* `has_nvidia_legacy_340_graphics_card`: Whether this host has a card
  which requires a 340.x (and no newer) NVIDIA driver.

* `has_nvidia_legacy_304_graphics_card`: Whether this host has a card
  which requires a 304.x (and no newer) NVIDIA driver.

* `has_nvidia_legacy_17314_graphics_card`: Whether this host has a
  card which requires a 173.14.x (and no newer) NVIDIA driver.

* `using_nouveau_driver`: Whether the Nouveau driver is presently
  active on this host. The NVIDIA proprietary driver will not install
  if Nouveau is running.

* `nvidia_ko_exists`: Whether the NVIDIA kernel module exists for the
  version of the kernel presently being run by this host. This module
  is built by the driver installer.

* `nvidia_libGL_installed`: Whether libGL.so appears to be the version
  belonging to the proprietary NVIDIA driver. If the driver is
  installed, and then Mesa is upgraded, Mesa can overwrite the library
  file put in place by the driver.

* `nvidia_glx_extension_installed`: Whether the X.org GLX extension
  appears to be the version installed by the proprietary NVIDIA
  driver. This file can be overwritten by upgrades to X server
  packages. N.B. This fact is a stub at present.


##Limitations

OS compatibility: RHEL 6, CentOS 6, or similar.

Puppet version compatibility: `>=2.7, <4`: Not tested with 4.x.

You can't put a legacy NVIDIA card and a newer one in the same machine
and have it work right, because only one version of the NVIDIA driver
can be installed at once. NVIDIA defines "legacy" on [this
page](http://www.nvidia.com/object/IO_32667.html).

You may not be able to put an NVIDIA driver newer than 256.53 on a
RHEL 5 box, and there are NVIDIA cards too new for 256.53 to support.


##Development

Contributions are welcome! Please fork
https://github.com/jaredjennings/puppet-nvidia_graphics and send a
pull request, or file issues at
https://github.com/jaredjennings/puppet-nvidia_graphics/issues.

The module is made available under the Apache 2.0 license.


##Release Notes/Contributors/Etc **Optional**

*0.2* Added a new legacy category: support for using a 340.xx driver (and no newer) where necessary.
