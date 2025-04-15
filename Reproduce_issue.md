# NVS erase after framework update

## Introduction
When updating from framework version 4.2 to 4.4.8, a weird issue is encountered: the partition labeled "nvs" gets erased and other partitions are not affected.

## How to reproduce
This directory holds two projects, both copied from the esp-idf examples directory. One from the v4.2 checkout and one from the v4.4.8 checkout. Open this top-level directory in Visual Studio Code (vscode) and you will be able to start two different containers to build the two projects.

1. Open in vscode
2. Run the first task:
    - F1 > Run task > Buildcontainer v4.2
3. The projects are configured to enable secure boot and flash encryption. To prevent your development-kit from encrypting, be sure to build and flash an unsecure bootloader first.
    - Run: ```idf.py menuconfig```
    - Go to: Security features
    - Disable "Enable hardware Secure Boot in bootloader"
    - Disable "Enable flash encryption on boot"
    - Save the configuration
    - Run: ```idf.py bootloader```
    - On successful build, connect your device, put in download mode and run: ```idf.py -p \<your-serial-device\> bootloader-flash```
    - Your ESP32 now contains only the unsecure bootloader and prevents the device from encryption when running an application that is built with the security features enabled.
4. Revert the changes to the sdkconfig
5. Rebuild the whole project by running: ```idf.py build```
6. Flash the appliction by running: ```idf.py -p <your-serial-device> flash```
7. After successful flash, run: ```idf.py -p <your-serial-device> monitor```
8. Powercycle the device and see the application print a restart counter every 10 seconds
9. Powercycle the device again and see the restart counter is still increasing and didn't say ```Reading restart counter from NVS ... The value is not initialized yet!```

Now comes the issue: when you update to newer framework, you do updates over the air and you only update your application, not the second-stage bootloader. This is what we mimic in the next part.

1. Run the second task:
    - F1 > Run task > Buildcontainer v4.4.8
2. Build the project by running: ```idf.py build```
3. Flash the appliction by running: ```idf.py -p <your-serial-device> flash```
    - (The bootloader will not be flashed due to the fact that security features are enabled, so this mimics an over-the-air update of the application)
4. After successful flash, run: ```idf.py -p <your-serial-device> monitor```
5. Powercycle the device and see the application prints ```Reading restart counter from NVS ... The value is not initialized yet!```. This means the partition labeled "nvs" is wiped.

This example does not include the proof that other partitions are not affected. If anyone is interested in that, just ask and we can add that to this example.
