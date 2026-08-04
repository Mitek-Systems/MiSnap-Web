# SDK_ERROR

Triggered when an error prevents capture or analysis from continuing. The handler may be called more than once for a single command if multiple errors occur (for example, when component preload fails for several components).

## Signature

```javascript
mitekScienceSDK.on("SDK_ERROR", (error) => {
  // error: { code: number, type: string, status: string, cause?: Error }
});
```

## TypeScript

Import `MitekSdkError` and `MitekSdkErrorType` from `mitek-science-sdk.js` (see [Installation — TypeScript](../INSTALLATION.md#typescript-both-options)). The `on("SDK_ERROR", …)` callback is typed as `(error: MitekSdkError) => void`.

## Error Object

| Property | Type | Description |
|----------|------|-------------|
| `code` | `number` | Numeric error code |
| `type` | `string` | Error type identifier |
| `status` | `string` | Always `"failure"` |
| `cause` | `Error` | (Optional) Original exception |

## Example

```javascript
mitekScienceSDK.on("SDK_ERROR", (err) => {
  console.error(`Error ${err.code}: ${err.type}`);
  
  switch (err.code) {
    case 4006:
      showMessage("Invalid license. Please contact support.");
      break;
    case 113:
      showMessage("Camera permission denied. Please allow camera access.");
      break;
    default:
      showMessage("An error occurred. Please try again.");
  }
});
```

## Error Codes

### Camera/Device Errors

| Code | Type | Description |
|------|------|-------------|
| 111 | `CONSTRAINT_NOT_SATISFIED` | Unsupported device |
| 112 | `NO_CAMERA_FOUND` | No video camera detected |
| 113 | `CAMERA_PERMISSION_DENIED` | Camera permission denied |
| 120 | `CAMERA_UNKNOWN_DEVICE_ISSUE` | Camera failed to start |
| 130 | `MEDIA_RECORDER_NOT_SUPPORTED` | MediaRecorder not available |

### SDK/Browser Errors

| Code | Type | Description |
|------|------|-------------|
| 331 | `UNKNOWN_METHOD` | Unknown command called |
| 332 | `INVALID_COMMAND_SIGNATURE` | Wrong arguments to cmd |
| 333 | `USER_MEDIA_NOT_SUPPORTED` | getUserMedia not available |
| 334 | `WASM_NOT_SUPPORTED` | WebAssembly not available |
| 335 | `DEVICE_NOT_SUPPORTED` | Device not supported |
| 336 | `WEBGL_NOT_SUPPORTED` | WebGL not available |
| 338 | `UNKNOWN_ERROR` | Unidentified exception |
| 339 | `WEBGL_SHADER_NOT_SUPPORTED` | WebGL shader not supported |

### Document/Input Errors

| Code | Type | Description |
|------|------|-------------|
| 400 | `INVALID_DOCUMENT_TYPE` | Unknown document type |
| 500 | `VIDEO_RECORDING_EXCEEDED_MAX_TIME` | Recording time limit reached |

### Image Errors

| Code | Type | Description |
|------|------|-------------|
| 4001 | `IMAGE_SMALLER_THAN_MIN_SIZE` | Image under 640x480 pixels |
| 4002 | `CORRUPT_IMAGE` | Image file is corrupt |
| 4003 | `UNSUPPORTED_IMAGE_FORMAT` | Invalid image format |
| 4004 | `INVALID_IMAGE_DATA` | Cannot extract image data |
| 4005 | `INVALID_STREAM_TRACK` | Video stream tracks not found |

### License Errors

| Code | Type | Description |
|------|------|-------------|
| 4006 | `INVALID_LICENSE` | License format invalid |
| 4007 | `INVALID_LICENSE_EXPIRED` | License expired |
| 4008 | `INVALID_LICENSE_FEATURE` | Feature not in license |
| 4040 | `INVALID_BARCODE_LICENSE` | Barcode license missing |

### Device/Voice Errors

| Code | Type | Description |
|------|------|-------------|
| 4009 | `MIC_UNKNOWN_DEVICE_ISSUE` | Microphone issue |
| 4113 | `DEVICE_PERMISSION_DENIED` | Device permission denied |

### Parameter Errors

| Code | Type | Description |
|------|------|-------------|
| 5001 | `PARAMETER_NOT_SET` | Required parameter missing |
| 5002 | `PARAMETER_TYPE_ERROR` | Wrong parameter type |
| 5003 | `PARAMETER_REQUIRED_DATA_NOT_PROVIDED` | Incomplete parameter |
| 5004 | `OPTIONS_PARAMETER_INVALID_VALUE` | Value out of bounds |

### Integration Errors

| Code | Type | Description |
|------|------|-------------|
| 5420 | `EVENT_CALLBACK_INTEGRATION_ERROR` | Error in your callback |
| 5421 | `EVENT_SUBSCRIPTION_INTEGRATION_ERROR` | Error in event subscription |

### Feature Errors

| Code | Type | Description |
|------|------|-------------|
| 5501 | `AI_BASED_RTS_ERROR` | AI-based RTS internal error |
| 5502 | `AI_BASED_RTS_LOST_FOCUS` | AI-based RTS closed (e.g. browser lost focus) |
| 5511 | `PAYLOAD_PROCESSING_FAILED` | RTS or VDS payload processing failed; payload was not generated |

**Examples that would trigger `AI_BASED_RTS_LOST_FOCUS` (5502) when the camera is open:**

- The `window` detects a [blur event](https://developer.mozilla.org/en-US/docs/Web/API/Element/blur_event)
- The user focuses outside the page (e.g. address bar, dev tools, another window)
- If the app runs in an `iframe`, this can happen when top-level page loses focus

## Common Error Handling

### License Errors

```javascript
if (err.code >= 4006 && err.code <= 4008) {
  // License issue - contact support
  showLicenseError(err.type);
}
```

### Permission Errors

```javascript
if (err.code === 113 || err.code === 4113) {
  // Permission denied - prompt user
  showPermissionInstructions();
}
```

### Device Compatibility

```javascript
if ([334, 335, 336, 339].includes(err.code)) {
  // Browser/device not supported
  showCompatibilityMessage();
}
```

### Integration Errors

```javascript
if (err.code === 5420) {
  // Your callback code threw an error
  console.error("Callback error:", err.cause);
}
```

## Best Practices

1. **Always register `SDK_ERROR` first** before any commands
2. **Log all errors** for debugging
3. **Show user-friendly messages** based on error type

## Related

- [Commands Reference](../commands/README.md) - Commands that may error
- [Troubleshooting Guide](../guides/troubleshooting.md) - Common issues
