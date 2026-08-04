# Release Notes

Version history for the MiSnap Web SDK.

---

## 5.12.0

### Updates

- **SELFIE capture improvements**
  - Improved face angle detection (pitch, yaw, and roll). Emits `MISNAP_HEAD_SKEWED` in `FRAME_PROCESSING_FEEDBACK` event.
  - The `MITEK_ERROR_FOCUS` hint key is no longer emitted for `SELFIE` auto capture.
    - After extensive testing, it was determined that focus detection for `SELFIE` was unreliable and causing unnecessary friction during auto capture. Users will now experience a smoother capture process.
  - See [Selfie Capture — Hint Messages](guides/selfie-capture.md#hint-messages).

- **Auto capture — `FRAME_CAPTURE_COMPLETE` event**
  - New event emitted during `AUTO_CAPTURE` when the capture is complete. This is emitted before the `customSuccessMessage` (if configured), teardown, and `FRAME_CAPTURE_RESULT` event. See [FRAME_CAPTURE_COMPLETE](events/FRAME_CAPTURE_COMPLETE.md).

- **IDLive to MiSnap transition guide and new AI-based RTS capture options**
  - New guide for integrators moving from IDLive Web Capture to MiSnap Web SDK for AI-based RTS. See [IDLive to MiSnap Transition Guide](guides/idlive-to-misnap-transition.md).
  - New command options when AI-based RTS is enabled: `aiBasedRtsPayloadSize`, `aiBasedRtsEncryptionKey` (valid base64 `publicKey` and optional `keyId`), and `aiBasedRtsAdditionalData` (optional external metadata string). See [Capture options — AI-Based RTS](configuration/capture-options.md#ai-based-rts-real-time-security).

- **SDK build variants**
  - Two packaged builds are available. See [SDK build variants](INSTALLATION.md#sdk-build-variants):
    - **development** (`-dev`): Useful for local dev and testing with AI-based RTS (allows use of browser DevTools)
    - **production** (`-prod`): Uses AI-based RTS frontend protections
  - The **development** build must never be deployed to a production environment.

- **TypeScript support**
  - The production package now includes TypeScript declaration files (`mitek-science-sdk.d.ts` and supporting `types/`). See [Installation — Including the SDK](INSTALLATION.md#including-the-sdk) for script-tag and bundler examples.

- **Improved experience for Android TalkBack users**
  - The Android screen reader (TalkBack) was overly chatty before announcing hints. This resolves the issue.

---

## 5.11.0

### Updates

- **Back camera selfie**
  - New `backCameraSelfie` option for `SELFIE` capture. When `true`, the SDK uses the back (environment) camera instead of the front camera.
  - If AI-based RTS (`aiBasedRtsEnabled`) is enabled, the front camera is always used (RTS requires it); `backCameraSelfie` is ignored in that case.
  - On devices with no back camera or only one camera, the SDK falls back to the front or only available camera.
  - The video preview is not mirrored when using the back camera for selfie. See the [Selfie Capture Guide](guides/selfie-capture.md#back-camera-selfie) for details.

- **VDS (Visible Digital Seal) header parsing for barcode scans**
  - Barcode scan responses now include a `vds` property.
  - For most barcodes: `vds: { isVds: false }`.
  - For barcodes with a valid ICAO 9303-13 VDS header (e.g. MiDNI QR codes in Spain): `vds` includes `isVds: true`, parsed `header` fields (`issuingCountry`, `featureDefinitionReference`, `typeCategory`), and when applicable an encrypted `payload` for backend processing.
  - See the [Barcode Scanning Guide](guides/barcode-scanning.md#vds-and-midni) for details and examples.

- **License Status**
  - New `getLicenseInfo` API to validate an SDK license.
  - See [Checking license status](INSTALLATION.md#checking-license-status) in the Installation guide.

- **Documentation format**
  - Product documentation is now provided as **Markdown** (`.md`) under the `documentation/` tree in the package instead of a PDF. 
  - You can browse files directly or run the local doc server for navigation and search. See [Viewing the Documentation](README.md#viewing-the-documentation).

---

## 5.10.0

### Updates

- **Selfie capture condition strategies**
  - **Detect Smile** (default): can be disabled with `disableSmileDetection`
  - **Countdown Timer** (NEW): shows 3...2...1 before capture, disable with `disableSelfieCountdown`
  - **Good IQA**: captures first successful frame after 1 second delay, disable with `disableCaptureDelay`

- **Option to require MRZ for `DL_FRONT`**
  - New `mrzRequired` option for driver's licenses with MRZ sections
  - If enabled, MRZ must be present for successful capture
  - Can be used for ID cards with MRZ to distinguish between front and back
  - Does not return MRZ data in the result
  - This setting has no effect on `PASSPORT`, where MRZ is required by default

- **Improved guide frame sizing**
  - Aspect ratio adjusts based on document type

- **Accessibility improvements**
  - Improved hint message examples
  - New `accessibilityMode` option for sensible screen reader defaults
  - New `announceDuplicateHints` option for re-announcing unchanged hints

- **Demo page updates**
  - Accessibility Mode checkbox
    - Changes settings to improve the experience for those using screen readers
    - These are suggestions. Individual settings can still be changed.
    - If capture fails, a warning with the last hint will be announced
  - Disable Capture Delay checkbox
    - Removes the one second delay upon successful auto capture
    - This can be useful for visually impaired users, as it reduces friction during the capture process
  - Selfie Capture Conditions selector
    - Selection changes the command options used for `SELFIE` auto capture
  - MRZ Required checkbox
    - Sets `mrzRequired` option for `DL_FRONT` capture

### Fixes

- Fixed blue-tinted images on some phones (e.g., Moto G84)
- Fixed barcode type detection. Only configured type (QR or PDF417) is now detected

---

## 5.9.0

### Updates

- Added support for loading cross-origin web worker
- Updated facial recognition for faster performance on older devices
- Improved sharpness detection for identity documents (except generic)
- Split demo page into separate HTML, CSS, JS files
  - **Note:** Integrators must link/import the CSS
- Decreased AI-based RTS payload size

### Fixes

- Selfie images in manual mode now have correct orientation
- Microsoft Surface devices no longer mirror video
- Support for CSP `style-src` directive

---

## 5.8.3

### Updates

- Enhanced camera selection logic for updated Chrome and Safari versions

---

## 5.8.2

### Fixes

- Improvements for `SELFIE` preload

---

## 5.8.1

### Fixes

- Fixed video recording sometimes missing last chunk when using `videoRecordingEnabled`
- Fixed inline `data:` URLs causing CSP violations

---

## 5.8.0

### Updates

- New `customSuccessMessage` option to show message after successful capture
- Upgraded dependencies for security patches
- Improved sharpness detection for checks
- Added sample SVG guide images
- Accessibility improvements

---

## 5.7.0

### Updates

- Upgraded dependencies for security patches

### Digital Fraud Defender

- New AI-based RTS encrypted payload for advanced injection attack detection (`SELFIE` images)
- **Note:** Requires front camera on mobile devices
- **Note:** Digital Fraud Defender is a paid feature. Please contact Customer Success for details.

---

## 5.6.4

### Updates

- Added support for Micro-frontend (MFE) architecture
- Upgraded dependencies for security patches

---

## 5.6.3

### Updates

- Introduced delay for user readiness to enable sharper image capture on SDK load

---

## 5.6.2

### Updates

- Support for iPhone 16, 16 Plus, 16 Pro, 16 Pro Max with iOS 18 high-end camera

---

## 5.6.1

### Updates

- Support for RTS in images marked with failed IQA

---

## 5.6.0

### Updates

- **Overexposure detection** - Detects overly bright/washed out images
- Updated accessibility for voice capture
- **Barcode scanning increased accuracy** - Adaptive algorithm for difficult barcodes
- iPhone 15 orientation issue resolved (vertical to horizontal)
- **Selfie distance tolerance** - Works with smaller face sizes at greater distances
- **Stream ended event handling** - Detects interrupted video streams
- Enhanced RTS security for Android devices

### Important

- `CAPTURE_FRAME` command is deprecated and will be removed in next major version (see [Deprecation Notices](#deprecation-notices)).

---

## 5.5.0

### Updates

- **ATP (Advanced Threshold Presets)** - Optional `glareLevel` and `blurLevel` settings
- Updated camera selection for Huawei devices (fixed black/white camera default)
- iPhone 15 back camera fixes
- iPhone 15 Pro/Pro Max back camera adjustment for iOS 17.x

---

## 5.4.0

### Updates

- **RTS (Real Time Security)** support. Please contact support for details.
- **NLD 3-line MRZ redaction** - BSN redaction for Dutch ID documents with QR codes

### Fixes

- Fixed WASM error when selfie and documents preloaded simultaneously
- Fixed duplicate file loads on slow 3G connections

---

## 5.3.0

### Updates

- **Device ID signal feature** - Optional device signals (`useDeviceIdSignal`)
- **Face IQA enhancements**
  - New selfie blur detection
  - Optimized selfie angle detection
  - Updated hint relevance
  - 90% compression default
- **New barcode library**
  - Text license format (eliminates PNG and CORS issues)
  - Single combined license for SDK + barcode
- **ClientActivityDataLog** - Official structure for device/session info
- Original image timestamp in EXIF metadata
- Updated error message for undersized images in direct process

---

## 5.2.3

### Updates

- iOS 16.4 camera blur fixes for multi-camera iPhones
- Resolution update with 1080p default, 720p fallback
- Canvas usage optimizations for Safari
- Chrome 2D context warning prevention

---

## 5.2.2

### Fixes

- Samsung A8/A9 resolution error during document capture
- Selfie preload hint speed
- Session recording timer during multiple captures

---

## 5.2.1

### Fixes

- **iPhone 14 Pro/Pro Max support** - Enables auto processing in iOS 16+ for new back camera hardware

---

## 5.2.0

### Updates

- **MRZ optional data redaction** - BSN redaction for Dutch passports
  - Enable with `redactMRZOptionalData: true`
  - Helps compliance with Dutch GDPR (UAVG section 46)

### Fixes

- Support for custom paths including URLs for all capture types

---

## 5.1.0

### Updates

- **Voice biometrics** - New `VOICE` capture type
  - New `CAPTURE_AND_PROCESS_VOICE` command
  - New error code 4009 for voice capture issues
- Support for selfie custom path during preload
- Adjustments for face detection at greater distances
- Chrome canvas warning addressed
- Updated hint logic for force update with longer display times

---

## 5.0.0

Complete rebuild with backwards compatibility.

### Updates

- **License required** - Valid license needed for SDK use
- **Component preload** - Faster UX with pre-downloaded dependencies
- **`QR_BARCODE`** - New document type for QR code capture
- **Passport MRZ** - Returns MRZ information in SDK result
- **No default guide images** - Reduces UI lag
- `disablePerpendicularCapture` - Match document/device orientation
- `disableMirroring` - Disable selfie preview mirroring
- **1080p default resolution** - Falls back to 720p if unsupported
- **50%+ CPU reduction** - Performance enhancements
- **Improved memory management** - Reduced Web Worker memory bloating
- **5-minute internal timeout** - Stops idle/abandoned SDK processes
- Command option validation with development warnings
- `SDK_REMOVE_LISTENERS` - Remove registered event listeners
- New `SHOW_HINT` format with duration override

---

## Testing

The MiSnap SDK is tested across all major browsers on a variety of Apple iOS, Android, and desktop devices.

---

## Deprecation Notices

### `CAPTURE_FRAME` (Deprecated)

**Status:** Will be removed in v6.0.0

**Issue:** Returns the last evaluated frame regardless of IQA status, which may not meet quality requirements.

**Alternatives:**

1. **Use `SDK_STOP`** - Stop the SDK and retrieve failed image from the result
2. **Fall back to manual mode** - Better image quality with native camera resolution

```javascript
// Instead of CAPTURE_FRAME, use SDK_STOP
mitekScienceSDK.cmd("SDK_STOP");
// Then check result.response.failedImage
```
