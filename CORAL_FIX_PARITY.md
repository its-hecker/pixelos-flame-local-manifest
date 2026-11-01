# PixelOS Coral → Flame fix parity

Reference: Coral manifest `355fb30d0ae6b2d4c9af52a27d99e71bcd6bb2e2`, shared device
`86de5f048b6f1a3ba3ef723aee2b9446dd872601`, adapted Coral vendor
`fc4a5b6b20c40954c2ad99d543f8c1e61e86d03a`. Flame's vendor base is Nahida
`lineage-24.0` at `a57f01d5642a55250e48b244f51be9322659eeed`.

Flame device changes are isolated on **`pixelos-flame`**. Stable Coral **`main`**
remains unchanged. Every dependency is pinned in `floral.xml`.

| Coral fix | Flame integration |
| --- | --- |
| PixelOS inheritance, source mainline modules, Google artifact exceptions | Complete Coral product adapted to `custom_flame`, own identity/vendor/hardware |
| HardwareInfo coredomain/battery sysfs neverallow | Same shared device policy |
| HWUI property in system rather than vendor | Same shared property files |
| GoogleParts eSIM resources without Lineage SDK | Same shared implementation |
| Hardware Face ID and Motion Sense after vendor inheritance | Same plugin commits and product/board order; ParanoidSense disabled |
| HIDL Face startup, sensor/callback order, reconnect, challenge/revoke and enrollment fixes | Same guarded framework adapter in the stable Face ID plugin |
| Face ABI, crypto/calibration and camera compatibility | Same immutable plugin inputs; Flame vendor explicitly selected |
| Eight duplicate stock Face service/library installers | Same eight `no_full_install: true` flags, preserving dependency modules |
| Shared v33 base/crypto/stagefright multilib providers | Same plugin provider checks and both architectures |
| Stock AIDL V1 camera isolated from newer source service | Shared `-coral` alias in Flame vendor/device/extraction list, original executable stem and V1 libraries |
| Camera provider startup | Flame prebuilt owns original `init_rc`; redundant Make copy/source attachment removed |
| Duplicate binary/VINTF full installers | Same source installer adapter, backups and ownership regressions |
| 64-bit legacy camera thermal interface | Explicitly added to `flame/device.mk` |
| Original signed Camera base + nine splits, no Aperture/Camera2 | Same shared module, immutable hashes and output checker; Flame PixelCameraServices retained |
| Motion Sense country/capability and settings privilege | Same product override, vendor-init rule and narrow system_ext WRITE_SECURE_SETTINGS allowlist |
| Oslo plugin allowlist, sensor overlays, stale-source rejection | Same preparation with Flame's selected overlay/vendor paths |
| PixelOS stock lockscreen clock application | Same stock-provider allowlist adapter |
| Scene/status-bar double tap to sleep | Same guarded adapter, current-user toggle, lifecycle and doze/bouncer/falsing guards |
| Dolby Make/sysconfig collision | Same `pixelos-compat` pin and filename fix |
| CAF duplicate modules/audio defaults/namespace ordering | Same namespace preflight, pinned plugins and backed-up recovery helpers |
| Missing tracked/sparse checkout files and Trusty root copy | Same recovery tools and build preflight |
| CNB KernelSU/namespace/seccomp build fixes | Exact same Coral kernel revision and KernelSU gitlink |
| Bounded early Metadata evidence | Same debug-only logging, existing debug domain and storage/runtime guards |

The full Flame vendor build files are compared with the adapted Coral versions
after only device path, PixelCameraServices identity and target-gate normalization.
All Flame proprietary files, original radio LFS pointers and checksums are kept.
Both actual product/device Make files are evaluated by the integration probe.

This records source parity. Full Android compilation and Pixel 4 runtime testing
remain separate requirements.
