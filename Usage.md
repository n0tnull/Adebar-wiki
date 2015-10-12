## General Usage
`adebar-cli` requires at least one parameter, which will be used to

* detect a matching (device-specific) configuration file (see [[Configuration]])
* define the desired output-directory (residing below the configured
  `STORAGE_BASE`)

You can pass it an optional second parameter, which will then be appended to the
output directory name. This might be especially useful if you want to keep scripts
from a specific run, and not have them overwriting your „default set“.

(Note that for *always* keeping „historical sets“, there's an easier way: see the
`TIMESTAMPED_SUBDIRS` parameter in the [[Configuration]].)

As this might be easier to understand with an example: Let's say you've got two
devices, one Motorola and one HTC. For each of them, you wish different settings
to be applied. So for a „shortcut“, you name the first „moto“ and the second „htc“.
For easier handling, you could do the following:

* create the `config` directory below the directory `adebar-cli` resides in
* copy `doc/config.sample` to `config/moto` resp. `config/htc`
* adjust the two files to your needs
* call `adebar-cli` with...
  * `./adebar-cli moto` to create the scripts in `$STORAGE_BASE/moto`
  * `./adebar-cli htc _20141101` to create them in `$STORAGE_BASE/htc_20141101`

Calling `adebar-cli` without specifying a parameter will cause it to tell you
its syntax – and list available configuration files you have created in the
`config/` directory.

## Exit codes
* 1: Syntax error (missing or wrong arguments when calling the script)
* 2: No device found
* 3: Multiple devices found, but no SERIAL specified in config
* 4: SERIAL specified, but no matching device connected
* 5: target directory doesn't exist, and neither can be created


## How to use the generated scripts
For an overview of which scripts are generated, please see the [[Files]] page.

### Backup
There are multiple backup scripts generated:

* `sysbackup`/`userbackup`: these scripts facilitate the `adb backup` command.
  As simply creating one full backup containing all your data would result in
  the fact you could only do a full restore with ADB (so it's an „all or nothing“,
  unless you use third-party-tools), *Adebar* splits this up into multiple
  pieces: a separate backup archive for each app, so you can do a selective
  restore.  
  Check the script, comment out what you don't want to be backed up („shared
  storage“, i.e. your SD cards, is already wrapped asking you at runtime
  whether you want to run that or not). Once you're done, make sure your device
  is connected, then start the script. Approve each backup call on the device
  (from the second call onwards, only do so after the previous call is finished,
  or ADB might choke). When the script is finished, you will find your `.ab`
  backup files in the corresponding directories.
* `partBackup`: this scipt creates images of all your device's partitions.
  As with the ADB backups, check the script and see if you need them all;
  I've tried my best to give the images „speaking names“ – but the reality will
  only be as good as your device's system permits.

### Restore
As with the backup, we again have multiple scripts here:

* `sysrestore`/`userrestore`: counterpart to the corresponding backup scripts.
  Deal with them the same way: comment out what you don't want to restore, then
  run the script.
* `disable`: disable the apps which have been disabled (frozen) at the time the
  backup was created
* `deadReceivers.sh`: similar, but for app components. Use this with care.
* `conf/*`: I'm not sure whether those files should be restored directly. You
  certainly can do so with the `wpa_supplicant.conf` file (to have all your
  configured WiFi APs back), same with the `hosts` and `gps.conf` files. But
  you for sure should *not* do so with the `packages.xml` and `build.prop`,
  which are more intended for reference.

You might have noticed there's no counterpart for the partition backup. That's
intentional. If you don't know what to do with those, better read on it first.
Which image belongs to which partition you can check in the corresponding backup
script.

### Documentation
The files in the `doc/` directory are your device's documentation, using
[Markdown] format. Please see the linked Wikipedia page for details. There
are several viewers/editors/convertors able to deal with this format – and
even without any, they should be easy to read in any text viewer/editor.


[Markdown]: https://en.wikipedia.org/wiki/Markdown "Wikipedia: Markdown"
