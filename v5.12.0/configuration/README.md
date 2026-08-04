# Configuration Reference

The MiSnap SDK accepts configuration options through the `options` object passed to capture commands.

## Configuration Structure

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  mitekSDKPath: "./mitekSDK/",
  options: {
    license: "your-license-here",
    // Configuration options go here
  }
});
```

## Configuration Topics

| Document | Description |
|----------|-------------|
| [Document Types](./document-types.md) | Available document types for capture |
| [Capture Options](./capture-options.md) | All capture configuration options |
| [Accessibility](./accessibility.md) | Accessibility and screen reader options |

## Quick Reference

### Required Options

| Option | Type | Description |
|--------|------|-------------|
| `license` | `string` | SDK license key (required for all captures) |
| `frame` | `string` | Image data (required for `PROCESS_FRAME` only) |
| `phrase` | `string` | Spoken phrase (required for `CAPTURE_AND_PROCESS_VOICE` only) |

### General Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `qualityPercent` | `number` | `90` | JPEG quality (0-100) |
| `hintFrequencyMS` | `number` | `1200` | Hint display duration (ms) |
| `videoContainerId` | `string` | - | Custom container element ID |
| `disableCaptureDelay` | `boolean` | `false` | Skip 1-second capture delay |
| `disableMirroring` | `boolean` | `false` | Don't mirror preview |
| `useDeviceIdSignal` | `boolean` | `false` | Include encrypted device ID signals in result |

### Document Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `disablePerpendicularCapture` | `boolean` | `false` | Require orientation match |
| `glareLevel` | `number` | `2` | 1=lenient, 2=moderate, 3=strict |
| `blurLevel` | `number` | `2` | 1=lenient, 2=moderate, 3=strict |
| `mrzRequired` | `boolean` | `false` | Require MRZ detection for a successful capture of `DL_FRONT` |
| `redactMRZOptionalData` | `boolean` | `false` | Redact Dutch BSN |

### Selfie Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `disableSmileDetection` | `boolean` | `false` | Skip smile requirement |
| `disableSelfieCountdown` | `boolean` | `false` | Skip 3-2-1 countdown |
| `faceDetectionLevel` | `number` | `1` | 1=lenient, 2=moderate, 3=strict |
| `aiBasedRtsEnabled` | `boolean` | `false` | Enable AI-based RTS |
| `aiBasedRtsPayloadSize` | `string` | `"small"` | `"small"` or `"normal"` (AI Based RTS payload size) |
| `aiBasedRtsEncryptionKey` | `AiBasedRtsEncryptionKeyOption` | *(unset)* | Custom AI Based RTS encryption: `publicKey` (valid base64), optional `keyId` |
| `aiBasedRtsAdditionalData` | `string` | *(unset)* | Optional external metadata string for AI-based RTS |

### Recording Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `videoRecordingEnabled` | `boolean` | `false` | Record video |
| `audioRecordingEnabled` | `boolean` | `false` | Include audio |
| `videoQuality` | `number` | `70` | Video bitrate (0-100) |
| `videoRecordingMessage` | `string` | `"Recording"` | Recording indicator |

### Accessibility Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `hintMessageSize` | `number` | `2` | 1=small, 2=medium, 3=large |
| `hintAriaLive` | `number` | `2` | 0=off, 1=polite, 2=assertive |
| `accessibilityMode` | `boolean` | `false` | Apply accessibility defaults |
| `announceDuplicateHints` | `boolean` | `false` | Re-announce hints |

### UI Messages

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `customSuccessMessage` | `string` | `""` | Message on success |
| `processingImageMessage` | `string` | `"Document found, wait."` | Processing message |

## Example Configurations

### Standard Document Capture

```javascript
options: {
  license: "your-license",
  qualityPercent: 90
}
```

### Accessibility-Focused

```javascript
options: {
  license: "your-license",
  accessibilityMode: true,
  hintMessageSize: 3,
  hintFrequencyMS: 5000
}
```

### Selfie with 3-2-1 Countdown

```javascript
options: {
  license: "your-license",
  disableSmileDetection: true
}
```

### Strict Document Quality (ATP)

```javascript
options: {
  license: "your-license",
  glareLevel: 3,  // Strict
  blurLevel: 3    // Strict
}
```

## Related

- [Commands Reference](../commands/README.md) - Commands accepting configuration
- [Installation Guide](../INSTALLATION.md) - License configuration
