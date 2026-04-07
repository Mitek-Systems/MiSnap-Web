# Troubleshooting Guide

Common issues and solutions for the MiSnap Web SDK.

## Quick Diagnostics

### Check SDK Version

```javascript
console.log("SDK Version:", mitekScienceSDK.getVersion());
```

### Check Browser Support

```javascript
// Check required APIs
const checks = {
  getUserMedia: !!navigator.mediaDevices?.getUserMedia,
  WebAssembly: typeof WebAssembly === "object",
  WebGL: !!document.createElement("canvas").getContext("webgl"),
  MediaRecorder: typeof MediaRecorder !== "undefined"
};

console.log("Browser support:", checks);
```

## Common Issues

### Webpack build warnings ("Critical dependency...")

When bundling the SDK with webpack, you may see "Critical dependency: the request of a dependency is an expression". This is expected and harmless; the SDK ships a config snippet to suppress it. See [Installation – Bundling with Webpack](../INSTALLATION.md#bundling-with-webpack) for how to suppress this warning.

### Camera Not Starting

**Symptoms:**
- Blank screen after starting capture
- No camera preview

**Solutions:**

1. **Check HTTPS** - Camera requires secure context
   ```
   http://localhost:8080  ✓ (localhost exempt)
   https://example.com    ✓
   http://example.com     ✗
   ```

2. **Check permissions** - User may have denied camera
   ```javascript
   navigator.mediaDevices.getUserMedia({ video: true })
     .then(() => console.log("Camera OK"))
     .catch(err => console.log("Camera denied:", err));
   ```

3. **Check error code**
   | Code | Issue |
   |------|-------|
   | 112 | No camera found |
   | 113 | Permission denied |
   | 120 | Camera failed to start |

### License Errors

**Error 4006: `INVALID_LICENSE`**

```javascript
mitekScienceSDK.on("SDK_ERROR", (err) => {
  if (err.code === 4006) {
    // License format invalid
  }
});
```

**Solutions:**
1. Verify license string is complete (no truncation)
2. Check for extra whitespace or line breaks
3. Ensure license matches your domain
4. Contact Mitek support for new license

**Error 4007: `INVALID_LICENSE_EXPIRED`**

License has expired. Contact Mitek for renewal.

**Error 4040: `INVALID_BARCODE_LICENSE`**

Barcode capture requires additional license. Check:
1. Combined text license includes barcode
2. Or `mitek_license.png` is in `./mitekSDK/`

### WASM Loading Errors

**Symptoms:**
- SDK hangs on initialization
- Console shows WASM errors

**Solutions:**

1. **Check MIME type** - Server must serve `.wasm` as `application/wasm`
   
   nginx:
   ```nginx
   types { application/wasm wasm; }
   ```
   
   Apache:
   ```apache
   AddType application/wasm .wasm
   ```

2. **Check file paths** - Verify `mitekSDKPath` is correct
   ```javascript
   mitekSDKPath: "./mitekSDK/"  // Relative to HTML file
   mitekSDKPath: "/assets/mitekSDK/"  // Absolute path
   ```

3. **Check network tab** - Verify WASM files load with 200 status

### Blank Screen / Missing UI

**Symptoms:**
- Camera works but no guide frame or hints
- Capture area is empty

**Solutions:**

1. **Link CSS file** (required since v5.9.0)
   ```html
   <link rel="stylesheet" href="mitekSDK/app.css" />
   ```

2. **Check custom container** - If using `videoContainerId`, ensure element exists
   ```html
   <div id="myContainer"></div>
   ```
   ```javascript
   options: { videoContainerId: "myContainer" }
   ```

### Hints Not Showing / Frozen

**Symptoms:**
- First hint appears but never updates
- "Initializing..." stuck on screen

**Solutions:**

1. **Check browser WebAssembly support**
   - Samsung Internet has known issues
   - Implement timeout fallback:
   ```javascript
   setTimeout(() => {
     mitekScienceSDK.cmd("SDK_STOP");
     showManualMode();
   }, 30000);
   ```

2. **Check for JavaScript errors** - Callback errors can freeze hints
   ```javascript
   mitekScienceSDK.on("SDK_ERROR", (err) => {
     if (err.code === 5420) {
       console.error("Callback error:", err.cause);
     }
   });
   ```

### Selfie Issues

**No face detected:**
- Improve lighting
- Clean camera lens
- Face the camera directly

**WebGL errors:**
- Some older devices don't support WebGL
- Check error code 336 (WEBGL_NOT_SUPPORTED)
- Fallback to manual mode

**AI-based RTS errors:**
- Close browser dev tools
- Check minimum browser version (Chrome 103+, etc.)

### Video Recording Issues

**Recording missing last chunk:**
- Fixed in v5.8.1 - update SDK

**MediaRecorder not supported:**
- iOS requires 14.3+
- Chrome requires 89+
- Error code 130

### Cross-Origin Issues

**CORS errors:**
- Use `loadWorkerCrossOrigin: true` for loading SDK worker cross-origin
- Ensure server hosting the SDK files returns proper CORS headers (`Access-Control-Allow-Origin`) 

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "https://host.example.com/mitekSDK/",
  loadWorkerCrossOrigin: true,
  preloadComponents: ["DOCUMENTS"],
  options: { license }
});
```

**Why is this needed?** Browsers block Web Workers loaded from a different origin due to the [same-origin policy](https://developer.mozilla.org/en-US/docs/Web/Security/Same-origin_policy). When `loadWorkerCrossOrigin` is enabled, the SDK fetches the worker script and loads it as a [Blob URL](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL), bypassing the cross-origin restriction.

## Device Compatibility

### Unsupported Devices

These devices have known WebGL compatibility issues (selfie auto-processing):

- Galaxy S3, S4 Mini, S5
- Galaxy J2 Prime, Tab S, Mega
- HTC One M8, M9
- Nexus 7, 9
- Moto X 2nd gen, Moto G
- Xperia Z3
- Asus Zenfone

**Fallback:** Use manual capture mode for these devices.

### iOS-Specific Issues

**Blurry Images in Safari Private Browsing:**
- Private browsing can affect the camera selection logic in older SDK versions
- Update to SDK v5.8.3 or later

**Camera blur on iOS 16:**
- Update to latest iOS version
- SDK v5.2.3+ includes fixes

**Safari 13:**
- Limited WebAssembly support
- Upgrade to Safari 14+

**Voice capture:**
- Requires Safari 16+

### Android-Specific Issues

**Android Firefox Camera Issues:**
1. Camera is rotated vs UI, e.g. an Android device in landscape mode displays the camera stream in portrait orientation ([bug 1802849](https://bugzilla.mozilla.org/show_bug.cgi?id=1802849))
2. The camera stream appears "zoomed in" due to a bug with how media constraints are applied ([bug 1992882](https://bugzilla.mozilla.org/show_bug.cgi?id=1992882))

These are issues with the Firefox browser itself. Until they are fixed, please use a different browser.

**Huawei black/white camera:**
- Fixed in v5.5.0+
- SDK now selects correct camera

**Samsung Internet:**
- Some versions have WebAssembly issues
- Implement timeout fallback

## Error Code Reference

### Camera Errors (100s)

| Code | Type | Solution |
|------|------|----------|
| 111 | `CONSTRAINT_NOT_SATISFIED` | Device not supported |
| 112 | `NO_CAMERA_FOUND` | Check camera connection |
| 113 | `CAMERA_PERMISSION_DENIED` | Request permission again |
| 120 | `CAMERA_UNKNOWN_DEVICE_ISSUE` | Try different browser |

### Browser Errors (300s)

| Code | Type | Solution |
|------|------|----------|
| 334 | `WASM_NOT_SUPPORTED` | Update browser |
| 335 | `DEVICE_NOT_SUPPORTED` | Use different device |
| 336 | `WEBGL_NOT_SUPPORTED` | Manual mode fallback |

### License Errors (4000s)

| Code | Type | Solution |
|------|------|----------|
| 4006 | `INVALID_LICENSE` | Check license string |
| 4007 | `INVALID_LICENSE_EXPIRED` | Renew license |
| 4040 | `INVALID_BARCODE_LICENSE` | Add barcode license |

## Debug Mode

Enable console logging:

```javascript
// Log all events
["SDK_ERROR", "CAMERA_DISPLAY_STARTED", "FRAME_PROCESSING_STARTED", 
 "FRAME_PROCESSING_FEEDBACK", "FRAME_CAPTURE_RESULT"].forEach(event => {
  mitekScienceSDK.on(event, (data) => {
    console.log(`[${event}]`, data);
  });
});
```

## Getting Support

1. **Check version** - `mitekScienceSDK.getVersion()`
2. **Collect error codes** - From `SDK_ERROR` events
3. **Note device/browser** - Include OS version
4. **Network tab** - Screenshot failed requests
5. **Contact:** https://www.miteksystems.com/support

## Related

- [SDK_ERROR](../events/SDK_ERROR.md) - Error codes
- [Installation](../INSTALLATION.md) - Setup requirements
- [Browser Requirements](../README.md#supported-browsers) - Compatibility
- [Known Issues](../KNOWN_ISSUES.md)
