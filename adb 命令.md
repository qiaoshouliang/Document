### adb shell procrank 产看内存
```
qiaoshouliangdeMacBook-Pro:~ qiaoshouliang$ adb shell procrank
  PID       Vss      Rss      Pss      Uss  cmdline
  626   442480K   51220K   26367K   23136K  com.android.systemui
  816   461616K   46436K   22241K   18996K  com.dnake.desktop
  564   494264K   44776K   19629K   15872K  system_server
  134   394504K   43204K   12922K    6976K  zygote
 1238   451920K   37300K   12260K    8556K  com.android.settings
  731   451776K   37180K   11602K    8164K  com.android.phone
  713   419972K   31852K    7966K    5176K  com.google.android.inputmethod.pinyin
  850   408404K   27592K    6748K    4636K  android.process.acore
  700   407272K   27932K    6550K    4500K  android.process.media
  786   408204K   24736K    4685K    2892K  com.dnake.smart
  757   406128K   24640K    4579K    2788K  com.dnake.security
  771   408188K   24036K    4198K    2624K  com.dnake.talk
  801   407500K   23692K    4045K    2504K  com.dnake.eSettings
  136    37032K    7692K    4037K    3212K  /system/bin/mediaserver
  515    61752K    9340K    3895K    2420K  /var/bin/dnake_talk
  744   406040K   23184K    3689K    2168K  com.dnake.apps
  994   403952K   22664K    3580K    1848K  com.dnake.d400
  552    43888K    8740K    3359K    1888K  /var/bin/dnake_media
  135    12552K    5028K    2763K    2416K  /system/bin/drmserver
 1450     2424K    1908K    1675K    1668K  procrank
  133    57656K    3380K    1504K    1188K  /system/bin/surfaceflinger
  138     3944K    1264K     603K     552K  /system/bin/keystore
  131     9752K    1244K     565K     488K  /system/bin/netd
  130     4676K    1144K     491K     424K  /system/bin/vold
    1      668K     528K     429K     356K  /init
  526     7300K     620K     351K     336K  /var/bin/dnake_control
  185     1116K     608K     339K     328K  logcat
  140     3524K     504K     229K     220K  /system/bin/sdcard
   69      584K     304K     228K     156K  /sbin/ueventd
  141     5620K     232K     220K     220K  /sbin/adbd
  491     1004K     456K     190K     180K  /var/bin/dnake_nvs
  137      996K     460K     167K     156K  /system/bin/installd
  492     1384K     200K     157K     156K  /var/bin/monitor
  132     1032K     420K     155K     144K  /system/bin/debuggerd
  490      904K     276K     146K     140K  /var/bin/mini_httpd
  562     1436K     184K     141K     140K  /var/bin/upgrade
  127     1424K     144K     140K     140K  /sbin/healthd
  153     1912K     132K     132K     132K  /system/bin/busybox
  129     1004K     336K     119K     112K  /system/bin/servicemanager
  561      856K     352K     106K      96K  logwrapper
  514      856K     352K     106K      96K  logwrapper
  525      856K     352K     106K      96K  logwrapper
  550      856K     348K     102K      92K  logwrapper
                           ------   ------  ------
                          173532K  128388K  TOTAL

RAM: 425876K total, 52636K free, 3488K buffers, 164656K cached, 324K shmem, 38128K slab
```
其中total指的就是系统的内存大小，512M


### adb shell wm size 获取屏幕分辨率

```
qiaoshouliangdeMacBook-Pro:~ qiaoshouliang$ adb shell wm size
Physical size: 1280x800
```
### adb shell am adb shell am start -n com.hismart.intercom/.StartupActivity --ei START_INTENT 1000

启动一个Acitivy with extras