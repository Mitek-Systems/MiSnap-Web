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

| Option                     | Type                            | Default   | Description                                                                                                                        |
| -------------------------- | ------------------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `aiBasedRtsEnabled`        | `boolean`                       | `false`   | Enable injection attack detection                                                                                                  |
| `aiBasedRtsPayloadSize`    | `string`                        | `"small"` | RTS payload size: `"small"` or `"normal"`                                                                                          |
| `aiBasedRtsEncryptionKey`  | `AiBasedRtsEncryptionKeyOption` | *(unset)* | `"DEFAULT_IAD"` for the stock on-prem IAD key, or `{ publicKey, keyId? }` for a custom key. See notes below. |
| `aiBasedRtsAdditionalData` | `string`                        | *(unset)* | Optional external metadata string for AI-based RTS. See notes below. |


> **Note:** When `aiBasedRtsEncryptionKey` is omitted, the SDK encrypts the AI-based RTS payload with a **default public key paired to Mobile Verify**. For use with an on-prem IAD Server that uses the stock default key, set `aiBasedRtsEncryptionKey: "DEFAULT_IAD"`. For a custom server key, pass `{ publicKey, keyId? }` that matches your server.
>
> `aiBasedRtsEncryptionKey` and `aiBasedRtsAdditionalData` are intended for on-prem IAD deployments only. **MiVIP** and **Mobile Verify** do not currently support these options. A custom key or `"DEFAULT_IAD"` sent to a Mitek SaaS backend will not decrypt successfully. See the [IDLive to MiSnap Transition Guide](../guides/idlive-to-misnap-transition.md) for details.

```javascript
options: {
  license,
  aiBasedRtsEnabled: true,
  aiBasedRtsPayloadSize: "normal",
  // Stock on-prem IAD key:
  // aiBasedRtsEncryptionKey: "DEFAULT_IAD",
  // Or a custom key:
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
- **`aiBasedRtsEncryptionKey`:** Omit this option if sending AI-based RTS capture data to Mobile Verify. Use `"DEFAULT_IAD"` for the stock on-prem IAD Server public key, or supply `{ publicKey, keyId? }` that matches your server. If you pass an object with an empty `publicKey`, the SDK uses the Mobile Verify default key and logs a console validation warning. A non-empty `publicKey` must be valid base64. Invalid values fail validation.
- **`aiBasedRtsAdditionalData`:** Must be a string when set. Invalid types are ignored with a console warning; capture continues. Use this for optional metadata your application needs to supply in the format your backend expects (for example a JSON string).

> **Note:** AI-based RTS uses anti-debugging techniques that prevent the use of browser DevTools when integrated with a **production** (`-prod`) SDK package. This is intentional protection against injection attacks. For local development and debugging, use the **development** (`-dev`) SDK package. Do **not** deploy the **`-dev`** package to production. See [SDK build variants](./INSTALLATION.md#sdk-build-variants) in the Installation guide.

## Video Recording Options

When `videoRecordingEnabled` is `true`, MiSnap returns the **last N seconds** of the capture session (default `5`, use `videoRecordingDuration` for `5`–`10`). If the session is shorter than that window, the returned clip is the full session. For example, a 3-second capture returns 3 seconds of video.

Use `videoBitrate` to set recording bitrate. The deprecated `videoQuality` option is still supported when `videoBitrate` is omitted. If both are set, `videoBitrate` takes precedence. The configured bitrate is a target. The actual encoded bitrate is not guaranteed and may vary by browser and device.

This supports eIDAS 2.0 identity-proofing video expectations. See [ETSI TS 119 461 V2.1.1](https://www.etsi.org/deliver/etsi_ts/119400_119499/119461/02.01.01_60/ts_119461v020101p.pdf) (sections 8.3.3 and 8.4.2).

| Option                   | Type      | Default       | Description                                      |
| ------------------------ | --------- | ------------- | ------------------------------------------------ |
| `videoRecordingEnabled`  | `boolean` | `false`       | Record capture session                           |
| `videoRecordingDuration` | `number`  | `5`           | Trailing window in seconds (`5`–`10`). Out-of-range values are clamped. Sessions shorter than this return the full recording. Actual length may be ±1s due to browser encoding. |
| `audioRecordingEnabled`  | `boolean` | `false`       | Include audio in recording                       |
| `videoBitrate`           | `number`  | `1300000`     | Target video bitrate in bits per second (`1300000`–`2200000`). Out-of-range values are clamped. Actual encoded bitrate is not guaranteed. |
| `videoQuality`           | `number`  | `10`          | **Deprecated.** Use `videoBitrate`. Quality percentage (`10`–`100`) mapped to bitrate range when `videoBitrate` is omitted. Out-of-range values are clamped. |
| `videoRecordingMessage`  | `string`  | `"Recording"` | Recording indicator text                         |

```javascript
options: {
  license,
  videoRecordingEnabled: true,
  videoRecordingDuration: 5,
  audioRecordingEnabled: false,
  videoBitrate: 1300000,
  videoRecordingMessage: "Recording in progress"
}
```

> **Note:** If session recording fails, capture can still complete successfully. If this happens, the response `videoData` is `null`, and `warnings.recording` will include `code: "VRF"` and a `message` describing the failure. See [FRAME_CAPTURE_RESULT](../events/FRAME_CAPTURE_RESULT.md).

**Requirements:**

- MediaRecorder API support
- iOS 14.3+ / Chrome 89+

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

