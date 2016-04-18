Lenovo S12 Mint/Ubuntu suspend fix.
===================================
After scouring google for a while I wrote this
tiny script to fix the issue I have with my
Lenovo S12 running Mint 17 (and apparently
also present on Ubuntu 14.04).

The Problem
-----------
The big symptom was that, whenever suspend was initiated, either
by closing the lid, or typing `pm-suspend`, or by pressing the power
button, the screen would go dark for a few seconds and then
wake up by itself.

Apparently it's caused by some USB device drivers not going
into a power-saving mode or whatever, I really didn't dig
that far down. Those devices prevented the kernel from
completely going into suspend mode, causing the computer
to wake up immediately after suspending.

The Solution
------------
It's quite simple once you know where to look.

The file `/proc/acpi/wakeup` contains a list of the
various devices and their status; some of them will have
an 'enabled' or a 'disabled' code on them, for example:

```
$ cat /proc/acpi/wakeup
Device	S-state	  Status   Sysfs node
PCI0	  S5	*disabled  no-bus:pci0000:00
USB0	  S3	*enabled   pci:0000:00:04.0
USB1	  S3	*enabled   pci:0000:00:04.1
USB2	  S3	*enabled   pci:0000:00:06.0
USB3	  S3	*enabled   pci:0000:00:06.1
MAC0	  S5	*disabled
AZA	  S5	*disabled  pci:0000:00:08.0
XVR3	  S5	*disabled  pci:0000:00:15.0
P2P0	  S5	*disabled  pci:0000:00:09.0
LID0	  S3	*enabled   platform:PNP0C0D:00
SLPB	  S3	*enabled   platform:PNP0C0E:00
$_```

The problem devices are those that have 'enabled' next
to them.

If you `echo USB# > /proc/acpi/wakeup`, the kernel will
set the corresponding line to `disabled`, which appears
to be the appropriate state for either suspending or
hibernating.

So, the solution would be to set all `enabled` devices
to `disabled` before suspending or hibernating, and then
resetting them to `enabled` after recovering from the
power saving state.

The Script
----------
This script does exactly that: before suspending or
hibernating, it obtains a list of the `enabled` devices
and sets them to `disabled`, and saves the list
in `/var/tmp`.  When returning from the power-saving
state, it'll fetch the device list and set them to
`enabled`.

Place the script in `/etc/pm/sleep.d` and set it
to mode 755.

Enjoy!

And, as expected, I disclaim any responsibility over the
consequences of using this script. I'm merely sharing this
because it worked for me, no guarantee that it will work
for you as well.

Have fun!

-gca
