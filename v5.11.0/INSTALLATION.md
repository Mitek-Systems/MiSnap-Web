# Installation Guide

Complete setup instructions for the MiSnap Web SDK.

## Browser Requirements

### Supported Browsers

| Browser | Minimum Version
|---------|-----------------|
| Chrome | 70+ |
| Chrome (Desktop) | 70+ |
| Safari | 14+ |
| Firefox | 70+ |
| Edge | 70+ |
| Samsung Internet | 7.2-8.2, 9.2+ |

### Required Web APIs

The SDK requires these browser APIs:

| API | Purpose |
|-----|---------|
| [MediaStream](https://developer.mozilla.org/en-US/docs/Web/API/MediaStream) (`getUserMedia`) | Camera and microphone access |
| [MediaRecorder](https://developer.mozilla.org/en-US/docs/Web/API/MediaRecorder) | Video/audio recording |
| [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly) | Document processing |
| [Web Workers](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) | Background processing |
| [WebGL](https://developer.mozilla.org/en-US/docs/Web/API/WebGL_API) | Selfie auto-processing |

### Camera Requirements

- Minimum resolution: **1080p** for documents, **720p** for selfies
- HTTPS required for camera access (localhost exempt for development)

### Video Recording Requirements

- Requires MediaRecorder API
- iOS: version 14.3+
- Chrome: version 89+

## Web Server Configuration

### MIME Types

Configure your server to serve these MIME types:

| Extension | MIME Type | Purpose |
|-----------|-----------|---------|
| `.wasm` | `application/wasm` | Document/barcode processing |
| Selfie shards | `application/octet-stream` | Selfie capture models |

### Example: nginx

```nginx
types {
    application/wasm wasm;
    application/octet-stream bin data;
}
```

### Example: Apache

```apache
AddType application/wasm .wasm
```

> **Note:** These are simplified examples. Consult your web server documentation for the correct configuration syntax and location for your setup.

## CSS Stylesheet (Required)

Starting with v5.9.0, you must link the SDK stylesheet:

### HTML Link

```html
<head>
  <link rel="stylesheet" href="mitekSDK/app.css" />
</head>
```

### Module Import (Bundler)

```javascript
import "path/to/mitekSDK/app.css";
```

> **Note:** Previous versions embedded styles in a `<style>` tag, which caused CSP `style-src` violations. The separate CSS file resolves this.

## Including the SDK

### Script Tag

```html
<script type="text/javascript" src="mitek-science-sdk.js"></script>
```

### ES Module

```javascript
import * as mitekScienceSDK from "./mitek-science-sdk.js";
```

### CommonJS

```javascript
const mitekScienceSDK = require("./mitek-science-sdk.js");
```

The SDK can be bundled using any modern bundler (Webpack, Vite, Rollup, Parcel, esbuild, etc.).

### Bundling with Webpack

When you bundle `mitek-science-sdk.js` with webpack, you may see:

- `Critical dependency: the request of a dependency is an expression`

The SDK includes dependencies that contain code that uses dynamic `require()` for Node-only paths. Webpack cannot statically resolve these. In the browser that code path never runs, so the warning is safe to suppress.

**How to suppress:**

Add the following to your config under `ignoreWarnings`:

```javascript
ignoreWarnings: [
  /Critical dependency: the request of a dependency is an expression/,
],
```

## License Installation

A valid license is required for all capture operations.

### Obtaining a License

1. Contact your Mitek Account team
2. Provide the domain names where the SDK will run
3. You'll receive a text license that can be passed to the SDK.

> **Note:** Older versions of the SDK (< 5.3.0) used an image license for barcode scanning (`mitek_license.png`). If the barcode license is missing or invalid, scanning will produce asterisk-filled data.

### Installing the License

Pass the license in the `options` object:

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["DOCUMENTS"],
  options: {
    license: "eyJhbGciOiJIUzI1NiIs..."  // Your license string
  }
});
```

### License Types

| Type | Format | Includes |
|------|--------|----------|
| Text (v5.3.0+) | Base64 string | SDK + Barcode combined |
| PNG (legacy) | Image file | Barcode only |

### Text License (Recommended)

As of v5.3.0, a single text license covers both SDK and barcode functionality:

```javascript
options: {
  license: "your-combined-text-license"
}
```

### PNG Barcode License (Legacy)

If using a separate PNG license for barcode:

1. Place `mitek_license.png` in `./mitekSDK/`
2. The SDK automatically loads it from that path

> **Note:** When upgrading to text license, remove any existing PNG license files.

### Checking license status

Starting with v5.11.0, you can validate a text (base64) SDK license **without** starting capture or preloading components. This is useful for setup checks or support.

```javascript
const status = await mitekScienceSDK.getLicenseInfo(licenseBase64);
```

| Return value | Meaning |
|--------------|---------|
| `VALID` | License is usable for the **current hostname**. |
| `EXPIRED` | License has expired but is still within the **grace period**. |
| `DISABLED` | License is past expiry and grace period. |
| `NOT_VALID` | License could not be decoded or verified, input is not valid base64, or the verifier failed to load or run. |
| `NOT_VALID_HOSTNAME` | Current hostname is not allowed by the license. |
| `PLATFORM_NOT_SUPPORTED` | This license does not allow the **web** platform. |

**Behavior:**

- Pass the same base64 string you use in `options.license` for `COMPONENT_PRELOAD` and capture commands.
- The SDK evaluates the license for the **current page hostname**. You may see `NOT_VALID_HOSTNAME` if the page is opened from another host that is not allowed by the license. **NOTE**: `localhost` is valid for local development.
- Malformed input or loading failures resolve to `NOT_VALID`.

**Deployment:** The `getLicenseInfo` check loads `license_verifier_bin.js` and `license_verifier_bin.wasm` from your SDK asset directory (the same base path as your other `mitekSDK` files). Ensure those files are deployed next to your main SDK bundle; if the verifier fails to load, you will get `NOT_VALID`.

### License Errors

| Code | Type | Meaning |
|------|------|---------|
| 4006 | `INVALID_LICENSE` | License format is invalid |
| 4007 | `INVALID_LICENSE_EXPIRED` | License has expired |
| 4008 | `INVALID_LICENSE_FEATURE` | License missing required feature |
| 4040 | `INVALID_BARCODE_LICENSE` | No barcode license found |

## Component Preloading

Preload components for faster user experience:

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["DOCUMENTS", "SELFIE"],
  options: { license: "your-license" }
});
```

### Available Components

| Component | Description |
|-----------|-------------|
| `DOCUMENTS` | ID documents, checks, trailing documents |
| `SELFIE` | Face detection models |
| `BARCODE` | PDF417 and QR code scanning |
| `AI_BASED_RTS` | AI-based Real Time Security for selfies |
| `ALL` | All components _except_ `AI_BASED_RTS`. Use selectively. |

### Cross-Origin Worker Loading

If loading SDK files from a different origin, you must set `loadWorkerCrossOrigin: true`.

**Why is this needed?** Browsers block Web Workers loaded from a different origin due to the [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy). When `loadWorkerCrossOrigin` is enabled, the SDK fetches the worker script and loads it as a [Blob URL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL), bypassing the cross-origin restriction.

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "https://host.example.com/mitekSDK/",
  loadWorkerCrossOrigin: true,
  preloadComponents: ["DOCUMENTS"],
  options: { license: "your-license" }
});
```

> **Note:** The server hosting the SDK files must return proper CORS headers (`Access-Control-Allow-Origin`) for the SDK to fetch the worker script.

## Next Steps

- [Quick Start](./QUICK_START.md) - First capture tutorial
- [Commands Reference](./commands/README.md) - Available SDK commands
- [Configuration Options](./configuration/README.md) - Customize behavior
