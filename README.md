# improved lg4ff and  lg3ff drivers

This repository merges the changes of the new-lg4ff driver (https://github.com/berarma/new-lg4ff) and G940-linux driver (lg3ff, https://github.com/chrisboyle/G940-linux). So both changed driver can used together.

For further instructions for the drivers see the origin Readme from the Repos.

Tested with Debian 13

## Requirements

- dkms
- GMake
- GCC
- The `linux-headers` package matching the installed kernel version.

## Install

The module will be installed in the `/usr/src` directory. Removal, updates, and
rebuilds for kernel upgrades will be managed by `dkms`. The module will load
automatically at system reboot.

Follow these steps:

- Install `dkms` from the package manager in your system.
- Download the project to `/usr/src/new-lg3ff-lg4ff`.
- Install the module:

`$ dkms install /usr/src/new-lg3ff-lg4ff`

When using DKMS the module will be installed as `hid-logitech` so it
automatically replaces the old module. Once loaded, it will be displayed
as `hid-logitech-new` though.

NOTE: If you had previously installed the module using the manual method then
you must delete the module by hand: `$ sudo rm /lib/modules/$(uname
-r)/extra/hid-logitech-new.ko`

#### Updating

When updating the module, remove the old module, update the code in
`/usr/src/new-lg4ff` and install again.

```
$ sudo dkms remove new-lg4ff/<version> --all
$ sudo dkms install /usr/src/new-lg4ff
```

Replace ` <version>` with the version you want to remove.