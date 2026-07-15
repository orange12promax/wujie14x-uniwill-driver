# wujie14x-uniwill-driver

Model-specific Linux platform driver work for the **MECHREVO WUJIE14XA**.
This repository carries a narrow, hardware-verified extension of the mainline
`uniwill_laptop` driver and packages it with DKMS for local use.

The project is intentionally not a generic vendor control center. A feature is
enabled for this DMI only after its EC or WMI behaviour has been verified on the
real machine and mapped to an existing Linux subsystem.

## Supported hardware

The driver binds only on this exact identity:

- System vendor: `MECHREVO`
- Product name: `WUJIE14XA`
- Board name: `WUJIE14-GX4HRXL`

The board belongs to the Uniwill family used by the TUXEDO InfinityBook Pro
14/15 Gen9 AMD (`GXxHRXx`). The family relationship is useful as an
implementation reference, but it is not permission to enable every sibling
feature: the physical fan layout and EC capability set are not identical.

## Current implementation

The current source exposes the three verified EC performance modes through the
standard Linux `platform_profile` interface:

| platform profile | EC register `0x0751` | Sustained power limit |
|---|---:|---:|
| `low-power` | `0xA0` | 25 W |
| `balanced` | `0x00` | 45 W |
| `performance` | `0x10` | 65 W |

Fn+X cycles through the profiles, and the selected mode is preserved across
suspend and resume. `power-profiles-daemon` and KDE/GNOME can use the backend
without a model-specific userspace service.

The driver also exposes read-only hwmon telemetry from EC registers verified
on this machine: CPU temperature (`temp1_input`, register `0x043e`), main-fan
speed (`fan1_input`, `0x0464`/`0x0465`), and main-fan duty as a read-only
`pwm1` (`0x075b`). No fan-control writes are implemented. The GPU temperature
and secondary-fan channels stay disabled: there is no discrete GPU, and the
second fan channel is not trustworthy on this board.

## Source and patch layout

- `uniwill-acpi.c`, `uniwill-wmi.c`, `uniwill-wmi.h` — vendored driver source.
- `Kbuild`, `dkms.conf`, `PKGBUILD` — local DKMS/Arch packaging.
- `patches/` — reviewable deltas against the pinned upstream source, applied
  in order.

The source is based on Linux v7.1 commit
`8cd9520d35a6c38db6567e97dd93b1f11f185dc6` from
`drivers/platform/x86/uniwill/`. The deltas are documented in
[patches/platform-profile-vs-v7.1.patch](patches/platform-profile-vs-v7.1.patch)
(platform_profile support) and
[patches/hwmon-telemetry.patch](patches/hwmon-telemetry.patch) (read-only
hwmon telemetry on top of it).

## Build on Arch Linux

Install DKMS and the headers for every kernel that should receive the module,
then build without installing:

```bash
makepkg --cleanbuild --force
```

The resulting package is named `wujie14x-uniwill-dkms`.

### Install

```bash
sudo pacman -U ./wujie14x-uniwill-dkms-*.pkg.tar.zst
sudo modprobe uniwill_laptop
sudo systemctl restart power-profiles-daemon
```

Do not load the legacy `tuxedo-drivers` Uniwill modules beside this
`uniwill_laptop` module; both drive the same EC.

Verify after installation:

```bash
dkms status wujie14x-uniwill
modinfo -n uniwill_laptop
cat /sys/class/platform-profile/platform-profile-0/{name,choices,profile}
powerprofilesctl list
sensors uniwill-*
```

## License and credits

GPL-2.0-or-later, matching the upstream driver. See [LICENSE](LICENSE).

- The mainline `uniwill_laptop` driver is maintained by Armin Wolf.
- TUXEDO's Uniwill driver is a hardware-family reference and its contribution
  requirements must be respected when code is derived from it.
- [LongSang01/wujie14X-Linux-Driver](https://github.com/LongSang01/wujie14X-Linux-Driver)
  provided initial EC-register clues for this machine.

## 中文说明

本仓库是机械革命无界 14XA 的 Uniwill Linux 平台驱动适配仓库，通过标准
`platform_profile` 接口提供三档功耗模式（25 W / 45 W / 65 W），支持 Fn+X 循环
切换，挂起恢复后保持所选模式，可直接配合 `power-profiles-daemon` 与
KDE/GNOME 的电源模式使用；并通过标准 hwmon 接口提供只读的 CPU 温度、主风扇
转速与占空比监控（`sensors` 可直接读取），不包含任何风扇写入控制。
