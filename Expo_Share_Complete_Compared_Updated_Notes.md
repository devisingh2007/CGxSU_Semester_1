# Expo Share API — Clean & Complete Notes

> Easy revision notes for React Native + Expo, based on the supplied material targeting **Expo SDK 54**. Repeated examples have been removed; important APIs, outputs, options, props, platform differences, and workflows are retained.
>
> **Version note:** Check the official documentation for your installed Expo/React Native version before relying on platform-specific behavior or less-common options.

## 1. What is the Share API?

The Share API opens the device's native sharing interface so the user can send content to another app.

Common uses:
- Share text, website links, referral links, product details, contacts, and order details.
- Share local files such as images, PDFs, videos, and documents.

### Which API should I use?

| Need | API | Purpose |
|---|---|---|
| Share text or a URL | `Share` from `react-native` | Opens the native share sheet |
| Share a local file | `expo-sharing` | Shares a local file URI |
| Select an image first | `expo-image-picker` | Opens the image library and returns an asset URI |
| Select a document first | `expo-document-picker` | Lets the user choose a file and returns its URI |

**Mental model**

```text
Text / URL ──> Share.share() ───────────────┐
                                            ├─> Native Share Sheet
Local file ──> Sharing.shareAsync(fileUri) ─┘
```

The operating system controls which destination apps appear. Your app does not directly control WhatsApp, Gmail, Telegram, or other destination apps.

The native **Share Sheet** is created by Android/iOS, not by your React Native UI. Its appearance and available targets can vary with the OS version, device manufacturer, and installed apps.

---

## 2. Installation and imports

### React Native Share

No additional package installation is required.

```js
import { Share } from "react-native";
```

### Share local files with Expo

```bash
npx expo install expo-sharing
```

```js
import * as Sharing from "expo-sharing";
```

### Select images or documents

```bash
npx expo install expo-image-picker expo-document-picker
```

```js
import * as ImagePicker from "expo-image-picker";
import * as DocumentPicker from "expo-document-picker";
```

Install only the picker package your feature needs.

---

## 3. React Native `Share.share()`

### Syntax

```js
Share.share(content, options);
```

- `content`: what you want to share.
- `options`: optional configuration for the native share operation.

It is asynchronous and returns a Promise. Use `await` when you need the result.

### Basic example

```js
const result = await Share.share({
  message: "Hello from my React Native app!",
});

console.log(result);
```

This opens the native share sheet. The user chooses a destination or dismisses the sheet.

### Important `content` properties

| Property | Purpose | Platform note from the supplied notes |
|---|---|---|
| `message` | Main text content to share | Core property for text sharing |
| `url` | URL content | Documented in the supplied notes as an iOS content property |
| `title` | Title for the share operation/dialog | Documented for Android; not guaranteed to be shared as recipient-facing text |

#### `message` — main shareable text

```js
await Share.share({
  message: "Check out this course!",
});
```

Expected shared text:

```text
Check out this course!
```

Dynamic message:

```js
const name = "Rajesh";

await Share.share({
  message: `Hello ${name}, check out this app!`,
});
```

Expected text:

```text
Hello Rajesh, check out this app!
```

#### Share a URL

For the URL to be part of the text across platforms, put it inside `message`:

```js
await Share.share({
  message: "Check this website:\nhttps://example.com",
});
```

Expected text:

```text
Check this website:
https://example.com
```

You may also pass `url`:

```js
await Share.share({
  message: "Check out this website!",
  url: "https://example.com",
});
```

However, do not assume `url` behaves identically on Android and iOS. The supplied notes describe it as an iOS content property.

#### `title` vs `message`

```js
await Share.share({
  title: "React Native Course",
  message: "Check out this course!",
});
```

- `message`: the main shareable text.
- `title`: information supplied to the Android share operation/dialog.
- A title is **not guaranteed** to appear in the recipient's message or visibly in every device's share UI.

### `options`

The supplied notes show `dialogTitle` as an option:

```js
await Share.share(
  { message: "Hello!" },
  { dialogTitle: "Share this message" }
);
```

The operating system controls the actual share-sheet appearance.

---

## 4. `Share.share()` result and outputs

Example:

```js
const result = await Share.share({
  message: "Hello!",
});

console.log(result);
```

A result may look like:

```js
{
  action: "sharedAction",
  activityType: null
}
```

### Result properties

| Property / constant | Meaning |
|---|---|
| `result.action` | Indicates the share operation outcome |
| `Share.sharedAction` | Constant used to check the shared action |
| `Share.dismissedAction` | Constant used to check whether the share dialog was dismissed |
| `result.activityType` | iOS sharing activity/service information; can be `null` where unavailable |

Check the action using the constants, rather than hardcoding strings:

```js
const result = await Share.share({
  message: "Hello!",
});

if (result.action === Share.sharedAction) {
  console.log("Share operation completed");
}

if (result.action === Share.dismissedAction) {
  console.log("Share dialog dismissed");
}
```

Possible messages:

```text
Share operation completed
```

or

```text
Share dialog dismissed
```

The result depends on what happens in the native share UI and platform behavior. In particular, `activityType` is mainly useful on iOS; `null` is not proof of failure. Use `result.action` and compare it with `Share.sharedAction` / `Share.dismissedAction` when you need to branch on the outcome.

**Important:** `activityType: null` does not by itself mean sharing failed. To inspect the operation outcome, check `result.action`.

### `async/await` and error handling

```js
try {
  const result = await Share.share({
    message: "Hello React Native!",
  });

  console.log(result);
} catch (error) {
  console.error("Unable to share:", error);
}
```

A Promise can also be handled with `.then()` and `.catch()`:

```js
Share.share({ message: "Hello!" })
  .then((result) => {
    console.log(result);
  })
  .catch((error) => {
    console.error("Unable to share:", error);
  });
```

Both approaches work. `async/await` with `try/catch` is often easier to read. Without `await` or `.then()`, logging the immediate return value shows a Promise rather than the resolved result.

---

## 5. Share text from a `TextInput`

Important React Native props:
- `value`: current input value.
- `onChangeText`: callback receiving the new text.
- `placeholder`: hint shown when the input is empty.
- `onPress`: callback invoked when a button is pressed.
- `title`: text displayed on a React Native `Button`.
- `style`: controls appearance and layout.

```jsx
import { useState } from "react";
import { View, TextInput, Button, Share, Alert } from "react-native";

export default function ShareTextScreen() {
  const [text, setText] = useState("");

  const handleShare = async () => {
    const message = text.trim();

    if (!message) {
      Alert.alert("Empty text", "Enter text before sharing.");
      return;
    }

    try {
      await Share.share({ message });
    } catch (error) {
      Alert.alert("Error", "Unable to share text.");
      console.error(error);
    }
  };

  return (
    <View style={{ padding: 20 }}>
      <TextInput
        value={text}
        onChangeText={setText}
        placeholder="Enter text to share"
        style={{ borderWidth: 1, padding: 12, marginBottom: 12 }}
      />
      <Button title="Share" onPress={handleShare} />
    </View>
  );
}
```

`trim()` removes whitespace from the beginning and end. Checking `!message` prevents sharing a string that contains only spaces.

---

## 6. `expo-sharing` — share local files

Use `expo-sharing` when sharing a file stored on the device, such as an image, PDF, video, or document.

### `Sharing.isAvailableAsync()`

Checks whether file sharing is available.

```js
const available = await Sharing.isAvailableAsync();
console.log(available);
```

**Output:** `true` or `false`.

Example:

```js
const available = await Sharing.isAvailableAsync();

if (!available) {
  console.log("File sharing is not available");
  return;
}
```

### `Sharing.shareAsync(fileUri, options?)`

Opens the native sharing interface for a **local file URI**.

```js
await Sharing.shareAsync(fileUri);
```

The supplied notes do not specify a useful content result for this method; treat it as an asynchronous operation and handle errors with `try/catch`.

Example with options:

```js
await Sharing.shareAsync(fileUri, {
  mimeType: "application/pdf",
  dialogTitle: "Share Report",
});
```

### Options covered in the supplied notes

| Option | Purpose | Example |
|---|---|---|
| `mimeType` | Describes the file type | `"application/pdf"` |
| `dialogTitle` | Title for the share dialog where supported | `"Share Report"` |

Common MIME types:

| File type | MIME type |
|---|---|
| PDF | `application/pdf` |
| JPEG | `image/jpeg` |
| PNG | `image/png` |
| MP4 video | `video/mp4` |

### Local URI vs remote URL

A local file URI points to a file already accessible on the device:

```text
file:///path/to/report.pdf
```

A remote URL points to a server:

```text
https://example.com/report.pdf
```

Do not pass a remote web URL to `Sharing.shareAsync()` as if it were a local file URI. For a remote file, download it first, then share the resulting local URI.

```text
Remote URL → Download file → Local file URI → shareAsync()
```

### Safe file-sharing pattern

```js
const shareFile = async (fileUri) => {
  try {
    const available = await Sharing.isAvailableAsync();

    if (!available) {
      console.log("Sharing is unavailable");
      return;
    }

    await Sharing.shareAsync(fileUri);
  } catch (error) {
    console.error("Unable to share file:", error);
  }
};
```

---

## 7. Select and share an image

Use `expo-image-picker` to select an image, then `expo-sharing` to share the returned URI.

### Important picker properties and result fields

| Name | Purpose |
|---|---|
| `mediaTypes: ["images"]` | Requests images from the library |
| `allowsEditing` | Controls whether editing is offered |
| `quality` | Image quality setting in the picker options |
| `result.canceled` | `true` if the user cancelled |
| `result.assets[0]` | First selected asset |
| `result.assets[0].uri` | URI of the selected image |
| `result.assets[0].mimeType` | MIME type, when available |

### Example

```js
import * as ImagePicker from "expo-image-picker";
import * as Sharing from "expo-sharing";

const pickAndShareImage = async () => {
  try {
    const available = await Sharing.isAvailableAsync();

    if (!available) {
      console.log("File sharing is unavailable");
      return;
    }

    const result = await ImagePicker.launchImageLibraryAsync({
      mediaTypes: ["images"],
      allowsEditing: false,
      quality: 1,
    });

    if (result.canceled || !result.assets?.length) {
      return;
    }

    const image = result.assets[0];

    await Sharing.shareAsync(image.uri, {
      mimeType: image.mimeType || "image/jpeg",
      dialogTitle: "Share Image",
    });
  } catch (error) {
    console.error("Unable to pick or share image:", error);
  }
};
```

### What does the picker result look like?

Illustrative result:

```js
{
  canceled: false,
  assets: [
    {
      uri: "file:///.../image.jpg",
      width: 1920,
      height: 1080,
      mimeType: "image/jpeg"
    }
  ]
}
```

- If `canceled` is `true`, stop; no image was selected.
- `assets[0].uri` is the selected image's URI.
- The MIME type can be missing, so the example uses a fallback.

**Remember:** `expo-image-picker` selects the image; `expo-sharing` opens the share sheet for its URI.

---

## 8. Select and share a document or other file

Use `expo-document-picker` for PDFs, documents, ZIP files, and other file types.

### Important properties and result fields

| Name | Purpose |
|---|---|
| `type: "*/*"` | Allows a broad range of file types |
| `copyToCacheDirectory: true` | Copies the selected file to the app cache, useful when another API needs to access it |
| `result.canceled` | Indicates cancellation |
| `result.assets[0]` | First selected file |
| `file.uri` | Selected file URI |
| `file.mimeType` | MIME type, when available |

### Example

```js
import * as DocumentPicker from "expo-document-picker";
import * as Sharing from "expo-sharing";

const pickAndShareFile = async () => {
  try {
    const available = await Sharing.isAvailableAsync();

    if (!available) {
      console.log("File sharing is unavailable");
      return;
    }

    const result = await DocumentPicker.getDocumentAsync({
      type: "*/*",
      copyToCacheDirectory: true,
    });

    if (result.canceled || !result.assets?.length) {
      return;
    }

    const file = result.assets[0];

    await Sharing.shareAsync(file.uri, {
      mimeType: file.mimeType,
      dialogTitle: "Share File",
    });
  } catch (error) {
    console.error("Unable to pick or share file:", error);
  }
};
```

Flow:

```text
Document picker → result.assets[0] → file.uri → Sharing.shareAsync()
```

---

## 9. SDK 54 installation quick reference

For a project using Expo SDK 54, install Expo packages with `npx expo install` so Expo can choose compatible package versions.

```bash
npx expo install expo-sharing
npx expo install expo-image-picker
npx expo install expo-document-picker
```

Install only the packages required by your feature. React Native's built-in `Share` needs no separate installation.

## 10. Real-world text-sharing pattern

The Share API does not need to understand whether the message represents a product, contact, order, course, profile, or referral. Build a normal text message from your data, then pass it to `Share.share()`.

```js
const product = {
  name: "React Native Course",
  price: 999,
  url: "https://example.com/course",
};

const message =
  `Check out this product!\\n\\n` +
  `Name: ${product.name}\\n` +
  `Price: ₹${product.price}\\n\\n` +
  product.url;

await Share.share({ message });
```

Example shared text:

```text
Check out this product!

Name: React Native Course
Price: ₹999

https://example.com/course
```

The same pattern works for contact details, order summaries, or referral links: format the data into a string and share the string. Avoid including private or sensitive details unless the user intends to share them.

## 11. Important API and output revision table

| API | Input | Output / result |
|---|---|---|
| `Share.share(content, options?)` | Content object and optional options | Promise resolving to a result object with `action`; `activityType` may also be present |
| `Share.sharedAction` | Constant | Used to compare with `result.action` |
| `Share.dismissedAction` | Constant | Used to compare with `result.action` |
| `Sharing.isAvailableAsync()` | None | Promise resolving to `true` or `false` |
| `Sharing.shareAsync(fileUri, options?)` | Local file URI and optional options | Asynchronous file-sharing operation; source notes do not specify a meaningful result value |
| `ImagePicker.launchImageLibraryAsync(options?)` | Picker options | Result with `canceled` and, when selected, `assets` |
| `DocumentPicker.getDocumentAsync(options?)` | Document picker options | Result with `canceled` and, when selected, `assets` |

## 12. Props and options at a glance

### React Native UI props

| Prop | Component | Meaning |
|---|---|---|
| `value` | `TextInput` | Current input value |
| `onChangeText` | `TextInput` | Receives new text |
| `placeholder` | `TextInput` | Hint for empty input |
| `title` | `Button` | Button label |
| `onPress` | `Button` / `Pressable` | Runs when pressed |
| `style` | UI components | Layout and appearance |

### Share API content and options

| Name | Used with | Meaning |
|---|---|---|
| `message` | `Share.share()` content | Main text to share |
| `url` | `Share.share()` content | URL property; platform-specific behavior |
| `title` | `Share.share()` content | Android share-operation/dialog title; not guaranteed recipient text |
| `dialogTitle` | Share options shown in source | Dialog title where supported |
| `mimeType` | `Sharing.shareAsync()` options | File MIME type |

### Picker options

| Option | API | Meaning |
|---|---|---|
| `mediaTypes: ["images"]` | Image Picker | Select images |
| `allowsEditing` | Image Picker | Whether editing is allowed |
| `quality` | Image Picker | Image quality setting |
| `type: "*/*"` | Document Picker | Broad file-type selection |
| `copyToCacheDirectory: true` | Document Picker | Copies selected file into cache |

---

## 13. Common mistakes to avoid

1. Using `Share.share()` to share a local file when `expo-sharing` is the appropriate API.
2. Passing a remote `https://...` URL to `Sharing.shareAsync()` as if it were a local file URI.
3. Forgetting `await` when you need a Promise's result.
4. Treating `title` as guaranteed message text for the recipient.
5. Assuming the `url` property behaves identically on Android and iOS.
6. Assuming `activityType: null` means sharing failed.
7. Forgetting to check `result.canceled` before reading a picker asset.
8. Assuming `result.assets[0]` exists when the user cancelled.
9. Hardcoding a file path that does not point to a real, accessible file.
10. Skipping `Sharing.isAvailableAsync()` when availability matters.
11. Forgetting `try/catch` around asynchronous operations.
12. Assuming all devices show the same share sheet or destination apps.

## 14. Final mental model

```text
Need to share text or URL?
    └─> Share.share({ message, ... })
            └─> Check result.action if needed

Need to share a local file?
    └─> Sharing.isAvailableAsync()
            └─> Sharing.shareAsync(fileUri, options?)

Need to let the user choose an image?
    └─> ImagePicker.launchImageLibraryAsync()
            └─> Check canceled
                    └─> assets[0].uri
                            └─> Sharing.shareAsync(uri)

Need to let the user choose a document?
    └─> DocumentPicker.getDocumentAsync()
            └─> Check canceled
                    └─> assets[0].uri
                            └─> Sharing.shareAsync(uri)
```

## 15. Best study order

1. Difference between React Native `Share` and Expo `Sharing`.
2. `Share.share()` content: `message`, `url`, `title`.
3. Share result: `action`, `sharedAction`, `dismissedAction`, `activityType`.
4. Promise, `await`, and `try/catch`.
5. `Sharing.isAvailableAsync()` and `Sharing.shareAsync()`.
6. Local file URI vs remote URL; MIME types.
7. Image Picker result and cancellation handling.
8. Document Picker result and cancellation handling.
9. React Native props and final revision tables.

**Final reminder:** These notes retain the important concepts, methods, outputs, props, options, and workflows from the supplied material while removing repeated examples. Platform behavior and exact API options can vary by SDK version, so check the documentation for the version installed in your project.
