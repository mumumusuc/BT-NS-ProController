# Bluetooth Joycon

## Android(Need Root?也许不需要，需验证)

*需要Android9-Pie，tested on my Nexus5x with PixelExperience9.0*

1. 设置蓝牙PID、VID
    ```
    // remount /system as "rw"
    # adb shell
    bullhead:$ su
    bullhead:# mount -o remount,rw /system
    
    // pull file
    bullhead:# cp /etc/bluetooth/bt_did.conf /sdcard
    # adb pull /sdcard/bt_did.conf .
    
    // modify file
    @ bt_did.conf
    - productId = 0x1200
    + productId = 0x2009
    + vendorId = 0x057E
    
    // push file
    # adb push bt_did.conf /sdcard
    bullhead:# cp /sdcard/bt_did.conf /etc/bluetooth/ 
    ```
    
2. 安装Xposed

    Android9.0可用的[Xposed](https://github.com/ElderDrivers/EdXposed)。
    
    因为这个项目会使用Xposed禁用除了Hid-Host之外的所有蓝牙服务，所以若不能安装Xposed框架则需修改编译你的系统源码。源码相关修改将在下节进行说明。
    
    
3. NFC功能&蓝牙服务

    因为Pro手柄的nfc数据需要362字节的InputReport，而在AOSP9.0-Pie的蓝牙源码中将HID_DEV的Report数据大小设置为了64字节。如果需要实现这一功能，你需要修改编译AOSP9.0-Pie源码。
    
    *建议修改device项目，而非直接修改AOSP代码*
    
    ```
    @ device/lge/bullhead/bluetooth/conf/bt_did.conf
    - productId = 0x1200
    + productId = 0x2009
    + vendorId = 0x057E
    
    @ device/lge/bullhead/bluetooth/bdroid_buildcfg.h
    + #define HID_DEV_MTU_SIZE 512 // 362+ is OK
    
    @ device/lge/bullhead/aosp_bullhead.mk
    + PRODUCT_COPY_FILES += device/lge/bullhead/bluetooth/conf/bt_did.conf:system/etc/bluetooth/bt_did.conf
    
    // build Android
    # . build/envsetup.sh
    # lunch aosp_bullhead-userdebug
    # mka bacon -j8
    ```
    
    **(也许不需要，需验证)**
    
    若之前不能安装Xposed框架，则需要修改如下内容(这会影响你手机蓝牙的正常使用，请谨慎)：
    ```
    @ packages/apps/Bluetooth/res/values/config.xml
    - <bool name="profile_supported_a2dp">true</bool>
    - <bool name="profile_supported_a2dp_sink">false</bool>
    - <bool name="profile_supported_hdp">true</bool>
    - <bool name="profile_supported_hs_hfp">true</bool>
    - <bool name="profile_supported_hfpclient">false</bool>
    - <bool name="profile_supported_hid_host">true</bool>
    - <bool name="profile_supported_opp">true</bool>
    - <bool name="profile_supported_pan">true</bool>
    - <bool name="profile_supported_pbap">true</bool>
    - <bool name="profile_supported_gatt">true</bool>
    - <bool name="pbap_include_photos_in_vcard">true</bool>
    - <bool name="pbap_use_profile_for_owner_vcard">true</bool>
    - <bool name="profile_supported_map">true</bool>
    - <bool name="profile_supported_avrcp_target">true</bool>
    - <bool name="profile_supported_avrcp_controller">false</bool>
    - <bool name="profile_supported_sap">false</bool>
    - <bool name="profile_supported_pbapclient">false</bool>
    - <bool name="profile_supported_mapmce">false</bool>
    - <bool name="profile_supported_hid_device">true</bool>
    - <bool name="profile_supported_hearing_aid">false</bool>
    + <bool name="profile_supported_a2dp">false</bool>
    + <bool name="profile_supported_a2dp_sink">false</bool>
    + <bool name="profile_supported_hdp">false</bool>
    + <bool name="profile_supported_hs_hfp">false</bool>
    + <bool name="profile_supported_hfpclient">false</bool>
    + <bool name="profile_supported_hid_host">true</bool>
    + <bool name="profile_supported_opp">false</bool>
    + <bool name="profile_supported_pan">false</bool>
    + <bool name="profile_supported_pbap">false</bool>
    + <bool name="profile_supported_gatt">false</bool>
    + <bool name="pbap_include_photos_in_vcard">false</bool>
    + <bool name="pbap_use_profile_for_owner_vcard">false</bool>
    + <bool name="profile_supported_map">false</bool>
    + <bool name="profile_supported_avrcp_target">false</bool>
    + <bool name="profile_supported_avrcp_controller">false</bool>
    + <bool name="profile_supported_sap">false</bool>
    + <bool name="profile_supported_pbapclient">false</bool>
    + <bool name="profile_supported_mapmce">false</bool>
    + <bool name="profile_supported_hid_device">true</bool>
    + <bool name="profile_supported_hearing_aid">false</bool>
    ```
    

## Linux

[bluez-ns-controller](https://github.com/mumumusuc/bluez-ns-controller)
