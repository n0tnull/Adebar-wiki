## Requirements
* **ADB** installed (and configured for your device) on your computer. This can either be the [complete Android SDK](https://developer.android.com/sdk/index.html "Android SDK at Android Developers"), or a [minimal installation of ADB](http://android.stackexchange.com/q/42474/16575 "Android.SE: Is there a minimal installation of ADB?").
* **Bash** (version 4 or higher). As this is a very common shell environment, it's available by default on most Linux distributions. If you're a Windows user: sorry, the only windows I have are for light and fresh air.
* **Android 4.0+**: As the `adb backup` and `adb restore` commands have not been present before Android 4.0, *Adebar* will not be of much use with devices running older versions – except for, maybe, creating a „device documentation“.

I have not tested *Adebar* with Android versions before 4.0. It might work, though, for [[device documentation|example deviceInfo.md]] and [[list of installed apps|example userApps.md]], and maybe even some more. Though it might as well create the backup and restore scripts, those will be of no use – as `adb backup` and `adb restore` have only be added with Android 4.0.


## Obtaining *Adebar*
There are multiple ways to get hold of a copy of the code:

* you can clone the Git repository (see the [main page](https://github.com/IzzySoft/Adebar "Adebar at Github"))
* you can download the repository code as `.zip` file (from the same place)
* you can download the latest release from the [IzzyOnDroid Download Area](http://android.izzysoft.de/downloads "IzzyOnDroid Download Area")

The `master` branch here at Github should always reflect the latest "stable" code. The `devel` branch might hold newer stuff which was not yet much tested. The choice is entirely yours :)


## Installation
Just unpack the contents of the downloaded `.zip`/`.tar.gz` file into an empty directory of your chosing, no special installation required. If you cloned the repo, you can run `adebar-cli` directly from within your clone, and also create your `config/` and `cache/` directories there (these are contained in the `.gitignore` file, and thus should cause no conflicts).