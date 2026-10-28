# Expo Clipboard (`expo-clipboard`) — Clean Complete Notes

> **Purpose:** Easy-to-revise notes with the important concepts, methods, options, props, examples, and expected outputs.
>
> These notes are based on the supplied material, which targets **Expo SDK 54**. Clipboard APIs and platform support can change between SDK versions. Check the documentation for the SDK installed in your project before using less-common APIs.

## 1. What is a Clipboard?

The **clipboard** is temporary, system-level storage used for copying and pasting content.

```text
Select content → Copy → System Clipboard → Paste → Same app or another app
```

Common examples:
- OTPs and verification codes
- Phone numbers and email addresses
- URLs and referral links
- Text, codes, and supported image content

**Important:** The clipboard is not limited to your app. Content copied by one app may be pasted into another. Avoid copying sensitive values without a clear user action.

## 2. Install and import

Install the version compatible with your Expo SDK:

```bash
npx expo install expo-clipboard
```

Import the API:

```js
import * as Clipboard from "expo-clipboard";
```

Most methods are asynchronous, so use `await` inside an `async` function.

---

## 3. Main API methods at a glance

| Method | Purpose | Output described in the supplied SDK 54 notes |
|---|---|---|
| `setStringAsync(text, options?)` | Copy text | Promise; resolves to `true` on Android/iOS; on Web indicates whether saving succeeded |
| `getStringAsync(options?)` | Read clipboard text | A string |
| `hasStringAsync()` | Check for string content | `true` or `false`; can also be true for rich text/HTML |
| `setUrlAsync(url)` | Set clipboard content as a URL | Promise; iOS-specific in the supplied notes |
| `getUrlAsync()` | Read URL content | URL string or `null`; iOS-specific in the supplied notes |
| `hasUrlAsync()` | Check URL content | `true` or `false`; iOS-specific in the supplied notes |
| `hasImageAsync()` | Check for clipboard image | `true` or `false` |
| `getImageAsync(options)` | Read clipboard image | Image object or `null` |
| `setImageAsync(base64Image)` | Write an image to clipboard | Mentioned as deprecated in the supplied SDK 54 notes; avoid treating it as the preferred approach |
| `addClipboardListener(callback)` | Listen for clipboard changes | An event subscription |
| `removeClipboardListener(subscription)` | Remove a listener | Removes the supplied subscription; the returned subscription's `.remove()` is also shown in the supplied notes |

### Quick distinction

- **Set** = write/copy content.
- **Get** = read content.
- **Has** = check whether a supported content type exists.
- **Listener** = react to clipboard changes.
- **ContentType** = identify the kind of content.
- **StringFormat** = choose plain text or HTML representation.

---

## 4. `setStringAsync()` — copy text

### Syntax

```js
await Clipboard.setStringAsync(text);
```

- `text`: the string to copy.
- Optional `options`: the supplied notes describe an `inputFormat` option for choosing a string format.

### Example

```js
await Clipboard.setStringAsync("Hello React Native");
```

**Clipboard content after the call:**

```text
Hello React Native
```

You can copy a phone number, email, URL, OTP, or multiple lines in the same way:

```js
await Clipboard.setStringAsync("+91 9876543210");
await Clipboard.setStringAsync("rahul@example.com");
await Clipboard.setStringAsync("https://example.com");
await Clipboard.setStringAsync("482931");
```

Each call replaces the previous clipboard content.

### Validate before copying

```js
const copyText = async (text) => {
  if (typeof text !== "string" || !text.trim()) {
    console.log("Nothing to copy");
    return;
  }

  try {
    const result = await Clipboard.setStringAsync(text);
    console.log("Copy result:", result);
  } catch (error) {
    console.error("Copy failed:", error);
  }
};
```

**Expected output:** on Android/iOS according to the supplied notes, `Copy result: true`. On Web, the result indicates whether the string was saved successfully.

### `setStringAsync()` options

The supplied notes show this format-aware example:

```js
await Clipboard.setStringAsync("Hello React Native", {
  inputFormat: Clipboard.StringFormat.PLAIN_TEXT,
});
```

For normal text copying, the simple call without options is usually enough. Confirm option names and support against your installed SDK.

---

## 5. `getStringAsync()` — read clipboard text

### Syntax

```js
const text = await Clipboard.getStringAsync();
```

If the clipboard contains `Hello React Native`, then:

```js
console.log(text);
```

**Output:**

```text
Hello React Native
```

### Read plain text explicitly

The supplied notes show `preferredFormat` as an optional choice:

```js
const text = await Clipboard.getStringAsync({
  preferredFormat: Clipboard.StringFormat.PLAIN_TEXT,
});
```

### Read HTML representation

```js
const html = await Clipboard.getStringAsync({
  preferredFormat: Clipboard.StringFormat.HTML,
});

console.log(html);
```

The returned value depends on what is available in the clipboard and the platform. For a normal app, `getStringAsync()` without options is the basic pattern.

### Clean pasted text

```js
const text = (await Clipboard.getStringAsync()).trim();
```

`trim()` removes whitespace from the beginning and end. Validate clipboard data before using it as an OTP, URL, phone number, or other structured value.

---

## 6. `hasStringAsync()` — check for text

### Syntax

```js
const hasText = await Clipboard.hasStringAsync();
console.log(hasText);
```

**Possible outputs:**

```js
true
```

or

```js
false
```

The supplied SDK 54 notes say this can return `true` for plain text and rich text such as HTML.

### Check, then read

```js
const pasteText = async () => {
  try {
    const hasText = await Clipboard.hasStringAsync();

    if (!hasText) {
      console.log("Clipboard is empty");
      return;
    }

    const text = await Clipboard.getStringAsync();
    console.log(text);
  } catch (error) {
    console.error("Clipboard read failed:", error);
  }
};
```

**Example output if text exists:**

```text
Hello World
```

**Example output if no string content exists:**

```text
Clipboard is empty
```

Checking that content exists does **not** prove that it is valid for your application.

---

## 7. Complete Copy-and-Paste app

This small example combines the three core string methods. It also shows the most important React Native props.

```jsx
import { useState } from "react";
import {
  View,
  Text,
  TextInput,
  Button,
  Alert,
} from "react-native";
import * as Clipboard from "expo-clipboard";

export default function App() {
  const [text, setText] = useState("");
  const [pastedText, setPastedText] = useState("");

  const copyText = async () => {
    if (!text.trim()) {
      Alert.alert("Empty text", "Enter text before copying.");
      return;
    }

    try {
      await Clipboard.setStringAsync(text);
      Alert.alert("Copied!", "Text copied to clipboard.");
    } catch (error) {
      Alert.alert("Error", "Unable to copy text.");
      console.error(error);
    }
  };

  const pasteText = async () => {
    try {
      const hasText = await Clipboard.hasStringAsync();

      if (!hasText) {
        Alert.alert("Clipboard empty", "Nothing to paste.");
        return;
      }

      const value = await Clipboard.getStringAsync();
      setPastedText(value);
    } catch (error) {
      Alert.alert("Error", "Unable to read clipboard.");
      console.error(error);
    }
  };

  return (
    <View style={{ flex: 1, padding: 20, justifyContent: "center" }}>
      <TextInput
        value={text}
        onChangeText={setText}
        placeholder="Enter text"
        style={{ borderWidth: 1, padding: 12, marginBottom: 12 }}
      />

      <Button title="Copy" onPress={copyText} />
      <Button title="Paste" onPress={pasteText} />

      <Text style={{ marginTop: 20 }}>
        Pasted text: {pastedText || "Nothing pasted yet"}
      </Text>
    </View>
  );
}
```

### Example behavior

1. Enter `Hello World`.
2. Press **Copy** → alert says `Copied!`.
3. Press **Paste** → screen displays `Pasted text: Hello World`.

### React Native props used

| Prop | Used on | Meaning |
|---|---|---|
| `value` | `TextInput` | Current input value |
| `onChangeText` | `TextInput` | Callback receiving the new text |
| `placeholder` | `TextInput` | Hint shown when input is empty |
| `style` | `View`, `TextInput`, `Text` | Controls layout and appearance |
| `title` | `Button` | Button label |
| `onPress` | `Button` | Function called when pressed |

These are React Native component props, not `expo-clipboard` options.

---

## 8. URLs in the clipboard

A URL is a web address, for example `https://example.com`.

The supplied SDK 54 notes describe these URL-specific methods as **iOS-only**:

### `setUrlAsync(url)`

```js
await Clipboard.setUrlAsync("https://example.com");
```

Sets the clipboard content specifically as a URL on iOS.

### `getUrlAsync()`

```js
const url = await Clipboard.getUrlAsync();
console.log(url);
```

The supplied notes describe the result as a URL string or `null` if no URL is available.

### `hasUrlAsync()`

```js
const hasURL = await Clipboard.hasUrlAsync();
console.log(hasURL);
```

**Output:** `true` or `false`.

### Android / cross-platform fallback from the supplied notes

For ordinary URL copying, use the string methods:

```js
await Clipboard.setStringAsync("https://example.com");

const value = await Clipboard.getStringAsync();
console.log(value);
```

**Output:**

```text
https://example.com
```

### Important comparison

| Need | Method |
|---|---|
| Copy a URL as ordinary text | `setStringAsync(url)` |
| Read copied text/URL | `getStringAsync()` |
| Use the URL-specific clipboard API | `setUrlAsync()`, `getUrlAsync()`, `hasUrlAsync()` where supported |

Do not assume URL-specific APIs work on every platform. Verify the current SDK documentation.

---

## 9. Images in the clipboard

The supplied notes describe two main image-reading methods.

### `hasImageAsync()`

```js
const hasImage = await Clipboard.hasImageAsync();
console.log(hasImage);
```

**Output:** `true` if image content is available, otherwise `false`.

### `getImageAsync(options)`

```js
const image = await Clipboard.getImageAsync({
  format: "png",
});
```

The supplied notes describe the result as an image object or `null` when no image is available.

**Options shown in the source:**

| Option | Example | Meaning |
|---|---|---|
| `format` | `"png"` or `"jpeg"` | Requested image format |
| `jpegQuality` | `0.8` | JPEG quality from `0` to `1`; source notes say the default is `1` and `1` means highest quality |

### Important image result properties

The supplied notes describe:

```js
image.data
image.size
```

- `image.data`: Base64-encoded image data with a data-URI prefix, for example `data:image/png;base64,...`.
- `image.size.width`: image width.
- `image.size.height`: image height.

Example result shape:

```js
{
  data: "data:image/png;base64,...",
  size: {
    width: 1080,
    height: 720
  }
}
```

This is an illustrative shape; the actual image and dimensions depend on clipboard content.

### Display a clipboard image

```jsx
import { Image } from "react-native";

<Image
  source={{ uri: image.data }}
  style={{ width: 200, height: 200 }}
/>
```

**Props used:**

- `source={{ uri: image.data }}`: supplies the image URI.
- `style`: sets display dimensions.

### Safe image-reading example

```js
const pasteImage = async () => {
  try {
    const hasImage = await Clipboard.hasImageAsync();

    if (!hasImage) {
      console.log("No image in clipboard");
      return;
    }

    const image = await Clipboard.getImageAsync({ format: "png" });

    if (!image?.data) {
      console.log("Image could not be read");
      return;
    }

    console.log("Image size:", image.size);
    // Store image in state to display it with <Image source={{ uri: image.data }} />
  } catch (error) {
    console.error("Unable to read clipboard image:", error);
  }
};
```

**Expected output when an image is available:**

```text
Image size: { width: 1080, height: 720 }
```

The exact dimensions will vary.

### PNG vs JPEG

- `png`: commonly useful when transparency or lossless image data matters.
- `jpeg`: commonly useful for photographic images and often smaller files.
- Image format availability and conversion behavior can vary by platform.

### `setImageAsync()` caution

The supplied material states that `setImageAsync(base64Image)` is deprecated in its SDK 54 context. Do not treat it as the preferred production method without checking the current Expo documentation.

### Performance and privacy

- Base64 image data can be large; avoid unnecessarily keeping multiple copies in React state.
- Clipboard data may have originated outside your app.
- On Web, image clipboard access may require browser support or permission.

---

## 10. Clipboard change listeners

Use a listener when the app needs to react to clipboard changes automatically. For a simple **Paste** button, manual reading with `getStringAsync()` is usually enough.

### `addClipboardListener(callback)`

```js
const subscription = Clipboard.addClipboardListener((event) => {
  console.log(event.contentTypes);
});
```

The supplied notes describe the return value as an event subscription.

### `event.contentTypes`

The supplied notes list these content types:

- `plain-text`
- `image`
- `url`
- `html`

The event describes the kinds of content currently available; it is not necessarily the full content itself.

### Detect content type

```js
const subscription = Clipboard.addClipboardListener((event) => {
  if (event.contentTypes.includes(Clipboard.ContentType.PLAIN_TEXT)) {
    console.log("Clipboard has text");
  }

  if (event.contentTypes.includes(Clipboard.ContentType.IMAGE)) {
    console.log("Clipboard has an image");
  }

  if (event.contentTypes.includes(Clipboard.ContentType.URL)) {
    console.log("Clipboard has a URL");
  }

  if (event.contentTypes.includes(Clipboard.ContentType.HTML)) {
    console.log("Clipboard has HTML");
  }
});
```

**Possible console output:**

```text
Clipboard has text
```

or

```text
Clipboard has an image
```

The output depends on the clipboard event.

### Read text when the listener reports text

```js
const subscription = Clipboard.addClipboardListener(async (event) => {
  const hasText = event.contentTypes.includes(
    Clipboard.ContentType.PLAIN_TEXT
  );

  if (!hasText) return;

  const text = await Clipboard.getStringAsync();
  console.log("Clipboard text:", text);
});
```

### Remove the listener

The supplied notes show both ways:

```js
subscription.remove();
```

and:

```js
Clipboard.removeClipboardListener(subscription);
```

In a React component, clean up in `useEffect`:

```jsx
import { useEffect, useState } from "react";
import { Text, View } from "react-native";
import * as Clipboard from "expo-clipboard";

export default function ClipboardMonitor() {
  const [text, setText] = useState("");

  useEffect(() => {
    const subscription = Clipboard.addClipboardListener(async (event) => {
      if (!event.contentTypes.includes(Clipboard.ContentType.PLAIN_TEXT)) {
        return;
      }

      const value = await Clipboard.getStringAsync();
      setText(value);
    });

    return () => {
      subscription.remove();
    };
  }, []);

  return (
    <View style={{ padding: 20 }}>
      <Text>{text || "Nothing copied yet"}</Text>
    </View>
  );
}
```

**Why cleanup matters:** it prevents a component from leaving an unnecessary listener active after it unmounts.

The supplied notes state that listener methods are no-ops on Web in the SDK 54 context. Confirm current platform behavior for your installed version.

---

## 11. `ContentType` vs `StringFormat`

These concepts are related but not the same.

### `Clipboard.ContentType`

Used to identify the **kind** of clipboard content.

The supplied notes list:

| Constant | Meaning |
|---|---|
| `Clipboard.ContentType.PLAIN_TEXT` | Normal text |
| `Clipboard.ContentType.IMAGE` | Image content |
| `Clipboard.ContentType.URL` | URL content |
| `Clipboard.ContentType.HTML` | HTML content |

Example:

```js
event.contentTypes.includes(Clipboard.ContentType.IMAGE);
```

This expression returns a Boolean (`true` or `false`).

### `Clipboard.StringFormat`

Used to select the **representation** of clipboard string content.

The supplied notes list:

| Constant | Meaning |
|---|---|
| `Clipboard.StringFormat.PLAIN_TEXT` | Plain text |
| `Clipboard.StringFormat.HTML` | HTML-formatted text |

Example:

```js
const text = await Clipboard.getStringAsync({
  preferredFormat: Clipboard.StringFormat.PLAIN_TEXT,
});
```

### Simple difference

- **ContentType:** “What kind of content is present?”
- **StringFormat:** “Which string representation do I want?”

For ordinary copy/paste apps, the default string methods are normally sufficient. Use explicit formatting only when your application needs rich-text/HTML handling, and verify the options for your installed SDK.

---

## 12. Reusable Copy Button

A reusable component avoids repeating the same copy logic.

```jsx
import { Button, Alert } from "react-native";
import * as Clipboard from "expo-clipboard";

export default function CopyButton({ text }) {
  const handleCopy = async () => {
    if (typeof text !== "string" || !text.trim()) {
      Alert.alert("Nothing to copy", "No text is available.");
      return;
    }

    try {
      await Clipboard.setStringAsync(text);
      Alert.alert("Copied!", "Text copied to clipboard.");
    } catch (error) {
      console.error(error);
      Alert.alert("Error", "Unable to copy text.");
    }
  };

  return <Button title="Copy" onPress={handleCopy} />;
}
```

Usage:

```jsx
<CopyButton text="+91 9876543210" />
<CopyButton text="rahul@example.com" />
<CopyButton text="https://example.com" />
```

**Important prop:** `text` is a custom prop defined by this component, not a built-in `Button` prop.

---

## 13. Copy multiple values or JSON

### Multiple lines

```js
const contactText = `Name: Rahul Sharma
Phone: +91 9876543210
Email: rahul@example.com`;

await Clipboard.setStringAsync(contactText);
```

Clipboard content:

```text
Name: Rahul Sharma
Phone: +91 9876543210
Email: rahul@example.com
```

### Copy an object as JSON text

```js
const user = {
  name: "Rahul",
  email: "rahul@example.com",
};

const jsonText = JSON.stringify(user, null, 2);
await Clipboard.setStringAsync(jsonText);
```

Clipboard content:

```json
{
  "name": "Rahul",
  "email": "rahul@example.com"
}
```

`JSON.stringify()` converts an object into a string. The clipboard stores text, not the original JavaScript object.

---

## 14. Error handling pattern

Clipboard methods are asynchronous. When appropriate, use `try...catch`:

```js
try {
  await Clipboard.setStringAsync(text);
  console.log("Copied successfully");
} catch (error) {
  console.error("Clipboard operation failed:", error);
}
```

Use clear user feedback for:
- Empty input
- No content available
- Invalid pasted content
- Failed read/write operations

Do not silently assume that copied content is valid.

---

## 15. Security and privacy

- Treat clipboard content as external input.
- Validate pasted OTPs, URLs, phone numbers, and referral codes before using them.
- Do not copy passwords, tokens, API keys, or other sensitive information without a clear reason and explicit user action.
- Avoid displaying or logging sensitive clipboard content unnecessarily.
- Read clipboard data only when your app needs it.
- A `has...` method checks availability, not whether the content is safe or valid.

Example six-digit OTP validation:

```js
const value = (await Clipboard.getStringAsync()).trim();

if (!/^\d{6}$/.test(value)) {
  console.log("Invalid OTP");
  return;
}

console.log("Valid six-digit OTP format");
```

This checks the format only; it does not verify that the OTP is authentic or correct.

---

## 16. Common mistakes

1. Forgetting to install `expo-clipboard`.
2. Forgetting to import `* as Clipboard`.
3. Calling an async clipboard method without `await` when the result is needed.
4. Confusing `setStringAsync()` (write) with `getStringAsync()` (read).
5. Treating `hasStringAsync()` as the actual text instead of a Boolean.
6. Assuming a clipboard URL is valid just because it exists.
7. Assuming all URL-specific methods work on Android and Web.
8. Assuming an image is always available; handle `null` and missing `image.data`.
9. Forgetting to remove a clipboard listener.
10. Treating `ContentType` and `StringFormat` as interchangeable.
11. Assuming the clipboard contains only data copied by your own app.
12. Using deprecated APIs without checking current SDK documentation.

---

## 17. Final revision tables

### Methods and return values

| Method | Input/options | Result |
|---|---|---|
| `setStringAsync(text)` | Text string | Promise; supplied notes describe `true` on Android/iOS |
| `getStringAsync()` | Optional format options | String |
| `hasStringAsync()` | None | Boolean |
| `setUrlAsync(url)` | URL string | Promise; iOS-specific per supplied notes |
| `getUrlAsync()` | None | URL string or `null`; iOS-specific per supplied notes |
| `hasUrlAsync()` | None | Boolean; iOS-specific per supplied notes |
| `hasImageAsync()` | None | Boolean |
| `getImageAsync({ format })` | `"png"` or `"jpeg"` in supplied notes | Image object or `null` |
| `addClipboardListener(callback)` | Callback receives event | Subscription |
| `removeClipboardListener(subscription)` | Subscription | Removes listener |

### Important image properties

| Property | Meaning |
|---|---|
| `image.data` | Base64/data-URI image string, as described in the supplied notes |
| `image.size.width` | Image width |
| `image.size.height` | Image height |

### Important props and options

| Name | Belongs to | Purpose |
|---|---|---|
| `value` | React Native `TextInput` | Current input value |
| `onChangeText` | React Native `TextInput` | Updates input state |
| `placeholder` | React Native `TextInput` | Empty-field hint |
| `title` | React Native `Button` | Button label |
| `onPress` | React Native `Button` | Press callback |
| `source` | React Native `Image` | Image source |
| `style` | React Native components | Layout and appearance |
| `format` | `getImageAsync` options in supplied notes | `"png"` or `"jpeg"` |
| `preferredFormat` | `getStringAsync` options shown in supplied notes | Preferred plain-text/HTML representation |
| `inputFormat` | `setStringAsync` option shown in supplied notes | Input string format |

## 18. Best study order

1. Clipboard concept and installation
2. `setStringAsync()`, `getStringAsync()`, `hasStringAsync()`
3. The complete copy/paste app and React Native props
4. URL methods and platform differences
5. Image methods and `image.data` / `image.size`
6. Clipboard listeners and cleanup
7. `ContentType` vs `StringFormat`
8. Error handling, validation, and privacy



---

## 19. Additional important details from the original notes

### A. `jpegQuality` — control JPEG quality

When requesting a JPEG clipboard image, the original material also describes the `jpegQuality` option:

```js
const image = await Clipboard.getImageAsync({
  format: "jpeg",
  jpegQuality: 0.8,
});
```

- `format`: requests `"png"` or `"jpeg"` in the supplied notes.
- `jpegQuality`: a value from `0` to `1` in the supplied notes.
- The supplied material says the default is `1`; `1` means highest JPEG quality, while lower values allow more compression.
- Verify this option against the Expo SDK version installed in your project.

### B. Validate a URL before using it

A clipboard value may be empty or may not be a valid web URL. The original notes demonstrate checking for an HTTP or HTTPS prefix after reading a URL:

```js
const url = await Clipboard.getUrlAsync();

if (
  !url?.startsWith("https://") &&
  !url?.startsWith("http://")
) {
  console.log("Invalid URL");
  return;
}

console.log("URL can be checked further:", url);
```

For ordinary cross-platform clipboard text, you can read with `getStringAsync()` instead, then apply the same basic check. A prefix check is only a basic filter; it does not prove that the full URL is valid or safe to open. URL-specific methods may be platform-limited, as described earlier.

### C. `ClipboardEvent.content` is deprecated in the supplied SDK 54 notes

The original material states that `ClipboardEvent.content` is deprecated and returns an empty string in its SDK 54 context. To retrieve the actual clipboard text, use `getStringAsync()`:

```js
const subscription = Clipboard.addClipboardListener(async (event) => {
  if (event.contentTypes.includes(Clipboard.ContentType.PLAIN_TEXT)) {
    const text = await Clipboard.getStringAsync();
    console.log("Clipboard text:", text);
  }
});
```

Do not rely on `event.content` to obtain the clipboard text. Confirm the exact event API for your installed SDK.

### D. `event.contentTypes` can contain multiple types

`event.contentTypes` is an array, so it may contain more than one content type. Check each type separately rather than assuming there is only one:

```js
const subscription = Clipboard.addClipboardListener((event) => {
  if (event.contentTypes.includes(Clipboard.ContentType.PLAIN_TEXT)) {
    console.log("Text is available");
  }

  if (event.contentTypes.includes(Clipboard.ContentType.IMAGE)) {
    console.log("Image is available");
  }

  if (event.contentTypes.includes(Clipboard.ContentType.URL)) {
    console.log("URL is available");
  }

  if (event.contentTypes.includes(Clipboard.ContentType.HTML)) {
    console.log("HTML is available");
  }
});

// Clean up when the listener is no longer needed:
subscription.remove();
```

The event indicates available content types; it does not necessarily contain the actual content itself.

### E. Base64 image data — why `image.data` can be used directly

The supplied notes describe `image.data` as a Base64-encoded image string with a data-URI prefix, for example:

```text
data:image/png;base64,...
data:image/jpeg;base64,...
```

Use it directly as an image URI:

```jsx
<Image
  source={{ uri: image.data }}
  style={{ width: 200, height: 200 }}
/>
```

Do not add another `data:image/...;base64,` prefix if the returned value already contains it. Base64 strings can be large, so avoid keeping unnecessary copies in state.

### F. `TextInput` state vs system clipboard

These are different storage locations:

- `TextInput` value / React state (for example, `text`) holds the value displayed and edited inside your app.
- The system clipboard holds copied content that may be pasted into your app or another app.
- `setStringAsync(text)` writes text to the system clipboard.
- `getStringAsync()` reads text from the system clipboard.
- `setText(value)` updates React state; it does not copy anything to the system clipboard by itself.

Typical flow:

```text
TextInput state → setStringAsync() → System clipboard
System clipboard → getStringAsync() → setText(value) → TextInput
```

---

**Final reminder:** This document cleans up the repeated examples from the supplied material while retaining the important topics. The API signatures, outputs, and platform notes above reflect that source's Expo SDK 54 framing; verify them against the current official documentation and the SDK installed in your project.
