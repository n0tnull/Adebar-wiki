## What needs to be configured?
In principle: nothing. *Adebar* should run fine out-of-the-box, as it comes with
a decent „default configuration“ coded into the main script itself. So if you
first just want to give it a look, you can skip this section and start *Adebar*
straight away. Come back here if you feel the need to fine-tune things.


## Where does *Adebar* check for configuration files?
When running, *Adebar* first sets up „default values“, and then checks for an existing config file:

* if `config/` is a directory, and contains a file with the same name
  as specified by the first command-line parameter, this is used. This
  allows for device-specific config files if you have more than one
  device.
* otherwise, if `config/` is a directory, and contains a file named
  `default`, this will be used. You might want to have that for e.g.
  "guest devices" or as a default for possible „new ones“
* otherwise, if `config` is a file, that will be used. Makes it easier
  if you only have one device, and don't plan for more.

A sample configuration file is included as `doc/config.sample`, which you
simply can copy and adjust. Your config file has only to contain the
settings you wish to change; this will usually be the `DEVICE_IP` (which is not
set by default) and/or the `STORAGE_BASE`.


## What settings are available, and what do they mean?
A short information for each setting is contained in the example config file, where all available settings are included. Here comes a more detailed description:

### Directory Settings
* `STORAGE_BASE`: that's the „base directory“ everything goes below. Here
  the command-line provided `OUTDIR` will be appended to. By default, this
  variable is empty, so `OUTDIR` is relative to the execution path. If you
  e.g. set it as `STORAGE_PATH=/home/me/backups`, and pass `moto_x` as
  parameter to `adebar_cli`, your files should end up below
  `/home/me/backups/moto_x`.
* `USERDIR`: sub-directory where the backup scripts will place the ADB backups
  of your user-apps into (relative to where they're run). Default is "userApps".
* `SYSDIR`: Similar, for the data backups of your system apps. Defaults to "sysApps".
* `PARTBACKUPDIR`: again similar, for partition images
* `DOCDIR`: Where to place created documentary files. Default is "docs", which results
  in `${STORAGE_PATH}/${OUTDIR}/docs`
* `CONFDIR`: Similar for pulled config files (`conf/`). Default is "conf", which has
  the device's config files placed in `${STORAGE_PATH}/${OUTDIR}/conf`.
* `CACHEDIR`: In the cache directory, Adebar stores data between runs –
  so it does not have to re-evaluate them. This includes e.g. names of apps
  in the `appnames` sub-directory. Leave it empty (default) to disable caching.

### Device specifics
* `DEVICE_NAME`: A name you can recognize your device by. This is used
  for the documentory files such as [[deviceInfo.md|example deviceInfo.md]],
  but also in scripts (mainly for headers in both cases). Defaults to "MyDroid".
* `SERIAL`: The device's serial as returned by `adb devices`. By default this
  is empty. It's not strictly needed as long as you only have one device
  connected when running `adebar-cli`. If there are multiple devices connected,
  *Adebar* uses the `SERIAL` to tell ADB which one to use.

### Settings for Titanium Backup
*Titanium Backup* features an integrated web server to obtain device contents from.
*Adebar* can create a script to do that for you, which is what these settings are for.

* `DEVICE_IP`: IP address of your device. By default this is empty, so the TiBu
  part will be skipped.
* `TIBU_PORT`: port the *TiBu* web server listens on. Default is "8080".
* `TIBU_SDINT`: URL path to obtain contents of the internal SD. Default is
  "/storage/INTERNAL/Storage-ALL.zip"
* `TIBU_SDEXT`: URL path to obtain contents of the external SD. Default is
  "/storage/SAMSUNG_EXT_SD_CARD/Storage-ALL.zip"
* `TIBU_BACKUPS`: URL path to obtain the *TiBu* backups. Default is "/TitaniumBackup-ALL.zip"

### Features
While *Adebar* can create a lot of files containing details from your device, you might
not have use for all of them, and thus wish to „skip“ some parts. You can do so by enabling
or disabling the corresponding feature. These settings are „booleans“, so `0` means „off“
and `1` stands for „on”. By default, all features are enabled (i.e. set to `1`).

* `MK_APPDISABLE`: the script to „freeze/disable“ apps
* `MK_USERBACKUP`/`MK_SYSBACKUP`: create the script to backup user apps+data /
  system app-data
* `PULL_SETTINGS`: pull settings/configs from the device (currently just
  `wpa_supplicant.conf`, but there might be more in the future)
* `MK_TIBU`: create the script to pull stuff from the TiBu web server.
  Even if set to `1`, this will be disabled if you didn't set a `DEVICE_IP`
  (see above in „Settings for Titanium Backup“).
* `MK_PKG_DATA`: pull `packages.xml` (if possible) and also create files
  with package details: Creates the *script* to deal with "disabled components",
  plus the markup document holding details on installed apps and their sources
  ([[userApps.md|example userApps.md]]).
* `MK_INSTALLLOC`: Deal with the default-install-location (where apps should
  be installed by default: 0=auto (system decides), 1=device, 2=sdcard).
  Creates a 1-liner script to set that again.
* `MK_PARTBACKUP`: Create a script to backup storage partitions (raw backup
  images). Turned OFF by default.
    * `PARTITION_SRC`: Where to obtain the partition details from. Reliability
      of sources differs between manufacturers, devices, ROMs, and possible
      other modifications. If this is set to "auto", *Adebar* tries to figure
      the best source itself. It currently considers `/proc/mtd`, `/proc/emmc`,
      `/dev/block/platform/*/by-name`, `/proc/partitions`, and `/proc/mounts`.
      Moreover, found details are sometimes combined from different sources.
      If for your device, *Adebar* seems to use the wrong source, or you want
      to "speed up" things a little, you can define the source to be used with
      this setting. Available options are currently: auto|mtd|emmc|byname|parts.
      Most reliable details seem to be available on MediaTek devices, but I
      have none at my disposal to check. You might get a „green hint“ if that
      source was detected. If that happens, please get in contact with me, so I
      can improve *Adebar* with your help.
* `MK_DEVICEINFO`: Create a (Markdown) document containing device information.
  For an example, see the [[example deviceInfo.md]].
    * `MK_DEVICEINFO_SENSORS`: Sensor details are part of device-info; if you
      wish to ommit it, this is the switch for it. I found this service
      sometimes crashes on one of my devices (`adb shell sensorservice` should
      restart it then)
    * `MK_DEVICEINFO_PMLISTFEATURES`: Though this part is done quickly, you
      might decide it's not useful to you (as it looks a bit cryptic). So I
      decided to give you the change to switch it off :)
    * `MK_DEVICEINFO_STATUS`: Device status details, such as battery status
    * `MK_PARTINFO`: Include details on storage partitions

### UserApp specifics
When an app is installed, Android records the „installer“ together with other
„package details“. This detail is not always available (it is e.g. missing
for pre-installed apps, when you „sideload“ an app via `adb install` or by
installing it from an `.apk` file stored on your device), so we unfortunately
cannot apply that to all apps...

This section has some places pre-configured – but you of course can complement
these settings: the „installer“, if available, is listed with the package
details. Apps are grouped by installers in [[userApps.md|example userApps.md]].

* `APP_INSTALL_SRC[installer.package.name.here]`: How the installer should
  appear in „human readable format“ with your listings.
* `APP_MARKET_URL[installer.package.name.here]`: The full URL of the
  corresponding market. `%s` will be replaced by the package name.

### Miscellaneous Settings
* `PROGRESS`: Show some progress while the script is running, so you know
   what's going on (and don't think it got „stuck“). By default, this is
   enabled. To disable, set it to `0`. To increase verbosity, place a higher
   number (currently used are values from 1 to 4 – feel free to use larger
   numbers for possibly added more-verbose-levels or debug info).
* `USE_ANSI`: Using ANSI sequences, *Adebar* can highlight important parts
   of the progress output: e.g. mark error messages red, titles bold, etc.
   This is fine for interactive use, but might not be wished in scripts.
   Set it to `0` to disable (Default: `1`, i.e. enabled)
* `TIMESTAMPED_SUBDIRS`: Is you want to keep multiple "generations" of the
   created files, this option can come in handy. It will cause *Adebar* to
   create a time-stamped sub-directory on each run. If you e.g. would call
   it `adebar-cli mydevice`, it would store the generated files in a
   directory `mydevice/201412010950` (i.e. YYYYMMDDHHMI as name below the
   name passed at command-line). By default, this is disabled. Note further
   that the optional second command-line parameter would override this.
* `LINK_LATEST_SUBDIR`: With `TIMESTAMPED_SUBDIRS=1`, this option would
   cause *Adebar* to keep a symlink pointing to the always latest copy.
* `KEEP_SUBDIR_GENERATIONS`: With `TIMESTAMPED_SUBDIRS=1`, and when set
   to a value other than `0`, *Adebar* automatically deletes older
   „generations“ and only keeps the specified number. If a „generation“
   seems to contain „real backups“ (i.e. has files in the `$USERDIR`
   of `$SYSDIR` directories, or contains any `.ab` (ADB Backup) or `.gz`
   (GZipped files, e.g. ADB Backups converted using `ab2tar`) files, it
   will not be removed: instead, *Adebar* renames it to `*.Backup` and
   issues a corresponding warning. You will have to take care for those
   yourself then.  
   By default, this is set to `0`, so no automatic purging takes place.
* `POSTRUN_CMD`: When *Adebar* is done extracting all information from
   your device, it optionally can run another command. This could be a
   script, any executable (with or without parameters), or even a function
   you've set up in your config file. This way you could e.g. zip the files
   and send them by mail, or whatever you think useful :)
* `APPNAME_CMD`: Similar to `POSTRUN_CMD` a command to retrieve app names.
  Must return a string only (the app name or, if it fails to retrieve it,
  the package name as passed to it). Only parameter is the package name.
* `ROOT_COMPAT`: Root compatibility mode. By default turned off (`=0`).
  Turn this on (`=1`) when your device is rooted, but the ADB daemon (`adbd`)
  is not running in root mode (i.e. your corresponding device property is
  set to `ro.secure=1` – which you can check with `adb shell getprop ro.secure`;
  note that some devices seem to "lie" here, so the final test would be
  `adb shell ps adb`, which should report whether adbd is run by root). When
  in compatibility mode, some "secured config files" are not simply "pulled"
  (which would remain their timestamps), but "cat" – as that can be done via
  `adb shell` using the `su -c` command.

## User-defined Functions
As described for the `POSTRUN_CMD` and `APPNAME_CMD` above, you can use
„user-defined functions“ to enhance *Adebar*. The example config file
even holds two of them. If you do so, it is recommended to have their
names always starting with `uf_` (for „user function“). This makes sure
they won't collide with any functions defined in `adebar-cli` itself –
whether functions already available there, or added in the future.


## App names
If you ran *Adebar* out-of-the-box, you will notice that even in
[[userApps.md|example userApps.md]], apps are only referenced by their
corresponding package name (e.g. `com.keramidas.TitaniumBackup`). So you will
certainly miss the name you remember the app by (for the given example:
„Titanium Backup“). The reason is quite simple: There is no generic way to
retrieve the names of apps – be it from the device itself, or any other source.
ADB does offer no such feature.

But still, there are ways: to have your `userApps.md` reveal the names for
packages, you can manually create the corresponding cache files:

1. Create a cache directory and inside that, an additional directory
   `appnames`
1. Inside the `appnames` directory, for each app create a corresponding
   file with the name of the package, e.g. `appnames/com.foobar.app`,
   which holds nothing but the app name (e.g. "Foobar App")
1. edit your config, and point the `CACHEDIR` to the cache directory. You can
   use an absolute path (e.g. `/home/me/adebar-cache`), or a relative one (e.g.
   `cache`); in the latter case, this is interpreted as relative to the path
   you're in when running *Adebar* (usually the one `adebar-cli` resides in).

Now *Adebar* will automatically check for each app whether its name is known
here, and use it accordingly.

