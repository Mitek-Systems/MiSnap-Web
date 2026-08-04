# Capture Options

Complete reference for all configuration options passed in the `options` object.

## Required Options


| Option    | Type     | Commands                                                                                       | Description                |
| --------- | -------- | ---------------------------------------------------------------------------------------------- | -------------------------- |
| `license` | `string` | `COMPONENT_PRELOAD`, `CAPTURE_AND_PROCESS_FRAME`, `PROCESS_FRAME`, `CAPTURE_AND_PROCESS_VOICE` | SDK license key            |
| `frame`   | `string` | `PROCESS_FRAME`                                                                                | Image data URL or blob URL |
| `phrase`  | `string` | `CAPTURE_AND_PROCESS_VOICE`                                                                    | Phrase for user to speak   |


## Image Quality


| Option           | Type     | Default | Description                      |
| ---------------- | -------- | ------- | -------------------------------- |
| `qualityPercent` | `number` | `90`    | JPEG compression quality (0-100) |


```javascript
options: {
  license,
  qualityPercent: 95  // Higher quality, larger file
}
```

## Capture Delay

To prevent captures from happening "too fast", by default there's a one second delay before capturing the first good frame. This behavior can be disabled.


| Option                | Type      | Default | Description             |
| --------------------- | --------- | ------- | ----------------------- |
| `disableCaptureDelay` | `boolean` | `false` | Skip 1-second IQA delay |


## Mirroring

The MiSnap SDK will automatically determine whether mirroring of the video preview is needed. There's also an option to explicitly disable it.


| Option             | Type      | Default | Description                                               |
| ------------------ | --------- | ------- | --------------------------------------------------------- |
| `disableMirroring` | `boolean` | `false` | Disable mirroring of video preview in `AUTO_CAPTURE` mode |


## Advanced Threshold Presets (ATP)

Control strictness of glare and blur detection for documents.


| Option       | Type     | Default | Values  | Description     |
| ------------ | -------- | ------- | ------- | --------------- |
| `glareLevel` | `number` | `2`     | 1, 2, 3 | Glare tolerance |
| `blurLevel`  | `number` | `2`     | 1, 2, 3 | Blur tolerance  |



| Level | Name     | Effect                              |
| ----- | -------- | ----------------------------------- |
| 1     | Lenient  | Faster capture, lower quality       |
| 2     | Moderate | Balanced (recommended)              |
| 3     | Strict   | Higher quality, longer capture time |


```javascript
options: {
  license,
  glareLevel: 3,  // Strict
  blurLevel: 2    // Moderate
}
```

## Document Capture Options


| Option                        | Type      | Default | Description                                                  |
| ----------------------------- | --------- | ------- | ------------------------------------------------------------ |
| `disablePerpendicularCapture` | `boolean` | `false` | Require document/device orientation match                    |
| `mrzRequired`                 | `boolean` | `false` | Require MRZ detection for a successful capture of `DL_FRONT` |
| `redactMRZOptionalData`       | `boolean` | `false` | Redact Dutch BSN from passport/ID MRZ                        |


### Perpendicular Capture

When enabled, document orientation must match device orientation:

```javascript
options: {
  license,
  disablePerpendicularCapture: true  // Landscape doc needs landscape device
}
```

### MRZ Required

Flag to require MRZ detection for `DL_FRONT` (default: `false`).

- Setting this flag requires MRZ detection for a successful capture
- Can be used for ID cards with MRZ to distinguish between front and back
- Does not return MRZ data in the result
- This setting has no effect on `PASSPORT`, where MRZ is required by default

```javascript
options: {
  license,
  mrzRequired: true  // Fail if MRZ not detected
}
```

### MRZ Redaction

For Dutch documents with BSN in MRZ:

```javascript
options: {
  license,
  redactMRZOptionalData: true  // Black out BSN in output image
}
```

## Selfie Capture Options


| Option                   | Type      | Default | Description               |
| ------------------------ | --------- | ------- | ------------------------- |
| `disableSmileDetection`  | `boolean` | `false` | Skip smile requirement    |
| `disableSelfieCountdown` | `boolean` | `false` | Skip 3-2-1 countdown      |
| `faceDetectionLevel`     | `number`  | `1`     | Face detection strictness |


### Capture Condition Strategies

**1. Smile Detection (default)**

```javascript
options: {
  license,
  disableSmileDetection: false  // Require smile + neutral
}
```

**2. Countdown Timer**

```javascript
options: {
  license,
  disableSmileDetection: true,
  disableSelfieCountdown: false  // Show 3-2-1
}
```

**3. Good IQA**

```javascript
options: {
  license,
  disableSmileDetection: true,
  disableSelfieCountdown: true,
  disableCaptureDelay: false  // 1-second delay before capturing good frame
}
```

### Face Detection Level


| Level | Name     | Description          |
| ----- | -------- | -------------------- |
| 1     | Lenient  | Easiest to pass      |
| 2     | Moderate | Balanced             |
| 3     | Strict   | Highest requirements |


```javascript
options: {
  license,
  faceDetectionLevel: 2  // Moderate strictness
}
```

## AI-Based RTS (Real Time Security)

> **Note:** `aiBasedRtsEncryptionKey` and `aiBasedRtsAdditionalData` are intended for on-prem deployments only. **MiVIP** and **Mobile Verify** do not currently support these options; a custom key sent to a Mitek SaaS backend will not decrypt successfully. See the [IDLive to MiSnap Transition Guide](../guides/idlive-to-misnap-transition.md) for details.


| Option                     | Type                            | Default   | Description                                                                                                                        |
| -------------------------- | ------------------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `aiBasedRtsEnabled`        | `boolean`                       | `false`   | Enable injection attack detection                                                                                                  |
| `aiBasedRtsPayloadSize`    | `string`                        | `"small"` | RTS payload size: `"small"` or `"normal"`                                                                                          |
| `aiBasedRtsEncryptionKey`  | `AiBasedRtsEncryptionKeyOption` | *(unset)* | `publicKey`: valid base64 (DER public key) and optional `keyId`.                                                                   |
| `aiBasedRtsAdditionalData` | `string`                        | *(unset)* | Optional external metadata string for AI-based RTS. |


```javascript
options: {
  license,
  aiBasedRtsEnabled: true,
  aiBasedRtsPayloadSize: "normal",
  aiBasedRtsEncryptionKey: { publicKey: "YWJjZGVmZ2hpams=", keyId: "my-key" },
  aiBasedRtsAdditionalData: JSON.stringify({ sessionId: "abc-123" })
}
```

**Requirements:**

- `SELFIE` document type only
- `AUTO_CAPTURE` mode only
- Front camera on mobile

**Behavior and validation:**

- **`aiBasedRtsPayloadSize`:** Invalid values (anything other than `"small"` or `"normal"`) are ignored; the SDK keeps the default and logs a validation warning to the console. Capture continues.
- **`aiBasedRtsEncryptionKey`:** Omit the option for the SDK default. If you pass an object with an empty `publicKey`, the SDK still uses the default key and logs a console validation warning. Non-empty `publicKey` must be valid base64. Invalid values fail validation.
- **`aiBasedRtsAdditionalData`:** Must be a string when set. Invalid types are ignored with a console warning; capture continues. Use this for optional metadata your application needs to supply in the format your backend expects (for example a JSON string).

> **Note:** AI-based RTS uses anti-debugging techniques that prevent the use of browser DevTools when integrated with a **production** (`-prod`) SDK package. This is intentional protection against injection attacks. For local development and debugging, use the **development** (`-dev`) SDK package. Do **not** deploy the **`-dev`** package to production. See [SDK build variants](./INSTALLATION.md#sdk-build-variants) in the Installation guide.

## Video Recording Options


| Option                  | Type      | Default       | Description                |
| ----------------------- | --------- | ------------- | -------------------------- |
| `videoRecordingEnabled` | `boolean` | `false`       | Record capture session     |
| `audioRecordingEnabled` | `boolean` | `false`       | Include audio in recording |
| `videoQuality`          | `number`  | `70`          | Video bitrate (0-100)      |
| `videoRecordingMessage` | `string`  | `"Recording"` | Recording indicator text   |


```javascript
options: {
  license,
  videoRecordingEnabled: true,
  audioRecordingEnabled: false,
  videoQuality: 80,
  videoRecordingMessage: "Recording in progress"
}
```

**Requirements:**

- MediaRecorder API support
- iOS 14.3+ / Chrome 89+
- Session timeout reduced to 45 seconds

## Hint Options


| Option            | Type     | Default | Description                |
| ----------------- | -------- | ------- | -------------------------- |
| `hintMessageSize` | `number` | `2`     | 1=small, 2=medium, 3=large |
| `hintFrequencyMS` | `number` | `1200`  | Hint display duration (ms) |


```javascript
options: {
  license,
  hintMessageSize: 3,     // Large hints
  hintFrequencyMS: 2000   // Show for 2 seconds
}
```

## Custom Messages


| Option                   | Type     | Default                   | Description                      |
| ------------------------ | -------- | ------------------------- | -------------------------------- |
| `customSuccessMessage`   | `string` | `""`                      | Hint message shown after successful capture |
| `processingImageMessage` | `string` | `"Document found, wait."` | Processing indicator             |


```javascript
options: {
  license,
  customSuccessMessage: "Success",
  processingImageMessage: "Hold steady"
}
```

## Custom Container


| Option             | Type     | Default | Description             |
| ------------------ | -------- | ------- | ----------------------- |
| `videoContainerId` | `string` | -       | ID of container element |


```javascript
// HTML: <div id="captureArea"></div>
options: {
  license,
  videoContainerId: "captureArea"
}
```

**Note:** Container should have aspect ratio close to 16:9.

## Device Signals


| Option              | Type      | Default | Description                                   |
| ------------------- | --------- | ------- | --------------------------------------------- |
| `useDeviceIdSignal` | `boolean` | `false` | Include encrypted device ID signals in result |


```javascript
options: {
  license,
  useDeviceIdSignal: true  // Include encrypted device ID signals
}
```

> **Note:** Requires user consent. Used for device+biometric binding. There is no personal identifying information (PII) involved.

## Accessibility Options

See [Accessibility Configuration](./accessibility.md) for full details.


| Option                   | Type      | Default | Description                  |
| ------------------------ | --------- | ------- | ---------------------------- |
| `hintAriaLive`           | `number`  | `2`     | 0=off, 1=polite, 2=assertive |
| `accessibilityMode`      | `boolean` | `false` | Apply accessibility defaults |
| `announceDuplicateHints` | `boolean` | `false` | Re-announce unchanged hints for those using screen readers  |

## Related

- [Document Types](./document-types.md) - Available document types
- [Accessibility](./accessibility.md) - Screen reader options
- [CAPTURE_AND_PROCESS_FRAME](../commands/CAPTURE_AND_PROCESS_FRAME.md) - Capture command

