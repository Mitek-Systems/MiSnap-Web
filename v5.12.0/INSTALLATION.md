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

## SDK Build Variants

Beginning with MiSnap version 5.12.0, Mitek ships two SDK package variants: `dev` and `prod`. This can be helpful for integrating and testing with AI-based RTS, which has frontend protections that prevent using browser DevTools. You can use the `dev` build for local development if you need to use DevTools.

| Package suffix | Production use | CSP requirement | DevTools |
|----------------|----------------|-----------------|----------------|
| `-dev` | **Never** | `unsafe-eval` | Allowed |
| `-prod` | Yes | `wasm-unsafe-eval` | Blocked |

> **IMPORTANT:** Do **not** deploy the **development** (`-dev`) package to a production environment. Use **`-prod`** instead.

**Choosing a variant:**

- **`-prod`**: For all production deployments. Requires the `mitekSDK/idlive-capture/` directory in your deployment.
- **`-dev`**: For local integration and debugging only.

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

> **Note:** Versions prior to v5.9.0 had embedded styles, which caused CSP `style-src` violations. The separate CSS file resolves this.

## Including the SDK

The SDK ships as a UMD bundle (`mitek-science-sdk.js`). Choose **one** loading method below — both work with or without a JS bundler.

| Method | When to use |
|--------|-------------|
| [Script tag](#option-a-script-tag) | Load from HTML `<script>` tag. Keeps the SDK outside your app bundle. |
| [Bundler import](#option-b-bundler-import) | Include the SDK in your app build via an import |

### Deploy these files together

| Path | Purpose |
|------|---------|
| `mitek-science-sdk.js` | Main SDK bundle |
| `mitek-science-sdk.d.ts` | TypeScript declarations |
| `types/` | Supporting declaration files |
| `mitekSDK/` | Workers, WASM, models, `app.css` |

Also link `mitekSDK/app.css` (see [CSS Stylesheet](#css-stylesheet-required)).

**Important:** Do not bundle files inside `mitekSDK/`. They are loaded at runtime. Configure the `mitekSDKPath` in [SDK commands](./commands/README.md).

### Option A: Script tag

Load the SDK with a `<script>` tag in your HTML page or in the HTML template your bundler serves (for example, `index.html`). This exposes the global `mitekScienceSDK` object.

```html
<link rel="stylesheet" href="mitekSDK/app.css" />
<script src="mitek-science-sdk.js"></script>
```

```typescript
mitekScienceSDK.on("SDK_ERROR", (err: unknown) => {
  console.error(err);
});

mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["DOCUMENTS"],
  options: { license: "your-license" },
});
```

### Option B: Bundler import

Import the SDK in your application entry so your bundler includes it in your build.

```typescript
// Bundler will include this in your app and register the global mitekScienceSDK
import "./mitek-science-sdk.js";

mitekScienceSDK.on("SDK_ERROR", (err: unknown) => {
  console.error(err);
});

mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["DOCUMENTS"],
  options: { license: "your-license" },
});
```

Use a path relative to your source file, or configure your bundler to resolve `./mitek-science-sdk.js` from your SDK asset directory.

The `mitekSDK/` directory must still be deployed as static assets and referenced via `mitekSDKPath`. Only `mitek-science-sdk.js` is pulled in through the module graph.

> **Note:** `mitek-science-sdk.js` is not a native ES module. The import runs the UMD wrapper, which assigns `window.mitekScienceSDK`.

### TypeScript

The type declaration files ship next to `mitek-science-sdk.js`. Import **types** from the JS path — TypeScript resolves the adjacent `.d.ts`:

```typescript
import type {
  MitekCaptureOptionsForDoc,
  MitekCmdOptions,
  MitekCmdParams,
  MitekCmdParamsFor,
  MitekDocType,
  MitekDocumentCaptureOptions,
  MitekEventCallback,
  MitekFrameCaptureResult,
  MitekFrameProcessingFeedback,
  MitekSelfieCaptureOptions,
  MitekSdkError,
} from "./mitek-science-sdk.js";
```

For events that pass data to listeners, `on()` infers the callback payload type from the event name (for example `FRAME_CAPTURE_RESULT` event payload → `MitekFrameCaptureResult` type). 

Similarly, `cmd()` infers the parameters object from the command name. For capture commands, `options` is also narrowed by `documentType`. Use `MitekCmdParamsFor<"COMMAND_NAME">` or `MitekCaptureOptionsForDoc<"SELFIE">` when you need an explicit type.

Command names, event names, and document types are string literals (e.g. `"DL_FRONT"`, `"COMPONENT_PRELOAD"`). Annotate with the types above:

```typescript
const documentType: MitekDocType = "DL_FRONT";
const options: MitekCmdOptions = { license: "your-license" };

const params: MitekCmdParamsFor<"CAPTURE_AND_PROCESS_FRAME"> = {
  mode: "AUTO_CAPTURE",
  documentType,
  mitekSDKPath: "./mitekSDK/",
  options,
};
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
