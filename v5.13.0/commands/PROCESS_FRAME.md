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
    frame: string | File | ImageData
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
| `options.frame` | `string \| File \| ImageData` | Yes | - | Image to process (see formats below) |

## Image Formats

| Format | Example |
|--------|---------|
| Data URL | `"data:image/jpeg;base64,/9j/4AAQ..."` |
| `File` object | From `<input type="file">` — pass `files[0]` directly |
| `ImageData` | From a `<canvas>` context |

## Examples

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

Pass the selected `File` directly:

```javascript
const fileInput = document.getElementById("imageUpload");

fileInput.addEventListener("change", (e) => {
  const file = e.target.files[0];

  mitekScienceSDK.cmd("PROCESS_FRAME", {
    mode: "DIRECT",
    documentType: "PASSPORT",
    mitekSDKPath: "./mitekSDK/",
    options: {
      license: "your-license-here",
      frame: file
    }
  });
});
```

### From ImageData

Draw an image to a canvas and pass `getImageData()` result:

```javascript
const img = new Image();
img.addEventListener("load", () => {
  const canvas = document.createElement("canvas");
  canvas.width = img.width;
  canvas.height = img.height;
  const context = canvas.getContext("2d");
  context.drawImage(img, 0, 0);

  mitekScienceSDK.cmd("PROCESS_FRAME", {
    mode: "DIRECT",
    documentType: "DL_FRONT",
    mitekSDKPath: "./mitekSDK/",
    options: {
      license: "your-license-here",
      frame: context.getImageData(0, 0, canvas.width, canvas.height)
    }
  });
});
img.src = "images/sample-license.jpg";
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
