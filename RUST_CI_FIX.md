# OpenWrt x86 Rust CI Build Fix - Summary

## Issue
The GitHub Actions workflow was failing with:
```
thread 'main' panicked at src/bootstrap/src/core/config/config.rs:1681:21:
`llvm.download-ci-llvm` cannot be set to `true` on CI. Use `if-unchanged` instead.
```

This occurred when building the Rust package for OpenWrt, as the default Rust package configuration had `llvm.download-ci-llvm = true`, which is incompatible with CI environments.

## Solution Overview

The fix implements a multi-layered approach to ensure the Rust package is properly configured:

### 1. Custom Patch File
**File**: `openwrt-patches/feeds/packages/lang/rust/900-ci-llvm-download-fix.patch`

This patch modifies the Rust package's `config.toml.in` to change:
- `llvm.download-ci-llvm = true` → `llvm.download-ci-llvm = "if-unchanged"`
- `[llvm] download-ci-llvm = true` → `[llvm] download-ci-llvm = "if-unchanged"`

### 2. Workflow Modifications
**File**: `.github/workflows/build-openwrt.yml`

#### a) Enhanced Patch Step (`patch-rust`)
- Copies custom patches from repository to the build feeds directory
- Patches all patch files in the Rust feeds directory that contain the problematic setting
- Uses flexible sed patterns to handle various whitespace and quote formats

#### b) Verification Step (`verify-rust-patch`)
- Lists the contents of the patches directory
- Searches for any remaining `true` values (should be none)
- Searches for `if-unchanged` values (confirms patch was applied)

#### c) Enhanced Build Step (`build`)
- Sets `RUSTC_LLVM_DOWNLOAD=if-unchanged` environment variable
- Sets `RUST_BOOTSTRAP_CONFIG` with the proper configuration
- Exports `CARGO_BUILD_JOBS` setting

## How It Works

1. **During build preparation**:
   - OpenWrt source and feeds are cloned
   - Custom Rust patches are copied to the feeds directory
   - Both custom patches and any existing Rust patches are verified/patched to use `if-unchanged`

2. **During verification**:
   - The workflow checks that all patches have been properly updated
   - Provides diagnostic output showing what was patched

3. **During build**:
   - The Rust package is compiled with the corrected configuration
   - Environment variables provide additional safeguards
   - The bootstrap process now accepts the `if-unchanged` setting on CI

## Key Files Modified
- `.github/workflows/build-openwrt.yml` - Workflow with new patch and build steps
- `openwrt-patches/feeds/packages/lang/rust/900-ci-llvm-download-fix.patch` - Rust package patch
- `openwrt-patches/feeds/packages/lang/rust/README.md` - Documentation

## Testing
When the workflow runs, it will:
1. Copy patches from the repository to the build environment
2. Verify patches are correctly applied
3. Build OpenWrt with the corrected Rust configuration
4. Generate firmware images successfully

If the build still fails with the same Rust error, check:
- That the patch files are in the correct location in the repository
- That the patch patterns match the actual content of the Rust package source
- That the workflow steps are executing in the correct order
