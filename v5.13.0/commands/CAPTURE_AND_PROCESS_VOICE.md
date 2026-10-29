# CAPTURE_AND_PROCESS_VOICE

Starts a voice capture session for voice biometrics enrollment or verification.

## Signature

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_VOICE", {
  mitekSDKPath?: string,
  options: {
    license: string,
    phrase: string
  }
});
```

## Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mitekSDKPath` | `string` | No | `"./mitekSDK/"` | Path to SDK files |
| `options.license` | `string` | Yes | - | SDK license key |
| `options.phrase` | `string` | Yes | - | Phrase for user to speak |

## Example

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_VOICE", {
  mitekSDKPath: "./mitekSDK/",
  options: {
    license: "your-license-here",
    phrase: "Time flies when you're having fun."
  }
});
```

## UI States

Voice capture has two UI states:

### 1. Initial State

- Microphone permission check occurs
- Microphone icon appears muted (not recording)
- User can cancel via back button or custom cancel button
- Event: `VOICE_PROCESSING_STARTED`

### 2. Recording State

- User taps/clicks microphone icon to begin
- Icon animates as audio is received
- User speaks the displayed phrase
- SDK auto-detects sufficient audio (~2 seconds)
- Green checkmark on success
- Event: `VOICE_RECORDING_STARTED`

## Events

| Event | When |
|-------|------|
| `VOICE_PROCESSING_STARTED` | Initial UI displayed, awaiting user action |
| `VOICE_RECORDING_STARTED` | User tapped mic, recording in progress |
| `VOICE_CAPTURE_RESULT` | Recording complete (success or failure) |
| `SDK_ERROR` | Error occurred (e.g., permission denied) |

## Result Object

### Success

```javascript
{
  request: "VOICE",
  response: {
    docType: "VOICE",
    audioData: "data:audio/wav;base64,UklGRiQg...",
    mibiData: { Platform: "Web", ... },
    mode: "AUTO",
    status: "success"
  }
}
```

### Failure

Failures occur due to:

- **Timeout** - Initial UI: 5 minutes, Recording: 2 minutes
- **User cancel** - Via `SDK_STOP` command
- **Permission denied** - Device/browser denied microphone
- **No recording capability** - Device lacks microphone

## Timeouts

| State | Timeout |
|-------|---------|
| Initial UI | 5 minutes (global timeout) |
| Recording | 2 minutes |

## Requirements

- **iOS:** Safari 16+ required
- **Microphone permission** must be granted
- **HTTPS** required for microphone access (localhost exempt)

## Error Codes

| Code | Type | Description |
|------|------|-------------|
| 4009 | `MIC_UNKNOWN_DEVICE_ISSUE` | Microphone/recording device issue |
| 4113 | `DEVICE_PERMISSION_DENIED` | Microphone permission denied |

## Complete Example

```javascript
// Setup event handlers
mitekScienceSDK.on("SDK_ERROR", (err) => {
  console.error("Voice capture error:", err);
});

mitekScienceSDK.on("VOICE_PROCESSING_STARTED", () => {
  console.log("Tap the microphone to start recording");
});

mitekScienceSDK.on("VOICE_RECORDING_STARTED", () => {
  console.log("Recording... speak the phrase clearly");
});

mitekScienceSDK.on("VOICE_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    // Send audioData to your server for MiPass processing
    sendToServer(result.response.audioData);
  } else {
    console.log("Voice capture failed");
  }
});

// Start voice capture
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_VOICE", {
  mitekSDKPath: "./mitekSDK/",
  options: {
    license: "your-license-here",
    phrase: "My voice is my password."
  }
});

// Optional: Add cancel button
document.getElementById("cancelBtn").addEventListener("click", () => {
  mitekScienceSDK.cmd("SDK_STOP");
});
```

## Related

- [SDK_STOP](./SDK_STOP.md) - Cancel voice capture
- [Events Reference](../events/README.md) - All events
