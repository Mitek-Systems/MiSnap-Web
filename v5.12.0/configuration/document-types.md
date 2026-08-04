# Document Types

The `documentType` parameter specifies what type of document or biometric data to capture.

## Available Document Types

### Identity Documents

| Type | Description | Use Case |
|------|-------------|----------|
| `DL_FRONT` | Driver's license or ID card | US/international driver's licenses, ID cards |
| `PASSPORT` | Passport photo/MRZ page | International passports (folio type) |

### Checks

| Type | Description | Use Case |
|------|-------------|----------|
| `CHECK_FRONT` | Front of check | Check deposit |
| `CHECK_BACK` | Back of check | Endorsement capture |

### Barcodes

| Type | Description | Use Case |
|------|-------------|----------|
| `PDF417_BARCODE` | 2D barcode on DL/ID back | North American DL/ID data extraction |
| `QR_BARCODE` | QR code | Netherlands passport BSN, general QR |

### Generic Documents

| Type | Description | Use Case |
|------|-------------|----------|
| `DOCUMENT` | Generic/trailing document | Utility bills, bank statements, any rectangular document |

### Biometrics

| Type | Description | Use Case |
|------|-------------|----------|
| `SELFIE` | Self-portrait | Face biometrics, liveness |
| `VOICE` | Voice recording | Voice biometrics (via [`CAPTURE_AND_PROCESS_VOICE`](../commands/CAPTURE_AND_PROCESS_VOICE.md)) |

## Document Type Details

### `DL_FRONT`

Captures the front of driver's licenses, identity cards, or any document with:
- Approximately 1.59:1 aspect ratio
- Rounded corners

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  options: { license }
});
```

**Options:**
- `mrzRequired: true` - Require MRZ to be present if document has one

### `PASSPORT`

Captures the photo and MRZ page of international passports.

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "PASSPORT",
  options: {
    license,
    redactMRZOptionalData: true  // Redact Dutch BSN
  }
});
```

**Result includes:**
- `mrz.countryCode` - 3-letter ISO country code
- `mrz.hasOptionalData` - Whether optional data exists

**See:** [MRZ Redaction Guide](../guides/mrz-redaction.md)

### `PDF417_BARCODE`

Captures the 2D barcode on the back of North American driver's licenses and ID cards.

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "PDF417_BARCODE",
  options: { license }
});
```

**Result includes:**
- `response.code` - Extracted barcode data

**Note:** Requires valid barcode license. Invalid license produces asterisk-filled data.

### `QR_BARCODE`

Captures QR codes. Used for Netherlands passport BSN extraction.

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "QR_BARCODE",
  options: { license }
});
```

**Result includes:**
- `response.code` - Extracted QR data

### `CHECK_FRONT` / `CHECK_BACK`

Captures check images for mobile deposit.

```javascript
// Front
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "CHECK_FRONT",
  options: { license }
});

// Back
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "CHECK_BACK",
  options: { license }
});
```

### `DOCUMENT`

Generic document capture for any rectangular document.

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DOCUMENT",
  options: { license }
});
```

### `SELFIE`

Captures a self-portrait for face biometrics.

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "SELFIE",
  options: {
    license,
    disableSmileDetection: false,  // Require smile
    // Enable AI-based RTS (*)
    aiBasedRtsEnabled: true
  }
});
```

&#42; Additional AI-based RTS settings: [Capture options — AI-Based RTS](./capture-options.md#ai-based-rts-real-time-security).

**Capture conditions:**
- Smile detection (default)
- Countdown timer (`disableSmileDetection: true`)
- Good IQA (`disableSmileDetection: true, disableSelfieCountdown: true`)

**See:** [Selfie Capture Guide](../guides/selfie-capture.md)

### `VOICE`

Captures voice for biometrics. Uses the [`CAPTURE_AND_PROCESS_VOICE`](../commands/CAPTURE_AND_PROCESS_VOICE.md) command:

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_VOICE", {
  options: {
    license,
    phrase: "Time flies when you're having fun."
  }
});
```

**See:** [CAPTURE_AND_PROCESS_VOICE](../commands/CAPTURE_AND_PROCESS_VOICE.md)

## Preload Components by Document Type

| Document Type | Preload Component |
|---------------|-------------------|
| `DL_FRONT`, `PASSPORT`, `CHECK_*`, `DOCUMENT` | `DOCUMENTS` |
| `PDF417_BARCODE`, `QR_BARCODE` | `BARCODE` |
| `SELFIE` | `SELFIE` |
| `SELFIE` with AI-based RTS | `SELFIE`, `AI_BASED_RTS` |

```javascript
// Preload components for ID and Selfie
mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
  preloadComponents: ["DOCUMENTS", "SELFIE"],
  options: { license }
});
```

## Related

- [CAPTURE_AND_PROCESS_FRAME](../commands/CAPTURE_AND_PROCESS_FRAME.md) - Capture command
- [Capture Options](./capture-options.md) - Configuration options
- [COMPONENT_PRELOAD](../commands/COMPONENT_PRELOAD.md) - Preload components
