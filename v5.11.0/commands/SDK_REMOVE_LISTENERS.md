# SDK_REMOVE_LISTENERS

Removes all event listeners registered via `mitekScienceSDK.on()`.

## Signature

```javascript
mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");
```

## Parameters

None.

## Example

```javascript
// Remove all listeners
mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");
```

## Behavior

When called:

1. Unregisters all event handlers added via `mitekScienceSDK.on()`
2. SDK stops sending events to your application
3. Does **not** stop an active capture session

## Warning

After calling `SDK_REMOVE_LISTENERS`, you will not receive any SDK events until you re-register handlers.

```javascript
// After this, no events will be received
mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");

// Must re-register to receive events again
mitekScienceSDK.on("SDK_ERROR", handleError);
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", handleResult);
```

## When to Use

### 1. User Cancels Upload (Manual Mode)

When a user cancels a file upload dialog, the SDK has no way to know the listeners are no longer needed:

```javascript
const fileInput = document.getElementById("upload");

fileInput.addEventListener("change", (e) => {
  if (e.target.files.length === 0) {
    // User cancelled - clean up listeners
    mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");
  }
});
```

### 2. Component Unmount (SPA)

Clean up when navigating away in a single-page application:

```javascript
// React example
useEffect(() => {
  mitekScienceSDK.on("FRAME_CAPTURE_RESULT", handleResult);
  
  return () => {
    mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");
  };
}, []);
```

### 3. Reset Between Flows

```javascript
function startNewFlow() {
  // Clean up previous flow's listeners
  mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");
  
  // Register fresh listeners
  mitekScienceSDK.on("SDK_ERROR", handleError);
  mitekScienceSDK.on("FRAME_CAPTURE_RESULT", handleResult);
  
  // Start capture
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", config);
}
```

## SDK_REMOVE_LISTENERS vs SDK_STOP

| Command | Effect |
|---------|--------|
| `SDK_REMOVE_LISTENERS` | Removes event handlers, capture continues |
| `SDK_STOP` | Stops capture session, keeps event handlers |

### Correct Order for Full Cleanup

```javascript
// Stop capture first
mitekScienceSDK.cmd("SDK_STOP");

// Then remove listeners
mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");
```

## Do Not Use During Active Capture

Calling `SDK_REMOVE_LISTENERS` during an active capture will prevent you from receiving the result:

```javascript
// BAD: Will never receive result
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", config);
mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");  // Lost the result handler!

// GOOD: Stop capture first, then clean up
mitekScienceSDK.cmd("SDK_STOP");
mitekScienceSDK.cmd("SDK_REMOVE_LISTENERS");
```

## Related

- [SDK_STOP](./SDK_STOP.md) - Stop capture session
- [Events Reference](../events/README.md) - All events
