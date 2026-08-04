# Events Reference

The MiSnap SDK communicates with your application through events. Register event handlers using `mitekScienceSDK.on()` to receive notifications about capture status, errors, and results.

## Registering Event Handlers

```javascript
mitekScienceSDK.on("EVENT_NAME", (data) => {
  // Handle the event
});
```

## TypeScript payload types

Three events publish named payload types in `mitek-science-sdk.d.ts`:

| Event | Type |
|-------|------|
| [FRAME_CAPTURE_RESULT](./FRAME_CAPTURE_RESULT.md) | `MitekFrameCaptureResult` |
| [FRAME_PROCESSING_FEEDBACK](./FRAME_PROCESSING_FEEDBACK.md) | `MitekFrameProcessingFeedback` |
| [SDK_ERROR](./SDK_ERROR.md) | `MitekSdkError` |

`mitekScienceSDK.on()` infers the callback payload type from the name for these events. See [Installation — TypeScript](../INSTALLATION.md#typescript-both-options).

## Available Events

### Preload Events

| Event | When Triggered | Data |
|-------|----------------|------|
| [COMPONENT_PRELOAD_COMPLETE](./COMPONENT_PRELOAD_COMPLETE.md) | Component preload finished | None |

### Capture Events

| Event | When Triggered | Data |
|-------|----------------|------|
| [CAMERA_DISPLAY_STARTED](./CAMERA_DISPLAY_STARTED.md) | Camera preview visible | None |
| [FRAME_PROCESSING_STARTED](./FRAME_PROCESSING_STARTED.md) | Frame analysis has begun | None |
| [FRAME_PROCESSING_FEEDBACK](./FRAME_PROCESSING_FEEDBACK.md) | Continuous IQA feedback | Feedback object |
| [FRAME_CAPTURE_COMPLETE](./FRAME_CAPTURE_COMPLETE.md) | Auto capture session finished | None |
| [FRAME_CAPTURE_RESULT](./FRAME_CAPTURE_RESULT.md) | Capture complete | Result object |

### Voice Events

| Event | When Triggered | Data |
|-------|----------------|------|
| [VOICE_PROCESSING_STARTED](./VOICE_EVENTS.md#voice_processing_started) | Voice UI displayed | None |
| [VOICE_RECORDING_STARTED](./VOICE_EVENTS.md#voice_recording_started) | Recording begun | None |
| [VOICE_CAPTURE_RESULT](./VOICE_EVENTS.md#voice_capture_result) | Voice capture complete | Result object |

### Error Events

| Event | When Triggered | Data |
|-------|----------------|------|
| [SDK_ERROR](./SDK_ERROR.md) | Error stops capture | Error object |

## Event Sequence

### Document/Selfie Capture

```
1. CAMERA_DISPLAY_STARTED     → Camera preview visible
2. FRAME_PROCESSING_STARTED   → Analysis begins
3. FRAME_PROCESSING_FEEDBACK  → Continuous (until capture)
4. FRAME_CAPTURE_COMPLETE     → Auto capture only — session finished, before optional `customSuccessMessage` delay
5. FRAME_CAPTURE_RESULT       → Success or failure (result payload)
```

For **manual** or **direct** capture, step 4 is omitted; `FRAME_CAPTURE_RESULT` follows the last relevant feedback event.

### Voice Capture

```
1. VOICE_PROCESSING_STARTED   → UI ready, awaiting user
2. VOICE_RECORDING_STARTED    → User tapped mic
3. VOICE_CAPTURE_RESULT       → Recording complete
```

## Basic Setup

Always register `SDK_ERROR` before any commands:

```javascript
// 1. Error handler first
mitekScienceSDK.on("SDK_ERROR", (err) => {
  console.error("SDK Error:", err.code, err.type);
});

// 2. Result handler
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    handleSuccess(result.response.imageData);
  } else {
    handleFailure(result.response.warnings);
  }
});

// 3. Now safe to call commands
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", config);
```

## Removing Event Handlers

Remove all registered handlers:

```javascript
mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");
```

> **Note:** This removes ALL handlers. You cannot remove individual handlers.

## Error Handling in Callbacks

If your callback throws an error, the SDK will:

1. Catch the exception
2. Emit `SDK_ERROR` with code `5420` (EVENT_CALLBACK_INTEGRATION_ERROR)
3. Include error details in `response.error.cause`

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  // If this throws, SDK_ERROR is emitted with details
  someUndefinedFunction();  // Error!
});

mitekScienceSDK.on("SDK_ERROR", (err) => {
  if (err.code === 5420) {
    console.error("Callback error:", err.cause);
  }
});
```

## Related

- [Commands Reference](../commands/README.md) - Commands that trigger events
- [SDK_REMOVE_LISTENERS](../commands/SDK_REMOVE_LISTENERS.md) - Remove handlers
