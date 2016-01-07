## Some apps simply don't want to be backed up via ADB (41 byte files)
In my tests I've encountered several apps which seem to simply refuse being
backed up via ADB. In those cases, backups result in a 41 byte file (which is
just the backup file header). So watch out for those, and see to have them
backed up by other means; as *Adebar* here is no more than a "front-end" to the
`adb backup` command, I don't see what it could fix here. As the example of
[DavDroid](https://github.com/rfc2822/davdroid) shows, this is rather to be
fixed on the corresponding Android app's end.

You should be able to identify any affected app in your created
[userApps.md](https://github.com/IzzySoft/Adebar/wiki/example-userApps.md) by
its „flags“: it will have no `ALLOW_BACKUP` and no `ALLOWBACKUP` flag reported.
Apps which I found having this issue include the following:

* installed as system apps:
    - GMail
    - SuperSU
* installed as user apps:
    - AppMonster Pro
    - DavDroid (pre-0.6.4; fixed [on DavDroid's end](https://github.com/rfc2822/davdroid/releases/tag/v0.6.4))
    - JuiceSSH
    - MobilityMap
    - WakeLockDetector

If your device is rooted, you can work around this using [Xposed](http://repo.xposed.info/module/de.robv.android.xposed.installer)
with the [Backup All Apps](http://repo.xposed.info/module/com.pyler.backupallapps)
module, which I tested successfully. The [XInstaller](http://repo.xposed.info/module/com.pyler.xinstaller)
module also contains a corresponding switch, and thus might work as well (not
tested by me).


## ADB Backup not working for *any* app (0/41 byte files)
Again no issue of *Adebar* – but an incompatibility on the ADB backend: the
version running on the device doesn't want to play with the version running
on your PC.

In the best case, simply updating the ADB binaries on your computer should solve
this – but there are cases reported where one must *downgrade* instead (see
[issue #7](https://github.com/IzzySoft/Adebar/issues/7#issuecomment-161903472) and,
in more detail, [Android.SE](http://android.stackexchange.com/q/83080/16575)).

If you're in the trouble having multiple devices and each requires its own ADB
version (rare case), a work-around (so each one gets its correct `adb` binary)
is to add a line to the corresponding config file: `alias adb='/path/to/adb'`,
to point to the corresponding binary for each device.


## Backup of each app has to be confirmed separately
Yes. That's a security measure so no stranger could simply connect an USB cable
to your device and steal your data. But there's a way to work around this
restriction using „key events“. If you want to go this way, add
`AUTO_CONFIRM=1` to your configuration file. Note you will still have to
unlock your device; the „key events“ won't register while the lock screen is
active.


## Backup passwords containing spaces causing issues
Yes they will. For details, please see [this pull request](https://github.com/IzzySoft/Adebar/pull/12).
As working around that issue is tricky at best, it's rather recommended to not
use spaces in passwords at all.


## `packages.xml` (and/or other files) not retrievable
On some devices (all 4.1+ devices with the ADB daemon running in non-root mode?),
`packages.xml` can not be pulled. Hence *Adebar* obtains related information via
`dumpsys` instead – which is a bit slower, but worked on all devices tested so far.

This applies to some other files as well, especially those with sensitive details
(e.g. `wpa_supplicant.conf`). If you want to pull them, you'll have to root your
device (a must) and make sure either the ADB daemon runs in root-mode (which can
e.g. be achieved using chainfire's [adbd Insecure](http://play.google.com/store/apps/details?id=eu.chainfire.adbd))
or you've set `ROOT_COMPAT=1` in your config.



## `disable` script not working?
It seems many (all?) `adb pm disable` commands are not performed when `adbd` is
running in non-root mode, at least when run against pre-installed apps (aka
„bloatware“; confirmations/dementis welcome, as the list of devices at my
disposal is pretty limited, so I can not say whether that's a general rule).
Nothing we can do about that.


## Apps are always listed by their package names
Even in `doc/userApps.md`, which is a documentation file, apps are only listed
with their package names (e.g. `eu.chainfire.adbd` for chainfire's *adbd Insecure*).
I wish there were means to list the „human readable app name“ along – but apart
from pulling the entire `.apk` file and dissecting it via `aapt`, or trying to
look up the package in the app stores' web pages, I know of no simple way to get
hold on it (if your device has `aapt` installed, as some CyanogenMod based ROMs
with Android 4.4+ seem to have it, that's used by *Adebar* automatically; if not,
you can install it manually [from here][1] if your device is rooted).

To work around that, you can manually create the corresponding cache files, as
described in [[Configuration]]. For apps available at *Google Play*, you can
also try the example `uf_getAppname()` user function defined in the example
config file, enabling it by setting `APPNAME_CMD="uf_getAppname"` in your config.
This will, whenever an app name was not found in the cache, retrieve it from the
HTML TITLE attribute of the app's *Playstore* web page.

Alternative suggestions welcome.


## Difficulties dealing with multiple devices having the same serial
This mostly goes with custom ROMs or with „cheap devices“, and is nothing to
blame upon *Adebar:* I've encountered multiple devices/ROMs from different
vendors using the serial `0123456789ABCDEF`, which is obviously some „dummy“.
Not a big deal if you call the *Adebar* script with the right config for the
connected device – but it might become tricky if you want to make use of the
`--auto` feature, where *Adebar* checks for connected devices using the `adb
devices` command, and tries to figure the correct config file by the serials
reported.

Nothing we can do about that on the end of *Adebar* – but if your device is
rooted, you may be able to fix its serial yourself: The serial is usually taken
from `/sys/class/android_usb/android0/iSerial`. If you can manage to have a
unique value written there automatically at boot, that would fix it.

    echo -n MyUniqueSerial > /sys/class/android_usb/android0/iSerial

would do the job – you just need to find out how to set that automatically.
Examples of possibilities are:

* integrating it into `/system/etc/install-recovery.sh` if that file exists
  (works on some devices – but not on all, even if the file exists)
* integrate it into an init script if your device and kernel support it
  (those devices usually have a `/system/etc/init.d` directory); also see:
  [How can I run a script on boot?](http://android.stackexchange.com/q/6558/16575)
  and [How to run a script on boot](http://android.stackexchange.com/a/115595/16575)
  (the latter is about how to add `init.d` support to your device if it's not there)
* according to [this SO post](http://stackoverflow.com/a/29389115/2533433) it might
  work to simply issue a `setprop persist.usb.serialno MyUniqueSerial` as root,
  and then reboot the device.
* run a script at boot time by other means, e.g. utilizing [Script
  Manager](https://play.google.com/store/apps/details?id=os.tools.scriptmanager)
  or [Tasker](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm)


[1]: http://android.izzysoft.de/downloads "IzzyOnDroid: Android Downloads"
