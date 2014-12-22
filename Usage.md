## General Usage
`adebar-cli` requires at least one parameter, which will be used to

* detect a matching (device-specific) configuration file (see [[Configuration]])
* define the desired output-directory (residing below the configured
  `STORAGE_BASE`)

You can pass it an optional second parameter, which will then be appended to the
output directory name. This might especially useful if you want to keep scripts
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


## Exit codes
* 1: Syntax error (missing or wrong arguments when calling the script)
* 2: No device found
* 3: Multiple devices found, but no SERIAL specified in config
* 4: SERIAL specified, but no matching device connected
* 5: target directory doesn't exist, and neither can be created
