---
layout:     post
title:      "【Android】在Ubuntu 18.04上建置AOSP編譯環境"
subtitle:   "Build AOSP system on Ubuntu 18.04"
date:       2019-04-18 18:31:12
author:     "Tabaco"
header-img: "img/in-post/post-build-aosp-on-ubuntu/post-bg-build-aosp-on-ubuntu.png"
header-mask: 0.3
catalog:    true
tags:
    - Android
    - AOSP
    - Ubuntu
---

## AOSP(Android Open Source Project)
[AOSP(Android Open Source Project)][AOSP]是由Google推出的Android系統原始碼，裡面包括手機硬體控制及Android SDK的原始碼等，從官網下載原始碼編譯後的Android系統可以安裝到Nexus系列的手機，雖然官網有建立環境跟編譯的說明，但資料比較分散，因此這篇就來整理一下，順便作個記錄(つ´ω`)つ。

## Ubuntu 18.04環境建置
由於整個原始碼非常龐大，加上編譯之後資料夾大小會超過100GB，因此如果使用虛擬主機的話盡量將硬碟空間超過200GB比較保險，假如要編譯多個不同版本的AOSP，那硬碟空間就要更多才行。
1. 安裝套件
```note
$ sudo apt update
$ sudo apt install git git-core gnupg flex bison gperf build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z-dev libgl1-mesa-dev libxml2-utils xsltproc unzip
```

2. 安裝JDK
根據要編譯的AOSP版本不同，所需要的JDK版本也要跟著變換。
這篇以編譯Android 7.0以上的系統作為範例，因此需要安裝openJDK 8。
```note
$ sudo apt install openjdk-8-jdk
```
> * Android 7.0 (Nougat) - Android 8.0 (Oreo): Ubuntu - [OpenJDK 8][openJDK], Mac OS - [jdk 8u45 or newer][jdk-8u45]
> * Android 5.x (Lollipop) - Android 6.0 (Marshmallow): Ubuntu - [OpenJDK 7][openJDK], Mac OS - [jdk-7u71-macosx-x64.dmg][jdk-7u71]
> * Android 2.3.x (Gingerbread) - Android 4.4.x (KitKat): Ubuntu - [Java JDK 6][JDK6], Mac OS - [Java JDK 6][MacJDK6]
> * Android 1.5 (Cupcake) - Android 2.2.x (Froyo): Ubuntu - [Java JDK 5][JDK6]

## 下載AOSP源始碼
1. 初始化repo
```note
$ mkdir ~/bin
$ echo 'export PATH=~/bin:$PATH' >> ~/.bashrc
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```
2. 設定git
```note
$ git config --global user.name "Your Name"
$ git config --global user.email "you@example.com"
```
3. 選擇AOSP版本
```
```
到 [Source Code Tags and Builds][SourceTag] 選擇要下載的AOSP版本，測試手機是Nexus 5X，版本為7.1.1，所以選擇android-7.1.1_r24來編譯。
![](/img/in-post/post-build-aosp-on-ubuntu/android_tag.png)
```note
$ mkdir android-7.1.1_r24
$ cd android-7.1.1_r24
$ repo init -u https://android.googlesource.com/platform/manifest -b android-7.1.1_r24
```
4. 開始下載
下載時間取決於你的網路速度，但都需要好幾個小時(´_ゝ`)
```note
$ repo sync
```

## 下載Driver
如果要將AOSP系統flash到實體手機上，需要下載對應的driver作安裝[(Driver)][driver]，不安裝的話也可以flash上實體手機，只是會無法偵測到SIM卡。
本篇測試手機為Nexus 5X，AOSP版本為android-7.1.1_r24，所以要下載這個[Driver][N4F26T]。

## 編譯AOSP
開始編譯之前要選擇你的 [device Code Name][deviceCodeName] ，如Nexus 5X的Code Name為```bullhead```。
AOSP系統可選擇三種系統格式，分別是user、userdebug及eng，詳細可以參考 [Target][Target]。
```note
$ cd android-7.1.1_r24
$ source build/envsetup.sh
$ lunch aosp_bullhead-user
```
以上設定完沒有出錯就可以開始編譯AOSP系統，其中j```N```，N代表要使用多少個CPU核心作編譯。
```note
$ make -j4
```

###### 錯誤解決方法
1. LC_ALL
```note
FAILED: out/target/product/fugu/obj/STATIC_LIBRARIES/libedify_intermediates/lexer.cpp
/bin/bash -c “prebuilts/misc/linux-x86/flex/flex-2.5.39 -oout/target/product/fugu/obj/STATIC_LIBRARIES/libedify_intermediates/lexer.cpp bootable/recovery/edify/lexer.ll”
flex-2.5.39: loadlocale.c:130:_nl_intern_locale_data: ?? ‘cnt < (sizeof (_nl_value_type_LC_TIME) / sizeof (_nl_value_type_LC_TIME[0]))’ ???
Aborted (core dumped)
[  6% 3452/56388] //frameworks/base/libs/androidfw:libandroidfw clang ResourceTypes.cpp [linux]
ninja: build stopped: subcommand failed.
09:56:41 ninja failed with: exit status 1
```
去除所有本地化的設定，讓指令能正確執行：
```note
$ export LC_ALL=C
```
2. jack-server
```note
[ 45% 16221/35670] Building with Jack:...k_intermediates/with-local/classes.dex
FAILED: /bin/bash out/target/common/obj/JAVA_LIBRARIES/framework_intermediates/with-local/classes.dex.rsp
Out of memory error (version 1.2-rc4 'Carnac' (298900 f95d7bdecfceb327f9d201a1348397ed8a843843 by android-jack-team@google.com)).
GC overhead limit exceeded.
Try increasing heap size with java option '-Xmx'.
Warning: This may have produced partial or corrupted output.
[ 45% 16221/35670] Building with Jack:...colorpicker_intermediates/classes.jack
ninja: build stopped: subcommand failed.
build/core/ninja.mk:148: recipe for target 'ninja_wrapper' failed
make: *** [ninja_wrapper] Error 1
#### make failed to build some targets (01:26:54 (hh:mm:ss)) ####
```
修改```~/.jack-settings```，新增一行下列的指令
```note
JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4096m"
```
接著重啟jack-server
```note
$ prebuilts/sdk/tools/jack-admin kill-server
$ prebuilts/sdk/tools/jack-admin start-server
```

## Flashing a device
在重新flash Android系統前需要將手機的bootloader解鎖，可以參考 [Unlocking the bootloader][Unlocking the bootloader]。
1. fastboot
```
```
將bootloader解鎖後回到fastboot模式。
```note
$ adb reboot bootloader
```
2. flash AOSP
```note
$ fastboot flashall -w
```
最後重開機進Android畫面就代表成功更換成AOSP系統了！

[AOSP]: https://source.android.com/
[openJDK]: http://openjdk.java.net/install/
[jdk-8u45]: http://www.oracle.com/technetwork/java/javase/downloads/java-archive-javase8-2177648.html#jdk-8u45-oth-JPR
[jdk-7u71]: https://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html#jdk-7u71-oth-JPR
[JDK6]: https://www.oracle.com/technetwork/java/javase/archive-139210.html
[MacJDK6]: http://support.apple.com/kb/dl1572
[SourceTag]: https://source.android.com/setup/start/build-numbers.html#source-code-tags-and-builds
[deviceCodeName]: https://source.android.com/setup/build/running#selecting-device-build
[Target]: https://source.android.com/setup/build/building.html#choose-a-target
[Unlocking the bootloader]: https://source.android.com/setup/build/running#unlocking-the-bootloader
[driver]: https://developers.google.com/android/drivers
[N4F26T]: https://developers.google.com/android/drivers#bullheadn4f26t