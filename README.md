# Xenia Canary: optional USB Skylanders portal

This project builds the public Xenia Canary source at commit `4f04bc7cc76b89bfcf7327b597d9770eaaddf935` with one narrowly scoped change: a runtime setting can skip construction of the optional USB Skylanders portal.

The change addresses an observed startup/hotplug stall while the portal scans USB devices. It does not change the controller backend, CPU, GPU, game patch loader or save format. It is not an official Xenia release.

## Runtime configuration

Before launching the custom build, put these values in the active configuration:

```toml
[HID]
hid = "xinput"
enable_hardware_portal = false
```

The setting must be loaded before application setup. A per-game override loaded after setup is too late. The default remains `true`, preserving upstream behavior unless explicitly disabled. Disabling this option also disables physical Skylanders portal support.

## Build scope and provenance

`portal-optional.patch` modifies only `src/xenia/hid/input_system.cc`. `SOURCE-PROVENANCE.json` records the expected upstream and modified source hashes. The manual workflow pins the upstream revision, verifies the source, applies the patch, builds only the application on a standard Windows runner, and uploads an explicit executable/license/receipt file list.

No games, profiles, save files, device reports, personal photos, keys or account credentials belong in this repository or its build inputs. The workflow needs no user-provided secrets and has read-only repository permissions.

## Validation limits

Source review and a successful compilation are not proof of a runtime fix. The resulting build still needs a local test with the USB controller connected before startup and after reconnecting. Preserve the prior executable, configuration and saves before testing. Graphics improvements and the existing ultrawide game patch are separate from this source change.

Upstream sources and context:

- https://github.com/xenia-canary/xenia-canary/tree/4f04bc7cc76b89bfcf7327b597d9770eaaddf935
- https://github.com/libusb/libusb/issues/1163
- https://github.com/libusb/libusb/pull/1882

The upstream BSD license is included in `LICENSE`.
