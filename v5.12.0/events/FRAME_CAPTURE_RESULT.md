# FRAME_CAPTURE_RESULT

Triggered when a capture session completes, either with a successful image or a failure.

For **auto capture**, [`FRAME_CAPTURE_COMPLETE`](./FRAME_CAPTURE_COMPLETE.md) fires first (empty payload) when the session ends but before any optional success-message delay and before this event.

## Signature

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  // result: { request: string, response: object }
});
```

## TypeScript

Import `MitekFrameCaptureResult` from `mitek-science-sdk.js` (see [Installation — TypeScript](../INSTALLATION.md#typescript-both-options)). The `on("FRAME_CAPTURE_RESULT", …)` callback is typed as `(result: MitekFrameCaptureResult) => void`. Narrow on `result.response.status` (`"success"` | `"failure"`) for success vs failure fields.

## Result Object

| Property | Type | Description |
|----------|------|-------------|
| `request` | `string` | Document type requested |
| `response` | `object` | Capture response (see below) |

## Response Object - Success

| Property | Type | Description |
|----------|------|-------------|
| `docType` | `string` | Document type captured |
| `status` | `string` | `"success"` |
| `mode` | `string` | `"AUTO"`, `"MANUAL"`, or `"DIRECT"` |
| `imageData` | `string` | Base64 JPEG data URL |
| `fourCornerA` | `object` | `{ x, y }` top-left corner |
| `fourCornerB` | `object` | `{ x, y }` top-right corner |
| `fourCornerC` | `object` | `{ x, y }` bottom-right corner |
| `fourCornerD` | `object` | `{ x, y }` bottom-left corner |
| `glareTop` | `number` | Glare position (0 if none) |
| `glareBottom` | `number` | Glare position (0 if none) |
| `glareLeft` | `number` | Glare position (0 if none) |
| `glareRight` | `number` | Glare position (0 if none) |
| `warnings` | `object` | `{ status: "success" }` |
| `mibiData` | `object` | Internal telemetry |
| `ClientActivityDataLog` | `object` | Session info (v5.3.0+) |
| `RTS` | `string` | Encrypted Real Time Security data (v5.4.0+) |

### Passport-Specific Properties

| Property | Type | Description |
|----------|------|-------------|
| `mrz` | `object` | `{ countryCode, hasOptionalData }` |

### Barcode-Specific Properties

| Property | Type | Description |
|----------|------|-------------|
| `code` | `string` | Extracted barcode data |
| `type` | `string` | `PDF417` or `QR` |
| `bytes` | `array` | Raw barcode byte array |

## Response Object - Failure

The response object for a failure will contain the same fields as the success object, _except_:

- `imageData`  is missing
- `failedImage` is added
- `warnings` contains a `code` and `key`
- `error` is present, only if an `SDK_ERROR` occurred

| Property | Type | Description |
|----------|------|-------------|
| `failedImage` | `string` | Base64 JPEG of failed frame |
| `warnings` | `object` | `{ status, code, key }` |
| `error` | `object` | Optional, if `SDK_ERROR` occurred |

## Examples

### Basic Handler

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    // Show captured image
    document.getElementById("preview").src = result.response.imageData;
    
    // Send to server
    sendToServer(result.response.imageData);
  } else {
    console.log("Capture failed:", result.response.warnings.key);
    showRetryOption();
  }
});
```

### Success - Driver's License

```javascript
{
  request: "DL_FRONT",
  response: {
    docType: "DL_FRONT",
    status: "success",
    mode: "AUTO",
    imageData: "data:image/jpeg;base64,/9j/4QQ+RXhpZgAATU0AKg...",
    fourCornerA: { x: 382, y: 154 },
    fourCornerB: { x: 1148, y: 157 },
    fourCornerC: { x: 1148, y: 655 },
    fourCornerD: { x: 372, y: 655 },
    glareBottom: 0,
    glareLeft: 0,
    glareRight: 0,
    glareTop: 0,
    warnings: { status: "success" },
    mibiData: { Platform: "Web", MibiVersion: "2.2", ... },
    ClientActivityDataLog: { ... },
    RTS: "WyJ3bFBnc..."
  }
}
```

### Success - Passport with MRZ

```javascript
{
  request: "PASSPORT",
  response: {
    docType: "PASSPORT",
    status: "success",
    mode: "AUTO",
    imageData: "data:image/jpeg;base64,...",
    mrz: {
      countryCode: "NLD",
      hasOptionalData: true
    },
    // ...other properties
  }
}
```

### Success - Selfie

```javascript
{
  request: "SELFIE",
  response: {
    docType: "SELFIE",
    status: "success",
    mode: "AUTO",
    imageData: "data:image/jpeg;base64,...",
    warnings: { status: "success" },
    mibiData: { ... }
  }
}
```

### Failure

```javascript
{
  request: "CHECK_FRONT",
  response: {
    docType: "CHECK_FRONT",
    status: "failure",
    mode: "MANUAL",
    failedImage: "data:image/jpeg;base64,...",
    warnings: {
      status: "failure",
      code: "NF",
      key: "MITEK_ERROR_FOUR_CORNER"
    },
    // ...other properties
  }
}
```

## ClientActivityDataLog (v5.3.0+)

Structured session data for client use:

```javascript
{
  ClientActivityDataLog: {
    DeviceInfo: {
      Type: "DESKTOP",
      UserAgent: "Mozilla/5.0...",
      ID: ""
    },
    SessionInfo: {
      "0": {
        Type: "Document",
        Mode: "Auto",
        JpegQuality: 90,
        AutoTries: 1,
        ManualTries: 0,
        DirectTries: 0,
        DocType: "ID_Front",
        MrzExtracted: false,
        TotalDuration: 10764,
        DeviceOrientation: "Landscape",
        DocumentOrientation: "Landscape",
        Warnings: [],
        ClassificationType: "",
        OptionalDataRedacted: false
      }
    }
  }
}
```

## Handling Netherlands Passports

For Dutch passports, check if QR code scanning is needed:

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success" && 
      result.response.docType === "PASSPORT") {
    
    const { countryCode, hasOptionalData } = result.response.mrz || {};
    
    if (countryCode === "NLD" && !hasOptionalData) {
      // Need to scan QR code for BSN
      startQRCapture();
    } else {
      // No QR needed
      proceedToNextStep();
    }
  }
});
```

## Best Practices

1. **Only submit successful images** - Don't send a `failedImage` as it's highly likely to fail in the backend
2. **Check status first** - Always verify `response.status === "success"`
3. **Use ClientActivityDataLog** - Prefer this over mibiData for client analytics

## Event Sequence

```
CAMERA_DISPLAY_STARTED
         ↓
FRAME_PROCESSING_STARTED
         ↓
FRAME_PROCESSING_FEEDBACK (continuous)
         ↓
FRAME_CAPTURE_COMPLETE  ← AUTO_CAPTURE only (see FRAME_CAPTURE_COMPLETE)
         ↓
FRAME_CAPTURE_RESULT    ← You are here
```

For **manual** or **direct** capture, `FRAME_CAPTURE_COMPLETE` is not emitted; `FRAME_PROCESSING_FEEDBACK` is followed directly by `FRAME_CAPTURE_RESULT`.

## Related

- [FRAME_CAPTURE_COMPLETE](./FRAME_CAPTURE_COMPLETE.md) - Auto capture session finished (before result payload)
- [FRAME_PROCESSING_FEEDBACK](./FRAME_PROCESSING_FEEDBACK.md) - Feedback during capture
- [CAPTURE_AND_PROCESS_FRAME](../commands/CAPTURE_AND_PROCESS_FRAME.md) - Start capture
- [MRZ Redaction Guide](../guides/mrz-redaction.md) - Dutch passport handling
