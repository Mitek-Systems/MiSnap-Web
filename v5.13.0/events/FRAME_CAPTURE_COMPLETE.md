# FRAME_CAPTURE_COMPLETE

Emitted during **auto capture** (`AUTO_CAPTURE`) when the capture session has finished (success or failure), **before** showing the configured `customSuccessMessage` and **before** the [`FRAME_CAPTURE_RESULT`](./FRAME_CAPTURE_RESULT.md) event is emitted.

This event can be used to handle initial UI cleanup (e.g. hide a cancel button) before receiving the full result object.

This event is **not** emitted for manual capture, direct processing, or voice capture.

## Signature

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_COMPLETE", () => {
  // No result payload — use FRAME_CAPTURE_RESULT for capture data.
});
```

## Payload

The callback receives an empty object `{}`. Do not rely on `request` or `response` here; subscribe to **`FRAME_CAPTURE_RESULT`** for the capture outcome and image or failure details.

## Ordering

For a typical **auto** capture session:

1. [`CAMERA_DISPLAY_STARTED`](./CAMERA_DISPLAY_STARTED.md) — preview is shown  
2. [`FRAME_PROCESSING_STARTED`](./FRAME_PROCESSING_STARTED.md) — analysis begins  
3. [`FRAME_PROCESSING_FEEDBACK`](./FRAME_PROCESSING_FEEDBACK.md) — repeated hint feedback while capturing  
4. **`FRAME_CAPTURE_COMPLETE`** — session finished; optional success-message delay may still run before the camera tears down  
5. [`FRAME_CAPTURE_RESULT`](./FRAME_CAPTURE_RESULT.md) — final result object (success or failure)

## Event Sequence

**Auto capture** (`AUTO_CAPTURE`) — same flow as in [FRAME_CAPTURE_RESULT — Event Sequence](./FRAME_CAPTURE_RESULT.md#event-sequence), with this event before the result:

```
CAMERA_DISPLAY_STARTED
         ↓
FRAME_PROCESSING_STARTED
         ↓
FRAME_PROCESSING_FEEDBACK (continuous)
         ↓
FRAME_CAPTURE_COMPLETE  ← You are here
         ↓
FRAME_CAPTURE_RESULT
```

For **manual** or **direct** capture, `FRAME_CAPTURE_COMPLETE` is not emitted; use [`FRAME_CAPTURE_RESULT`](./FRAME_CAPTURE_RESULT.md) after the last relevant feedback.

## Related

- [`FRAME_CAPTURE_RESULT`](./FRAME_CAPTURE_RESULT.md) — Full result payload  
- [Capture options — custom success message](../configuration/capture-options.md) — `customSuccessMessage` and success hint timing  
