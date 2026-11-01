# PixelOS Flame source validation

Validation date: 1 November 2026. This records source and host checks, not an
Android ROM compilation or a Pixel 4 phone test.

| Check | Result |
| --- | --- |
| Inherited host regression suite | 69 tests passed |
| Actual `custom_flame` / `custom_coral` product + device Make probe | Both resolve the correct identity, display, CHRE namespace, vendor, overlays and plugin order |
| Complete adapted vendor build files | Match the stable Coral fixes after only Flame path, PixelCameraServices identity and target-gate normalization |
| Original Flame proprietary files/radio pointers/checksums | No changes; only Android.bp and flame-vendor.mk are adapted, plus provenance/documentation |
| Real Motion Sense and Face ID preparation | Passed against Flame vendor and pinned PixelOS inputs |
| Repeated preparation | Identical generated outputs; stale-source changes correctly rejected |
| Native installers | One full installer for each of 13 destinations; all eight restored Face duplicates rejected |
| Camera init ownership | Sole selected-prebuilt installer; reintroduced Make copy correctly rejected |
| Original Camera APK set | Base + nine signed splits verified against immutable hashes |
| Patched QQS listener | Compiled; all 17 toggle/user/doze/bouncer/falsing/action/region cases passed |
| Face framework patch regressions | 13 host tests passed; extracted Java methods passed 42 challenge/revoke and 28 enrollment assertions, plus startup/callback ordering probes |
| Metadata boot capture | Seven bounded-runtime, storage, recovery and debug-variant tests passed |
| CAF namespace recovery, globally visible audio defaults, plugin namespace order | Passed against real pinned sources |
| Dolby exported ETC names and installed filenames | No collision |
| Manifest pins and shell/Python source checks | Passed |
| Branch isolation | Flame selects `pixelos-flame`; stable Coral `main` remains at `86de5f048b6f1a3ba3ef723aee2b9446dd872601` |

The same Face, stock-clock and tap-to-sleep adapters were also checked against
PixelOS `seventeen` framework `044b965aa2d30d056b930e5ff013ba5303cde9fa`, still the
current branch head at verification. The fixes applied idempotently; QQS and
Face method regressions passed there too.

The source/plugin smoke test uses selected real inputs and host platform doubles.
Its product-routing probe stubs external platform Make includes; the actual
vendor/plugin entries are parsed separately. These checks cannot establish full
Soong/Kati compilation, successful boot, or runtime hardware behavior.

The first complete build and Pixel 4 validation remain required. Follow README.md
for the ServerHive/RBE build command and the phone test list.
