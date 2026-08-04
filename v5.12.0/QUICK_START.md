# Quick Start Guide

Get your first MiSnap capture working in under 5 minutes.

## Prerequisites

1. MiSnap Web SDK package files
2. Valid license key (contact your Mitek Account team)
3. Web server (HTTPS required for camera access, except localhost)

## Step 1: Extract Package Files

Extract the SDK package to your web server root:

```
/your-web-root/
├── mitek-science-sdk.js
├── mitekSDK/
│   ├── app.css
│   ├── license_verifier_bin.js
│   ├── *.bundle.js
│   ├── *.wasm
│   └── ...
└── your-app.html
```

## Step 2: Create Hello World Page

Create a new file `helloworld.html`:

```html
<!doctype html>
<head>
  <link rel="stylesheet" href="mitekSDK/app.css" />
</head>
<body>
  <h2>MiSnap Hello World</h2>
  <p>Check the browser console for SDK output.</p>
  
  <script>
    function loadSDKSetup() {
      console.log("MiSnap Web SDK version: ", mitekScienceSDK.getVersion());
      
      // Listen for SDK errors
      mitekScienceSDK.on("SDK_ERROR", (err) => {
        console.error("SDK Error:", err);
        if (err.code === 4006) {
          console.warn("Invalid license - check your license key");
        }
      });
      
      // Preload document capture components
      mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
        mitekSDKPath: "./mitekSDK/",
        preloadComponents: ["DOCUMENTS"],
        options: {
          license: "your-license-here"
        }
      });
    }
    
    window.addEventListener("load", loadSDKSetup);
  </script>
  <script src="mitek-science-sdk.js"></script>
</body>
</html>
```

## Step 3: Install Your License

Replace `"your-license-here"` with your actual license key:

```javascript
options: {
  license: "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."  // Your license
}
```

## Step 4: Verify Installation

1. Open `http://localhost:8080/helloworld.html` in your browser
2. Open the browser developer tools (F12)
3. Check the **Console** tab for SDK version output
4. Check the **Network** tab for loaded files:
   - `mitek-science-sdk.js`
   - `mitek-mobile-web.bundle.js`
   - `mitekMobileWeb.worker.js`
   - `mitekMobileWeb.wasm`

### Expected Console Output

```
MiSnap Web SDK version: 5.12.0
```

### If You See License Errors

```
Error: {code: 4006, type: "INVALID_LICENSE", status: "failure"}
```

This means your license is invalid or expired. Contact Mitek support.

## Step 5: Add Your First Capture

Extend the page to capture a driver's license with hint messages:

```html
<!doctype html>
<head>
  <link rel="stylesheet" href="mitekSDK/app.css" />
</head>
<body>
  <h2>MiSnap Document Capture</h2>
  <button id="captureBtn">Capture Driver's License</button>
  <div id="result"></div>
  
  <script>
    const license = "your-license-here";
    
    // Hint messages for user guidance during capture
    const captureHints = {
      MITEK_ERROR_GLARE: "Too much glare on document",
      MITEK_ERROR_FOUR_CORNER: "Document not found yet",
      MITEK_ERROR_TOO_DARK: "Not enough light",
      MITEK_ERROR_FOCUS: "Focusing",
      MITEK_ERROR_MRZ_MISSING: "Bottom text should be visible",
      MITEK_ERROR_MIN_PADDING: "Too close",
      MITEK_ERROR_HORIZONTAL_FILL: "Too far away",
      MITEK_ERROR_LOW_CONTRAST: "Background not dark enough",
      MITEK_ERROR_BUSY_BACKGROUND: "Background too busy",
      MITEK_ERROR_SKEW_ANGLE: "Angle too large",
      MITEK_ERROR_PERPENDICULAR_DOCUMENT: "Document not aligned"
    };
    
    function setupSDK() {
      // Handle errors
      mitekScienceSDK.on("SDK_ERROR", (err) => {
        console.error("Error:", err);
      });
      
      // Handle real-time feedback and show hints
      mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
        const { key } = result;
        if (!key) return;
        
        const hintText = captureHints[key];
        
        if (hintText) {
          mitekScienceSDK.cmd("SHOW_HINT", {
            options: { hintText }
          });
        }
      });
      
      // Handle capture result
      mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
        if (result.response.status === "success") {
          document.getElementById("result").innerHTML = 
            "<img src=\"" + result.response.imageData + "\" width=\"400\" />";
        } else {
          console.log("Capture failed:", result.response.warnings);
        }
      });
      
      // Preload components
      mitekScienceSDK.cmd("COMPONENT_PRELOAD", {
        mitekSDKPath: "./mitekSDK/",
        preloadComponents: ["DOCUMENTS"],
        options: { license }
      });
      
      // Button click handler
      document.getElementById("captureBtn").addEventListener("click", () => {
        mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
          mode: "AUTO_CAPTURE",
          documentType: "DL_FRONT",
          mitekSDKPath: "./mitekSDK/",
          options: { license }
        });
      });
    }
    
    window.addEventListener("load", setupSDK);
  </script>
  <script src="mitek-science-sdk.js"></script>
</body>
</html>
```

The `FRAME_PROCESSING_FEEDBACK` event fires continuously during capture with a `key` indicating the current issue. Use this to display helpful hints that guide users to a successful capture.

## Step 6: Test the Capture

1. Access the page via **HTTPS** (or `http://localhost` for development)
2. Click "Capture Driver's License"
3. Allow camera permission when prompted
4. Follow the on-screen hints to capture your document
5. The captured image appears in the result div

## Try the Included Demo Page

The SDK package includes a fully-featured demo page that showcases all capture types and configuration options.

### Running the Demo

1. **Serve the SDK package** from a local web server:

   ```bash
   # Using Node.js (npx)
   npx serve -p 8080
   ```

2. **Open the demo** at `http://localhost:8080/`

The demo page includes a license that works on localhost, so you can test all features immediately.

> **Note:** To deploy the demo on your own domain, replace the license in `index.html` with one configured for your domain.

### Demo Features

The demo page allows you to test:

- **Document types** - `DL_FRONT`, `PASSPORT`, `CHECK`, `BARCODE`, `SELFIE`, `VOICE`
- **Capture modes** - Auto, Manual, and Direct (image upload)
- **Configuration options** - Quality, hints, accessibility, video recording
- **Selfie options** - Smile detection, countdown timer, AI-based RTS
- **Results display** - Captured images and SDK response data

### Demo Files

```
/
├── index.html              # Main demo page
└── /demo
    ├── mitek-sdk-demo.js   # Demo application logic
    ├── mitek-sdk-demo.css  # Demo styles
    └── /images             # Sample test images
```

Use the demo as a reference implementation when building your own integration.

## Next Steps

- [Installation Guide](./INSTALLATION.md) - Full setup and configuration
- [Commands Reference](./commands/README.md) - All available SDK commands
- [Events Reference](./events/README.md) - Handling SDK events
- [Configuration Options](./configuration/README.md) - Customize capture behavior

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Camera not starting | Ensure HTTPS (or localhost) and camera permissions |
| License error (4006) | Verify license key is correct |
| WASM error | Check server serves `.wasm` as `application/wasm` |
| Blank screen | Link `mitekSDK/app.css` stylesheet |
