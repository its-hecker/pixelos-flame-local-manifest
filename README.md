# PixelOS Android 17 — Pixel 4 (flame)

PixelOS `seventeen` for **Pixel 4 (`flame`)**, using the same device repository
and exact CNB kernel revision as the working PixelOS Coral configuration.
Flame changes live on the **separate `pixelos-flame` device branch**. Coral's
stable `main` branch and existing manifest remain untouched.

This manifest uses Flame's own Nahida vendor snapshot, adapted with the Coral
fixes. The actual PixelOS product is `custom_flame`.

## Fresh build from mkdir

Run in ServerHive's initialized Byobu/Tmux/RBE shell. **Keep RBE enabled.**
The host needs normal AOSP dependencies, Repo, Git LFS and Python 3. Use a fresh
Flame directory, separate from Coral or HeckerOS.

```bash
mkdir -p ~/pixelos-flame &&
cd ~/pixelos-flame &&
git lfs install &&
repo init -u https://github.com/PixelOS-AOSP/android_manifest.git -b seventeen --git-lfs &&
git clone --branch main --single-branch https://github.com/its-hecker/pixelos-flame-local-manifest.git .repo/local_manifests &&
repo sync -c -j8 --fail-fast --no-clone-bundle --no-tags &&
PIXEL_BUILD_VARIANT=userdebug PIXEL_BUILD_JOBS=8 bash .repo/local_manifests/build.sh
```

The script checks checkout markers, the Trusty `lk_inc.mk` copy and CAF namespace
exports, resolves Flame's radio LFS objects, prepares the plugins/camera, validates
the sources, runs `breakfast flame userdebug`, and builds with `m pixelos`.
PixelOS chooses the lunch release from synced sources; do not hard-code one.
Output: `out/target/product/flame/PixelOS_*.zip`.

## Sources and branch isolation

All local projects are pinned to full commit IDs in `floral.xml`.

| Checkout | Repository / branch |
| --- | --- |
| `device/google/coral` | [Shared PixelOS device repository](https://github.com/its-hecker/pixelos_android_device_google_coral/tree/pixelos-flame), **`pixelos-flame`** |
| `vendor/google/flame` | [Adapted Flame vendor](https://github.com/its-hecker/pixelos-flame-vendor), `main` |
| `kernel/google/msm-4.14` | [Same CNB kernel as Coral](https://github.com/its-hecker/infinity_cnb_kernel_google_msm-4.14), `cnb-Latest-stable`, exact pin `aaaa03e40200825133d21402cad8f2a86939a778` |
| `packages/apps/ElmyraService` | LineageOS ElmyraService, same Coral pin |
| `vendor/lunaris/dolby` | its-hecker Lunaris Dolby, same `pixelos-compat` pin |
| `vendor/hecker/floral/motionsense` | its-hecker Motion Sense plugin, same stable Coral pin |
| `vendor/hecker/floral/faceid` | its-hecker 3D Face ID plugin, same stable Coral pin |

The shared checkout path must remain `device/google/coral`; the source modules
reference that path. The Flame product selects `flame/device.mk`, its 1080 × 2280
screen, Flame board/vendor, CHRE namespace, display, thermal/audio configuration,
overlays and stock fingerprint. `custom_coral.mk` is retained unchanged on this
branch as a regression reference. Stable Coral builds continue using `main`.

The camera module suffix `-coral` and app name `GoogleCameraCoral` are shared
build identifiers. Flame's original camera HAL binary and its own
`PixelCameraServicesFlame` are retained. Do not substitute Coral vendor blobs.

Flame's vendor comes from `Nahida-Prjkt/proprietary_vendor_google_flame`,
`lineage-24.0`, at `a57f01d5642a55250e48b244f51be9322659eeed`. Original proprietary
files, radio checksums and LFS pointers are unchanged. `.lfsconfig` retains the
upstream radio object endpoint. The kernel includes its exact KernelSU gitlink
and uses `sync-s="true"`; no different HeckerOS kernel is selected.

## Fixes carried from Coral

[CORAL_FIX_PARITY.md](CORAL_FIX_PARITY.md) records each inherited fix and exact
reference revisions. The preparation/recovery tools originate from Coral local
manifest `355fb30d0ae6b2d4c9af52a27d99e71bcd6bb2e2`, with product, vendor,
extraction-list and device-overlay paths adapted for Flame.

This includes camera AIDL V1 isolation and startup, duplicate native/init/VINTF
installer fixes, the eight Face ID replacement install rules, shared v33
multilib providers, HIDL Face startup/challenge/enrollment repairs, Motion Sense
permissions/country availability, original signed Google Camera splits, stock
lockscreen clocks, scene/status-bar double tap to sleep, Dolby sysconfig,
HardwareInfo SELinux, HWUI properties, GoogleParts resources, mainline source
builds, Google artifact exceptions, Trusty/CAF checkout recovery and bounded
Metadata logging on debug builds. Flame also packages the same legacy thermal
interface and preserves the camera extraction module suffix.

## Update and rebuild

After the current build finishes:

```bash
cd ~/pixelos-flame &&
git -C .repo/local_manifests pull --ff-only &&
repo sync -c -j8 --fail-fast --no-clone-bundle --no-tags &&
PIXEL_BUILD_VARIANT=userdebug PIXEL_BUILD_JOBS=8 bash .repo/local_manifests/build.sh
```

Preparation reruns after every sync. It checks immutable plugin inputs, applies
guarded framework fixes with backups, and regenerates selected policy/overlays.
Unfamiliar source is rejected for review. Pass `--offline` to use existing verified
camera/plugin caches. The Android compilation itself performs no plugin fetches.
This inherited diagnostic configuration supports `userdebug` or `eng`, and uses
PixelOS's default signing without private signing keys.

Separate investigation commands:

```bash
git -C vendor/google/flame lfs pull
python3 .repo/local_manifests/tools/prepare.py
python3 .repo/local_manifests/tools/validate.py --rom-root "$PWD"
source build/envsetup.sh
breakfast flame userdebug
m pixelos
```

## Checkout recovery

For missing tracked files or Repo copy/link exports:

```bash
python3 .repo/local_manifests/tools/restore_checkout.py --restore &&
repo sync -l -j1 --fail-fast build/make trusty/vendor/google/aosp hardware/qcom-caf/common &&
PIXEL_BUILD_JOBS=8 bash .repo/local_manifests/build.sh
```

The helper preserves existing edits, staged deletions and build outputs. If
Trusty's root copy is still missing or mismatched, restore its source, run its
local sync and compare `trusty/vendor/google/aosp/lk_inc.mk` with `lk_inc.mk`.

For prior manual CAF namespace workarounds:

```bash
python3 .repo/local_manifests/tools/recover_namespace_edits.py &&
python3 .repo/local_manifests/tools/recover_caf_namespaces.py --apply &&
repo sync -l -j1 --fail-fast hardware/qcom-caf/common &&
PIXEL_BUILD_JOBS=8 bash .repo/local_manifests/build.sh
```

Keep printed backup directories. These tools preserve unrelated edits and
ServerHive's RBE configuration. Fresh Flame checkouts already use the committed
kernel backports and do not need Coral's historical kernel migration procedure.

## Validation and first phone test

```bash
python3 -m unittest discover -s tests -p 'test_*.py'
python3 tools/validate.py
python3 tools/integration_smoke.py
```

The source integration test evaluates both real product/device Make files,
prepares the real plugins twice against Flame's vendor and pinned PixelOS
sources, checks native/APK inputs, framework fixes, overlays, installer ownership
and stale-input rejection, compiles the patched QQS listener with platform test
doubles, and runs the Face framework Java regressions and boot-capture tests.
These tests do not compile Android or boot a phone.

**The first complete Flame ROM build and Pixel 4 phone validation are still
required.** Coral's working phone test does not establish Flame runtime success.
After flashing on a Pixel 4, check boot, cameras/flashlight, calls/data, Wi-Fi,
Bluetooth, eSIM, audio, Face enrollment/unlock/BiometricPrompt, Motion Sense
controls/glow/settings, stock clock selection and enabled/disabled status-bar
and lockscreen double tap to sleep.

The post-build checker verifies camera HAL binary/init/VINTF installation, all
ten signed Google Camera APKs, absence of Aperture/Camera2, and the single Motion
Sense product override in actual build outputs.

```bash
repo manifest -r -o out/pixelos-flame-source-manifest.xml
adb logcat -b all -d > pixelos-flame-logcat.txt
adb logcat -b crash -d > pixelos-flame-crash.txt
adb shell dumpsys face > pixelos-flame-face-service.txt
adb shell lshal > pixelos-flame-hal-services.txt
adb shell cmd overlay list --user 0 > pixelos-flame-overlays.txt
```

Original upstream copyright/license attribution is preserved. The phone's own
persist/calibration data remain required.
