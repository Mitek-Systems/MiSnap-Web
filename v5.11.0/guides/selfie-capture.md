# Selfie Capture Guide

Best practices for implementing selfie capture with the MiSnap SDK.

## Overview

Selfie capture uses the front-facing camera to capture a face image for biometric verification. The SDK provides real-time face detection with guided feedback.

## Basic Implementation

```javascript
// Preload selfie components
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  preloadComponents: ["SELFIE"],
  options: { license }
});

// Start selfie capture
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "SELFIE",
  mitekSDKPath: "./mitekSDK/",
  options: { license }
});
```

## Capture Condition Strategies

The SDK supports three capture strategies:

### 1. Smile Detection (Default)

Requires the user to smile, then captures a neutral expression.

```javascript
options: {
  license,
  disableSmileDetection: false  // Default
}
```

### 2. Countdown Timer

Shows a 3-2-1 countdown before capture.

```javascript
options: {
  license,
  disableSmileDetection: true,
  disableSelfieCountdown: false
}
```

### 3. Good IQA (Fastest)

Captures immediately when quality requirements are met, with optional 1-second delay.

```javascript
options: {
  license,
  disableSmileDetection: true,
  disableSelfieCountdown: true,
  disableCaptureDelay: false  // 1-second delay before capture
}
```

**Without delay (fastest):**
```javascript
options: {
  license,
  disableSmileDetection: true,
  disableSelfieCountdown: true,
  disableCaptureDelay: true  // Capture immediately
}
```

## Face Detection Level

Control how strictly the SDK evaluates face positioning:

```javascript
options: {
  license,
  faceDetectionLevel: 1  // 1=lenient, 2=moderate, 3=strict
}
```

## Back camera selfie

By default, selfie capture uses the front-facing (user) camera. You can optionally use the back (environment) camera instead with the `backCameraSelfie` option:

```javascript
options: {
  license,
  backCameraSelfie: true  // Use back camera for selfie (default: false)
}
```

**Behavior:**

- **Camera selection:** When `backCameraSelfie` is `true`, the SDK prefers the same back camera used for document capture (e.g. main/rear camera on phones). On devices with only one camera or no back camera, the SDK automatically falls back to the front or only available camera.
- **AI-based RTS:** If `aiBasedRtsEnabled` is `true`, the front camera is always used; `backCameraSelfie` is ignored. AI-based RTS requires the front camera.
- **Preview mirroring:** The video preview is mirrored when using the front camera (default selfie) and not mirrored when using the back camera. 

## Guide Frame Customization

Update the guide frame color based on hint feedback:

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  const { key, actionType } = result;
  
  if (actionType !== "SELFIE") return;
  
  const guide = document.getElementById("mitekGuide");
  const activeHints = ["MISNAP_SMILE", "MISNAP_STOP_SMILING", "MISNAP_READY_POSE"];
  const isReady = activeHints.includes(key);
  guide?.classList.toggle("active", isReady);
});
```

The CSS for the active state is in `app.css`. You can override this, if needed:
```css
#mitekGuide.active {
  border-color: #58ce54;  /* Green when ready */
}
```

## Hint Messages

Selfie-specific feedback keys and suggested messages. For deprecated/unused keys (e.g. `MISNAP_AXIS_ANGLE`), see [FRAME_PROCESSING_FEEDBACK — Deprecated / Unused Hint Keys](../events/FRAME_PROCESSING_FEEDBACK.md#deprecated--unused-hint-keys).

| Key | Suggested Message |
|-----|-------------------|
| `NO_FACE_FOUND` | "No face detected" |
| `MISNAP_HEAD_OUTSIDE` | "Slowly place face in oval and wait" |
| `MISNAP_HEAD_SKEWED` | "Look straight ahead" |
| `MISNAP_HEAD_TOO_CLOSE` | "Too close" |
| `MISNAP_HEAD_TOO_FAR` | "Too far away" |
| `MISNAP_STAY_STILL` | "Hold still in a well lit space" |
| `MISNAP_SMILE` | "Hold a smile" |
| `MISNAP_STOP_SMILING` | "Stop smiling" |
| `MISNAP_READY_POSE` | "Hold still" |
| `MITEK_ERROR_FOCUS` | "Focusing" |

## AI-Based RTS (Real Time Security)

Enable enhanced injection attack detection:

```javascript
// Preload AI-Based RTS components
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  preloadComponents: ["SELFIE", "AI_BASED_RTS"],
  options: { license }
});

// Enable AI-Based RTS
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "SELFIE",
  options: {
    license,
    aiBasedRtsEnabled: true
  }
});
```

### AI-based RTS Requirements

- `SELFIE` document type only
- `AUTO_CAPTURE` mode only
- Front camera on mobile devices
- Minimum browser versions:
  - Chrome desktop: 103+
  - Chrome Android: 112+
  - Firefox desktop: 103+
  - Firefox Android: 110+
  - Safari desktop: 15.1+
  - Safari iOS: 15.0+
  - Edge: 103+
  - Samsung Internet: 11.1+

### Development Note

Please note that the usage of browser dev tools will not work properly with `aiBasedRtsEnabled`. It will trigger debugger breakpoints and result in an `AI_BASED_RTS_ERROR`. This is an intentional feature to protect against injection attacks in the frontend.

If dev tools are needed, we recommend temporarily disabling `aiBasedRtsEnabled`. Once dev tools are closed, it can be re-enabled.

## Accessibility

For users with accessibility needs:

```javascript
options: {
  license,
  accessibilityMode: true  // Applies sensible defaults
}
```

Or configure individually:

```javascript
options: {
  license,
  disableSmileDetection: true,
  disableSelfieCountdown: true,
  disableCaptureDelay: true,
  faceDetectionLevel: 1,
  hintFrequencyMS: 5000,
  hintAriaLive: 2
}
```

## Complete Example

```javascript
const license = "your-license-here";

// Error handling
mitekScienceSDK.on("SDK_ERROR", (err) => {
  console.error("Selfie error:", err);
  showError(err.type);
});

// Hint display with guide frame update
mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  const { key, actionType } = result;
  if (actionType !== "SELFIE" || !key) return;
  
  const hints = {
    "NO_FACE_FOUND": "No face detected",
    "MISNAP_HEAD_OUTSIDE": "Slowly place face in oval and wait",
    "MISNAP_HEAD_SKEWED": "Look straight ahead",
    "MISNAP_HEAD_TOO_CLOSE": "Too close",
    "MISNAP_HEAD_TOO_FAR": "Too far away",
    "MISNAP_STAY_STILL": "Hold still in a well lit space",
    "MISNAP_SMILE": "Hold a smile",
    "MISNAP_STOP_SMILING": "Stop smiling",
    "MISNAP_READY_POSE": "Hold still",
    "MITEK_ERROR_FOCUS": "Focusing",
  };

  const hintText = hints[key];
  if (!hintText) return;
  
  // Force hint to "Hold still"
  const hintForceUpdate = key === "MISNAP_READY_POSE";
  
  // Update guide frame
  const guide = document.getElementById("mitekGuide");
  const activeHints = ["MISNAP_SMILE", "MISNAP_STOP_SMILING", "MISNAP_READY_POSE"];
  const isReady = activeHints.includes(key);
  guide?.classList.toggle("active", isReady);
  
  // Show hint
  mitekScienceSDK.cmd("SHOW_HINT", {
    options: {
      hintText,
      hintForceUpdate,
    }
  });
});

// Handle result
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    showPreview(result.response.imageData);
    submitSelfie(result.response.imageData);
  } else {
    showRetryButton();
  }
});

// Preload
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  preloadComponents: ["SELFIE"],
  options: { license }
});

// Start capture with 3-2-1 countdown
function startSelfieCapture() {
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
    mode: "AUTO_CAPTURE",
    documentType: "SELFIE",
    options: {
      license,
      disableSmileDetection: true,
      faceDetectionLevel: 1,
      qualityPercent: 90
    }
  });
}
```

## Related

- [CAPTURE_AND_PROCESS_FRAME](../commands/CAPTURE_AND_PROCESS_FRAME.md) - Capture command
- [Capture Options](../configuration/capture-options.md) - All options
- [Accessibility](../configuration/accessibility.md) - Accessibility options
