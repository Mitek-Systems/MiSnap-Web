# CAPTURE_AND_PROCESS_FRAME

Starts a camera capture session and performs real-time image quality assessment (IQA) on captured frames.

## Signature

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE" | "MANUAL_CAPTURE",
  documentType: string,
  mitekSDKPath?: string,
  loadWorkerCrossOrigin?: boolean,
  options: {
    license: string,
    // ...additional options
  }
});
```

## Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mode` | `string` | Yes | - | `"AUTO_CAPTURE"` or `"MANUAL_CAPTURE"` |
| `documentType` | `string` | Yes | - | Document type to capture (see below) |
| `mitekSDKPath` | `string` | No | `"./mitekSDK/"` | Path to SDK files |
| `loadWorkerCrossOrigin` | `boolean` | No | `false` | Load worker as Blob URL for cross-origin |
| `options` | `object` | Yes | - | Capture configuration (see [Options](#options)) |

## Document Types

| Type | Description |
|------|-------------|
| `DL_FRONT` | Driver's license or ID card front |
| `PASSPORT` | Passport photo/MRZ page |
| `CHECK_FRONT` | Front of check |
| `CHECK_BACK` | Back of check |
| `DOCUMENT` | Generic/trailing document |
| `PDF417_BARCODE` | North American DL/ID barcode |
| `QR_BARCODE` | QR code |
| `SELFIE` | Self-portrait for face biometrics |

## Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `license` | `string` | *required* | SDK license key |
| `qualityPercent` | `number` | `90` | JPEG quality (1-100) |
| `videoContainerId` | `string` | - | Custom container element ID |
| `disablePerpendicularCapture` | `boolean` | `false` | Require orientation match |
| `disableMirroring` | `boolean` | `false` | Disable mirroring of video preview in `AUTO_CAPTURE` mode |
| `disableSmileDetection` | `boolean` | `false` | Disable smile requirement for selfie |
| `disableSelfieCountdown` | `boolean` | `false` | Disable 3-2-1 countdown for selfie |
| `disableCaptureDelay` | `boolean` | `false` | Disable 1-second capture delay |
| `faceDetectionLevel` | `number` | `1` | 1=lenient, 2=moderate, 3=strict |
| `hintMessageSize` | `number` | `2` | 1=small, 2=medium, 3=large |
| `hintFrequencyMS` | `number` | `1200` | Hint display duration (ms) |
| `hintAriaLive` | `number` | `2` | 0=off, 1=polite, 2=assertive |
| `videoRecordingEnabled` | `boolean` | `false` | Record video during capture (45 second limit) |
| `audioRecordingEnabled` | `boolean` | `false` | Include audio with video recording |
| `videoQuality` | `number` | `70` | Video quality percentage (1-100) |
| `videoRecordingMessage` | `string` | `"Recording"` | Recording indicator text |
| `glareLevel` | `number` | - | 1=lenient, 2=moderate, 3=strict |
| `blurLevel` | `number` | - | 1=lenient, 2=moderate, 3=strict |
| `useDeviceIdSignal` | `boolean` | `false` | Include encrypted device ID signals in result |
| `aiBasedRtsEnabled` | `boolean` | `false` | Enable AI-based RTS (selfie only) |
| `redactMRZOptionalData` | `boolean` | `false` | Redact Dutch BSN from MRZ |
| `mrzRequired` | `boolean` | `false` | Require MRZ detection for a successful capture of `DL_FRONT` |
| `accessibilityMode` | `boolean` | `false` | Apply accessibility defaults |
| `announceDuplicateHints` | `boolean` | `false` | Re-announce unchanged hints |
| `customSuccessMessage` | `string` | `""` | Message shown on successful capture |
| `processingImageMessage` | `string` | `"Document found, wait."` | Processing indicator text |

## Examples

### Auto Capture - Driver's License

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  mitekSDKPath: "./mitekSDK/",
  options: {
    license: "your-license-here",
    qualityPercent: 90
  }
});
```

### Manual Capture - Passport

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "MANUAL_CAPTURE",
  documentType: "PASSPORT",
  mitekSDKPath: "./mitekSDK/",
  options: {
    license: "your-license-here",
    redactMRZOptionalData: true  // Redact Dutch BSN
  }
});
```

### Selfie with AI-based RTS

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "SELFIE",
  mitekSDKPath: "./mitekSDK/",
  options: {
    license: "your-license-here",
    aiBasedRtsEnabled: true,
    disableSmileDetection: true  // Will show 3-2-1 countdown timer
  }
});
```

### Custom Container

```javascript
// HTML: <div id="myContainer"></div>
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  options: {
    license: "your-license-here",
    videoContainerId: "myContainer"
  }
});
```

### With Video Recording

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  options: {
    license: "your-license-here",
    videoRecordingEnabled: true,
    audioRecordingEnabled: false,
    videoQuality: 70
  }
});
```

### Cross-Origin SDK Files

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  mitekSDKPath: "https://host.example.com/mitekSDK/",
  loadWorkerCrossOrigin: true,
  options: {
    license: "your-license-here"
  }
});
```

## Events

This command triggers the following events:

| Event | When |
|-------|------|
| [CAMERA_DISPLAY_STARTED](../events/CAMERA_DISPLAY_STARTED.md) | Camera preview is visible |
| [FRAME_PROCESSING_STARTED](../events/FRAME_PROCESSING_STARTED.md) | Frame analysis has begun |
| [FRAME_PROCESSING_FEEDBACK](../events/FRAME_PROCESSING_FEEDBACK.md) | Continuous IQA feedback |
| [FRAME_CAPTURE_RESULT](../events/FRAME_CAPTURE_RESULT.md) | Capture complete (success or failure) |
| [SDK_ERROR](../events/SDK_ERROR.md) | Error occurred |

## Result Object

The result is delivered via the `FRAME_CAPTURE_RESULT` event.

### Success

```javascript
{
  request: "DL_FRONT",
  response: {
    docType: "DL_FRONT",
    status: "success",
    mode: "AUTO",
    imageData: "data:image/jpeg;base64,...",
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
    ClientActivityDataLog: { ... }
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
    // ... other properties
  }
}
```

### Failure

```javascript
{
  request: "DL_FRONT",
  response: {
    docType: "DL_FRONT",
    status: "failure",
    mode: "AUTO",
    failedImage: "data:image/jpeg;base64,...",
    fourCornerA: { x: 0, y: 0 },
    fourCornerB: { x: 0, y: 0 },
    fourCornerC: { x: 0, y: 0 },
    fourCornerD: { x: 0, y: 0 },
    warnings: {
      status: "failure",
      code: "NF",
      key: "MITEK_ERROR_FOUR_CORNER"
    },
    mibiData: { ... }
  }
}
```

## Related

- [PROCESS_FRAME](./PROCESS_FRAME.md) - Process existing image
- [SDK_STOP](./SDK_STOP.md) - Stop capture session
- [Document Types](../configuration/document-types.md) - All document types
- [Capture Options](../configuration/capture-options.md) - All options
- [Selfie Capture Guide](../guides/selfie-capture.md) - Selfie best practices
