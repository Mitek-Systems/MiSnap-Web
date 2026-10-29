# CAMERA_DISPLAY_STARTED

Triggered once when the camera preview becomes visible during auto capture mode.

## Signature

```javascript
mitekScienceSDK.on("CAMERA_DISPLAY_STARTED", () => {
  // No data passed to callback
});
```

## When It Fires

- After `CAPTURE_AND_PROCESS_FRAME` with `AUTO_CAPTURE` mode
- When camera stream is active and visible to user
- Before `FRAME_PROCESSING_STARTED`

## Example

```javascript
mitekScienceSDK.on("CAMERA_DISPLAY_STARTED", () => {
  // Hide loading spinner
  document.getElementById("spinner").style.display = "none";
  
  // Show first hint
  mitekScienceSDK.cmd("SHOW_HINT", {
    options: {
      hintText: "Center on a dark background",
      hintDuration: 1500
    }
  });
});
```

## Use Cases

### 1. Hide Loading State

```javascript
mitekScienceSDK.on("CAMERA_DISPLAY_STARTED", () => {
  hideLoadingSpinner();
});
```

### 2. Show Initial Instructions

```javascript
mitekScienceSDK.on("CAMERA_DISPLAY_STARTED", () => {
  mitekScienceSDK.cmd("SHOW_HINT", {
    options: {
      hintText: "Center barcode",
      hintDuration: 1500
    }
  });
});
```

### 3. Track Capture Start Time

```javascript
let captureStartTime;

mitekScienceSDK.on("CAMERA_DISPLAY_STARTED", () => {
  captureStartTime = Date.now();
});

mitekScienceSDK.on("FRAME_CAPTURE_RESULT", () => {
  const duration = Date.now() - captureStartTime;
  analytics.track("capture_duration", duration);
});
```

### 4. Add Cancel Button

```javascript
mitekScienceSDK.on("CAMERA_DISPLAY_STARTED", () => {
  addCancelButton();
});
```

## Event Sequence

```
CAPTURE_AND_PROCESS_FRAME called
         ↓
CAMERA_DISPLAY_STARTED      ← You are here
         ↓
FRAME_PROCESSING_STARTED
         ↓
FRAME_PROCESSING_FEEDBACK (continuous)
         ↓
FRAME_CAPTURE_RESULT
```

## Related

- [FRAME_PROCESSING_STARTED](./FRAME_PROCESSING_STARTED.md) - Next in sequence
- [CAPTURE_AND_PROCESS_FRAME](../commands/CAPTURE_AND_PROCESS_FRAME.md) - Command that triggers this
- [SHOW_HINT](../commands/SHOW_HINT.md) - Show initial hint
