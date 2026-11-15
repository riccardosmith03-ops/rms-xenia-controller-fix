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

`portal-optional.patch` modifies only `src/xenia/hid/input_system.cc`. `SOURCE-PROVENANCE.json` records the expected upstream and modified source hashes. The manual workflow pins the upstream revision, verifies the source and dependencies, runs the host validation below, applies the production patch, and builds the application on a standard Windows runner. It uploads only the executable, license and public build receipt.

No games, profiles, save files, device reports, personal photos, keys or account credentials belong in this repository or its build inputs. The workflow needs no user-provided secrets and has read-only repository permissions.

## Host validation and the Windows fixture expectation

The first build compiled the application successfully but found one failure in the untouched upstream host suite: `PhysicalHeap vE0000000 alignment`, section `alloc with alignment larger than page_size is rejected` (`physical_heap_test.cc:173`). The stock suite reported 80 passing cases out of 81 and 2,231 passing assertions out of 2,232. On Windows, the host-view offset can compensate for the physical translation offset; the fixture's unconditional rejection expectation does not describe that mapping.

This workflow first runs the complete suite on byte-exact, unmodified upstream source and captures Catch2 XML plus the exit code. It proceeds only if all 81 stock cases pass, or if that exact single known failure and all counts reproduce. Any other result stops the job.

For the reproduced Windows failure, the workflow temporarily changes only that test section to check the actual host-view offset, physical and host alignment, and allocation bounds. It then reruns the entire suite with no test filters or skipped cases: all 81 cases must pass, the process must exit successfully, and there must be zero failed or expected-failure assertions and no failing expressions. Passing assertions must be at least the 2,231 passing checks already observed in the stock baseline. Catch2 section abort/re-entry can change the total assertion count, so the workflow records the actual counts rather than requiring a predicted fixed total. The fixture is restored to its exact upstream bytes before the portal patch and application build. If the stock suite already passes, no test override is used.

Each suite prints its exit code, actual counts, failed case names and failed expressions before the result gate. The known failing stock baseline still requires its exact 80/1 case and 2,231/1 assertion counts and assertion identity. The build receipt records both results, the original/corrected fixture hashes and restoration, and the unchanged production `memory.cc` hash. No production memory/allocator behavior is changed to satisfy a test. Only `input_system.cc` differs in the resulting application source.

## Validation limits

Source review and a successful compilation are not proof of a runtime fix. The resulting build still needs a local test with the USB controller connected before startup and after reconnecting. Preserve the prior executable, configuration and saves before testing. Graphics improvements and the existing ultrawide game patch are separate from this source change.

Upstream sources and context:

- https://github.com/xenia-canary/xenia-canary/tree/4f04bc7cc76b89bfcf7327b597d9770eaaddf935
- https://github.com/libusb/libusb/issues/1163
- https://github.com/libusb/libusb/pull/1882

The upstream BSD license is included in `LICENSE`.
