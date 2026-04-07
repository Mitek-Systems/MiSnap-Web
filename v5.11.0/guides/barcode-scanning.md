# Barcode Scanning Guide

Capture and decode PDF417 and QR barcodes with the MiSnap SDK.

## Overview

The SDK supports two barcode types:
- **PDF417** - 2D barcodes on North American driver's licenses and ID cards
- **QR Code** - Matrix barcodes, including Netherlands passport BSN codes and VDS/MiDNI (e.g. Spanish digital ID)

## Preloading

Preload barcode components before capture:

```javascript
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  preloadComponents: ["BARCODE"],
  options: { license }
});
```

## PDF417 Barcode (Driver's License Back)

### Basic Capture

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "PDF417_BARCODE",
  mitekSDKPath: "./mitekSDK/",
  options: { license }
});
```

### Result

The barcode data is in `response.code`:

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    const barcodeData = result.response.code;
    console.log("Barcode data:", barcodeData);
    // Parse AAMVA data
  }
});
```

### Licensing

PDF417 scanning requires a valid barcode license. If missing or expired:
- Capture still works
- Extracted data contains asterisks (`***`)

## QR Code

### Basic Capture

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "QR_BARCODE",
  mitekSDKPath: "./mitekSDK/",
  options: { license }
});
```

### Netherlands Passport BSN

For Dutch passports (2021+), the BSN may be in a QR code instead of the MRZ.

**Logic flow:**

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.docType === "PASSPORT" && 
      result.response.status === "success") {
    
    const { countryCode, hasOptionalData } = result.response.mrz || {};
    
    if (countryCode === "NLD" && !hasOptionalData) {
      // BSN not in MRZ - scan QR code
      startQRCapture();
    } else {
      // BSN in MRZ or non-Dutch passport
      proceedToNextStep();
    }
  }
});

function startQRCapture() {
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
    mode: "AUTO_CAPTURE",
    documentType: "QR_BARCODE",
    options: { license }
  });
}
```

## VDS and MiDNI

### What are VDS and MiDNI?

**VDS (Visible Digital Seal)** is an ICAO standard for machine-readable identity data in 2D barcodes. Barcodes that follow [ICAO 9303 Part 13](https://www2023.icao.int/publications/Documents/9303_p13_cons_en.pdf) start with a defined header; the SDK parses this header so you can detect and handle VDS barcodes (e.g. for digital identity schemes).

**MiDNI** (Ministerio del Interior – DNI en el móvil) is Spain’s national digital ID on a mobile phone. Spanish citizens can use a QR code that follows the VDS format. The QR format is described in [MiDNI Formato QR](https://www.midni.gob.es/assets/pdf/MiDNI-FormatoQR_v107_sc_PN.pdf).

When a user scans a QR code, the SDK checks for a valid VDS header and exposes the result in the `vds` property of the barcode response.

### The `vds` property on barcode results

Every successful barcode capture includes `response.vds`:

- **Non-VDS barcodes** (most QR/PDF417 codes):
  ```json
  { "isVds": false }
  ```

- **Valid VDS barcodes** (e.g. MiDNI):
  ```json
  {
    "isVds": true,
    "header": {
      "issuingCountry": "ES",
      "featureDefinitionReference": 7,
      "typeCategory": 9
    },
    "payload": "<encrypted payload>"
  }
  ```

  - `issuingCountry`: 3-letter code (can be less than 3 letters, e.g. `"ES"` for Spain).
  - `featureDefinitionReference`: document feature. For MiDNI: `7` = Simple, `8` = Complete, `9` = Age.
  - `typeCategory`: document type. For MiDNI this is `9` (DNI en el móvil).
  - `payload`: an encrypted payload that can be used for analysis in Mitek's backend.

### Example: scanning a QR code and handling VDS

Use the same QR capture flow as above; the result now includes `vds`:

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status !== "success") return;

  const { code, vds } = result.response;

  if (vds?.isVds) {
    // VDS barcode
    const country = vds.header?.issuingCountry;
    const featureRef = vds.header?.featureDefinitionReference;
    const typeCategory = vds.header?.typeCategory;

    if (country === "ES" && typeCategory === 9) {
      // MiDNI QR: Add any additional processing
      // For example: validate that the `featureRef` matches expected
      validateFeatureRef(featureRef);
      processVdsData(vds);
    }
  } else {
    // Non-VDS barcode: use code as before
    processBarcodeData(code);
  }
});
```

Scanning is unchanged: use `documentType: "QR_BARCODE"` (or `"PDF417_BARCODE"`) as in the [QR Code](#qr-code) and [PDF417](#pdf417-barcode-drivers-license-back) sections. Only the `response.vds` property is new.

## Hint Messages

Barcode-specific feedback:

| Key | Suggested Message |
|-----|-------------------|
| `CV_NO_BARCODE_FOUND` | "Barcode not found" |

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  if (result.key === "CV_NO_BARCODE_FOUND") {
    mitekScienceSDK.cmd("SHOW_HINT", {
      options: { hintText: "Barcode not found" }
    });
  }
});
```

## Complete Example

```javascript
const license = "your-license-here";

// Error handling
mitekScienceSDK.on("SDK_ERROR", (err) => {
  handleError(err);
});

// Feedback
mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  if (result.key === "CV_NO_BARCODE_FOUND") {
    mitekScienceSDK.cmd("SHOW_HINT", {
      options: { hintText: "Barcode not found" }
    });
  }
});

// Result
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "success") {
    const barcodeData = result.response.code;
    
    // Process barcode data
    processBarcodeData(barcodeData);
  }
});

// Preload
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  mitekSDKPath: "./mitekSDK/",
  preloadComponents: ["BARCODE"],
  options: { license }
});

// Capture PDF417
function scanLicenseBack() {
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
    mode: "AUTO_CAPTURE",
    documentType: "PDF417_BARCODE",
    options: { license }
  });
}

// Capture QR code
function scanQRCode() {
  mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
    mode: "AUTO_CAPTURE",
    documentType: "QR_BARCODE",
    options: { license }
  });
}
```

## Related

- [Document Types](../configuration/document-types.md) - All document types
- [MRZ Redaction](./mrz-redaction.md) - Netherlands passport handling
- [Installation](../INSTALLATION.md) - Barcode license setup
