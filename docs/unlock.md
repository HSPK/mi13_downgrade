> [!CAUTION]
> **免责声明**
>
> 本文档仅供学习和研究目的。解锁 Bootloader 及刷写分区表存在极高的变砖风险。作者不对因使用本文档内容而导致的任何损害或损失承担责任。请在操作前备份所有重要数据。

> [!WARNING]
> **风险警告**
>
> - 此操作**风险极高**，GPT 分区表重写失败将导致**硬砖**，且可能无法通过常规手段恢复。
> - 此操作将**清除设备上的所有数据**，请务必提前备份。
> - 此操作可能导致设备**保修失效**。
> - 请确保操作过程中设备电量充足（建议 > 50%），且不要中断刷机过程。
> - 如果您不了解相关风险或不具备相关知识，请**不要**进行此操作。

# Xiaomi 13 免官方渠道解锁 Bootloader

本方法利用与[免解锁降级](../README.md)相同的漏洞链（ABL cmdline 注入 + IMQSNative 提权）作为跳板，在此基础上进一步执行 BL 解锁、GPT 分区表重建和官方固件刷写，最终实现 **Bootloader 真正解锁**，绕过小米官方要求的账号绑定和 168 小时等待期。

> 漏洞原理详见 [exploit.md](exploit.md)。

---

## 与免解锁降级方案的区别

| | 免解锁降级（README） | BL 解锁（本文档） |
|---|---|---|
| **最终目标** | 系统降级，BL 保持锁定 | **真正解锁 Bootloader** |
| **BL 状态** | 🔒 锁定 | 🔓 解锁 |
| **步骤数** | 4 步 | 10 步 |
| **所需文件** | 工程包 + 目标降级包 | 降级小包 + 核心解锁文件 + 官方线刷包 |
| **GPT 分区表重写** | ❌ | ✅ |
| **变砖风险** | 中等 | 较高 |

---

## 准备工作

1. [下载](https://developer.android.com/tools/releases/platform-tools) Android SDK Platform Tools（adb、fastboot）
2. 下载并安装小米手机驱动
3. 下载对应机型的**降级小包**（包含 `abl` 文件、`flash_all.bat` 脚本及 `images/` 目录）
4. 下载**核心解锁文件**（包含用于执行 BL 解锁的 bat 脚本）
5. 从 [xiaomirom.com](https://xiaomirom.com) 下载与当前系统版本对应的**官方线刷包**

> [!NOTE]
> 不建议使用一键工具，请在命令行中手动逐条输入命令。

---

## 解锁步骤

### 第一阶段：漏洞利用 — 刷入工程 ABL

此阶段与免解锁降级方案完全相同，利用 ABL cmdline 注入 + IMQSNative 提权将工程 ABL 写入设备。

#### 1. 进入 fastboot 模式

```shell
adb reboot bootloader
```

或长按 **音量下 + 电源键** 进入。

#### 2. 注入 SELinux 宽容模式并启动系统

```shell
fastboot oem set-gpu-preemption 0 androidboot.selinux=permissive
fastboot continue
```

#### 3. 验证 SELinux 状态

```shell
adb shell getenforce
```

预期输出为 `Permissive`。如果返回 `Enforcing`，请重试步骤 1-2。

#### 4. 推送并写入工程 ABL

```shell
adb push <降级小包中abl文件路径> /data/local/tmp/abl
adb shell service call miui.mqsas.IMQSNative 21 i32 1 s16 "dd" i32 1 s16 'if=/data/local/tmp/abl of=/dev/block/by-name/abl_a' s16 '/data/mqsas/log.txt' i32 60
adb shell service call miui.mqsas.IMQSNative 21 i32 1 s16 "dd" i32 1 s16 'if=/data/local/tmp/abl of=/dev/block/by-name/abl_b' s16 '/data/mqsas/log.txt' i32 60
```

#### 5. 重启进入 fastboot

```shell
adb reboot bootloader
```

此时工程 ABL 已生效，Anti-Rollback 校验被禁用。

---

### 第二阶段：刷入降级小包

#### 6. 执行降级小包刷写脚本

在命令行中运行降级小包中的 `flash_all.bat`：

```shell
<降级小包路径>\flash_all.bat
```

等待运行完成。如果出现包含 `lock` 字样的报错，请重试第一阶段（从步骤 1 开始）。

#### 7. 重启进入 fastboot

刷写完成后设备可能进入 FTD 模式或黑屏。等待电脑识别到设备连接后输入：

```shell
fastboot reboot bootloader
```

如果持续黑屏，长按 **音量下 + 电源键** 手动进入 fastboot。

---

### 第三阶段：执行核心解锁

#### 8. 运行核心解锁脚本

在 fastboot 模式下，双击运行**核心解锁文件**中的 bat 脚本，等待运行完成。

> 此步骤是与免解锁降级方案的**关键分歧点**——核心解锁文件会执行真正的 BL unlock 操作。

#### 9. 重新进入 fastboot

运行完成后手机应处于白屏状态。长按 **音量下 + 电源键** 进入 fastboot。

---

### 第四阶段：重建 GPT 分区表 + 刷入降级小包

#### 10. 修改降级小包的 flash_all.bat

右键编辑降级小包中的 `flash_all.bat`，在脚本开头的刷写命令之前加入以下 GPT 分区表刷写命令：

```bat
fastboot %* flash partition:0 %~dp0images/gpt_both0.bin || @echo "Flash gpt_both0 error" && exit 1
fastboot %* flash partition:1 %~dp0images/gpt_both1.bin || @echo "Flash gpt_both1 error" && exit 1
fastboot %* flash partition:2 %~dp0images/gpt_both2.bin || @echo "Flash gpt_both2 error" && exit 1
fastboot %* flash partition:3 %~dp0images/gpt_both3.bin || @echo "Flash gpt_both3 error" && exit 1
fastboot %* flash partition:4 %~dp0images/gpt_both4.bin || @echo "Flash gpt_both4 error" && exit 1
fastboot %* flash partition:5 %~dp0images/gpt_both5.bin || @echo "Flash gpt_both5 error" && exit 1
```

> [!WARNING]
> GPT 分区表重写是**高风险操作**。`gpt_both0~5.bin` 文件必须与设备型号严格匹配，否则将导致硬砖。

#### 11. 运行修改后的刷写脚本

在命令行中运行修改后的 `flash_all.bat`，等待刷写完成。

---

### 第五阶段：刷入官方线刷包

#### 12. 进入 fastboot

刷写完成后设备进入 FTD 模式，输入：

```shell
fastboot reboot bootloader
```

#### 13. 刷入官方线刷包

在 fastboot 模式下，运行官方线刷包中的 `flash_all.bat`：

```shell
<官方线刷包路径>\flash_all.bat
```

等待刷写完成，设备即可正常开机。此时 Bootloader 已处于解锁状态。

---

## 完整流程总结

| 阶段 | 步骤 | 操作 | 目的 |
|------|------|------|------|
| 一 | 1-5 | SELinux 注入 + IMQSNative 写入工程 ABL | 绕过 Anti-Rollback |
| 二 | 6-7 | 刷入降级小包 | 将系统降级到可利用版本 |
| 三 | 8-9 | 运行核心解锁文件 | **真正解锁 Bootloader** |
| 四 | 10-11 | 重建 GPT 分区表 + 再次刷入降级小包 | 修复分区布局 |
| 五 | 12-13 | 刷入官方线刷包 | 在已解锁 BL 上恢复干净系统 |

---

## 常见问题

**Q: 刷入降级小包时报错包含 "lock" 字样？**
A: 说明 SELinux 未成功切换为 Permissive 或工程 ABL 未正确写入。请重试第一阶段。

**Q: 核心解锁后手机持续黑屏？**
A: 长按 **音量下 + 电源键** 强制进入 fastboot。

**Q: GPT 分区表刷写失败？**
A: 这是最危险的情况，可能导致硬砖。确认 `gpt_both*.bin` 文件来源正确且与设备型号匹配。

---

## 参考资料

- [Preempted: From ADB Service Call to Bootloader Unlock on Xiaomi](https://bestwing.me/preempted-unlocking-xiaomi-via-two-unsanitized-strings.html)
- [exploit.md — 漏洞原理分析](exploit.md)
- [README.md — 免解锁降级方案](../README.md)
