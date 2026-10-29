# FRAME_PROCESSING_STARTED

Triggered once when the SDK begins analyzing camera frames for quality assessment.

## Signature

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_STARTED", () => {
  // No data passed to callback
});
```

## When It Fires

- After `CAMERA_DISPLAY_STARTED`
- When frame analysis begins
- Before `FRAME_PROCESSING_FEEDBACK`

## Example

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_STARTED", () => {
  // Start custom timeout
  setTimeout(() => {
    mitekScienceSDK.cmd("SDK_STOP");
    showTimeoutMessage();
  }, 30000);  // 30 second timeout
});
```

## Use Cases

### 1. Custom Timeout

The SDK has a built-in 5-minute timeout, but you can implement a shorter one:

```javascript
let timeoutId;

mitekScienceSDK.on("FRAME_PROCESSING_STARTED", () => {
  timeoutId = setTimeout(() => {
    clearTimeout(timeoutId);
    mitekScienceSDK.cmd("SDK_STOP");
    showMessage("Capture timed out. Please try again.");
  }, 30000);
});

mitekScienceSDK.on("FRAME_CAPTURE_RESULT", () => {
  clearTimeout(timeoutId);
});
```

### 2. Analytics Tracking

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_STARTED", () => {
  analytics.track("capture_processing_started", {
    documentType: currentDocType,
    timestamp: Date.now()
  });
});
```

### 3. Disable UI During Processing

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_STARTED", () => {
  document.getElementById("switchMode").disabled = true;
});

mitekScienceSDK.on("FRAME_CAPTURE_RESULT", () => {
  document.getElementById("switchMode").disabled = false;
});
```

## Event Sequence

```
CAMERA_DISPLAY_STARTED
         ↓
FRAME_PROCESSING_STARTED    ← You are here
         ↓
FRAME_PROCESSING_FEEDBACK (continuous)
         ↓
FRAME_CAPTURE_RESULT
```

## Related

- [CAMERA_DISPLAY_STARTED](./CAMERA_DISPLAY_STARTED.md) - Previous in sequence
- [FRAME_PROCESSING_FEEDBACK](./FRAME_PROCESSING_FEEDBACK.md) - Next in sequence
- [SDK_STOP](../commands/SDK_STOP.md) - Stop capture with custom timeout
