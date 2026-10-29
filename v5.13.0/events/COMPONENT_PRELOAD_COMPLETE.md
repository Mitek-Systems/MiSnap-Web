# COMPONENT_PRELOAD_COMPLETE

Triggered when component preloading started by `COMPONENT_PRELOAD` has finished successfully. Register this event to know when it is safe to send capture commands (e.g. `CAPTURE_AND_PROCESS_FRAME`).

## Signature

```javascript
mitekScienceSDK.on("COMPONENT_PRELOAD_COMPLETE", () => {
  // Preload finished -> Safe to start capture
});
```

## When It Fires

- After all requested components (e.g. DOCUMENTS, SELFIE, BARCODE) have finished loading.
- The `COMPONENT_PRELOAD` command returns immediately with success (0); this event fires later when loading actually completes.

## When It Does Not Fire

- If preload fails (e.g. invalid license, WASM not supported, or component loader failure with code 5513), `SDK_ERROR` is emitted instead. No `COMPONENT_PRELOAD_COMPLETE` event is fired.

## Example

```javascript
mitekScienceSDK.on("SDK_ERROR", (err) => {
  console.error("SDK Error:", err.code);
});

mitekScienceSDK.on("COMPONENT_PRELOAD_COMPLETE", () => {
  console.log("Preload finished");
  enableCaptureButton();
});

mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  preloadComponents: ["DOCUMENTS", "SELFIE"],
  mitekSDKPath: "./mitekSDK/",
  options: { license: "your-license" }
});
```

## Related

- [COMPONENT_PRELOAD](../commands/COMPONENT_PRELOAD.md) - Command that starts preload
- [CAPTURE_AND_PROCESS_FRAME](../commands/CAPTURE_AND_PROCESS_FRAME.md) - Start capture after preload completes
