# 与 Xiaodu SEED-2301 (xps06e) 的驱动覆盖比对

比对时间：2026-10-02
方法：拿设备 `/proc/config.gz` 与设备实际加载的 112 个 `.ko`，
逐项与内核树的 Kconfig / Makefile 目标做匹配。

## 结论：覆盖度很高

设备与这棵树是**同 SoC（UMS9230）+ 同平台（Unisoc 1H10 公版）**，
设备 `uname` 报的内核 release 为 `5.4.161-ab20250825010819358`，
设备树 model 为 `Spreadtrum UMS9230 1H10 SoC`。

| 项目 | 数量 |
| --- | --- |
| 设备 `.ko` 总数 | 112 |
| 树里能找到对应构建目标的 | **98** |
| 需要处理命名差异或从 `kernel_modules/` 接入的 | 13 |
| **源码确实缺失的** | **1** |

## 唯一的真实缺口：AW87xxx 音频功放

```text
设备模块      snd-soc-aw87xxx.ko
设备 DT 节点  /soc/ap-apb/i2c@0x20110000/aw87xxx@5b/
              compatible = "awinic,aw87xxx_pa"
树里引用      arch/arm64/boot/dts/sprd/RMX3624-overlay.dts:674
              compatible = "awinic,aw87xxx_pa"
树里驱动      无
```

树里的 DTS 引用了这个 compatible，但驱动源码不在树中。
后果：**扬声器功放不工作**（编解码器 sc2730 仍在，耳机通路可能正常）。
修法：移植 awinic aw87xxx 驱动（各展锐 BSP 与 GitHub 上常见），
需匹配 `awinic,aw87xxx_pa` 这个 compatible。

## 其余 13 项都能解决

### A. 只是模块名不同（驱动在树里）

| 设备模块 | 树里的目标 | 说明 |
| --- | --- | --- |
| `chipone_tddi_9916.ko` | `chipone-tddi.o` | 触摸屏，设备输入设备名就是 `chipone-tddi`，驱动一致 |
| `sgm41510-charger.ko` | `sgm4151x-charger.o`（`CHARGER_SGM4151X`） | 同系列，需确认支持 41510 |
| `sprd_battery_info.ko` | `drivers/power/supply/sprd_battery_info.c` | 已在树里 |
| `sprd_vote.ko` | `drivers/power/supply/sprd_vote.c` | 已在树里 |

### B. 在 `kernel_modules/` 里，但该目录未接入构建

`kernel_modules/` 是展锐的“可选驱动暂存区”，顶层 Kconfig/Makefile 均未引用它，
需要手工 source / 作为外部模块构建：

| 设备模块 | 位置 |
| --- | --- |
| `mali_kbase.ko` | `kernel_modules/kernel5.4/gpu/gondul/mali/` |
| `sprd_sensor.ko` | `kernel_modules/`（27 个文件）+ `drivers/iio/sprd_hub/` |
| `flash_ic_aw3641.ko` | `kernel_modules/common/camera/flash/aw3641/` |
| `mmdvfs.ko` | `kernel_modules/common/camera/mmdvfs/` |
| `sprd_camera.ko` / `sprd_cpp.ko` | `kernel_modules/common/camera/{core,cpp}/` |
| `sprd_fm.ko` | `kernel_modules/kernel5.4/wcn/fm/` |
| `sprdbt_tty.ko` | `kernel_modules/kernel5.4/wcn/bluetooth/` |

### C. 上游版本差异导致的符号消失（无害）

`REFCOUNT_FULL`、`COMPAT_VDSO`、`GENERIC_COMPAT_VDSO`、`CRYPTO_LIB_BLAKE2S`、
`NET_CLS_TCINDEX`（安全修复，上游已移除）、`ZRAM_DEDUP`（展锐私有优化）。

## 另一个必须处理的问题：模块名要改

Android 的 `modules.load` 按**文件名**加载模块：

```text
/vendor/lib/modules/modules.load  →  sgm41510-charger.ko
                                     chipone_tddi_9916.ko
                                     snd-soc-aw87xxx.ko
```

而新内核编出来的是 `sgm4151x-charger.ko`、`chipone-tddi.ko`。
所以打包时必须二选一：改名输出，或改写 `modules.load`。

## 仍然存在的结构性风险（与驱动覆盖无关）

1. **vermagic 不匹配**：设备模块是 `5.4.161-...modversions`，新内核是 5.4.254+。
   `CONFIG_MODULE_SIG_FORCE=y` + `CONFIG_MODVERSIONS=y` ⇒
   **112 个模块必须全部重建并用新内核的密钥重签**。
2. **DT 在独立分区**：`dtb_a` / `dtbo_a`。新内核必须与设备现有 DT 的
   binding 兼容；树里的 `ums9230-1h10` 系列 DTS 可作参照，但不能直接替换。
3. **编译环境**：Termux(bionic) 无法完成内核主机工具编译，需在 glibc
   环境（CI / Linux 主机）构建。
