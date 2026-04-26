# Xiaomi 13 免解锁 BL 系统降级

## 准备工作
1. [下载](https://developer.android.com/tools/releases/platform-tools) Android SDK Platform Tools(adb, fastboot, etc.) 
2. [下载]()小米手机驱动并安装
3. [下载](https://gitee.com/geekflashtool)刷机匣
4. [下载]()工程包
5. [下载](https://xiaomirom.com)目标降级包，例如 14.0.5

## 降级步骤

1. 进入 fastboot 模式
```shell
adb reboot bootloader
```
2. 设置 selinux 宽容
```shell
fastboot oem set-gpu-preemption 0 androidboot.selinux=permissive
fastboot continue # 开机
```
2. 刷入工程包 abl 文件
```shell
adb push abl.elf /data/local/tmp/abl
adb shell service call miui.mqsas.IMQSNative 21 i32 1 s16 "dd" i32 1 s16 'if=/data/local/tmp/abl of=/dev/block/by-name/abl_a' s16 '/data/mqsas/log.txt' i32 60
adb shell service call miui.mqsas.IMQSNative 21 i32 1 s16 "dd" i32 1 s16 'if=/data/local/tmp/abl of=/dev/block/by-name/abl_b' s16 '/data/mqsas/log.txt' i32 60
adb reboot bootloader # 进入 fastboot 模式
```
3. 使用刷机匣刷入工程包（解压缩工程包，刷机匣选择 flash_all.bat 并执行命令）
4. 使用刷机匣刷入目标降级包（解压缩降级包，刷机匣选择 flash_all.bat 并执行命令）
