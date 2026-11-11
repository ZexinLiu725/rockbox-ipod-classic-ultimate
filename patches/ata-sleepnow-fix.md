# ATA `ata_sleepnow()` 断电等待修复：GitHub 发布草稿

**状态 / Status:** 待用户审稿 / Awaiting user review; not committed or published.  
**建议形式 / Recommended format: 单独提交说明 / standalone commit message.** 此改动集中在一个驱动函数和一条设备路径，适合独立审阅与摘取；单独提交也不会让读者误以为整个三线整合候选已经发布。 / The change is confined to one driver function and one device path, so it can be reviewed and picked independently. A standalone commit also avoids implying that the full three-line integration candidate has been released.

## 可直接复制的提交说明

```text
ATA: wait after cache flush before power-down

On non-CE-ATA devices that cannot enter ATA standby but support cache flush,
ata_sleepnow() flushes the cache and then reaches the existing power-down call
without an added settling delay. On affected devices, cached data may not have
finished reaching the medium, risking FAT corruption during sleep or shutdown.

Wait HZ/2 after the flush in the !ceata && !ata_disk_can_sleep() && canflush
path before the existing ata_power_down() call. Standby-capable devices and
CE-ATA keep their existing paths.

The author's Kingspec ZIF SSD takes the standby path and is not affected by
this case.

[中文]
ATA：缓存刷新后延迟断电

对于不支持 ATA standby、但支持缓存刷新的非 CE-ATA 设备，ata_sleepnow()
刷新缓存后会直接进入原有断电调用，没有额外的沉降等待。部分受影响设备的
缓存数据可能尚未完成写入介质；休眠或关机时过早断电可能造成 FAT 损坏。

在 !ceata && !ata_disk_can_sleep() && canflush 路径中，缓存刷新后、原有
ata_power_down() 调用前等待 HZ/2。支持 standby 的设备和 CE-ATA 保持原路径。

作者本机的 Kingspec ZIF SSD 走 standby 路径，不属于此次修复覆盖的情形。
```

## 修复说明与适用范围 / Fix and scope

- **Mechanism:** `ata_sleepnow()` flushes the ATA cache first. If a device cannot enter ATA standby but supports cache flush, the existing code proceeds to `ata_power_down()` without an added settling delay. On affected devices, power may be removed before cached data has finished reaching the medium, risking FAT corruption.
- **Scope:** The added wait runs only for non-CE-ATA devices where `ata_disk_can_sleep()` is false and `canflush` is true. Devices that can neither enter standby nor flush do not enter the new branch; the patch does not change their existing power-down condition.
- **Implementation:** In `firmware/target/arm/s5l8702/ipod6g/storage_ata-6g.c`, `ata_sleepnow()` adds `else if (canflush) { sleep(HZ / 2); }` after cache-flush handling and before the existing power-down call. The standby-capable and CE-ATA paths remain unchanged.
- **Author's device:** The author's Kingspec ZIF SSD follows the existing standby path and is outside the affected case. The fix is intended to benefit users of devices that do not support ATA power management; it should not be described as fixing a problem reproduced on the author's SSD.

- **机理：** `ata_sleepnow()` 先刷新 ATA 缓存。若设备不支持 ATA standby，原代码不会进入 standby 命令分支；当缓存刷新可用时，随后仍会调用 `ata_power_down()`，中间没有额外等待。对受影响设备，这个时序可能在设备完成缓存数据写入介质前切断供电，导致 FAT 损坏。
- **覆盖范围：** 新等待只在非 CE-ATA、`ata_disk_can_sleep()` 为假且 `canflush` 为真的路径生效，即不支持 ATA standby 但支持缓存刷新的设备。不能 standby 且也不能 flush 的设备不会进入新分支，也不因本补丁改变原断电条件。
- **原理与代码位置：** 在 `firmware/target/arm/s5l8702/ipod6g/storage_ata-6g.c` 的 `ata_sleepnow()` 中，缓存刷新处理后新增 `else if (canflush) { sleep(HZ / 2); }`，再执行原有断电调用。可 standby 设备的命令与等待、CE-ATA 路径均保持不变。
- **作者设备：** 作者本机 Kingspec ZIF SSD 走既有 standby 分支，不受此缺陷路径影响。该修复的价值在于覆盖不支持 ATA 电源管理设备的用户，不应表述为修复作者本人 SSD 上已复现的问题。

## 影响与风险 / Impact and risks

- The change is limited to the device path above. The added `HZ / 2` wait is about half a second, so sleep or shutdown may take correspondingly longer on that path.
- The patch does not change the cache-flush command, standby command, or existing power-down decision. Device write-settling times may vary; current evidence does not establish that this delay suits every device, and the patch should not be presented as eliminating all FAT-corruption risk.

- 影响局限于上述设备路径；新增等待为 `HZ / 2`（约半秒），因此该路径上的休眠或关机可能相应延长。
- 本补丁不改变缓存刷新命令、standby 命令或原有断电判定。不同设备的内部写入时序可能不同；当前证据不足以证明半秒等待适用于所有硬件，亦不应宣称可消除所有 FAT 损坏风险。

## 测试情况与边界 / Test status and limitations

- The three-line integration candidate containing this ATA change previously built for ipod6g/64 MB and produced its ZIP successfully.
- On 2026-11-11, a real-device shutdown/wake smoke test completed normally on the author's Kingspec ZIF SSD. That SSD follows the standby path, so the smoke test confirms only that the general shutdown/wake flow worked on that setup; it did **not** exercise or validate the new `!can_sleep && canflush` wait.
- No device without ATA power-management support has been tested. FAT corruption was not reproduced, and no cross-device compatibility, long-term reliability, or comprehensive shutdown testing has been completed. This smoke test is not evidence of target-device or full validation.

- 三线整合候选包含此 ATA 改动；该候选的 ipod6g/64 MB 构建及 ZIP 生成曾成功。
- 2026-11-11 真机关机/唤醒烟测正常，测试设备为作者本机 Kingspec ZIF SSD。该 SSD 走 standby 分支，所以烟测仅说明该设备上的关机/唤醒流程正常，**没有执行或验证新增的 `!can_sleep && canflush` 等待分支**。
- 尚未在不支持 ATA 电源管理的目标设备上测试，未复现 FAT 损坏，也未完成跨设备兼容性、长期稳定性或全面关机测试。此次烟测不能作为目标设备验证或全面验证的依据。

## 补丁来源

`out/ata-sleepnow-fix.patch` 是从三线整合候选的 `storage_ata-6g.c` 对照上游基线 `66f3075f9ea9b8919192ee53ac09047d14b8b677` 提取的单文件差异。该补丁仅包含本修复。
