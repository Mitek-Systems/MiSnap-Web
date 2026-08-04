# IDLive Web Capture → MiSnap Web SDK Migration Guide

This guide is for integrators moving from **IDLive Web Capture** (`<idlive-face-capture>`) to **MiSnap Web SDK** for AI-based RTS (Real Time Security).

## Before You Begin

Please see [Installation](../INSTALLATION.md), [Quick Start](../QUICK_START.md), and [AI-Based RTS](../configuration/capture-options.md#ai-based-rts-real-time-security) to familiarize yourself with how MiSnap is installed and configured.

Once you've done that, continue with this transition guide, which will outline how to map IDLive Face Plus functionality to MiSnap.

> MiSnap 5.12 introduces some new features intended for use with on-prem deployments.  These include use of a custom encryption key, the encryption key ID when multiple keys are being used, and per-capture optional metadata. **MiVIP** and **Mobile Verify** do not currently support these features. If a custom key is used in MiSnap and the payload is sent to a Mitek SaaS back-end, payload decryption will fail on the server side and the payload will not be accepted. These features are only compatible with on-prem IDLive docker solutions.
>
> If you are unsure which deployment model you are on, check with your Mitek account team before enabling this configuration.

In MiSnap you enable AI-based RTS with `aiBasedRtsEnabled: true` on an `AUTO_CAPTURE` session instead of embedding an IDLive element.

## Development Note

Integrators familiar with IDLive know that it uses anti-debugging techniques to protect against frontend injection attacks. These protections prevent the use of browser DevTools. 

MiSnap has two SDK build packages: **development** (`-dev`) and **production** (`-prod`).

For local development, use the **development** (`-dev`) package. Do **not** deploy the **`-dev`** package to production. See [SDK build variants](../INSTALLATION.md#sdk-build-variants) in the Installation guide.

---

## Attributes

[IDLive attributes reference](https://docs.idrnd.net/idlivefaceplus/web/usage/attributes/)


| IDLive                              | MiSnap Web SDK             | Notes                                                         |
| ----------------------------------- | -------------------------- | ------------------------------------------------------------- |
| `sdk_path`                          | `mitekSDKPath`             | Default `"./mitekSDK/"`                                       |
| `capture_mode="encryptedPayload"`   | `aiBasedRtsEnabled: true`  | Default IDLive mode                                           |
| `capture_mode="singleImage"`        | `aiBasedRtsEnabled: false` | Image only, no RTS bundle                                     |
| `payload_size`                      | `aiBasedRtsPayloadSize`    | IDLive defaults `"normal"`; MiSnap defaults `"small"`         |
| `device_id`                         | —                          | MiSnap selects an ideal camera                                |
| `mask_hidden`                       | CSS                        | Hide or style the guide yourself                              |
| `auto_close_disabled`               | —                          | Not supported. Auto capture closes after a successful capture |
| `auto_capture_disabled`             | —                          | Not supported                                                 |
| `audio_enabled`                     | —                          | Not supported                                                 |
| `face_not_centered_check_enabled`   | Built-in IQA               | Not individually configurable                                 |
| `sunglasses_detected_check_enabled` | Built-in IQA               | Not individually configurable                                 |
| `mouth_open_check_enabled`          | Built-in IQA               | Not individually configurable                                 |
| `face_blurry_check_enabled`         | Built-in IQA               | Not individually configurable                                 |
| `dark_image_check_enabled`          | Built-in IQA               | Not individually configurable                                 |
| `eyes_closed_check_enabled`         | Built-in IQA               | Not individually configurable                                 |


**Before:**

```html
<idlive-face-capture
  capture_mode="encryptedPayload"
  payload_size="normal"
  sdk_path="./idlive-capture/"
></idlive-face-capture>
```

**After:**

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "SELFIE",
  mitekSDKPath: "./mitekSDK/",
  options: {
    license,
    aiBasedRtsEnabled: true,
    aiBasedRtsPayloadSize: "normal"
  }
});
```

---

## Events

[IDLive events reference](https://docs.idrnd.net/idlivefaceplus/web/usage/events/)

Replace IDLive `addEventListener` handlers with `mitekScienceSDK.on(...)`.


| IDLive             | MiSnap Web SDK                                                        | Notes                                                                                                                                                                                                                                                                                             |
| ------------------ | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `beforeInitialize` | —                                                                     |                                                                                                                                                                                                                                                                                                   |
| `initialize`       | [`COMPONENT_PRELOAD_COMPLETE`](../events/COMPONENT_PRELOAD_COMPLETE.md) | Emitted once [`COMPONENT_PRELOAD`](../commands/COMPONENT_PRELOAD.md) has finished                                                                                                                                                                                                                 |
| `beforeOpen`       | —                                                                     |                                                                                                                                                                                                                                                                                                   |
| `open`             | [`CAMERA_DISPLAY_STARTED`](../events/CAMERA_DISPLAY_STARTED.md)       | No callback data — run your logic once the preview is visible. MiSnap attaches `<video id="mitekMediaStreamSource">` inside `<div id="mitekDisplayContainer">`. Use `videoContainerId` to mount in your own element. See [Custom Container](../configuration/capture-options.md#custom-container) |
| `faceDetection`    | [`FRAME_PROCESSING_FEEDBACK`](../events/FRAME_PROCESSING_FEEDBACK.md) |                                                                                                                                                                                                                                                                                                   |
| `detection`        | [`FRAME_PROCESSING_FEEDBACK`](../events/FRAME_PROCESSING_FEEDBACK.md) | Map `key` to your messages                                                                                                                                                                                                                                                                        |
| `beforeCapture`    | —                                                                     |                                                                                                                                                                                                                                                                                                   |
| `capture`          | [`FRAME_CAPTURE_RESULT`](../events/FRAME_CAPTURE_RESULT.md)           | `response.status === "success"`                                                                                                                                                                                                                                                                   |
| `close`            | After [`SDK_STOP`](../commands/SDK_STOP.md) or session end            |                                                                                                                                                                                                                                                                                                   |
| `error`            | [`SDK_ERROR`](../events/SDK_ERROR.md)                                 |                                                                                                                                                                                                                                                                                                   |
| `licenseInfo`      | —                                                                     | No event equivalent. See [Checking license status](../INSTALLATION.md#checking-license-status)                                                                                                                                                                                                     |


### Capture payload


| IDLive `capture` detail | MiSnap `FRAME_CAPTURE_RESULT`           |
| ----------------------- | --------------------------------------- |
| `encryptedFile`         | `response.aiBasedRts` — base64 data URL |
| `photo`                 | `response.imageData` — JPEG data URL    |


```javascript
mitekScienceSDK.on("SDK_ERROR", (err) => {
  console.error("SDK Error:", err.code, err.type);
});

mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status !== "success") return;
  const { aiBasedRts, imageData } = result.response;
  // continue your flow
});
```

#### Blob for on-prem IAD servers

```javascript
function dataURLToBlob(dataURL) {
  const [header, base64] = dataURL.split(",");
  const mime = header.match(/:(.*?);/)?.[1] || "application/octet-stream";
  const bytes = Uint8Array.from(atob(base64), (c) => c.charCodeAt(0));
  return new Blob([bytes], { type: mime });
}

const encryptedFile = dataURLToBlob(aiBasedRts);
```

---

## Actions

[IDLive actions reference](https://docs.idrnd.net/idlivefaceplus/web/usage/actions/)


| IDLive                            | MiSnap Web SDK                                                          | Notes                                                                |
| --------------------------------- | ----------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `openCamera()`                    | [`CAPTURE_AND_PROCESS_FRAME`](../commands/CAPTURE_AND_PROCESS_FRAME.md) |                                                                      |
| `closeCamera()`                   | [`SDK_STOP`](../commands/SDK_STOP.md)                                   |                                                                      |
| `takePhoto()`                     | —                                                                       | Not supported                                                        |
| `setLicense(license, type)`       | `options.license`                                                       | On `COMPONENT_PRELOAD` and capture commands                          |
| `getLicenseInfo(type)`            | [Checking license status](../INSTALLATION.md#checking-license-status)                                   | `mitekScienceSDK.getLicenseInfo(license)`                            |
| `setEncryptionKey(blob[, keyId])` | `aiBasedRtsEncryptionKey: { publicKey, keyId? }`                        | `publicKey` is base64 DER. Must match server when using a custom key |
| `setExternalMeta(string)`         | `aiBasedRtsAdditionalData`                                              | Optional string                                                      |
| `setCustomQualityFunction(fn)`    | Not supported                                                           |                                                                      |


When using a custom encryption key (`aiBasedRtsEncryptionKey`), it must match your server payload encryption configuration.

**Before:**

```html
<link rel="stylesheet" href="your-app.css" />
<script type="module">
  import "idlive-face-capture-web";
</script>

<idlive-face-capture id="capture"></idlive-face-capture>
```

```javascript
const el = document.getElementById("capture");
el.addEventListener("initialize", () => {
  el.setLicense(license, "faceDetector");
  el.setEncryptionKey(publicKeyBlob, "my-key-id");
  el.setExternalMeta("session-id");
  el.openCamera();
});
```

**After:**

```html
<link rel="stylesheet" href="mitekSDK/app.css" />
<script src="mitek-science-sdk.js"></script>
```

```javascript
mitekScienceSDK.on("SDK_ERROR", (err) => {
  console.error("SDK Error:", err.code, err.type);
});

mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["SELFIE", "AI_BASED_RTS"],
  options: { license }
});

mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "SELFIE",
  mitekSDKPath: "./mitekSDK/",
  options: {
    license,
    aiBasedRtsEnabled: true,
    aiBasedRtsEncryptionKey: {
      publicKey: "<base64-DER-public-key>",
      keyId: "my-key-id"
    },
    aiBasedRtsAdditionalData: "session-id"
  }
});
```

---

## References

- [Selfie Capture Guide](../guides/selfie-capture.md)
- [IDLive Face Plus — Attributes](https://docs.idrnd.net/idlivefaceplus/web/usage/attributes/)
- [IDLive Face Plus — Events](https://docs.idrnd.net/idlivefaceplus/web/usage/events/)
- [IDLive Face Plus — Actions](https://docs.idrnd.net/idlivefaceplus/web/usage/actions/)

