# Floran Kernel

A GitHub Actions-based automated build workflow for Android 6.1 GKI kernels, targeting `sm8650` / `pineapple` kernel sources and their OPlus GKI configurations.

The workflow pulls the kernel source for the selected ROM, integrates KernelSU, SUSFS, Droidspaces, networking features and other components as needed, and finally packages a flashable ZIP with AnyKernel3. It can also create a GitHub Release automatically.

> [!WARNING]
> The kernel source, modules, ROM, device and firmware versions must all match. Flashing a kernel carries risks of boot failure, data loss or other unexpected issues. Verify compatibility yourself and back up your data before flashing.

## Features

- Multiple ROM kernel sources:
  - YAAP (Android 16 / 17)
  - LineageOS (Android 16 / 17)
  - FrierenKernel (Android 17, LineageOS-optimized)
  - crDroid (Android 16 / 17)
  - PixelOS
  - DerpFest (Android 17)
  - AOSPA (Android 17)
- Multiple root solutions:
  - KernelSU Official
  - KernelSU-Next
  - BakaSU
  - SUSFS variants of the above (some combinations)
  - No KernelSU
- Optional features:
  - BBR, ECN and FQ queue scheduling
  - IPSet and IPv6 NAT
  - Droidspaces container support
  - Droidspaces Extended: virtual HCI, systemd-coredump related options and Lindroid EVDI DRM
  - Docker container support (single switch, off by default; ⚠️ may bootloop some devices, see the header of `patch/docker.config`)
  - BBRv3 congestion control (GKI 6.1 backport from WildKernels)
  - TTL target support (IPv4 TTL / IPv6 HL)
  - OPLUS SDP 900 mA charging (from WildKernels)
  - Unicode Bypass Fix (kernel 6.1+, from WildKernels)
  - NoMount VFS path-redirection subsystem (built-in, from maxsteeel/nomount)
- Built with AOSP Clang: `clang-r596125` for Android 17 sources, `clang-r563880c` for Android 16 sources
- Flashable ZIP output via AnyKernel3
- Uploads to Actions Artifacts and can create a GitHub Release automatically

## Usage

### Single build

1. Fork this repository.
2. Open the **Actions** tab of your fork.
3. Select **Build Android16 Kernel** for Android 16 kernels, or **Build Android17 Kernel** for Android 17 kernels.
4. Click **Run workflow** and fill in the build parameters as needed.
5. Wait for the workflow to finish:
   - Download the ZIP from **Artifacts** on the workflow run page;
   - If `Create GitHub Release` is enabled, download it from the repository's **Releases** page.

### Build parameters

| Parameter | Description |
|---|---|
| `Build Android16 Kernel` | Builds `YAAP`, `LineageOS`, `Crdroid` or `PixelOS` kernels; this workflow offers general options such as Droidspaces. |
| `Build Android17 Kernel` | Builds `YAAP`, `DerpFest`, `LineageOS`, `FrierenKernel`, `Crdroid` or `AOSPA` kernels; this workflow always uses `clang-r596125`. For FrierenKernel, only Droidspaces is kept among the optional enhancements. |
| `ROM Kernel Source Code` / `Android 17 Kernel Source Code` | Selects the kernel source in the workflow for the matching Android version. ROM names have no version suffix; the internal `android_version` determines the source branch and toolchain. |
| `KernelSU Version` | Selects the KernelSU integration, or `None` to build a non-KernelSU kernel. |
| `ksu_branch` | Overrides the source branch of the selected KernelSU variant for single builds; leave empty for the default. For example, use `sync-up` for BakaSU. |
| `bakasu_branch` / `kernelsu_next_branch` | Overrides the branches of BakaSU and KernelSU-Next independently in matrix builds; leave empty for their defaults. |
| `susfs_dev` | Boolean, off by default. On uses the 6.1 `gki-android14-6.1-dev` branch; off uses the stable `gki-android14-6.1` branch. Only used by `susfs` variants. |
| `Enable IPSET & IPv6_NAT` | Enables IPSet, IPv6 NAT and related Netfilter options; ignored by FrierenKernel. |
| `Enable BBR & ECN` | Enables BBR, ECN and FQ; ignored by FrierenKernel. |
| `Droidspaces Container Support` | `none`, `standard` or `extended`. YAAP does not receive the Droidspaces patches. |
| `Docker Container Support` | ⚠️ DANGER: appends the single `patch/docker.config` fragment (namespaces / cgroup v2 / USER_NS / overlay / btrfs); may bootloop some devices. |
| `enable_bbr3` | Applies `patch/bbrv3_android14-6.1.patch` (WildKernels' BBRv3 backport for android14-6.1, including TCP PLB) and writes `CONFIG_TCP_CONG_ADVANCED=y` / `CONFIG_TCP_CONG_BBR3=y`. Can be used together with `Enable BBR & ECN`. |
| `enable_ttl` | Appends `CONFIG_IP_NF_TARGET_TTL` / `CONFIG_IP6_NF_TARGET_HL` / `CONFIG_IP6_NF_MATCH_HL`. No patch file needed. |
| `enable_sdp900ma` | Applies `patch/oplus_sdp_900ma.patch`: an early_initcall updates the live device tree so the OPLUS charger module raises USB SDP input current from 500 mA to 900 mA (supports both V1 and V2 charger frameworks). |
| `enable_unicode_fix` | Applies `patch/unicode_bypass_fix_6.1+.patch`, fixing Unicode normalization handling in `fs/unicode/utf8-norm.c`. |
| `enable_nomount` | Integrates NoMount with its upstream automatic `setup.sh` (Method 1) and sets `CONFIG_NOMOUNT=y`; builds it directly into the `Image`. |
| `nomount_ref` | Branch, tag or commit passed to NoMount's `setup.sh` (default `dev`; leave empty to use the latest release tag). |
| `Custom Kernel Name` | Sets an additional kernel version name. The script adds the `-` prefix automatically. |
| `Create GitHub Release?` | Whether to create and upload a GitHub Release after a successful build. |

## ROM sources and branches

Single-build workflows use the following upstreams and branches:

The matrix workflow builds `YAAP-17`, `FrierenKernel`, `Crdroid-17`, `LineageOS` and `AOSPA`. YAAP-17, FrierenKernel, Crdroid-17 and AOSPA use the Android 17 `clang-r596125`; LineageOS uses the Android 16 `clang-r563880c`. The matrix no longer builds crDroid 16, which is still available through the single-build workflow. YAAP-17 skips Droidspaces and the generic performance tweaks; FrierenKernel skips BBR/ECN/FQ, IPSet/IPv6 NAT and the generic performance tweaks, but keeps the Droidspaces option.

Android 17 Clang toolchain artifact: [`clang-r596125.tar.gz`](https://github.com/AkiHaza/Floran-Kernel/releases/download/toolchain-r596125/linux-x86-refs_heads_android17-release-clang-r596125.tar.gz)

| Android version | ROM source | Kernel repo / branch | Modules repo / branch |
|---|---|---|---|
| 16 | YAAP | [`AkiHaza/android_kernel_oneplus_sm8650`](https://github.com/AkiHaza/android_kernel_oneplus_sm8650) / `sixteen` | [`AkiHaza/android_kernel_oneplus_sm8650-modules`](https://github.com/AkiHaza/android_kernel_oneplus_sm8650-modules) / `sixteen` |
| 17 | YAAP | [`AkiHaza/android_kernel_oneplus_sm8650`](https://github.com/AkiHaza/android_kernel_oneplus_sm8650) / `seventeen` | [`AkiHaza/android_kernel_oneplus_sm8650-modules`](https://github.com/AkiHaza/android_kernel_oneplus_sm8650-modules) / `seventeen` |
| 16 | LineageOS | [`LineageOS/android_kernel_oneplus_sm8650`](https://github.com/LineageOS/android_kernel_oneplus_sm8650) / `lineage-23.2` | [`LineageOS/android_kernel_oneplus_sm8650-modules`](https://github.com/LineageOS/android_kernel_oneplus_sm8650-modules) / `lineage-23.2` |
| 17 | LineageOS | [`LineageOS/android_kernel_oneplus_sm8650`](https://github.com/LineageOS/android_kernel_oneplus_sm8650) / `lineage-24.0` | [`LineageOS/android_kernel_oneplus_sm8650-modules`](https://github.com/LineageOS/android_kernel_oneplus_sm8650-modules) / `lineage-24.0` |
| 17 | FrierenKernel | [`AkiHaza/custom_kernel_oneplus_sm8650`](https://github.com/AkiHaza/custom_kernel_oneplus_sm8650) / `lineage-24.0` | [`AkiHaza/custom_kernel_oneplus_sm8650-modules`](https://github.com/AkiHaza/custom_kernel_oneplus_sm8650-modules) / `lineage-24.0` |
| 16 | Crdroid | [`crdroidandroid/android_kernel_oneplus_sm8650`](https://github.com/crdroidandroid/android_kernel_oneplus_sm8650) / `16.0` | [`crdroidandroid/android_kernel_oneplus_sm8650-modules`](https://github.com/crdroidandroid/android_kernel_oneplus_sm8650-modules) / `16.0` |
| 17 | Crdroid | [`crdroidandroid/android_kernel_oneplus_sm8650`](https://github.com/crdroidandroid/android_kernel_oneplus_sm8650) / `17.0` | [`crdroidandroid/android_kernel_oneplus_sm8650-modules`](https://github.com/crdroidandroid/android_kernel_oneplus_sm8650-modules) / `17.0` |
| 16 | PixelOS | [`PixelOS-Devices/android_kernel_oneplus_sm8650`](https://github.com/PixelOS-Devices/android_kernel_oneplus_sm8650) / `sixteen-qpr2` | [`PixelOS-Devices/android_kernel_oneplus_sm8650-modules`](https://github.com/PixelOS-Devices/android_kernel_oneplus_sm8650-modules) / `sixteen-qpr2` |
| 17 | DerpFest | [`ppanzenboeck/android_kernel_oneplus_sm8650`](https://github.com/ppanzenboeck/android_kernel_oneplus_sm8650) / `lineage-23.2` | [`ppanzenboeck/android_kernel_oneplus_sm8650-modules`](https://github.com/ppanzenboeck/android_kernel_oneplus_sm8650-modules) / `derp16.2-arb` |
| 17 | AOSPA | [`AkiHaza/AOSPA_kernel_oneplus`](https://github.com/AkiHaza/AOSPA_kernel_oneplus) / `standalone` | [`AkiHaza/AOSPA_kernel_oneplus_modules`](https://github.com/AkiHaza/AOSPA_kernel_oneplus_modules) / `standalone` |

In **Build Android17 Kernel**, DerpFest, LineageOS and AOSPA can choose `standard` or `extended` Droidspaces support. YAAP ignores this option. FrierenKernel keeps the selected root/SUSFS solution and only Droidspaces among the extra enhancements; even if checked, BBR/ECN/FQ, IPSet/IPv6 NAT and the generic performance options added by the workflow are skipped.

AOSPA's modules source lives inside the kernel tree at `sm8650/sm8650-modules`, matching the directory convention of its standalone build Makefile. When `None` is selected, the workflow removes the source's pre-existing KernelSU Kconfig/Makefile references so an uninitialized KernelSU submodule cannot break config generation; existing KernelSU source and submodule declarations are preserved. The current ZIP only replaces the kernel `Image`; it does not rebuild or replace the device's existing modules, so the module ABI must still match the corresponding ROM.

## KernelSU and SUSFS

The following KernelSU variants are available:

| Option | Contents |
|---|---|
| `KernelSU-Official` | Official KernelSU |
| `KernelSU-Official-susfs` | Official KernelSU + SUSFS |
| `KernelSU-Next` | KernelSU-Next |
| `KernelSU-Next-susfs` | KernelSU-Next + SUSFS |
| `BakaSU` | BakaSU |
| `BakaSU-susfs` | BakaSU + SUSFS |
| `None` | No KernelSU integration |

Single builds can override the branch of the chosen variant with `ksu_branch`. For example, selecting `BakaSU-susfs` and setting `ksu_branch=sync-up` uses BakaSU's `sync-up` branch. Matrix builds set the branches of the two variants independently with `bakasu_branch` and `kernelsu_next_branch`, so a BakaSU branch name is never applied to KernelSU-Next. When left empty, KernelSU Official and BakaSU use `main`, while KernelSU-Next uses the existing pershoot repository's `dev-susfs`. Branch input is ignored when `None` is selected.

The workflow clones the requested branch first, then runs that branch's `kernel/setup.sh` and verifies the final branch, commit and driver links. It aborts if the branch does not exist or if setup switches to a different branch. KernelSU retains full commit history so version codes are calculated correctly; the actual branch and commit are written to the Actions log and summary.

When a `susfs` variant is selected, the workflow always uses the `android14-6.1` branch series from [simonpunk/susfs4ksu](https://gitlab.com/simonpunk/susfs4ksu). `susfs_dev` only chooses between stable and dev branches; custom branch names are not supported. The ROM's Android 16 / 17 version does not affect this choice.

| `susfs_dev` switch | SUSFS branch |
|---|---|
| Off / `false` (default) | `gki-android14-6.1` |
| On / `true` | `gki-android14-6.1-dev` |

For example, set `ksu_branch=sync-up` plus `susfs_dev` in a single build, or `bakasu_branch=sync-up` plus `susfs_dev` in a matrix build. Both stable and dev use the patch file `50_add_susfs_in_gki-android14-6.1.patch`; the filename does not get a `-dev` suffix. Official and non-official SUSFS variants share the same switch.

All SUSFS variants validate and apply the kernel patch through `.github/scripts/apply-susfs-kernel.py` in a temporary directory. It handles YAAP's stack-allocated `linux_binprm` and `bprm_execve` argument differences as well as declaration contexts in `fs/namespace.c` and `fs/super.c`. Files are written back only after all hooks are verified; unknown conflicts abort the build instead of dropping functional patches. Matrix builds use the same handler.

The SUSFS credential-switching helpers and their callers are guarded together by `CONFIG_KSU_SUSFS_PRIORITY_UID_FILTER`, so BakaSU does not hit unused-function errors when that optional feature is missing. Kernel `Werror` checks are preserved. The compatibility patch aborts when the source layout is unknown or incomplete.

`KernelSU-Official-susfs` uses the unified entry point `.github/scripts/integrate-susfs.sh` with `source` / `config` / `built` stages: it checks and applies the upstream `KernelSU/10_enable_susfs_for_ksu.patch`, fills in the official KernelSU-side options and interfaces, and records both source commits. Kernel patches are written back only after every added fragment is verified in a temporary directory; only known `fs/exec.c` argument / stack-object differences and namespace/super declaration contexts are adapted. Any unresolved functional conflict aborts the build and keeps the reject file.

Only the official SUSFS variant verifies the final `out/.config` after merging vendor config, and checks key static-key symbols in `fs/susfs.o`, `vmlinux`, `Image` and `System.map` after compilation. It packages the ZIP only if verification passes and uploads `SUSFS_Evidence_*` (final config and symbol table). These checks do not replace runtime verification after flashing.

After flashing a kernel with KernelSU integrated, install the manager matching your chosen variant:

- [KernelSU](https://github.com/tiann/KernelSU)
- [KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next)
- [BakaSU](https://github.com/Baka-SU/BakaSU)

## Droidspaces container support

| Mode | Contents |
|---|---|
| `none` | No Droidspaces features added. |
| `standard` | Basic container support: PID, IPC and mount namespaces, SysV IPC, POSIX message queues, NTSync, etc. |
| `extended` | Adds virtual HCI, Lindroid EVDI DRM and related configuration on top of `standard`. |

> [!NOTE]
> The Droidspaces patches support non-YAAP sources such as DerpFest and are not applied to YAAP.

## WildKernels feature ports (BBRv3 / TTL / SDP / Unicode)

The following four features come from [WildKernels/kernel_patches](https://github.com/WildKernels/kernel_patches). Each has its own boolean switch and is available in both single-build workflows (Android 16 / 17) and the matrix build.

| Option | Patch | Description |
|---|---|---|
| `enable_bbr3` | `patch/bbrv3_android14-6.1.patch` | Backports BBRv3 to android14-6.1 (~124 KB, 16 files, adds `tcp_bbr3.c` and `tcp_plb.c`) and adds `CONFIG_TCP_CONG_BBR3=y`. Upstream declares Android KABI compliance. |
| `enable_ttl` | none (config only) | `CONFIG_IP_NF_TARGET_TTL`, `CONFIG_IP6_NF_TARGET_HL`, `CONFIG_IP6_NF_MATCH_HL`. |
| `enable_sdp900ma` | `patch/oplus_sdp_900ma.patch` | Adds `drivers/misc/oplus_sdp_900ma.c`, which updates the device tree's `qcom,input_current_usb_ma` (500 → 900 mA) and the V2 framework's `oplus_spec,input-power-mw` SDP slot (2500 → 4500 mW) at boot. Disable with `oplus_sdp_900ma=0`. |
| `enable_unicode_fix` | `patch/unicode_bypass_fix_6.1+.patch` | Fixes Unicode normalization in `fs/unicode/utf8-norm.c` to prevent path traversal / detection bypasses using non-printable Unicode code points. |

Patches are applied uniformly with `patch -p1 -F3 --forward` and checked for idempotency with `--dry-run` / `-R --dry-run`. If a patch cannot be applied, the build fails instead of silently producing a ZIP that claims the feature is enabled.

> [!WARNING]
> The BBRv3 patch modifies headers such as `include/linux/tcp.h` and `include/net/tcp.h`. Although upstream declares KABI compliance, prebuilt vendor modules that reference those structures may still face ABI risks — always verify networking and baseband on the device.
> SDP 900 mA is only meaningful on OnePlus/Oppo/Realme devices using the OPLUS charger framework; on other devices the patch safely skips its edits.

ZIP filenames append `-bbr3`, `-ttl`, `-sdp900` and `-unifix` for the enabled features.

## NoMount support

`enable_nomount` uses **Method 1 (Automatic Built-in Integration)** from NoMount's own documentation. The workflow downloads the upstream `kernel/setup.sh` at build time and runs it at the root of the kernel tree:

```bash
curl -fsSL https://raw.githubusercontent.com/maxsteeel/nomount/refs/heads/dev/kernel/setup.sh -o "$RUNNER_TEMP/nomount-setup.sh"
bash "$RUNNER_TEMP/nomount-setup.sh" "$NOMOUNT_REF"
```

`setup.sh` clones the NoMount repository into the kernel tree, creates the `fs/nomount` symlink to its `kernel/src`, appends `obj-$(CONFIG_NOMOUNT) += nomount/` to `fs/Makefile`, and inserts `source "fs/nomount/Kconfig"` into `fs/Kconfig`. The workflow then appends `CONFIG_NOMOUNT=y` to `gki_defconfig` and verifies that the symlink and both build-system entries actually exist. `nomount_ref` controls which branch/tag/commit of NoMount is checked out; an empty value makes the script use the latest `v*` tag.

> [!WARNING]
> Method 1 is network-dependent and version selection is not pinned by this repository: a new upstream tag or branch content can change what gets built. For fully reproducible/offline builds, vendor the source and use the manual method instead.

The NoMount kernel source is GPL-2.0, has explicit compatibility guards for kernel 6.1+, uses `fs_initcall`, and exports no new kernel symbols, so GKI module ABI is preserved.

> [!NOTE]
> For standard GKI 5.10+ kernels the NoMount release ZIP already ships prebuilt LKMs, so flashing the NoMount KernelSU/APatch metamodule usually needs **no kernel modification**. This option is for builds that want NoMount compiled into the `Image` instead (e.g. custom kernels that block standard LKMs).

> [!WARNING]
> NoMount intercepts VFS path resolution and directory iteration in RAM, while SUSFS also hooks VFS (`SUS_PATH` / `SUS_MOUNT`). Test the combination with your chosen KernelSU/SUSFS variant. To actually use NoMount after flashing, install the NoMount metamodule from the project's Releases; installing it over a kernel that already has NoMount built-in is supported.

## Output file naming

Single-build ZIPs roughly follow:

```text
<ROM source>-A<Android version>-<KSU variant>[-dss|-dss-ext][-docker][-bbr3][-ttl][-sdp900][-unifix][-nomount]-<UTC month/day>.zip
```

Where:

- `A16` / `A17`: the Android version of the workflow; Artifacts and Releases also show it so ROMs with the same name can be distinguished.
- `dss`: Droidspaces Standard
- `dss-ext`: Droidspaces Extended
- `docker`: Docker container support enabled
- `bbr3` / `ttl` / `sdp900` / `unifix`: the corresponding WildKernels feature options
- `nomount`: NoMount built into the kernel

## Local build

Main dependencies used by the workflow:

```bash
sudo apt update
sudo apt install -y \
  bc bison flex libssl-dev libelf-dev libdw-dev build-essential \
  lz4 git python3 curl dwarves cpio gcc-aarch64-linux-gnu
```

Android 17 kernels use the `clang-r596125` toolchain artifact from Releases; Android 16 kernels use `clang-r563880c`. Then run:

```bash
make O=out gki_defconfig vendor/pineapple_GKI.config vendor/oplus/pineapple_GKI.config
make -j"$(nproc)" O=out Image
```

For the complete source-fetching, patching, KernelSU/SUSFS integration and packaging steps, see [`.github/workflows/build-android16.yml`](.github/workflows/build-android16.yml) (Android 16) and [`.github/workflows/build-android17.yml`](.github/workflows/build-android17.yml) (Android 17); both share the build logic in [`.github/workflows/_build-kernel.yml`](.github/workflows/_build-kernel.yml).

## Credits

- [AnyKernel3](https://github.com/AkiHaza/AnyKernel3)
- [KernelSU](https://github.com/tiann/KernelSU)
- [KernelSU-Next](https://github.com/KernelSU-Next/KernelSU-Next)
- [BakaSU](https://github.com/Baka-SU/BakaSU)
- [SUSFS4KSU](https://gitlab.com/simonpunk/susfs4ksu)
- [WildKernels/kernel_patches](https://github.com/WildKernels/kernel_patches)
- [NoMount](https://github.com/maxsteeel/nomount)
- Maintainers of the various ROMs, kernel sources and patch projects
