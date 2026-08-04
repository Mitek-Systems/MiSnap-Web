# Known Issues

Known issues, limitations, and device compatibility notes for the MiSnap Web SDK.

## Device Compatibility

### Unsupported Devices (WebGL Issues)

The following devices have known WebGL compatibility issues that can affect selfie auto-processing. Please use manual capture mode as a fallback.

| Manufacturer | Models |
|--------------|--------|
| Samsung | Galaxy S3, S4 Mini, S5, J2 Prime, Tab S, Mega |
| HTC | One M8, M9 |
| Google | Nexus 7, Nexus 9 |
| Motorola | Moto X 2nd gen, Moto G |
| Sony | Xperia Z3 |
| Asus | Zenfone series |

**Workaround:** Implement a fallback to manual capture mode when WebGL is unavailable:

```javascript
mitekScienceSDK.on("SDK_ERROR", (err) => {
  if (err.code === 336) {  // WEBGL_NOT_SUPPORTED
    // Switch to manual capture
    startManualCapture();
  }
});
```

## iOS Issues

### Blurry Images in Safari Private Browsing

Private browsing can affect the camera selection logic in older SDK versions.

**Resolution:** Update to SDK v5.8.3 or later.

### Voice Capture Requires Safari 16+

Voice capture using `CAPTURE_AND_PROCESS_VOICE` requires Safari 16 or later on iOS devices. Earlier versions do not support the required MediaRecorder API.

### Safari 13 Limited Support

Safari 13 has limited WebAssembly support which may cause SDK initialization failures.

**Resolution:** Upgrade to Safari 14+.

## Android Issues

### Android Firefox Camera Issues

There are two known issues with the camera presentation in Android Firefox:

1. Camera is rotated vs UI, e.g. an Android device in landscape mode displays the camera stream in portrait orientation ([bug 1802849](https://bugzilla.mozilla.org/show_bug.cgi?id=1802849))
2. The camera stream appears "zoomed in" due to a bug with how media constraints are applied ([bug 1992882](https://bugzilla.mozilla.org/show_bug.cgi?id=1992882))

**Resolution:** These are issues with the Firefox browser itself. Until they are fixed, please use a different browser.

### Huawei Camera Selection

On some Huawei devices, the SDK previously selected the wrong camera, resulting in black or white camera output.

**Resolution:** Fixed in SDK v5.5.0+. Update to the latest SDK version.

## AI-Based RTS Issues

### Browser DevTools Interference

AI-based RTS uses anti-debugging techniques that prevent the use of browser DevTools when integrated with a **production** (`-prod`) SDK package. With `aiBasedRtsEnabled`, DevTools may trigger debugger breakpoints, page crashes, or `AI_BASED_RTS_ERROR`. This is intentional protection against injection attacks in the frontend.

For local development and debugging, use the **development** (`-dev`) SDK package. Do **not** deploy the **`-dev`** package to production. See [SDK build variants](./INSTALLATION.md#sdk-build-variants) in the Installation guide.

### Minimum Browser Versions for AI-based RTS

AI-based RTS requires newer browser versions than the base SDK:

| Browser | Minimum Version |
|---------|-----------------|
| Chrome (Desktop) | 103+ |
| Chrome (Android) | 112+ |
| Firefox (Desktop) | 103+ |
| Firefox (Android) | 110+ |
| Safari (Desktop) | 15.1+ |
| Safari (iOS) | 15.0+ |
| Edge | 103+ |
| Samsung Internet | 11.1+ |

## Video Recording Issues

### Missing Final Chunk

In SDK versions prior to v5.8.1, video recordings could be missing the final chunk of data.

**Resolution:** Update to SDK v5.8.1 or later.

### MediaRecorder Browser Support

Video recording requires MediaRecorder API support:

| Platform | Minimum Version |
|----------|-----------------|
| iOS | 14.3+ |
| Chrome | 89+ |

## MRZ Redaction Limitations

### Still Images Only

MRZ redaction (`redactMRZOptionalData: true`) applies to still images only. If video recording is enabled, the recorded video will **not** have MRZ data redacted.

**Recommendation:** Disable video recording when MRZ redaction is required:

```javascript
options: {
  license,
  redactMRZOptionalData: true,
  videoRecordingEnabled: false  // Video is NOT redacted
}
```

### Netherlands Documents Only

MRZ redaction only applies to Netherlands (NLD) documents:
- Netherlands Passport (v5.2.0+)
- Netherlands ID Card (v5.4.0+)

Non-Dutch documents are not affected by this setting.

## Barcode Scanning

### License Required for Data Extraction

PDF417 barcode scanning captures images without a barcode license, but the extracted data will contain asterisks (`***`) instead of actual values.

### Single Barcode Type Detection (v5.10.0+)

As of v5.10.0, the SDK only detects the specific barcode type configured:
- `PDF417_BARCODE` - Detects PDF417 only
- `QR_BARCODE` - Detects QR codes only

Previous versions could detect either type regardless of configuration.

## CSS Requirement (v5.9.0+)

### Missing UI Elements

Starting with v5.9.0, the SDK CSS file must be explicitly linked. Without it, guide frames and hints will not display.

**Required:**

```html
<link rel="stylesheet" href="mitekSDK/app.css" />
```

## Related

- [Troubleshooting Guide](./guides/troubleshooting.md) - Solutions for common issues
- [Release Notes](./RELEASE_NOTES.md) - Version history and fixes
- [SDK_ERROR Event](./events/SDK_ERROR.md) - Error code reference
