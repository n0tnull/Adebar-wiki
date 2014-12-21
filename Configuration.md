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

## User-defined Functions
As described for the `POSTRUN_CMD` and `APPNAME_CMD` above, you can use
„user-defined functions“ to enhance *Adebar*. The example config file
even holds two of them. If you do so, it is recommended to have their
names always starting with `uf_` (for „user function“). This makes sure
they won't collide with any functions defined in `adebar-cli` itself –
whether functions already available there, or added in the future.