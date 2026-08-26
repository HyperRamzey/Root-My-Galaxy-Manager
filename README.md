# Root-My-Galaxy Manager (spoofed KernelSU manager builds)

Builds the KernelSU manager app from the **latest upstream commit**
(`tiann/KernelSU`) with:

- **Spoofed identity** — application id `com.sec.android.systune`,
  launcher label `SystemTune`. No KernelSU string survives on the device
  (the upstream manager takes both via first-class `KSU_PACKAGE_NAME` /
  `KSU_NAME` gradle properties; no source fork).
- **Pipeline optimization flags** — release native build injected with
  `-O2 -flto=thin -march=armv8-a+crc+crypto -mtune=cortex-a715`
  (same tune as every other binary in the Root-My-Galaxy pipeline).
  R8 minify + resource shrinking are already upstream defaults.
- **Stable signature** — signed with the committed pipeline keystore
  (`release.keystore`, same cert as the Root-My-Galaxy app), so the
  kernel-side manager recognition that accepts our apps accepts this one
  too, and `pm install -r` updates work.

## Cadence

- Every push to `main` and every 6 hours: resolve upstream `main`,
  build, and publish a release tagged `mgr-<upstream-short-sha>`.
- Already-built upstream commits are skipped (delete the release to force
  a rebuild, or run the workflow with `force=true`).

## Consumption

`feed.json` (repo root + release asset) is the machine-readable pointer:

```json
{
  "pkg": "com.sec.android.systune",
  "label": "SystemTune",
  "versionCode": 1234,
  "versionName": "...",
  "upstreamSha": "...",
  "url": "https://github.com/HyperRamzey/Root-My-Galaxy-Manager/releases/download/mgr-<sha>/SystemTune-<ver>.apk",
  "sha256": "..."
}
```

The Root-My-Galaxy app polls this feed after each successful root apply
(before module activation) and installs/updates the APK via root shell,
recording `<versionCode> <pkg>` in `/data/adb/.rmg/ksumgr` — the spoofed
package cannot be discovered by name from outside, so the registry file
written at install time is the detection mechanism.
