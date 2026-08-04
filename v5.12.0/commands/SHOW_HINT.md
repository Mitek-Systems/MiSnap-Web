# SHOW_HINT

Displays a custom hint message during an auto capture session.

## Signature

```javascript
mitekScienceSDK.cmd("SHOW_HINT", {
  options: {
    hintText: string,
    hintDuration?: number,
    hintForceUpdate?: boolean
  }
});
```

## Parameters

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `options.hintText` | `string` | Yes | - | Message to display |
| `options.hintDuration` | `number` | No | `1200` | Display duration in milliseconds |
| `options.hintForceUpdate` | `boolean` | No | `false` | Show immediately, interrupting current hint |

> **Note:** If `hintFrequencyMS` is configured for `CAPTURE_AND_PROCESS_FRAME`, that value will be used as the default `hintDuration`.

## Examples

### Basic Hint

```javascript
mitekScienceSDK.cmd("SHOW_HINT", {
  options: {
    hintText: "Say cheese"
  }
});
```

### With Duration

```javascript
mitekScienceSDK.cmd("SHOW_HINT", {
  options: {
    hintText: "Center barcode",
    hintDuration: 3000  // Show for 3 seconds
  }
});
```

### Force Immediate Update

```javascript
mitekScienceSDK.cmd("SHOW_HINT", {
  options: {
    hintText: "Hold still!",
    hintForceUpdate: true  // Show immediately
  }
});
```

## Use Cases

### 1. First Hint on Camera Start

```javascript
mitekScienceSDK.on("CAMERA_DISPLAY_STARTED", () => {
  mitekScienceSDK.cmd("SHOW_HINT", {
    options: {
      hintText: "Position your ID in the frame",
      hintDuration: 2000
    }
  });
});
```

### 2. Custom Hints from Feedback

```javascript
const customHints = {
  "MITEK_ERROR_GLARE": "Too much glare on document",
  "MITEK_ERROR_FOCUS": "Focusing",
  "MITEK_ERROR_FOUR_CORNER": "Document not found yet",
  "MITEK_ERROR_TOO_DARK": "Not enough light"
};

mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  const hint = customHints[result.key];
  if (hint) {
    mitekScienceSDK.cmd("SHOW_HINT", {
      options: { hintText: hint }
    });
  }
});
```

### 3. Selfie Guidance

```javascript
mitekScienceSDK.on("FRAME_PROCESSING_FEEDBACK", (result) => {
  const { key } = result;
  
  // Force update for "hold still" to show immediately
  const forceUpdate = key === "MISNAP_READY_POSE";
  
  const hints = {
    "MISNAP_HEAD_TOO_CLOSE": "Too close",
    "MISNAP_HEAD_TOO_FAR": "Too far away",
    "MISNAP_SMILE": "Hold a smile",
    "MISNAP_READY_POSE": "Hold still"
  };
  
  if (hints[key]) {
    mitekScienceSDK.cmd("SHOW_HINT", {
      options: {
        hintText: hints[key],
        hintForceUpdate: forceUpdate
      }
    });
  }
});
```

## Hint Message Styling

Hints are displayed in an element with a class of `mitek-hint-message`. Customize with CSS:

```css
/* Size classes: small, medium, large */
div.mitek-hint-message.small {
  font-size: 0.75rem;
}

div.mitek-hint-message.medium {
  font-size: 1.5rem;
}

div.mitek-hint-message.large {
  font-size: 2rem;
}
```

> **Note:** The `mitek-hint-message` class is also applied to the `SELFIE` 3-2-1 countdown element.

## Related

- [HINT_MESSAGE_SIZE](./HINT_MESSAGE_SIZE.md) - Change hint font size
- [FRAME_PROCESSING_FEEDBACK](../events/FRAME_PROCESSING_FEEDBACK.md) - Feedback keys
- [Accessibility Options](../configuration/accessibility.md) - Hint accessibility
