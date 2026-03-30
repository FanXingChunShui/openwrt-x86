# OpenWrt Rust Build Fix

This directory contains patches for the OpenWrt Rust package to fix CI compatibility issues.

## Problem

The default Rust package in OpenWRT builds sets `llvm.download-ci-llvm = true` in the bootstrap configuration. However, Rust's bootstrap system rejects this setting on CI environments (detected via the `CI` environment variable), requiring the value to be `"if-unchanged"` instead.

Error:
```
thread 'main' panicked at src/bootstrap/src/core/config/config.rs:1681:21:
`llvm.download-ci-llvm` cannot be set to `true` on CI. Use `if-unchanged` instead.
```

## Solution

The patch file `900-ci-llvm-download-fix.patch` modifies the Rust package's `config.toml.in` to use `"if-unchanged"` instead of `true` for both:
- `llvm.download-ci-llvm` 
- `[llvm] download-ci-llvm`

This allows the Rust bootstrap to work correctly in CI environments.

## Application

These patches are automatically applied during the OpenWrt build process if placed in the correct directory structure:
- `feeds/packages/lang/rust/patches/`

The patches are copied into this location by the GitHub Actions workflow during the build.
