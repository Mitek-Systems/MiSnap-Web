# Accessibility Configuration

Options for improving the capture experience for users who rely on assistive technologies such as screen readers.

## Accessibility Mode

The `accessibilityMode` option applies a set of defaults optimized for assistive technology users:

```javascript
options: {
  license,
  accessibilityMode: true
}
```

### Applied Defaults

When `accessibilityMode: true`:

| Option | Value | Reason |
|--------|-------|--------|
| `hintFrequencyMS` | `5000` | More time for screen readers to announce hints |
| `hintAriaLive` | `2` (assertive) | Immediate announcement |
| `announceDuplicateHints` | `true` | Re-announce repeated hints |
| `disablePerpendicularCapture` | `true` | Simpler orientation handling |
| `disableSmileDetection` | `true` | Remove smile requirement for `SELFIE` |
| `disableSelfieCountdown` | `true` | No countdown timer for `SELFIE` |
| `disableCaptureDelay` | `true` | Faster capture |
| `faceDetectionLevel` | `1` (lenient) | Easier face detection for `SELFIE` |

### Overriding Defaults

Individual options can still be set explicitly:

```javascript
options: {
  license,
  accessibilityMode: true,
  hintFrequencyMS: 3000  // Override the 5000ms default
}
```

## Individual Options

### hintAriaLive

Controls how screen readers announce hint updates.

| Value | Mode | Behavior |
|-------|------|----------|
| `0` | Off | No announcements |
| `1` | Polite | Announce when idle |
| `2` | Assertive | Announce immediately |

```javascript
options: {
  license,
  hintAriaLive: 2  // Immediate announcements (default)
}
```

### hintFrequencyMS

How long each hint displays before the next one.

```javascript
options: {
  license,
  hintFrequencyMS: 3000  // 3 seconds per hint
}
```

**Recommendation:** Use 3000-5000ms for screen reader users.

### hintMessageSize

Font size for hint messages.

| Value | Size | CSS Font Size |
|-------|------|---------------|
| `1` | Small | `0.75rem` |
| `2` | Medium | `1rem` |
| `3` | Large | `1.5rem` |

```javascript
options: {
  license,
  hintMessageSize: 3  // Large hints
}
```

### announceDuplicateHints

By default, identical consecutive hints are not re-announced. Enable this to repeat them.

```javascript
options: {
  license,
  announceDuplicateHints: true  // Re-announce unchanged hints
}
```

**Use case:** Screen reader users may miss hints; repeating ensures they hear guidance.

## Selfie Accessibility

### Disable Smile Detection

Removes the requirement to detect a smile, which can be difficult for some users:

```javascript
options: {
  license,
  disableSmileDetection: true
}
```

### Disable Countdown

Removes the 3-2-1 visual countdown:

```javascript
options: {
  license,
  disableSelfieCountdown: true
}
```

### Disable Capture Delay

Removes the 1-second delay after finding a good frame:

```javascript
options: {
  license,
  disableCaptureDelay: true
}
```

**Benefit:** Reduces friction for visually impaired users.

### Lenient Face Detection

Uses the most forgiving face detection:

```javascript
options: {
  license,
  faceDetectionLevel: 1  // Lenient
}
```

## CSS Customization

Override hint font sizes in your CSS:

```css
/* Make hints larger for visibility */
div.mitek-hint-message.small {
  font-size: 1rem;
}

div.mitek-hint-message.medium {
  font-size: 1.5rem;
}

div.mitek-hint-message.large {
  font-size: 2rem;
}
```

**Tip:** Use `rem` units to respect browser font size settings.

## Capture Failure Handling

When accessibility mode is enabled and capture fails, announce the last hint:

```javascript
mitekScienceSDK.on("FRAME_CAPTURE_RESULT", (result) => {
  if (result.response.status === "failure") {
    const warning = result.response.warnings?.key;
    if (warning && isAccessibilityMode) {
      announceToScreenReader(`Capture failed: ${getHintText(warning)}`);
    }
  }
});

function announceToScreenReader(message) {
  const el = document.createElement("div");
  el.setAttribute("aria-live", "assertive");
  el.setAttribute("role", "alert");
  el.textContent = message;
  document.body.appendChild(el);
  setTimeout(() => el.remove(), 1000);
}
```

## Related

- [Capture Options](./capture-options.md) - All configuration options
- [SHOW_HINT](../commands/SHOW_HINT.md) - Display custom hints
- [FRAME_PROCESSING_FEEDBACK](../events/FRAME_PROCESSING_FEEDBACK.md) - Feedback keys
