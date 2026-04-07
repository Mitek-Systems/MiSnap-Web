# Commands Reference

The MiSnap SDK uses a command-based API accessed through the `cmd` method on the global `mitekScienceSDK` object.

## Usage

```javascript
mitekScienceSDK.cmd("COMMAND_NAME", configurationObject);
```

## Available Commands

| Command | Description |
|---------|-------------|
| [CAPTURE_AND_PROCESS_FRAME](./CAPTURE_AND_PROCESS_FRAME.md) | Start camera capture with auto/manual mode |
| [CAPTURE_AND_PROCESS_VOICE](./CAPTURE_AND_PROCESS_VOICE.md) | Capture voice sample for biometrics |
| [PROCESS_FRAME](./PROCESS_FRAME.md) | Process an existing image directly |
| [COMPONENT_PRELOAD](./COMPONENT_PRELOAD.md) | Preload SDK components for faster UX |
| [SDK_STOP](./SDK_STOP.md) | Stop the current capture session |
| [SHOW_HINT](./SHOW_HINT.md) | Display hint message during capture |
| [HINT_MESSAGE_SIZE](./HINT_MESSAGE_SIZE.md) | Change hint font size dynamically |
| [SDK_REMOVE_LISTENERS](./SDK_REMOVE_LISTENERS.md) | Remove all registered event listeners |

## Commands Requiring License

These commands require a valid license in the `options` object:

- `CAPTURE_AND_PROCESS_FRAME`
- `CAPTURE_AND_PROCESS_VOICE`
- `PROCESS_FRAME`
- `COMPONENT_PRELOAD`

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  mitekSDKPath: "./mitekSDK/",
  options: {
    license: "your-license-here"  // Required
  }
});
```

## Configuration Object

Most commands accept a configuration object with these common properties:

| Property | Type | Description |
|----------|------|-------------|
| `mode` | `string` | Capture mode (`AUTO_CAPTURE`, `MANUAL_CAPTURE`, `DIRECT`) |
| `documentType` | `string` | Type of document to capture (see [Document Types](../configuration/document-types.md)) |
| `mitekSDKPath` | `string` | Path to SDK files (default: `"./mitekSDK/"`) |
| `options` | `object` | Capture options including license (see [Capture Options](../configuration/capture-options.md)) |

## Capture Modes

| Mode | Description |
|------|-------------|
| `AUTO_CAPTURE` | Real-time camera with automatic capture when IQA passes |
| `MANUAL_CAPTURE` | Manually capture an image (**desktop**: opens file picker, **mobile**: opens camera) |
| `DIRECT` | Process an existing image |

## Timeout Behavior

The SDK has an internal **5-minute timeout** for auto processing. If exceeded, the capture stops automatically.

For video recording (`videoRecordingEnabled: true`), the timeout is reduced to **45 seconds**.

### Custom Timeout

To set a custom timeout less than 5 minutes, you can do the following:

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_STARTED", () => {
  // Stop after 30 seconds
  setTimeout(() => {
    mitekScienceSDK.cmd("SDK_STOP");
  }, 30000);
});
```

## Deprecated Commands

### `CAPTURE_FRAME` (Deprecated)

**Do not use.** This command is deprecated and will be removed in v6.0.0.

It returns the last frame regardless of IQA status, which may produce unusable images.

**Alternative:** Use `SDK_STOP` and retrieve `failedImage` from the result, or fall back to manual mode.

## Related

- [Events Reference](../events/README.md) - Handle command responses
- [Configuration Options](../configuration/README.md) - Customize command behavior
