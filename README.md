> [!CAUTION]
> **免责声明**
>
> 本文档仅供学习和研究目的。对系统进行降级操作存在使设备变砖、数据丢失、保修失效等风险。作者不对因使用本文档内容而导致的任何损害或损失承担责任。请在操作前备份所有重要数据。

> [!WARNING]
> **风险警告**
>
> - 降级操作可能导致设备**永久损坏（变砖）**，且无法恢复。
> - 此操作将**清除设备上的所有数据**，请务必提前备份。
> - 此操作可能导致设备**保修失效**。
> - 请确保操作过程中设备电量充足，且不要中断刷机过程。
> - 如果您不了解相关风险或不具备相关知识，请**不要**进行此操作。

# Xiaomi 13 免解锁 BL 系统降级

## 准备工作
1. [下载](https://developer.android.com/tools/releases/platform-tools) Android SDK Platform Tools(adb, fastboot, etc.) 
2. [下载]()小米手机驱动并安装
3. [下载]()工程包
4. [下载](https://xiaomirom.com)目标降级包，例如 14.0.5

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
3. 刷入工程包（解压缩工程包，在命令行中执行 `flash_all.bat`）
4. 刷入目标降级包（解压缩降级包，在命令行中执行 `flash_all.bat`）
