# Expo Local Authentication — Clean & Complete Notes

> Easy revision notes for React Native + Expo using `expo-local-authentication`. These notes keep the important concepts, methods, return values, options, platform differences, errors, configuration, and practical patterns while removing repeated explanations.
>
> **Version note:** API options and native behavior can vary by Expo SDK, OS, and device. The supplied material targets Expo SDK 54. Check the documentation matching your installed SDK before shipping.

## 1. What is Local Authentication?

`expo-local-authentication` lets an app ask the device operating system to authenticate the current user using an already configured device method, such as fingerprint, Face ID, or Touch ID.

The app **does not receive raw fingerprint or face data**. The operating system performs the biometric verification and returns a result.

### Common use cases
- Re-lock an app or private screen when it is reopened.
- Protect private notes, documents, or account information.
- Add a local check before a sensitive action, such as confirming a payment.
- Offer biometric unlock as a convenience after the user has logged into the app.

### Local authentication vs app/server login

| App/server authentication | Local authentication |
|---|---|
| Usually verifies account credentials with your backend | Asks the device OS to verify the device user |
| Can establish a server session | Does not create a backend session by itself |
| Example: email + password | Example: Face ID / fingerprint prompt |

Local authentication is usually an **extra local access gate**, not a replacement for backend authentication or server-side authorization.

## 2. Install and import

```bash
npx expo install expo-local-authentication
```

```js
import * as LocalAuthentication from "expo-local-authentication";
```

Use `npx expo install` so Expo can select a compatible package version. Installing the package does not automatically authenticate a user.

## 3. The five important questions

| Question | Method | Output |
|---|---|---|
| Is biometric hardware available? | `hasHardwareAsync()` | `Promise<boolean>` |
| Is an eligible credential enrolled? | `isEnrolledAsync()` | `Promise<boolean>` |
| Which biometric types are supported? | `supportedAuthenticationTypesAsync()` | `Promise<AuthenticationType[]>` |
| What enrolled security level is available? | `getEnrolledLevelAsync()` | `Promise<SecurityLevel>` |
| Did the authentication attempt succeed? | `authenticateAsync(options?)` | `Promise<LocalAuthenticationResult>` |

**Do not confuse these states:** hardware available ≠ credential enrolled ≠ authentication succeeded.

## 4. Check hardware

### `hasHardwareAsync()`

```js
const hasHardware =
  await LocalAuthentication.hasHardwareAsync();

console.log(hasHardware);
```

**Output:** `true` or `false`.

- `true`: the device reports biometric hardware availability.
- `false`: biometric hardware is unavailable for this use.
- It does **not** show an authentication prompt.
- It does **not** prove that a biometric is enrolled.

## 5. Check enrollment

### `isEnrolledAsync()`

```js
const enrolled =
  await LocalAuthentication.isEnrolledAsync();

console.log(enrolled);
```

**Output:** `true` or `false`.

- `true`: the device reports an eligible local authentication credential is enrolled.
- `false`: no eligible credential is enrolled.
- Hardware can be present while enrollment is false.

Example state:

```text
Hardware = true
Enrolled = false
```

This is valid: the phone has a sensor, but the user has not configured a biometric credential.

## 6. Supported authentication types

### `supportedAuthenticationTypesAsync()`

```js
const types =
  await LocalAuthentication.supportedAuthenticationTypesAsync();

console.log(types);
```

**Output:** an array of enum values. The array can contain zero, one, or multiple supported types depending on the device/platform.

Use named enum constants instead of hard-coded numbers:

```js
LocalAuthentication.AuthenticationType.FINGERPRINT
LocalAuthentication.AuthenticationType.FACIAL_RECOGNITION
LocalAuthentication.AuthenticationType.IRIS
```

The supplied material lists these enum values:

| Enum | Numeric value in supplied material |
|---|---:|
| `FINGERPRINT` | `1` |
| `FACIAL_RECOGNITION` | `2` |
| `IRIS` | `3` |

Prefer enum names over numeric values because names are easier to understand and maintain.

### Check for a specific type

```js
const types =
  await LocalAuthentication.supportedAuthenticationTypesAsync();

const supportsFingerprint = types.includes(
  LocalAuthentication.AuthenticationType.FINGERPRINT
);

const supportsFace = types.includes(
  LocalAuthentication.AuthenticationType.FACIAL_RECOGNITION
);

console.log({ supportsFingerprint, supportsFace });
```

**Output:** an object containing booleans, for example:

```js
{ supportsFingerprint: true, supportsFace: false }
```

This checks capability, not whether the user is enrolled or has just authenticated.

## 7. Enrolled security level

### `getEnrolledLevelAsync()`

```js
const level =
  await LocalAuthentication.getEnrolledLevelAsync();

console.log(level);
```

**Output:** a `SecurityLevel` enum value. Compare it using named constants:

```js
if (
  level === LocalAuthentication.SecurityLevel.BIOMETRIC_STRONG
) {
  console.log("Strong biometric level is enrolled.");
}
```

The supplied material lists these levels:

| Enum | Value | General meaning |
|---|---:|---|
| `NONE` | `0` | No enrolled authentication at the relevant level |
| `SECRET` | `1` | Non-biometric secret such as PIN, pattern, or passcode |
| `BIOMETRIC_WEAK` | `2` | Platform's weaker biometric security class |
| `BIOMETRIC_STRONG` | `3` | Platform's strong biometric security class |

Platform behavior and exact semantics can vary. A security level describes the enrolled method/class; it does **not** mean that the user has successfully authenticated in the current attempt.

## 8. Authenticate the user

### `authenticateAsync(options?)`

This starts the native authentication prompt.

```js
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: "Authenticate to continue",
});

console.log(result);
```

The important result property is `success`.

Successful result, conceptually:

```js
{ success: true }
```

Unsuccessful result, conceptually:

```js
{ success: false, error: "authentication_failed" }
```

The actual result may include additional properties such as an error or warning, depending on the outcome and SDK.

**Always check `result.success`.** Calling the method is only a request; it is not proof of success.

```js
if (result.success) {
  // Allow the protected action.
} else {
  // Keep the protected content/action locked.
  console.log("Authentication result:", result.error);
}
```

## 9. `authenticateAsync()` options

Pass an options object as the argument. Options customize the native prompt and behavior; they do not replace the need to inspect the result.

| Option | Platform in supplied material | Purpose / notes |
|---|---|---|
| `promptMessage` | Android + iOS | Main message explaining why authentication is needed |
| `cancelLabel` | Android + iOS | Label for the cancel action where supported |
| `disableDeviceFallback` | Android + iOS | Controls device-credential fallback behavior; exact behavior is OS-dependent |
| `fallbackLabel` | iOS | Custom label for the fallback button; an empty string can hide that button where supported |
| `promptDescription` | Android | Additional prompt description |
| `promptSubtitle` | Android | Prompt subtitle |
| `requireConfirmation` | Android | Requests confirmation behavior; the system/device may ignore it |
| `biometricsSecurityLevel` | Android | Minimum accepted biometric class: `"weak"` or `"strong"` |

### Examples

```js
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: "Authenticate to open private notes",
  cancelLabel: "Cancel",
});
```

Android-specific example:

```js
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: "Authenticate to access your account",
  promptDescription: "Confirm your identity",
  promptSubtitle: "Use your biometric",
  requireConfirmation: true,
  biometricsSecurityLevel: "strong",
});
```

iOS-specific fallback example:

```js
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: "Authenticate to continue",
  fallbackLabel: "Use Passcode",
});
```

### Important option details

- **`promptMessage`**: use a clear reason, e.g. `"Authenticate to confirm payment"`, not a vague message like `"Welcome"`.
- **`cancelLabel`**: customizes the cancel label where supported.
- **`disableDeviceFallback`**: controls fallback to the device credential. Be careful: disabling fallback may leave a user without an alternative when biometrics cannot be used.
- **`fallbackLabel`**: iOS-specific fallback button label; `""` can hide the fallback button where supported.
- **`promptDescription` / `promptSubtitle`**: Android-specific prompt text.
- **`requireConfirmation`**: asks Android to require confirmation, but the OS/device can ignore this setting.
- **`biometricsSecurityLevel`**:
  - `"weak"` permits the platform's weak class and stronger biometrics.
  - `"strong"` requests strong biometrics only.
  - This option is Android-specific in the supplied material.

Do not add every option automatically. Choose the options that match the product's user experience and security requirements.

## 10. Authentication errors

When `success` is false, inspect `result.error`. It is an error identifier, **not a user-facing sentence**. Convert it to a helpful message if needed.

| Error | Meaning / suggested handling |
|---|---|
| `not_enrolled` | No biometric credential enrolled; explain how to enable biometrics or offer fallback |
| `user_cancel` | User cancelled; usually keep the screen locked without treating it as a crash |
| `app_cancel` | App cancelled the authentication request |
| `not_available` | Authentication is unavailable on the device/platform |
| `lockout` | Authentication is locked after repeated attempts; ask user to follow system guidance or use an allowed fallback |
| `timeout` | Authentication timed out; offer retry if appropriate |
| `user_fallback` | User selected the fallback option |
| `passcode_not_set` | Required device passcode is not configured |
| `authentication_failed` | The attempt did not authenticate successfully |
| `system_cancel` | System interrupted/cancelled authentication |
| `no_space` | Required system space/resources unavailable |
| `unable_to_process` | The system could not process the request |
| `invalid_context` | Authentication context is invalid |
| `unknown` | Unspecified/unknown error |

A useful basic pattern:

```js
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: "Authenticate to continue",
});

if (result.success) {
  console.log("Authenticated");
} else {
  switch (result.error) {
    case "user_cancel":
      // User chose to cancel; keep protected content locked.
      break;
    case "not_enrolled":
      // Explain how to set up device authentication.
      break;
    case "lockout":
      // Do not repeatedly retry automatically.
      break;
    default:
      console.log("Authentication did not succeed:", result.error);
  }
}
```

### Failure vs cancellation

Both may have `success: false`, but they do not mean the same thing. A user cancellation is not necessarily a technical error. Avoid automatically retrying every error; some states require user action or waiting.

Use `try/catch` as well: a Promise can reject because an operation could not be started or another unexpected issue occurred.

## 11. Cancel an active authentication attempt

### `cancelAuthenticate()`

```js
LocalAuthentication.cancelAuthenticate();
```

- **Output:** `void` (no Promise result to await).
- The supplied material identifies this as **Android-specific**.
- It attempts to cancel an active authentication attempt; it does not start authentication.
- Check platform before calling it:

```js
import { Platform } from "react-native";

function cancelAuthentication() {
  if (Platform.OS === "android") {
    LocalAuthentication.cancelAuthenticate();
  }
}
```

## 12. iOS Face ID configuration

Face ID requires a native usage description. In an Expo project, configure it using the `expo-local-authentication` config plugin in `app.json` / `app.config.js`:

```json
{
  "expo": {
    "plugins": [
      [
        "expo-local-authentication",
        {
          "faceIDPermission": "Allow $(PRODUCT_NAME) to use Face ID."
        }
      ]
    ]
  }
}
```

`faceIDPermission` is a **build-time/native configuration value**, not an argument passed to `authenticateAsync()`.

After changing native config, rebuild the app/development build as required. The supplied material notes that Face ID cannot be fully tested in Expo Go; use a development build or appropriate native build for real testing.

Biometric authentication is not the same as asking for general camera permission. The OS handles the secure authentication flow.

## 13. Android permissions and platform differences

The supplied material lists these Android native permissions:

- `android.permission.USE_BIOMETRIC` — modern biometric permission.
- `android.permission.USE_FINGERPRINT` — older fingerprint-specific permission, deprecated on newer Android API levels.

In a normal Expo setup, you generally do not manually request these through `PermissionsAndroid.request()` just to use this module; the module's native configuration handles its requirements.

| Feature | Android | iOS |
|---|---|---|
| Main prompt message | Yes | Yes |
| `cancelLabel` | Yes | Yes |
| `disableDeviceFallback` | Supported option; behavior varies | Supported option; behavior varies |
| `fallbackLabel` | No | Yes |
| `promptDescription` | Yes | No |
| `promptSubtitle` | Yes | No |
| `requireConfirmation` | Yes | No |
| `biometricsSecurityLevel` | Yes | No |
| `cancelAuthenticate()` | Yes | No |
| Face ID usage description | Not applicable | Required for Face ID configuration |

Always test on the actual target platform. The native OS and device settings control the authentication UI and can affect behavior.

## 14. A safe practical flow

```text
User requests protected feature
        ↓
Check hardware (optional capability check)
        ↓
Check enrollment
        ↓
Call authenticateAsync(options)
        ↓
Inspect result.success
      /          \
   true           false
    ↓               ↓
Allow protected    Keep locked;
action             handle error/cancel
```

Do not rely on hardware checks alone to protect data. The actual protected operation must only proceed after successful authentication.

## 15. Complete React Native example

This example checks hardware and enrollment, starts authentication, handles the result, and resets loading state. It does not replace your app's backend login or server-side authorization.

```jsx
import { useState } from "react";
import {
  View,
  Text,
  Button,
  Alert,
  ActivityIndicator,
} from "react-native";
import * as LocalAuthentication from "expo-local-authentication";

export default function App() {
  const [message, setMessage] = useState("Not authenticated");
  const [loading, setLoading] = useState(false);

  const authenticateUser = async () => {
    try {
      setLoading(true);

      const hasHardware =
        await LocalAuthentication.hasHardwareAsync();

      if (!hasHardware) {
        setMessage("Biometric hardware is not available");
        Alert.alert("Not available", "This device has no biometric hardware.");
        return;
      }

      const enrolled =
        await LocalAuthentication.isEnrolledAsync();

      if (!enrolled) {
        setMessage("No biometric credentials enrolled");
        Alert.alert(
          "Not enrolled",
          "Set up a fingerprint, Face ID, or another supported credential in device settings."
        );
        return;
      }

      const result =
        await LocalAuthentication.authenticateAsync({
          promptMessage: "Authenticate to continue",
          cancelLabel: "Cancel",
        });

      if (result.success) {
        setMessage("Authentication successful");
        Alert.alert("Success", "Authentication successful.");
        // Perform the protected action here.
      } else {
        setMessage(`Authentication did not succeed: ${result.error}`);
        // Keep private content/actions locked.
      }
    } catch (error) {
      console.error("Authentication error:", error);
      setMessage("Something went wrong");
      Alert.alert("Error", "Unable to start authentication.");
    } finally {
      setLoading(false);
    }
  };

  return (
    <View
      style={{
        flex: 1,
        justifyContent: "center",
        alignItems: "center",
        padding: 20,
      }}
    >
      <Text style={{ fontSize: 24, marginBottom: 20 }}>
        Local Authentication
      </Text>

      <Text style={{ fontSize: 16, marginBottom: 24, textAlign: "center" }}>
        {message}
      </Text>

      {loading ? (
        <ActivityIndicator size="large" />
      ) : (
        <Button title="Authenticate" onPress={authenticateUser} />
      )}
    </View>
  );
}
```

### What the UI props mean

These are React Native props, not `expo-local-authentication` options:

| Prop / API | Meaning |
|---|---|
| `title` on `Button` | Button label |
| `onPress` | Function called when the button is pressed |
| `style` | Visual layout/appearance |
| `useState` | Stores UI state such as message/loading |
| `ActivityIndicator size="large"` | Shows a loading indicator |
| `Alert.alert(title, message)` | Shows a native alert |

## 16. Reusable authentication utility

If multiple screens need authentication, keep the common checks in one utility and let each screen decide what to show or where to navigate.

```js
import * as LocalAuthentication from "expo-local-authentication";

export async function authenticateUser() {
  try {
    const hasHardware =
      await LocalAuthentication.hasHardwareAsync();

    if (!hasHardware) {
      return { success: false, error: "hardware_unavailable" };
    }

    const enrolled =
      await LocalAuthentication.isEnrolledAsync();

    if (!enrolled) {
      return { success: false, error: "not_enrolled" };
    }

    return await LocalAuthentication.authenticateAsync({
      promptMessage: "Authenticate to continue",
    });
  } catch (error) {
    console.error("Local authentication error:", error);
    return { success: false, error: "unknown" };
  }
}
```

Usage:

```js
const result = await authenticateUser();

if (result.success) {
  // Continue with the protected action.
} else {
  // Keep the feature locked and handle result.error.
}
```

The utility returns a result; it should not automatically navigate or display UI unless that behavior is intentionally designed.

## 17. Common mistakes to avoid

1. Treating `hasHardwareAsync() === true` as proof that a biometric is enrolled.
2. Treating supported authentication types as proof of successful authentication.
3. Calling `authenticateAsync()` and allowing access without checking `result.success`.
4. Assuming local authentication creates a backend session or replaces account login.
5. Treating `result.error` as a user-friendly sentence.
6. Automatically retrying every failure, including lockout or cancellation.
7. Assuming all options behave identically on Android and iOS.
8. Assuming `cancelAuthenticate()` works on iOS; the supplied material describes it as Android-only.
9. Forgetting Face ID native configuration and rebuilding after config changes.
10. Storing raw biometric data yourself; the app receives the authentication result, not raw biometrics.
11. Using local authentication as the only protection for server-side actions. Sensitive server operations still need server-side authorization.

## 18. Method quick reference: inputs and outputs

| Method | Input | Output / effect |
|---|---|---|
| `hasHardwareAsync()` | None | `Promise<boolean>` |
| `isEnrolledAsync()` | None | `Promise<boolean>` |
| `supportedAuthenticationTypesAsync()` | None | `Promise<AuthenticationType[]>` |
| `getEnrolledLevelAsync()` | None | `Promise<SecurityLevel>` |
| `authenticateAsync(options?)` | Optional options object | `Promise<LocalAuthenticationResult>`; check `success`, then `error` if unsuccessful |
| `cancelAuthenticate()` | None | `void`; attempts to cancel active Android authentication |

## 19. Final revision checklist

- **Hardware:** `hasHardwareAsync()`
- **Enrollment:** `isEnrolledAsync()`
- **Supported types:** `supportedAuthenticationTypesAsync()`
- **Security level:** `getEnrolledLevelAsync()`
- **Start authentication:** `authenticateAsync(options)`
- **Success gate:** `if (result.success) { ... }`
- **Failure reason:** `result.error`
- **Cancel active Android prompt:** `cancelAuthenticate()`
- **Face ID native setup:** `faceIDPermission` config plugin value
- **Security rule:** never unlock the protected action unless authentication succeeds.


---

## 20. Extra details recovered from the original notes

These useful points appeared in the original material but were not clearly covered in the cleaned notes.

### A. `result.warning` is optional

The authentication result may include an optional `warning` property.

```js
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: "Authenticate to continue",
});

console.log(result.warning);
```

Remember:

- `result.success` is the primary value to check before allowing a protected action.
- `result.warning` is additional information when the API provides it.
- Do not treat the presence of a warning, by itself, as proof that authentication failed.
- Check the result shape supported by your installed Expo SDK.

### B. Convert error codes into user-friendly messages

`result.error` is an identifier for code, not ideal text to show directly to users. A helper can translate common errors:

```js
function getAuthenticationMessage(error) {
  switch (error) {
    case "user_cancel":
      return "Authentication cancelled.";
    case "authentication_failed":
      return "Authentication failed. Please try again.";
    case "not_enrolled":
      return "Biometric authentication is not set up on this device.";
    case "lockout":
      return "Authentication is temporarily locked. Follow your device's instructions.";
    case "timeout":
      return "Authentication timed out. Please try again.";
    case "passcode_not_set":
      return "A device passcode is not configured.";
    case "not_available":
      return "Local authentication is currently unavailable.";
    default:
      return "Authentication could not be completed.";
  }
}
```

Example use:

```js
const result = await LocalAuthentication.authenticateAsync({
  promptMessage: "Authenticate to continue",
});

if (result.success) {
  // Continue with the protected action.
} else if (result.error !== "user_cancel") {
  const message = getAuthenticationMessage(result.error);
  Alert.alert("Authentication", message);
}
```

A cancellation often does not need a second alert because the user deliberately cancelled. Adjust this behavior to fit your app.

### C. Keep the authentication utility separate from UI and navigation

A reusable utility should return a result; the calling screen should decide whether to show an alert, navigate, or continue.

```js
const result = await authenticateUser();

if (result.success) {
  // Screen decides what protected action to perform.
} else {
  // Screen decides how to handle the failure.
}
```

This keeps authentication logic reusable and avoids coupling it to one screen's UI.

### D. Distinguish module errors from errors created by your own utility

The utility example can return a custom error such as:

```js
{ success: false, error: "hardware_unavailable" }
```

`hardware_unavailable` is a label created by your application logic; it is not necessarily an error identifier returned by `authenticateAsync()`. Keep these two categories distinct when handling errors.

For example:

```js
switch (result.error) {
  case "hardware_unavailable":
    // Custom result from your pre-check.
    break;
  case "not_enrolled":
    // May come from a pre-check or authentication result.
    break;
  case "user_cancel":
    // Returned by the authentication flow where applicable.
    break;
  default:
    // Handle other cases.
}
```

### E. Android legacy security-level note

The original material notes a legacy Android detail: on Android versions before Android M, `SecurityLevel.SECRET` may be returned in a scenario involving SIM-lock enrollment. Do **not** interpret this as meaning that `authenticateAsync()` will display a SIM-lock prompt for your app. Security-level reporting and the authentication prompt are separate concepts. This is a platform/version-specific note; check the documentation for the Expo SDK and Android versions you support.

### F. Default options vs explicit options

Both calls request native authentication:

```js
await LocalAuthentication.authenticateAsync();
```

```js
await LocalAuthentication.authenticateAsync({
  promptMessage: "Authenticate to open private notes",
});
```

The second supplies an explicit prompt message. Options configure the request; they do not prove that hardware exists, that a credential is enrolled, or that authentication succeeded. Always inspect `result.success`.

---

## 21. Final comparison audit

The original notes contained many repeated explanations and examples. The cleaned notes already retained the core concepts, all six main APIs, their return types, authentication options, enum values, error identifiers, Android/iOS differences, Face ID configuration, security guidance, and practical React Native examples.

The additional sections above restore the notable useful details that were less explicit or missing:

- Optional `result.warning` property.
- A reusable helper for converting error identifiers into user-facing messages.
- Why user cancellation often should not trigger another alert.
- Separation between authentication utility logic and UI/navigation.
- Clarification that `hardware_unavailable` is a custom utility error label, not necessarily a native module error.
- Legacy Android `SECRET` / SIM-lock caveat.
- Difference between default authentication options and explicit options.

**Revision rule:** `result.success === true` is the gate for protected actions. Capability checks and configuration help prepare the flow, but they are not proof of successful authentication.
