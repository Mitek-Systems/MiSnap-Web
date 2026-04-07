# MiSnap Web SDK 5.11.0 Documentation

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
├── /integrations                 # Webpack config snippets (see Installation)
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
    └── /wasm                     # Face recognition WebAssembly modules
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

## Getting Help

- Contact your Mitek Account team for licensing
- Visit [Mitek Support](https://www.miteksystems.com/support) for technical assistance
