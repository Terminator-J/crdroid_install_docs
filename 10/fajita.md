_Starting in crDroid 10.x (Android 14.0) and onwards, we're using retrofitted dynamic partitions to avoid the constant threat of running out of space in the GPT system partition with any sizeable Google Apps package. Due to that, these installation steps might look different than what you're used to for this device._

_Please do not panic. Resist the temptation to read, or talk to loved ones._

_Have a cookie._

### Pre-Installation:
**Prerequisite phone & computer setup**

- Phone:
  - Ensure you're running with OxygenOS 11.1.2.2 firmware in your current slot. This doesn't mean you need to be running OxygenOS itself; it is not required to go back to stock first. This only means that the firmware-specific partitions have been updated to the last-available contents from OxygenOS 11.1.2.2, at some point in time since the last time you ran MSMTool or a similar "return to stock" fastboot ROM zip. Instructions for how to do this are beyond the scope of this guide, but you can always use MSMTool to go back to stock OxygenOS with locked bootloader and start updating via OTA until you're up to date, or follow the firmware updating guides for fajita from the LineageOS wiki **[here](https://wiki.lineageos.org/devices/fajita/fw_update/)**.

    If needed, you can download the correct OxygenOS 11.1.2.2 zip for fajita **[here](https://oxygenos.oneplus.net/OnePlus6TOxygen_34.J.62_OTA_0620_all_2111252336_f6eda340d7af4e3e.zip)**.

  - You will need an unlocked bootloader (see Funk Wizard's guides in XDA Forums for instructions if you need help to do this), and the understanding that crDroid does not support re-locking the bootloader after installation on this device.

- Computer:
  - You need a computer with current AOSP "platform-tools" installed or otherwise on your $PATH (the current official ones from developer.android.com, not some script-kiddie's sketchy all-in-one bundle or your distro's packages that are two years out of date), OnePlus USB drivers installed (works for adb and OEM bootloader `fastboot` commands), and Google's universal ADB drivers installed (needed for fastbootd mode within recovery). Linux and MacOS users: when it comes to drivers, may the odds be ever in your favor!

    If you do not have current platform-tools installed, get it from the official site here:

    **[platform-tools](https://developer.android.com/studio/releases/platform-tools)**

  - Windows-specific:

    If you do not have the OnePlus USB drivers for 6-series, get the installer here:

    **[OnePlus USB drivers](https://drive.google.com/file/d/1L7EZGx5mgeQYXO19Vsp9FWu9GXhF45Qs/view)**

    If you do not have the generic Google USB drivers, download them here:

    **[Google USB drivers](https://developer.android.com/studio/run/win-usb)**

    Installing Windows Terminal and PowerShell 7.x are highly recommended for the best experience, although these commands should also work from the included PowerShell versions that have shipped from Windows 7 onwards.

**Required files**

- crDroid files for your device can be found **[here](https://sourceforge.net/projects/crdroid/files/fajita/10.x/)**.

- You'll need a complete set of the following files:

  - `crDroidAndroid-xxx_boot.img`
  - `crDroidAndroid-xxx_dtbo.img`
  - `crDroidAndroid-xxx_super_empty.img`
  - `crDroidAndroid-xxx_vbmeta.img`
  - `crDroidAndroid-xxx.zip` (the actual ROM zip)

  Each release includes updated boot, dtbo, super_empty, and vbmeta img files. Make sure you get a matched set that all have the same version and date in the file name. Do not mix & match.

**Optional files**

- Recommended Google Apps:

  - MindTheGapps _should_ always be a good option (does not include PrivateCompute or SettingsIntelligence services):

    Download **[MindTheGapps 14.0.0-arm64](https://github.com/MindTheGapps/14.0.0-arm64/releases)**

  - NikGapps "crdroid-official" elite config by the dev team (slightly more Google integration):

    Download **[NikGapps recommended bundle](https://nikgapps.com/crdroid-official)**

- Recommended microG package (alternative to Google Apps/Play Services):

  - Shane's MinMicroG "Standard" package installer has been a historical favorite, due to its correct installer behavior in our recovery & addon.d OTA survival scripts working properly from booted System updates. Magisk module-based installation is not supported. Use the latest "Updately" versions for most up-to-date microG project components:

    Download **[MinMicroG](https://github.com/FriendlyNeighborhoodShane/MinMicroG-abuse-CI/releases)**

  Protip: Save a copy of the Google Apps or microG zip installer(s) somewhere safe for later. If you need to dirty flash from recovery at a later date, you'll need them!

- Magisk/crDroid++ Kernel for KernelSU support:

  - Wait until after first boot & finishing the Setup Wizard. Best to get a clean SafetyNet/Play Integrity attestation first, then reboot to recovery to flash root tools later on.

- Disable_Dm-Verity_ForceEncrypt:

  - _NO._

### First Time Installation (Clean Flash):
**Coming from any other ROM or major Android version**

**Notice that each command is prefixed with a `./`. This is important. Please keep that prefix when you run each command.**

_(unless you know how to add things to your $PATH and can verify which executable is running, of course)_

- Step 1: Unzip the `platform-tools-latest-xxx.zip` file you downloaded earlier to any directory you can run programs from.

- Step 2: Move the following files into the extracted platform-tools directory (so that they are in the same directory level as the `adb` and `fastboot` executable files), and rename the `.img` files to lose the `crDroidAndroid-xxx` prefixes:

  - `crDroidAndroid-xxx_boot.img` renamed to `boot.img`
  - `crDroidAndroid-xxx_dtbo.img` renamed to `dtbo.img`
  - `crDroidAndroid-xxx_super_empty.img` renamed to `super_empty.img`
  - `crDroidAndroid-xxx_vbmeta.img` renamed to `vbmeta.img`
  - the latest crDroid ROM zip for your device (`crDroidAndroid-xxx.zip`, do not rename)
  - (optional) the Google Apps or microG installer zip you decided on

- Step 3: Open your terminal and `cd` to the extracted platform-tools directory.

  - If you are on Windows, use PowerShell. Do **NOT** use Command Prompt.
  - If you are on macOS or Linux, then use your preferred terminal and shell.

- Step 4: Boot phone to the OEM bootloader.

  - Unplug USB cable from phone, power it off, then hold all three buttons (Vol Up + Vol Down + Power) until you see a screen with giant green text saying "Start" at the top of the screen.
  - Verify you have correctly installed OnePlus USB drivers by plugging the USB cable back in, and running `fastboot devices` from the terminal window. You should see your device's serial number in the list of devices attached. If not, try reinstalling drivers, try a different cable, or a different USB port, or a USB hub, etc until it works.

- Step 5: Run the following terminal command (yes it's one long line) which will flash the necessary partitions to boot crDroid recovery successfully with needed retrofit dynamic partition support for ROM install, and prepare GPT partitions that will become the backing store for super partition:

  ```
  ./fastboot flash boot_a boot.img && ./fastboot flash boot_b boot.img && ./fastboot flash dtbo_a dtbo.img && ./fastboot flash dtbo_b dtbo.img && ./fastboot flash vbmeta_a vbmeta.img && ./fastboot flash vbmeta_b vbmeta.img && ./fastboot erase system_a && ./fastboot erase system_b && ./fastboot erase odm_a && ./fastboot erase odm_b && ./fastboot erase vendor_a && ./fastboot erase vendor_b
  ```

- Step 6: Unplug the USB cable & reboot phone into recovery.

  - Use the volume buttons to page through the boot options until you see `Recovery mode`. Press the power button to select that option.

- Step 7: In recovery, choose `Advanced` then `Enter fastboot` to enter fastbootd ("userspace fastboot"), and plug the USB cable back into phone.

  - Run `fastboot devices` in terminal to make sure that the userspace fastbootd drivers are working. If not, unplug & re-plug the USB cable again or check for driver updates via Device Manager or Windows Update.

- Step 8: Run the following terminal command (initializes the retrofit super partitions with metadata for this ROM's configuration):

  ```
  ./fastboot --slot a wipe-super super_empty.img && ./fastboot --slot b wipe-super super_empty.img
  ```

  WARNING: If fastboot on the computer returns some error message about not recognizing `wipe-super` or prints a long help message instead, then that means you are either running a very old version of the fastboot executable or the phone was still in OEM bootloader instead of recovery-based fastbootd. Please go back and download the latest version of platform-tools as mentioned in the "Required files" section, and/or figure out your driver situation so that you can send fastboot commands while the phone is in "Enter fastboot" mode from within crDroid Recovery. This is NOT optional.

- Step 9: Choose `Enter recovery` on phone to return to recovery, then choose `Apply update` then `Apply from ADB` so that it is waiting to receive an installation package from the computer.

- Step 10: Run the following terminal command to sideload crDroid Android, substituting the actual zip file name:

  ```
  ./adb sideload crDroidAndroid-xxx.zip
  ```

  - During a successful sideload, the process will be paused at 47% on the computer, and on the phone you'll be prompted to reboot to recovery again in order to continue flashing Addons (like Google Apps or microG, since those are not built into crDroid). Tap on "Yes" to reboot into recovery (while `adb sideload` process on the computer will complete at 47%, possibly with an error message about "no error", and give you a prompt again. That's "normal", roll with it).

  If you encounter any other installation errors when sideloading besides ROM zip sideloading stopping at 47%, run `./adb pull /tmp/recovery.log` in terminal to get logs to share while asking for help.

- Step 11: Back in recovery, choose `Apply update` then `Apply from ADB` again.

- Step 12: Run the following terminal command to sideload crDroid Android to the other slot, again substituting the actual zip file name:

  ```
  ./adb sideload crDroidAndroid-xxx.zip
  ```

  - Choose "Yes" again on phone when prompted to reboot to recovery.

- Step 13: If you want to run with Google Apps or microG since we don't ship them built-in, now is the time to install. On the phone, choose `Apply update` then `Apply from ADB` again, and from the terminal window on your computer, run the following (substituting the actual zip file name):

  ```
  ./adb sideload <Google Apps or microG package file name>.zip
  ```

- Step 14: Since this is a clean install, navigate back to top level recovery menu on phone, and choose the `Factory reset` option & confirm in order to format the data partition & set up FBE encryption keys.

- Step 15: Choose `Reboot system now` and remember: **𓂀 We love you 3000!**

- Step 16 (Optional): Once you've completed initial installation & first boot Setup Wizard, it's a good time to reboot to recovery and adb sideload the KernelSU-enabled crDroid++ kernel zip or Magisk if you want to be rooted.

### Update Installation:
**Updating to a newer release of the same major Android version**

**Option 1: From OTA Updater**

_Processes addon.d OTA survival scripts, so addons such as Google Apps / microG / Magisk that install survival scripts to `/system/addon.d/` should be preserved automatically_

- Step 1: Open `Settings > System > System Updates` on the phone. Before loading an update to install, choose `Preferences` from the menu and turn on the `Prioritize update process` and `Delete updates when installed` toggles.

  Protip: Use the "Caffeine" QS tile to keep the screen on for at least 10 minutes, or temporarily enable "Stay awake while charging" in Developer Options, or just temporarily set the Display timeout to 10 minutes to speed up the process. Installation slows down drastically if the screen turns off.

- Step 2: Now either check for updates, or choose `Local update` from the menu & select the desired ROM zip file from your phone's internal storage. Once the installer has verified the update, choose `Install` to begin installation.

- Step 3: Reboot once installation has completed. If the screen has turned off or you navigated away from the `System updates` activity to use another app, then the only indication that updating has completed will be that the notification with progress bar has disappeared. Don't panic; just reboot the phone and you should see that the update has been applied.

- Step 4 (Optional): If using the crDroid++ kernel, reboot to recovery and adb sideload the zip that corresponds to the version of the ROM zip you just installed. This will never be automated with an addon.d OTA survival script. Since KernelSU support has to be compiled into the kernel image itself, including a script that restores an old version of the kernel image would defeat the whole point of updating.

**Option 2: From Recovery**

_Does NOT process OTA survival scripts, so you must manually re-flash installer zips for any addons you need, such as Google Apps_

- Step 1: Make sure you have the **same Google Apps or microG installation zips** (or updated versions of the same Android version/architecture for you preferred package) as when you did the initial clean installation available to sideload. If you try to change to different GApps now, things will break.

- Step 2: Reboot phone to recovery and connect USB cable to computer once you're in the crDroid recovery environment. Open a terminal on the computer and run `adb devices` to verify that the phone is visible to the computer.

- Step 3: On the phone, choose `Apply update` then `Apply from ADB` to begin the phone listening for an update. On the computer, run `./adb sideload path/to/crdroid.zip` (substituting the correct directory path & zip file name).

- Step 4: On the computer, the process will pause at 47%, and on the phone you'll see it's prompting you to reboot to recovery to install addons (like Google Apps). Choose `Yes` on the phone to reboot to recovery now, and notice that the adb sideload process has finished on the computer at 47%, possibly with an error message about "no error". This is "normal". _Thanks, Google!_

  DO NOT SKIP REBOOTING TO RECOVERY TO INSTALL ADDONS. This is due to how slot-switching works.

- Step 5: If you have any addons to flash (Google Apps, microG, custom kernel zips, etc) now is the time to sideload them. On the phone, choose `Apply update` then `Apply from ADB` to start the phone listening, and then run `./adb sideload path/to/installation.zip` from the computer for each thing you need to install.

  Again: Recovery does **not** process addon.d OTA survival scripts like the regular OTA updater in `Settings` does, so you CANNOT skip installing the Google Apps zip now if you previously had them installed, or it will ruin your current installation and you'll have to deal with either factory resetting or wiping runtime permissions.

- Step 6: On the phone, choose `Reboot system now` and remember: **𓂀 We love you 3000!**


_These instructions have been verified using both old/crusty and fresh installations of Windows 10, with both built-in PowerShell 1.x & 2.x and updated 7.x versions, on at least three different machines, including multiple combinations of AMD and Intel motherboards and various USB chipsets/hubs/ports/cables, as well as on a 2019 MacBook Pro (Intel) running MacOS Sonoma. If something isn't working right, then the problem is with your computer configuration, not the guide. We cannot fix your computer for you._

_(Guide adapted with permission from the Evolution-X installation guide originally by jabashque and AnierinBliss, with extensive adaptation & updating over the years by Terminator_J)._
