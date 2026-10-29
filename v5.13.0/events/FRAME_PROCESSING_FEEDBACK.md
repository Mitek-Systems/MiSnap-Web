# FRAME_PROCESSING_FEEDBACK

Triggered continuously during auto capture with image quality feedback. Use this to display hints guiding the user to a successful capture.

## Signature

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  // result: { key: string, actionType: string, actionMode: string }
});
```

## TypeScript

Import `MitekFrameProcessingFeedback` from `mitek-science-sdk.js` (see [Installation — TypeScript](../INSTALLATION.md#typescript-both-options)). The `on("FRAME_PROCESSING_FEEDBACK", …)` callback is typed as `(feedback: MitekFrameProcessingFeedback) => void`.

## Callback Data

| Property | Type | Description |
|----------|------|-------------|
| `key` | `string` | Feedback key indicating the issue |
| `actionType` | `string` | Document type being captured |
| `actionMode` | `string` | Capture mode (e.g. `AUTO`) |

## Example

```javascript
const hints = {
  "MITEK_ERROR_GLARE": "Too much glare on document",
  "MITEK_ERROR_FOCUS": "Focusing",
  "MITEK_ERROR_FOUR_CORNER": "Document not found yet",
  "NO_FACE_FOUND": "No face detected"
};

mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  const hint = hints[result.key];
  if (hint) {
    mitekScienceSDK.cmd("SHOW_HINT", {
      options: { hintText: hint }
    });
  }
});
```

## Feedback Keys

### Document Feedback

| Key | Description | Suggested Hint |
|-----|-------------|----------------|
| `MITEK_ERROR_FOUR_CORNER` | Cannot find document corners | "Document not found yet" |
| `MITEK_ERROR_GLARE` | Too much light/reflection | "Too much glare on document" |
| `MITEK_ERROR_FOCUS` | Image is blurry | "Focusing" |
| `MITEK_ERROR_HORIZONTAL_FILL` | Document too far | "Too far away" |
| `MITEK_ERROR_MIN_PADDING` | Document too close | "Too close" |
| `MITEK_ERROR_LOW_CONTRAST` | Poor background contrast | "Background not dark enough" |
| `MITEK_ERROR_BUSY_BACKGROUND` | Background too busy | "Background too busy" |
| `MITEK_ERROR_SKEW_ANGLE` | Document not level | "Angle too large" |
| `MITEK_ERROR_TOO_DARK` | Underexposed image | "Not enough light" |
| `MITEK_ERROR_PERPENDICULAR_DOCUMENT` | Orientation mismatch | "Document not aligned" |

### Passport-Specific

| Key | Description | Suggested Hint |
|-----|-------------|----------------|
| `MITEK_ERROR_MRZ_MISSING` | Cannot read MRZ | "Bottom text should be visible" |

### Barcode Feedback

| Key | Description | Suggested Hint |
|-----|-------------|----------------|
| `CV_NO_BARCODE_FOUND` | Barcode not detected | "Barcode not found" |

### Selfie Feedback

| Key | Description | Suggested Hint |
|-----|-------------|----------------|
| `NO_FACE_FOUND` | No face detected | "No face detected" |
| `MISNAP_HEAD_OUTSIDE` | Face outside guide | "Slowly place face in oval and wait" |
| `MISNAP_HEAD_SKEWED` | Face not straight | "Look straight ahead" |
| `MISNAP_HEAD_TOO_CLOSE` | Face too close | "Too close" |
| `MISNAP_HEAD_TOO_FAR` | Face too far | "Too far away" |
| `MISNAP_STAY_STILL` | Movement detected | "Hold still in a well lit space" |
| `MISNAP_SMILE` | Need smile | "Hold a smile" |
| `MISNAP_STOP_SMILING` | Stop smiling | "Stop smiling" |
| `MISNAP_READY_POSE` | Ready to capture | "Hold still" |

### Image Validation

| Key | Description | Suggested Hint |
|-----|-------------|----------------|
| `CORRUPT_IMAGE` | Image file corrupt | "Image is unreadable" |
| `IMAGE_SMALLER_THAN_MIN_SIZE` | Under 640x480 | "Image too small" |

### Deprecated / Unused Hint Keys

The following hint keys are **not emitted** by the SDK. They have been removed from the API. Use the alternatives below if you have existing hint maps:

| Removed Key | Use Instead |
|-------------|-------------|
| `MISNAP_AXIS_ANGLE` | `MISNAP_HEAD_SKEWED` (face tilt/straighten) |
| `MISNAP_SUCCESS` | `customSuccessMessage` in capture options |
| `MITEK_ERROR_TOO_FAR` | `MITEK_ERROR_HORIZONTAL_FILL` (document too far) |
| `MITEK_ERROR_TOO_CLOSE` | `MITEK_ERROR_MIN_PADDING` (document too close) |
| `MITEK_ERROR_NOT_CENTERED` | *(no replacement; not emitted)* |
| `MITEK_ERROR_FOCUS` | As of 5.12.0, no longer emitted for selfie auto capture. |

>**Note**: Document capture still emits `MITEK_ERROR_FOCUS` when focus quality is poor.

## Example with Selfie Guide

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  const { key, actionType } = result;
  if (!key) return;
  
  // Selfie hints
  const hints = {
    "MISNAP_HEAD_SKEWED": "Look straight ahead",
    "MISNAP_HEAD_TOO_CLOSE": "Too close",
    "MISNAP_HEAD_TOO_FAR": "Too far away",
    "NO_FACE_FOUND": "No face detected",
    "MISNAP_HEAD_OUTSIDE": "Slowly place face in oval and wait",
    "MISNAP_STAY_STILL": "Hold still in a well lit space",
    "MISNAP_STOP_SMILING": "Stop smiling",
    "MISNAP_SMILE": "Hold a smile",
    "MISNAP_READY_POSE": "Hold still",
  };
  
  const hintText = hints[key];
  if (!hintText) return;
  
  // Force update for "hold still" hint
  const hintForceUpdate = key === "MISNAP_READY_POSE";
  
  // Update guide frame color for selfie
  if (actionType === "SELFIE") {
    const guide = document.getElementById("mitekGuide");
    const activeHints = ["MISNAP_SMILE", "MISNAP_STOP_SMILING", "MISNAP_READY_POSE"];
    
    if (activeHints.includes(key)) {
      guide?.classList.add("active");  // Face in position
    } else {
      guide?.classList.remove("active");
    }
  }
  
  mitekScienceSDK.cmd("SHOW_HINT", {
    options: { hintText, hintForceUpdate }
  });
});
```

## Controlling Hint Frequency

Use `hintFrequencyMS` to control how often hints update:

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  options: {
    license: "your-license",
    hintFrequencyMS: 3000  // 3 seconds between hints
  }
});
```

## Event Sequence

```
CAMERA_DISPLAY_STARTED
         ↓
FRAME_PROCESSING_STARTED
         ↓
FRAME_PROCESSING_FEEDBACK (continuous)    ← You are here
         ↓
FRAME_CAPTURE_RESULT
```

## Related

- [SHOW_HINT](../commands/SHOW_HINT.md) - Display hint messages
- [FRAME_CAPTURE_RESULT](./FRAME_CAPTURE_RESULT.md) - Capture result
- [Accessibility Options](../configuration/accessibility.md) - Screen reader settings
