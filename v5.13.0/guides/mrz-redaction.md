# MRZ Redaction Guide

Redact sensitive data from passport and ID card MRZ sections for Dutch GDPR compliance.

## Overview

The MRZ (Machine Readable Zone) optional data section may contain the Dutch BSN (Citizen Service Number). Dutch data protection law (UAVG section 46) restricts BSN usage, so the SDK can redact this data automatically.

## Supported Documents

| Document | MRZ Lines | Redaction Support |
|----------|-----------|-------------------|
| Netherlands Passport | 2-line MRZ | v5.2.0+ |
| Netherlands ID Card | 3-line MRZ | v5.4.0+ |

## Enabling Redaction

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "PASSPORT",  // or "DL_FRONT" for ID cards
  options: {
    license: "your-license-here",
    redactMRZOptionalData: true
  }
});
```

## How It Works

When `redactMRZOptionalData: true`:

1. SDK captures the document
2. Detects if document is Dutch (NLD country code)
3. Checks if optional data section contains BSN
4. If present, blacks out the BSN region in the output image
5. Returns redacted image in `imageData`

## Result Verification

Check if redaction was applied:

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    // Check ClientActivityDataLog for redaction status
    const sessionInfo = result.response.ClientActivityDataLog?.SessionInfo?.["0"];
    const wasRedacted = sessionInfo?.OptionalDataRedacted;
    
    console.log("BSN redacted:", wasRedacted);
  }
});
```

## Netherlands Passport Flow

For Dutch passports, BSN may be in MRZ or QR code:

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.docType !== "PASSPORT" || 
      result.response.status !== "success") {
    return;
  }
  
  const { countryCode, hasOptionalData } = result.response.mrz || {};
  
  if (countryCode === "NLD") {
    if (hasOptionalData) {
      // BSN in MRZ - redaction applied if enabled
      console.log("Dutch passport with MRZ BSN");
    } else {
      // BSN in QR code - need to capture QR
      console.log("Dutch passport - scan QR for BSN");
      startQRCapture();
    }
  } else {
    // Non-Dutch passport - no BSN handling needed
    console.log("Non-Dutch passport");
  }
});
```

## Decision Table

| Country Code | hasOptionalData | Action |
|--------------|-----------------|--------|
| `NLD` | `true` | BSN in MRZ - redaction applied |
| `NLD` | `false` | BSN in QR code - scan `QR_BARCODE` |
| Other | any | No BSN handling needed |

## QR Code Capture (Netherlands 2021+)

For newer Dutch passports with QR-encoded BSN:

```javascript
function startQRCapture() {
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
    mode: "AUTO_CAPTURE",
    documentType: "QR_BARCODE",
    options: { license }
  });
}

mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.docType === "QR_BARCODE" && 
      result.response.status === "success") {
    // QR data in result.response.code
    const qrData = result.response.code;
    // Process QR data as needed
  }
});
```

## Important Notes

### Still Images Only

Redaction is applied to **still images only**, not the live camera stream or video recording:

```javascript
// Redaction will NOT apply to recorded video
options: {
  videoRecordingEnabled: true,  // Video NOT redacted
  redactMRZOptionalData: true   // Only still image redacted
}
```

### Non-Dutch Documents

For non-Dutch documents, redaction is not applied even if enabled:

```javascript
// Setting enabled, but only applies to NLD documents
options: {
  redactMRZOptionalData: true
}

// US passport  -> no redaction applied
// NLD passport -> redaction applied
```

### Passport Failure

If passport capture fails, no `mrz` property is returned:

```javascript
// Failure result - no mrz property
{
  response: {
    docType: "PASSPORT",
    status: "failure",
    failedImage: "data:image/jpeg;base64,...",
    warnings: { ... }
    // No mrz property
  }
}
```

## Complete Example

```javascript
const license = "your-license-here";
let capturedImages = [];

mitekScienceSDK.on("SDK_ERROR", (err) => {
  console.error("Error:", err);
});

mitekScienceSDK.on("FRAME_CAPTURE_RESULT", async (result) => {
  if (result.response.status !== "success") {
    showRetry();
    return;
  }
  
  const docType = result.response.docType;
  
  if (docType === "PASSPORT") {
    capturedImages.push({
      type: "passport",
      image: result.response.imageData
    });
    
    // Check for Netherlands
    const { countryCode, hasOptionalData } = result.response.mrz || {};
    
    if (countryCode === "NLD" && !hasOptionalData) {
      // Need QR code
      showMessage("Please scan the QR code");
      startQRCapture();
    } else {
      // Done with passport
      proceedToSelfie();
    }
  } else if (docType === "QR_BARCODE") {
    capturedImages.push({
      type: "qr",
      data: result.response.code
    });
    
    proceedToSelfie();
  }
});

function startPassportCapture() {
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
    mode: "AUTO_CAPTURE",
    documentType: "PASSPORT",
    options: {
      license,
      redactMRZOptionalData: true  // Enable BSN redaction
    }
  });
}

function startQRCapture() {
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
    mode: "AUTO_CAPTURE",
    documentType: "QR_BARCODE",
    options: { license }
  });
}
```

## Related

- [Document Types](../configuration/document-types.md) - `PASSPORT`, `QR_BARCODE`
- [Barcode Scanning](./barcode-scanning.md) - `QR_BARCODE` capture
- [FRAME_CAPTURE_RESULT](../events/FRAME_CAPTURE_RESULT.md) - Result structure
