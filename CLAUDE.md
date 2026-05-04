# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`mobile_scanner` là Flutter plugin đa nền tảng cho quét barcode/QR code, hỗ trợ Android, iOS, macOS và Web. Version hiện tại: 7.0.1.

- **Android**: CameraX + ML Kit Barcode Scanning
- **iOS/macOS**: AVFoundation + Vision API (shared Darwin codebase)
- **Web**: ZXing JavaScript library

---

## Commands

### Lint & Format
```bash
flutter analyze
dart format --set-exit-if-changed .
```

### Tests
```bash
# Toàn bộ test suite
flutter test --coverage

# Chạy một test file cụ thể
flutter test test/enums/barcode_format_test.dart
```

### Example App
```bash
cd example
flutter run
```

---

## Architecture

### Platform Interface Pattern

Plugin dùng `plugin_platform_interface` để tách biệt API khỏi implementation:

```
MobileScannerPlatform (abstract)
├── MobileScannerMethodChannel  ← default (Android/iOS/macOS)
└── MobileScannerWebPlugin      ← Web
```

### Luồng dữ liệu chính

```
Native Camera
    ↓ (MethodChannel / EventChannel)
MobileScannerMethodChannel
    ↓
MobileScannerController (ValueNotifier<MobileScannerState>)
    ↓
MobileScanner (widget) → onDetect callback
```

**Controller** (`lib/src/mobile_scanner_controller.dart`): Extends `ValueNotifier<MobileScannerState>`, quản lý lifecycle, expose `barcodes` stream.

**Widget** (`lib/src/mobile_scanner.dart`): Implements `WidgetsBindingObserver` để pause/resume theo app lifecycle.

### Key Source Directories

| Path | Mô tả |
|------|-------|
| `lib/src/enums/` | 11 enum types (BarcodeFormat, DetectionSpeed, TorchState...) |
| `lib/src/objects/` | 18 data models (Barcode, BarcodeCapture, ContactInfo, WiFi...) |
| `lib/src/method_channel/` | MethodChannel implementation + platform interface |
| `lib/src/overlay/` | BarcodeOverlay, ScanWindowOverlay UI components |
| `lib/src/web/` | Web implementation via JS interop |
| `android/src/main/kotlin/dev/steenbakker/mobile_scanner/` | Kotlin native code |
| `darwin/mobile_scanner/Sources/mobile_scanner/` | Swift native code (iOS + macOS shared) |

### Native Entry Points

- **Android**: `MobileScannerPlugin.kt` → `MobileScannerHandler.kt` → `MobileScanner.kt` (CameraX logic)
- **iOS/macOS**: `MobileScannerPlugin.swift` (AVFoundation + Vision API)

---

## Platform Setup Notes

### Android
- compileSdk 35, minSdk 21, Kotlin 1.8.0
- ML Kit bundled by default; dùng unbundled qua gradle property:
  ```
  dev.steenbakker.mobile_scanner.useUnbundled=true
  ```

### iOS
- iOS 12.0+, Swift 5.0
- Bắt buộc `NSCameraUsageDescription` trong Info.plist
- Dùng Vision API (không phải ML Kit)

### macOS
- macOS 10.14+
- Grant camera permission trong Xcode Signing & Capabilities

### Web
- ZXing script tự động load; có thể override URL:
  ```dart
  MobileScannerPlatform.instance.setBarcodeLibraryScriptUrl(url)
  ```

---

## Code Quality

- Linter: `very_good_analysis` (xem `analysis_options.yaml`)
- Không dùng bang operator `!` nếu có thể tránh
- Tất cả public API phải có dartdoc

---

## Commit Convention

Dùng Conventional Commits: `fix:`, `feat:`, `doc:`, `chore:`, `refactor:`
