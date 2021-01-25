# Google Play on Akuvox c315

This how to describes installation of Google Play (and other basic Google services) on Akuvox C315. I have Akuvox C315W specifically, however I believe that below steps should work for all C31x series. 

As a base for installation I used minimal [Open GApps](https://opengapps.org/) version (i.e. pico). You can probably use different ones as well (not tested). It appeared that Akuvox recovery partition don't have option to install ZIP updates as required by Open GApps project. To workaround it I installed all apps manually following [this guide](https://tinkerboarding.co.uk/forum/thread-553.html). The guide describes steps on Linux machine. You should be able to use Windows or macOS as well after finding appropriate alternatives to commands from step 9 onwards.

Note that you will be required to dismantle monitor's case and that you are doing everything at your own risk. You've been warned so let's begin :)

1. Change permission admin to be able to access applications as described [here](http://wiki.akuvox.com/doku.php?id=indoor_monitor:feature_guides:how_to_install_apk_on_android_indoor_monitor)
2. In monitor's web config go to Phone->Key/Display and configure one of areas to run Setings
```
Type: Cusom APK
Value: com.android.settings
Label: Settings
```
3. Open android settings via configre area (to close it later you can kill it via "Applications" menu)
4. Go to "System Information" and tap "Model Number" until "Developer's Options" will be enabled
5. Go up and enter "Developer's Options" menu
6. Check if "USB Debugging" is enabled
7. Dismantle monitor case. Insert micro USB cable to your monitor and attach it to your computer. After confirming pop-up on monitor screen you should be able to use ADB with your Akuvox now.
8. Download prebuilt Open GApps [here](https://sourceforge.net/projects/opengapps/files/arm/). Architecture: ARM, Android Version: 6.0 (I used open_gapps-arm-6.0-pico-20210123.zip)
9. Prepare temporary directories
```
mkdir ~/gapps
mkdir ~/gapps/pkg
mkdir ~/gapps/tmp
mkdir ~/gapps/sys
```
10. Unzip the downloaded gapps archive with
```
unzip [name of gapps zip file].zip -d gapps/pkg
```
11. Enter gapps directory and extract all the relevant packages
```
find -name "*.tar.[g|l|x]z" -o -name "*.tar" | xargs -n 1 tar -xC tmp -f
```
12. Create script.sh in gapps directory
```
#!/bin/bash
for dir in tmp/*/
    do
      pkg=${dir%*/}
      dpi=$(ls -1 $pkg | head -1)

      echo "Preparing $pkg/$dpi"
      rsync -aq $pkg/$dpi/ sys/
    done
```
13. Execute the script
```
chmod +x script.sh
./script.sh
```
14. Push prepared binaries to monitor
```
adb root
adb remount
adb push sys/. /system/.
```
15. Remove second package installer and existing runtime permissions (permissions will be recreated after reboot)
```
adb shell
rm -rf /system/priv-app/PackageInstaller /data/system/users/0/runtime-permissions.xml
exit
```
16. Reboot the board and enjoy Google Play!
```
adb reboot
```

