# debian-compat-drivers-dkms

Debian package wrapper for DKMS installer for the compat-drivers
backports of the latest Linux network drivers to older kernels.

## Introduction

Many people have to run an older kernel than the latest one, for example
they run the latest one supported by their installed Linux distribution.

The Linux kernel is improving drivers and fixing bugs all the time, but
those improvements are usually only available in the latest kernel version.

[compat-drivers](https://backports.wiki.kernel.org/index.php/Documentation/compat-drivers)
is the framework that pulls code from Linux kernel releases
and strives to backport them automatically for usage on older Linux kernel
releases. compat-drivers allows us to make releases of code from newer
Linux kernel releases and get the latest drivers to users without having
to recompile their entire kernel.

[DKMS](https://help.ubuntu.com/community/DKMS) provides support for
installing supplementary versions of kernel modules. The package compiles
and installs modules into the kernel tree. Uninstalling restores the
previous modules. Furthermore, DKMS is called automatically upon
installation of new Ubuntu kernel-image packages, and therefore modules
added to DKMS will be automatically carried across updates. 

This project uses DKMS to wrap the compat-drivers, so that they will be
built and installed automatically for each new kernel that you install,
and you will always be able to use the latest wireless drivers without
manually compiling anything.

This project makes it easy to build a Debian package, that anyone can
install on a Debian or Ubuntu system, to get access to the latest drivers.

I used [these instructions](http://tjworld.net/wiki/Linux/Ubuntu/Kernel/BuildDebianDKMSPackages)
to build the Debian package.

## Notes

DKMS needs to know the list of modules (compiled .ko files) that will be
built by compat-drivers, and that list must be supplied before the drivers
are compiled on the target system (where the DKMS package is to be
installed) in dkms.conf.

To avoid hard-coding that list, I instructed the debian package build
system (via debian/rules) to actually build the modules on the host system
(where the package is built). These binary modules are not included in
the package, but the list of files is needed to generate dkms.conf
automatically.

## Configuration

The Debian package is configured to build all modules by default, which
can take a long time (for each newly installed kernel), and might not be
what you want. For example, you may want only the latest wireless card
driver. To change that, you need to download the source for this package:

	git clone git@github.com:aptivate/debian-compat-drivers-dkms.git

Enter the `compat-drivers` directory:

	cd debian-compat-drivers-dkms/compat-drivers

Reconfigure `compat-drivers`. You can get a list of available configuration
options by running:

	./scripts/driver-select

And to select a particular driver, in this case `iwlwifi` (Intel Centrino
Wireless-N):

	./scripts/driver-select iwlwifi

Or to build all drivers:

	./scripts/driver-select restore

Then build a new Debian package:

	debuild -i -us -uc -b

This should build a `.deb` file in the parent directory:

	dpkg-deb: building package `compat-drivers-dkms' in `../compat-drivers-dkms_3.7-rc1-6_all.deb'.

If you have `ccache` installed, and `/usr/local/bin/gcc` is a link to
`/usr/bin/ccache`, then you can use it when you build the package to
make building faster, by running this command instead:

	debuild --prepend-path=/usr/local/bin -i -us -uc -b

You can install this package on any Debian-like system where you want the latest drivers.

When you install it, DKMS should automatically build the drivers for your
current running kernel:

	First Installation: checking all kernels...
	Building for 2.6.32.59+drm33.24-cw-custom-dsdt-120808-1 and 3.0.0-26-generic-pae
	Building for architecture i686

You can check the status of the drivers in DKMS with this command

	$ dkms status -m compat-drivers-dkms
	compat-drivers-dkms, 3.7-rc1-6: added 

* `added` means that the module was registered with DKMS, but failed to
  build. This is bad. Try to build it with this command and see what
  happens:

	dkms build -m compat-drivers-dkms -v 3.7-rc1-6

* `built` means that the module has been built (for a particular kernel),
  but is not installed in that kernel's module directory under
  `/lib/modules` any more. The kernel may have been uninstalled, or the
  modules removed from it with the `dkms uninstall` command. If the
  kernel is still in use, then the updated `compat-drivers` modules
  are not available for use with it, which is bad, but can be fixed by
  running `dkms install -m compat-drivers-dkms -v 3.7-rc1-6`.

* `installed` means that the modules are in the kernel modules directory
  under `/lib/modules` and will be used. This is good.

