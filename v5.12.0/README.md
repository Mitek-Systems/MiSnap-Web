# MiSnap Web SDK 5.12.0 Documentation

## Viewing the Documentation

To view this documentation with full navigation and search:

```bash
cd documentation
npm run serve
# Or yarn
yarn run serve
```

This will open the documentation at [http://localhost:3001](http://localhost:3001)

Alternatively, you can browse the markdown files directly in this directory.

## Overview

The MiSnap Web SDK is a JavaScript library for capturing and processing identity documents, selfies, barcodes, and voice samples directly in the browser. It provides real-time image quality assessment (IQA) with guided feedback to help users capture optimal images for identity verification workflows.

### Key Capabilities

- **Document Capture** - Driver's licenses, passports, ID cards, checks
- **Selfie Capture** - Face detection with smile detection, countdown timer, or IQA-based capture
- **Barcode Scanning** - PDF417 (North American DL/ID) and QR codes
- **Voice Capture** - Audio recording for voice biometrics
- **Real-Time Security (RTS)** - AI-based injection attack detection for selfies

## Quick Links

| Document | Description |
|----------|-------------|
| [Quick Start](./QUICK_START.md) | Get your first capture working in minutes |
| [Installation](./INSTALLATION.md) | Browser requirements, setup, and licensing |
| [Release Notes](./RELEASE_NOTES.md) | Version history and changelog |
| [Known Issues](./KNOWN_ISSUES.md) | Device compatibility and limitations |
| [Terms and Conditions](./TERMS_AND_CONDITIONS.md) | SDK usage terms and conditions |

### API Reference

| Section | Description |
|---------|-------------|
| [Commands](./commands/README.md) | SDK methods for capture and control |
| [Events](./events/README.md) | Event handlers for SDK communication |
| [Configuration](./configuration/README.md) | Options for customizing capture behavior |

### Guides

| Guide | Description |
|-------|-------------|
| [Selfie Capture](./guides/selfie-capture.md) | Smile detection, countdown, AI-based RTS |
| [IDLive to MiSnap Transition](./guides/idlive-to-misnap-transition.md) | Migrate from IDLive Web Capture to MiSnap for AI-based RTS |
| [Barcode Scanning](./guides/barcode-scanning.md) | PDF417 and QR code capture |
| [MRZ Redaction](./guides/mrz-redaction.md) | Dutch BSN redaction for passports |
| [Troubleshooting](./guides/troubleshooting.md) | Known issues and device compatibility |

## Supported Browsers

| Browser | Minimum Version
|---------|-----------------|
| Chrome | 70+ |
| Chrome (Desktop) | 70+ |
| Safari | 14+ |
| Firefox | 70+ |
| Edge | 70+ |
| Samsung Internet | 7.2-8.2, 9.2+ |

## Technology Requirements

The SDK requires the following browser APIs:

- **[MediaStream API](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream)** - `getUserMedia()` for camera/microphone access
- **[MediaRecorder API](https://developer.mozilla.org/en-US/docs/Web/API/MediaRecorder)** - For video/audio recording
- **[WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)** - For document processing
- **[Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API)** - For background processing
- **[WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API)** - For selfie auto-processing
- **HTTPS** - Required for camera/microphone permissions (localhost exempt)

## Package Contents

```
/
├── index.html                    # Sample demo page
├── mitek-science-sdk.js          # Core SDK interface
├── mitek-science-sdk.d.ts        # TypeScript entry point (global `mitekScienceSDK` + type re-exports)
├── /types                        # TypeScript declarations (commands, options, events, API)
├── /demo
│   ├── mitek-sdk-demo.js         # Demo JavaScript
│   ├── mitek-sdk-demo.css        # Demo styles
│   └── /images                   # Test images
├── /documentation                # Markdown documentation
├── /images/guide_images          # Sample SVG guide images
└── /mitekSDK
    ├── app.css                   # SDK styles (link required)
    ├── license_verifier_bin.js   # License verifier loader
    ├── *.bundle.js               # Core dependencies
    ├── *.wasm                    # WebAssembly modules (includes license verifier and other binaries)
    ├── *.worker.js               # Web workers
    ├── <face models>             # Face recognition models
    ├── /wasm                     # Face recognition WebAssembly modules
    └── /idlive-capture           # AI-based RTS assets (production package)
```

## Basic Usage

```javascript
// 1. Include the SDK
<script src="mitek-science-sdk.js"></script>
<link rel="stylesheet" href="mitekSDK/app.css" />

// 2. Preload components
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["DOCUMENTS"],
  options: { license: "your-license-here" }
});

// 3. Listen for events
mitekScienceSDK.on("SDK_ERROR", (err) => console.error(err));
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    // Handle capture response
  }
});

// 4. Start capture
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  mitekSDKPath: "./mitekSDK/",
  options: { license: "your-license-here" }
});
```

## Browser-Loaded SDK Files

The table below lists SDK assets the browser fetches. Base files are included via page `<script>` and `<link>` tags. Additional files load asynchronously when you call `COMPONENT_PRELOAD` (or on first capture if preload was skipped). Voice and license-check assets load on demand when their respective command is invoked. 

> **Note**: Gzip sizes approximate the network transfer size when the server sends `Content-Encoding: gzip`. Your compression method (e.g. Brotli) and results may vary.

| File | Size (disk) | Size (gzip) | Notes |
|------|-------------|-------------|-------|
| `mitek-science-sdk.js` | 420 kB | 114 kB | Always loaded |
| `mitekSDK/app.css` | 13 kB | 4 kB | Always loaded |
| `mitekSDK/mitekMobileWeb.worker.js` | 19 kB | 6 kB | DOCUMENTS |
| `mitekSDK/mitekMobileWeb.wasm` | 617 kB | 231 kB | DOCUMENTS |
| `mitekSDK/mwb.bundle.js` | 1682 kB | 633 kB | BARCODE |
| `mitekSDK/mitek_license.png` | 20 kB | 20 kB | BARCODE; image barcode license only |
| `mitekSDK/faceapi.bundle.js` | 2758 kB | 684 kB | SELFIE |
| `mitekSDK/tiny_face_detector_model-shard1` | 189 kB | 152 kB | SELFIE |
| `mitekSDK/tiny_face_detector_model-weights_manifest.json` | 3 kB | 1 kB | SELFIE |
| `mitekSDK/face_landmark_68_tiny_model-shard1` | 75 kB | 61 kB | SELFIE |
| `mitekSDK/face_landmark_68_tiny_model-weights_manifest.json` | 4 kB | 1 kB | SELFIE |
| `mitekSDK/face_expression_model-shard1` | 322 kB | 275 kB | SELFIE |
| `mitekSDK/face_expression_model-weights_manifest.json` | 6 kB | 1 kB | SELFIE |
| `mitekSDK/wasm/tfjs-backend-wasm.wasm` | 304 kB | 116 kB | SELFIE; one of (browser-dependent) |
| `mitekSDK/wasm/tfjs-backend-wasm-simd.wasm` | 415 kB | 134 kB | SELFIE; one of (browser-dependent) |
| `mitekSDK/wasm/tfjs-backend-wasm-threaded-simd.wasm` | 425 kB | 139 kB | SELFIE; one of (browser-dependent) |
| `mitekSDK/idlive-capture/main.js` | 409 kB | 118 kB | AI_BASED_RTS; **prod build only** |
| `mitekSDK/idlive-face-capture-web.bundle.js` | 1827 kB | 566 kB | AI_BASED_RTS |
| `mitekSDK/idlive-capture/encryption_module.wasm` | 689 kB | 206 kB | AI_BASED_RTS; **prod build only** |
| `mitekSDK/idlive-capture/css/style.css` | 1 kB | < 1 kB | AI_BASED_RTS; **prod build only** |
| `mitekSDK/mitek-mobile-web-vbm.bundle.js` | 2 kB | 1 kB | [CAPTURE_AND_PROCESS_VOICE](./commands/CAPTURE_AND_PROCESS_VOICE.md) |
| `mitekSDK/license_verifier_bin.js` | 64 kB | 25 kB | [`getLicenseInfo()`](./INSTALLATION.md#checking-license-status) |
| `mitekSDK/license_verifier_bin.wasm` | 130 kB | 52 kB | [`getLicenseInfo()`](./INSTALLATION.md#checking-license-status) |

See also: [Including the SDK](./INSTALLATION.md#including-the-sdk), [COMPONENT_PRELOAD](./commands/COMPONENT_PRELOAD.md).

## Getting Help

- Contact your Mitek Account team for licensing
- Visit [Mitek Support](https://www.miteksystems.com/support) for technical assistance
