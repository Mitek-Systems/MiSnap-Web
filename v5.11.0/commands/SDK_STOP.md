# SDK_STOP

Stops the current capture session and removes all SDK UI elements.

## Signature

```javascript
mitekScienceSDK.cmd("SDK_STOP");
```

## Parameters

None.

## Example

```javascript
// Stop button click handler
document.getElementById("cancelBtn").addEventListener("click", () => {
  mitekScienceSDK.cmd("SDK_STOP");
});
```

## Behavior

When called:

1. Stops any active capture session
2. Stops camera/microphone streams
3. Removes SDK UI elements from the DOM
4. Triggers `FRAME_CAPTURE_RESULT` with failure status (if capture was in progress)

## Use Cases

### 1. Cancel Button

```javascript
// User-initiated cancel
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  options: { license }
});

document.getElementById("cancelBtn").addEventListener("click", () => {
  mitekScienceSDK.cmd("SDK_STOP");
});
```

### 2. Custom Timeout

```javascript
let timeoutId;

mitekScienceSDK.on("FRAME_PROCESSING_STARTED", () => {
  // Stop after 30 seconds
  timeoutId = setTimeout(() => {
    mitekScienceSDK.cmd("SDK_STOP");
    showTimeoutMessage();
  }, 30000);
});

mitekScienceSDK.on("FRAME_CAPTURE_RESULT", () => {
  clearTimeout(timeoutId);
});
```

### 3. Retrieving Failed Image

After stopping, you can retrieve the last frame from the result:

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "failure") {
    // Get the failed image for review/retry
    const failedImage = result.response.failedImage;
    showRetryOption(failedImage);
  }
});

// User wants to stop and try manual mode
document.getElementById("switchToManual").addEventListener("click", () => {
  mitekScienceSDK.cmd("SDK_STOP");
});
```

## SDK_STOP vs SDK_REMOVE_LISTENERS

| Command | Effect |
|---------|--------|
| `SDK_STOP` | Stops capture session, keeps event listeners |
| `SDK_REMOVE_LISTENERS` | Removes event listeners, does not stop capture |

Use `SDK_STOP` to end a capture session. Event listeners remain active for the next capture.

## Internal Timeout

The SDK has a built-in **5-minute timeout** for auto processing. If this is reached, the SDK automatically stops.

For video recording (`videoRecordingEnabled: true`), the timeout is **45 seconds**.

## Related

- [SDK_REMOVE_LISTENERS](./SDK_REMOVE_LISTENERS.md) - Remove event listeners
- [CAPTURE_AND_PROCESS_FRAME](./CAPTURE_AND_PROCESS_FRAME.md) - Start capture
- [FRAME_CAPTURE_RESULT](../events/FRAME_CAPTURE_RESULT.md) - Result event
