# new-lg4ff for Linux

Improved Linux module driver for Logitech driving wheels.

Thanks to Oleg Makarenko for adding support for the Logitech G923 Racing
Wheel (PS4/PC version).

Supported devices:

- Logitech WingMan Formula GP (without force feedback)
- Logitech WingMan Formula Force GP
- Logitech Driving Force
- Logitech MOMO Force Feedback Racing Wheel
- Logitech Driving Force Pro
- Logitech G25 Racing Wheel
- Logitech Driving Force GT
- Logitech G27 Racing Wheel
- Logitech G29 Driving Force (switch in PS3 mode)
- Logitech G923 Racing Wheel for PlayStation 4 and PC (046d:c267,
  046d:c266)
- Logitech MOMO Racing
- Logitech Speed Force Wireless Wheel for Wii

This module is not compatible with the Logitech G920 Driving Force and XBOX/PC
version of the Logitech G923 (046d:c26d, 046d:c26e). Both wheels use the HID++
protocol and are supported by the HID++ driver as of kernel 6.3.

Reports on success or failure using this module on untested devices are
welcome.

## Differences with the in-tree module

It has all the features in the in-kernel `hid-logitech` module and adds the
following ones:

- Support for most effects defined in the Linux FF API (except inertia) rather
  than just constant the force effect.
- Asynchronous operations with realtime handling of effects.
- Rate limited FF updates with best possible latency.
- Tunable sprint, damper and friction effects gain.
- It can combine accelerator and clutch.
- Use the wheel leds as a FFBmeter to monitor clipping.
- Added a system gain setting that modulates the gain setting used by
  applications.
- SYSFS entries for gain, autocenter, spring/damper/friction effect gain and
  FFBmeter.

## Requirements

- GMake
- GCC
- The `linux-headers` package matching the installed kernel version.

## Build and install

The module can be installed with DKMS or manually. Follow just one of these
procedures.

### DKMS

The module will be installed in the `/usr/src` directory. Removal, updates, and
rebuilds for kernel upgrades will be managed by `dkms`. The module will load
automatically at system reboot.

Follow these steps:

- Install `dkms` from the package manager in your system.
- Download the project to `/usr/src/new-lg4ff`.
- Install the module:

`$ sudo dkms install /usr/src/new-lg4ff`

When using DKMS the module will be installed as `hid-logitech` so it
automatically replaces the old module. Once loaded, it will be displayed
as `hid-logitech-new` though.

NOTE: If you had previously installed the module using the manual method then
you must delete the module by hand: `$ sudo rm /lib/modules/$(uname
-r)/extra/hid-logitech-new.ko`

#### Update initrd image

If you have to load the module manually every time after booting, you need to
update the initrd image after the module is installed.

Users of distros using `initramfs` can do it with the command:

`sudo update-initramfs -u`

Users of distros using `dracut` can do it with the command:

`sudo dracut -f`

#### Updating

When updating the module, remove the old module, update the code in
`/usr/src/new-lg4ff` and install again.

```
$ sudo dkms remove new-lg4ff/<version> --all
$ sudo dkms install /usr/src/new-lg4ff
```

Replace ` <version>` with the version you want to remove.

### Manual method

Use only if DKMS doesn't work.

Type the following commands inside the project directory to build and install:

```
$ make
$ sudo make install
```

In some distributions, the install step might throw some SSL errors because
it's trying to sign the module. These errors can be ignored.

Now the module is installed but not loaded.

To load the module:

`$ sudo make load`

To unload the module (restoring the in-kernel module):

`$ sudo make unload`

### Check that the driver is loaded

Run:

`sudo dmesg`

You should see something like this (notice the version at the end of the module
description):

```
[347092.750524] logitech 0003:046D:C24F.000B: Force feedback support for Logitech Gaming Wheels (0.2b)
[347092.750525] logitech 0003:046D:C24F.000B: Hires timer: period = 2 ms
```

## Force Feedback clipping

There's been work done to be able to monitor FF clipping and avoid it. The FF
force level is calculated as a maximum since we can't know the exact amount of
force the wheel is applying because conditional effects are dynamic depending
on wheel movements.

Games using conditional effects will report the maximum force level as if the
conditional effects were playing at the maximum level. This way we may get a
higher value than real one but never lower. Since the goal is to avoid
clipping, this is the measure we are interested in.

Conditional effects gain can be tuned seperately for each effect type to leave
more dynamic range for the other effects. The default value seems close enough
to the gain used in the official Logitech driver for Windows.

Clipping isn't necesarily bad. To allow a wider dynamic range of forces to be
played, it may be good to have some very light clipping, and even lots of
clipping are acceptable when crashing or driving over very rough ground.

## Options

You can use `modinfo` to query for available options.

New options available:

- timer_msecs: Set the timer period. The timer is used to update the FF
  effects in the device. It changes the maximum latency and the maximum rate
  at which commands are sent. Maximum 4 commands every timer period get sent.
   The default value is 2ms, less will have no benefit and greater may add
   unrequired latency.

- fixed_loop: Set the firmware loop mode to fixed or fast. In fixed mode it
  runs at about 500Hz. In fast mode it runs as fast as it can. The default is
  fast loop to try to minimize latencies.

- timer_mode: Fixed (0), static (1) or dynamic (2). In fixed mode the timer
  period will not change. In static mode the period will increase as needed.
  In dynamic mode the timer period will be dynamic trying to maintain synch
  with the device to minimize latencies (default).

- profile: Enable debug messages when set to 1.

- spring_level: (see the corresponding SYSFS entry).

- damper_level: (see the corresponding SYSFS entry).

- friction_level: (see the corresponding SYSFS entry).

- ffb_leds: (see the corresponding SYSFS entry).

## New SYSFS entries

They are located in a special directory named after the driver, for example:

`/sys/bus/hid/drivers/logitech/XXXX:XXXX:XXXX.XXXX/`

Entries in this directory can be read and written using normal file commands to
get and set property values.

### combine_pedals

This entry already existed. It has been extended, setting the value to 2
combines the clutch and gas pedals in the same axis.

### gain

Get/set the global FF gain (0-65535). This property is independent of the gain
set by applications using the Linux FF API.

### autocenter

Get/set the autocenter strength (0-65535). This property can be overwritten by
applications using the Linux FF API.

### spring_level

Set the level (0-100) for the spring type effects.

### damper_level

Set the level (0-100) for the damper type effects.

### friction_level

Set the level (0-100) for the friction type effects.

### ffb_leds

Use the wheel leds (when present) to monitor FF levels.

Led combinations:

- All leds off: force < 7.5% (normally the force is lower than the wheel
  mechanical friction so it will be too weak to be noticed).
- 1 led on from outside: 7.5%-25% force.
- 2 leds on from outside: 25%-50% force.
- 3 leds on from outside: 50%-75% force.
- 4 leds on from outside: 75%-90% force.
- 5 leds on from outside: 90%-100% force.
- 1 led off from outside: 100%-110% force (some clipping but most probably
  unnoticeable).
- 2 leds off from outside: 110%-125% force (probably noticeable light clipping).
- 3 leds off from outside: 120%-150% force (clipping must be pretty noticeable).
- 4 leds off from outside: force > 150% (clipping hard).

### peak_ffb_level

It returns the maximum detected FF level value as an integer. It can be written
to reset the value and start reading again. Values read will be always greater
or equal than the last value written. Values between 0-32768 mean no clipping,
greater values mean there can be clipping.

## Contributing

Please, use the issues to discuss bugs, ideas, etc.

## Disclaimer

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
# G940-linux
Improvements to Linux kernel support for
[Logitech Flight System G940](https://support.logitech.com/en_gb/product/flight-system-g940)

This device has partial Linux support since 2010, but with bugs including some axes conflated/missing.

I have made both some relatively uncontroversial improvements,
[submitted upstream](https://patchwork.kernel.org/project/linux-input/list/?series=90297)
(fix axis/button mappings, support LEDs, remove hard autocenter) and some more experimental changes which are only here, to
support additional force feedback effects by including
[ff-memless-next from Michal Malý](https://patchwork.kernel.org/project/linux-input/list/?q=ff-memless-next&archive=both),
a sort of emulation layer for periodic and rumble effects which was submitted upstream circa 2014, never merged, but it still
works great for this.

The repository contains a replacement for the **hid_logitech** module called **hid_logitech_next** and a support module
called **ff_memless_next**.

## Support

Module is tested with kernel version `5.4.0`. Other kernels *might* work, but this is not tested. In any case, use of this module is at **your own risk**.

Module may be installed manually or by creating a DKMS package (for Ubuntu, Debian etc.).

## Build and install kernel module without packackage

This installation procedure must be run whenever the kernel has been upgraded.

*Preconditions: The repository has been cloned from GitHub, and current directory is the top level source directory.*

1.  Kernel headers for running kernel must be present in `/lib/modules/(VERSION)/`
2.  Compile and install module: `cd drivers/ && ./local_make.sh modules && sudo ./local_make.sh modules_install`
3.  Blacklist the old **hid_logitech** module: `sudo echo "blacklist hid_logitech" > /etc/modprobe.d/blacklist-hid-logitech.conf`
4.  Unload old module if loaded: `sudo rmmod hid_logitech`

## Build .deb DKMS package (for Debian, Ubuntu etc.)

The DKMS package will ensure that a new module is automatically built whenever the kernel is upgraded.

*Preconditions: The repository has been cloned from GitHub, and current directory is the top level source directory.*

1.  Install kernel headers package (`linux-headers-(VERSION)`)
2.  Install dependencies: `sudo apt install dkms debhelper dpkg-dev`
3.  Build package: `dpkg-buildpackage`
4.  Install package: `sudo dpkg -i ../g940-dkms*.deb`

NOTE : Other packages might be required for building, install those who are missing if step 3 fails.

## Check that correct module is loaded

1.  (re)connect device
2.  Run and verify that **hid_logitech_next** exists, but not **hid_logitech**: `lsmod | grep hid`
3.  If the old module is still loaded, perform a restart.
