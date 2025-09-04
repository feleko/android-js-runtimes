# Changelog

## 1.1.0
- Added `verifyLibCppBundled` task (fails build if required ABIs missing `libc++_shared.so`).
- Added configurable `jsc.requiredAbis` property.
- Documentation: troubleshooting section for missing `libc++_shared.so`.

## 1.0.22
- Improved discovery of `libc++_shared.so` for legacy and r23+ NDK layouts.

## 1.0.21
- Ensured NDK install on JitPack + `failIfNoLibCpp` flag.

## 1.0.20 .. 1.0.11 (earlier)
- Progressive improvements to publishing, symbolization, and single JSC packaging.