# COMPONENT_PRELOAD

Preloads SDK components asynchronously for faster capture startup. Call this early in your page lifecycle to reduce latency when users begin capture.

## Signature

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath?: string,
  loadWorkerCrossOrigin?: boolean,
  preloadComponents: string[],
  options: {
    license: string
  }
});
```

## Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mitekSDKPath` | `string` | No | `"./mitekSDK/"` | Path to SDK files |
| `loadWorkerCrossOrigin` | `boolean` | No | `false` | Load worker as Blob URL for cross-origin |
| `preloadComponents` | `string[]` | Yes | - | Components to preload |
| `options.license` | `string` | Yes | - | SDK license key |

## Components

| Component | Description | Files Loaded |
|-----------|-------------|--------------|
| `DOCUMENTS` | ID documents, checks, trailing docs | WASM, worker |
| `SELFIE` | Face detection AI models | Face API models, shards |
| `BARCODE` | PDF417 and QR code scanning | Barcode library |
| `AI_BASED_RTS` | AI-based Real Time Security | RTS models |
| `ALL` | All components | Everything _except_ `AI_BASED_RTS` |

## Examples

### Preload Documents Only

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["DOCUMENTS"],
  options: {
    license: "your-license-here"
  }
});
```

### Preload Multiple Components

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["DOCUMENTS", "BARCODE"],
  options: {
    license: "your-license-here"
  }
});
```

### Cross-Origin Worker Loading

If loading SDK files from a different origin, you must set `loadWorkerCrossOrigin: true`.

**Why is this needed?** Browsers block Web Workers loaded from a different origin due to the [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy). When `loadWorkerCrossOrigin` is enabled, the SDK fetches the worker script and loads it as a [Blob URL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL), bypassing the cross-origin restriction.

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "https://host.example.com/mitekSDK/",
  loadWorkerCrossOrigin: true,
  preloadComponents: ["DOCUMENTS"],
  options: {
    license: "your-license-here"
  }
});
```

### Preload for Selfie with AI-based RTS

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["SELFIE", "AI_BASED_RTS"],
  options: {
    license: "your-license-here"
  }
});
```

## Best Practices

### 1. Always Register Error Handler First

```javascript
// Handle errors before preloading
mitekScienceSDK.on("SDK_ERROR", (err) => {
  if (err.code === 4006) {
    console.error("Invalid license");
  }
});

// Then preload
mitekScienceSDK.cmd("COMPONENT_PRELOAD", config);
```

### 2. Preload on Page Load

```javascript
window.addEventListener("load", () => {
  mitekScienceSDK.on("SDK_ERROR", handleError);
  mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
    preloadComponents: ["DOCUMENTS"],
    options: { license }
  });
});
```

### 3. Only Preload What You Need

```javascript
// Good: Only preload what you'll use
preloadComponents: ["DOCUMENTS"]

// Avoid: Heavy load if you only need documents
preloadComponents: ["ALL"]
```

### 4. Wait for COMPONENT_PRELOAD_COMPLETE Before Capture

The command returns immediately; loading continues in the background. **Recommendation:** Register a listener for `COMPONENT_PRELOAD_COMPLETE` and start capture only after it fires. This ensures components are ready.

```javascript
mitekScienceSDK.on("COMPONENT_PRELOAD_COMPLETE", () => {
  // Preload finished -> Safe to start capture.
  enableCaptureButton();
});

mitekScienceSDK.on("SDK_ERROR", handleError);
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  preloadComponents: ["DOCUMENTS"],
  mitekSDKPath: "./mitekSDK/",
  options: { license }
});
```

If capture is initiated before preload completes on slow connections, the camera may not initialize properly. See [COMPONENT_PRELOAD_COMPLETE](../events/COMPONENT_PRELOAD_COMPLETE.md) for details.

## Error Handling

| Code | Type | Description |
|------|------|-------------|
| 4006 | `INVALID_LICENSE` | License format is invalid |
| 4007 | `INVALID_LICENSE_EXPIRED` | License has expired |
| 4008 | `INVALID_LICENSE_FEATURE` | License missing required feature |
| 5513 | `COMPONENT_LOADER_FAILED` | A requested component (e.g. DOCUMENTS, BARCODE, SELFIE, AI_BASED_RTS) failed to load during preload |

## Related

- [COMPONENT_PRELOAD_COMPLETE](../events/COMPONENT_PRELOAD_COMPLETE.md) - Event fired when preload finishes; wait for it before sending capture commands
- [CAPTURE_AND_PROCESS_FRAME](./CAPTURE_AND_PROCESS_FRAME.md) - Start capture after preload
- [Installation Guide](../INSTALLATION.md) - License setup
