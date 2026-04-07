# PROCESS_FRAME

Processes an existing image without using the camera. Use this for images already captured or uploaded.

## Signature

```javascript
mitekScienceSDK.cmd("PROCESS_FRAME", {
  mode: "DIRECT",
  documentType: string,
  mitekSDKPath?: string,
  options: {
    license: string,
    frame: string
  }
});
```

## Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `mode` | `string` | Yes | - | Must be `"DIRECT"` |
| `documentType` | `string` | Yes | - | Document type to process |
| `mitekSDKPath` | `string` | No | `"./mitekSDK/"` | Path to SDK files |
| `options.license` | `string` | Yes | - | SDK license key |
| `options.frame` | `string` | Yes | - | Image data (see formats below) |

## Image Formats

The `frame` option accepts:

| Format | Example |
|--------|---------|
| Data URL | `"data:image/jpeg;base64,/9j/4AAQ..."` |
| Blob URL | `"blob:https://example.com/..."` |
| File path | `"/path/to/image.jpg"` (relative or absolute) |

## Example

### From Data URL

```javascript
const imageData = "data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ...";

mitekScienceSDK.cmd("PROCESS_FRAME", {
  mode: "DIRECT",
  documentType: "DL_FRONT",
  mitekSDKPath: "./mitekSDK/",
  options: {
    license: "your-license-here",
    frame: imageData
  }
});
```

### From File Input

```javascript
const fileInput = document.getElementById("imageUpload");

fileInput.addEventListener("change", (e) => {
  const file = e.target.files[0];
  const reader = new FileReader();
  
  reader.addEventListener("load", (event) => {
    mitekScienceSDK.cmd("PROCESS_FRAME", {
      mode: "DIRECT",
      documentType: "PASSPORT",
      mitekSDKPath: "./mitekSDK/",
      options: {
        license: "your-license-here",
        frame: event.target.result
      }
    });
  });
  
  reader.readAsDataURL(file);
});
```

### From Blob URL

```javascript
// Assuming you have a blob from fetch or other source
const blob = await response.blob();
const blobUrl = URL.createObjectURL(blob);

mitekScienceSDK.cmd("PROCESS_FRAME", {
  mode: "DIRECT",
  documentType: "CHECK_FRONT",
  options: {
    license: "your-license-here",
    frame: blobUrl
  }
});

// Clean up after processing
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", () => {
  URL.revokeObjectURL(blobUrl);
});
```

## Events

| Event | When |
|-------|------|
| `FRAME_CAPTURE_RESULT` | Processing complete (success or failure) |
| `SDK_ERROR` | Error occurred (invalid image, etc.) |

## Minimum Image Requirements

| Requirement | Value |
|-------------|-------|
| Minimum dimensions | 640 x 480 pixels |
| Supported formats | JPEG, PNG |

## Error Handling

### Image Too Small

```javascript
mitekScienceSDK.on("SDK_ERROR", (err) => {
  if (err.code === 4001) {
    console.log("Image must be at least 640x480 pixels");
  }
});
```

### Corrupt Image

```javascript
mitekScienceSDK.on("SDK_ERROR", (err) => {
  if (err.code === 4002) {
    console.log("Image file is corrupt");
  }
});
```

## Result Object

Same structure as `CAPTURE_AND_PROCESS_FRAME`:

### Success

```javascript
{
  request: "DL_FRONT",
  response: {
    docType: "DL_FRONT",
    status: "success",
    imageData: "data:image/jpeg;base64,...",
    fourCornerA: { x: 382, y: 154 },
    fourCornerB: { x: 1148, y: 157 },
    fourCornerC: { x: 1148, y: 655 },
    fourCornerD: { x: 372, y: 655 },
    glareBottom: 0,
    glareLeft: 0,
    glareRight: 0,
    glareTop: 0,
    warnings: { status: "success" },
    mibiData: { ... }
  }
}
```

### Failure

```javascript
{
  request: "DL_FRONT",
  response: {
    docType: "DL_FRONT",
    status: "failure",
    failedImage: "data:image/jpeg;base64,...",
    warnings: { 
      status: "failure", 
      code: "NF", 
      key: "MITEK_ERROR_FOUR_CORNER" 
    }
  }
}
```

## Related

- [CAPTURE_AND_PROCESS_FRAME](./CAPTURE_AND_PROCESS_FRAME.md) - Camera capture
- [Document Types](../configuration/document-types.md) - All document types
- [FRAME_CAPTURE_RESULT](../events/FRAME_CAPTURE_RESULT.md) - Result event
