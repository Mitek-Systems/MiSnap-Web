# HINT_MESSAGE_SIZE

Dynamically changes the font size of hint messages during an active auto capture session.

## Signature

```javascript
mitekScienceSDK.cmd("HINT_MESSAGE_SIZE", size);
```

## Parameters

| Parameter | Type | Required | Values | Description |
|-----------|------|----------|--------|-------------|
| `size` | `number` | Yes | `1`, `2`, `3` | Font size level |

## Size Values

| Value | Size | CSS Class | Default Font Size |
|-------|------|-----------|-------------------|
| `1` | Small | `.small` | `0.75rem` |
| `2` | Medium | `.medium` | `1rem` |
| `3` | Large | `.large` | `1.5rem` |

## Example

```javascript
// Change to large hints
mitekScienceSDK.cmd("HINT_MESSAGE_SIZE", 3);

// Change to small hints
mitekScienceSDK.cmd("HINT_MESSAGE_SIZE", 1);
```

## Use Cases

### Responsive Adjustment

```javascript
// Larger hints on mobile
if (window.innerWidth < 768) {
  mitekScienceSDK.cmd("HINT_MESSAGE_SIZE", 3);
} else {
  mitekScienceSDK.cmd("HINT_MESSAGE_SIZE", 2);
}
```

### Accessibility Toggle

```javascript
document.getElementById("largeText").addEventListener("change", (e) => {
  const size = e.target.checked ? 3 : 2;
  mitekScienceSDK.cmd("HINT_MESSAGE_SIZE", size);
});
```

## Setting Initial Size

To set hint size before capture starts, use the `hintMessageSize` option:

```javascript
mitekScienceSDK.cmd("CAPTURE_AND_PROCESS_FRAME", {
  mode: "AUTO_CAPTURE",
  documentType: "DL_FRONT",
  options: {
    license: "your-license-here",
    hintMessageSize: 3  // Start with large hints
  }
});
```

## Custom CSS

Override default sizes in your stylesheet:

```css
div.mitek-hint-message.small {
  font-size: 1rem;    /* Override default 0.75rem */
}

div.mitek-hint-message.medium {
  font-size: 1.5rem;  /* Override default 1rem */
}

div.mitek-hint-message.large {
  font-size: 2rem;    /* Override default 1.5rem */
}
```

> **Tip:** Use `rem` units for accessibility - they scale with browser font settings.

## Related

- [SHOW_HINT](./SHOW_HINT.md) - Display hint messages
- [Accessibility Options](../configuration/accessibility.md) - Full accessibility configuration
