# Voice Events

Events specific to voice capture using `CAPTURE_AND_PROCESS_VOICE`.

## Event Sequence

```
CAPTURE_AND_PROCESS_VOICE called
         ↓
VOICE_PROCESSING_STARTED    → UI ready, mic permission granted
         ↓
VOICE_RECORDING_STARTED     → User tapped mic, recording
         ↓
VOICE_CAPTURE_RESULT        → Recording complete
```

---

## VOICE_PROCESSING_STARTED

Triggered when the voice capture UI is displayed and ready for user interaction.

### Signature

```javascript
mitekScienceSDK.on("VOICE_PROCESSING_STARTED", () => {
  // No data passed
});
```

### When It Fires

- Microphone permission has been granted
- Voice capture UI is visible
- Microphone icon shown as muted (not recording yet)

### Example

```javascript
mitekScienceSDK.on("VOICE_PROCESSING_STARTED", () => {
  showMessage("Tap the microphone when ready to record");
});
```

---

## VOICE_RECORDING_STARTED

Triggered when the user taps the microphone icon and recording begins.

### Signature

```javascript
mitekScienceSDK.on("VOICE_RECORDING_STARTED", () => {
  // No data passed
});
```

### When It Fires

- User has tapped/clicked the microphone icon
- Audio recording is in progress
- Microphone icon is animated

### Example

```javascript
mitekScienceSDK.on("VOICE_RECORDING_STARTED", () => {
  showMessage("Speak the phrase clearly");
});
```

---

## VOICE_CAPTURE_RESULT

Triggered when voice recording completes, either successfully or with failure.

### Signature

```javascript
mitekScienceSDK.on("VOICE_CAPTURE_RESULT", (result) => {
  // result: { request: string, response: object }
});
```

### Success Response

```javascript
{
  request: "VOICE",
  response: {
    docType: "VOICE",
    status: "success",
    mode: "AUTO",
    audioData: "data:audio/webm;base64,GkXfo59Ch...",
    mibiData: { Platform: "Web", MibiVersion: "2.2", ... },
    ClientActivityDataLog: { ... },
    RTS: "WyJ3bFBnc..."
  }
}
```

### Failure Response

```javascript
{
  request: "VOICE",
  response: {
    docType: "VOICE",
    status: "failure",
    mode: "AUTO",
    failedAudio: "data:audio/webm;base64,GkXfo59Ch...",
    mibiData: { Platform: "Web", MibiVersion: "2.2", ... },
    ClientActivityDataLog: { ... },
    RTS: "WyJ3bFBnc...",
    warnings: { status: "failure", ... }
  }
}
```

### Example

```javascript
mitekScienceSDK.on("VOICE_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    // Send audio to Mitek backend for voice biometrics
    sendVoiceData(result.response.audioData);
  } else {
    showMessage("Voice capture failed. Please try again.");
  }
});
```

---

## Complete Example

```javascript
const license = "your-license-here";
const phrase = "Time flies when you're having fun.";

// Setup all event handlers
mitekScienceSDK.on("SDK_ERROR", (err) => {
  if (err.code === 4113) {
    showMessage("Microphone permission denied");
  } else if (err.code === 4009) {
    showMessage("Microphone not available");
  } else {
    showMessage("Error: " + err.type);
  }
});

mitekScienceSDK.on("VOICE_PROCESSING_STARTED", () => {
  addCancelButton();

  mitekScienceSDK.cmd("SHOW_HINT", {
    options: {
      hintText: "Tap microphone button to start, then repeat the phrase",
    },
  });
});

mitekScienceSDK.on("VOICE_RECORDING_STARTED", () => {
  console.log("Recording started");

  mitekScienceSDK.cmd("SHOW_HINT", {
    options: {
      hintText: "Repeat the phrase above loud and clear",
      hintForceUpdate: true,
    },
  });
});

mitekScienceSDK.on("VOICE_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    console.log("Voice captured successfully");
    uploadVoiceData(result.response.audioData);
  } else {
    console.log("Voice capture failed");
    showRetryButton();
  }
});

// Start voice capture
function startVoiceCapture() {
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_VOICE", {
    mitekSDKPath: "./mitekSDK/",
    options: { license, phrase }
  });
}
```

## Timeouts

| State | Timeout |
|-------|---------|
| Waiting for user to tap mic | 5 minutes |
| Recording | 2 minutes |

## Error Codes

| Code | Type | Description |
|------|------|-------------|
| 4009 | `MIC_UNKNOWN_DEVICE_ISSUE` | Microphone problem |
| 4113 | `DEVICE_PERMISSION_DENIED` | Mic permission denied |

## Requirements

- **iOS:** Safari 16+ required
- **HTTPS:** Required for microphone access (localhost exempt)

## Related

- [CAPTURE_AND_PROCESS_VOICE](../commands/CAPTURE_AND_PROCESS_VOICE.md) - Voice capture command
- [SDK_STOP](../commands/SDK_STOP.md) - Cancel voice capture
