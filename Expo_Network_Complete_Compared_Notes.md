# Expo Network (`expo-network`) — Clean & Complete Notes

> Easy revision notes for React Native + Expo. Covers the important concepts, all main APIs, return values, network-state properties, enum values, platform differences, listener cleanup, and practical examples.
>
> **Version note:** Platform support and API behavior can vary by Expo SDK, OS, browser, and device. Check the documentation matching your installed Expo SDK before shipping.

## 1. What is `expo-network`?

`expo-network` is an Expo module that provides information about the device's network connection.

You can use it to check:
- Whether an active network connection exists.
- Whether internet connectivity is reported as reachable.
- The current network type, such as Wi-Fi or cellular.
- The device's current IP address.
- Whether airplane mode is enabled on supported platforms.
- When network state changes.

It **does not make API requests for you**. For HTTP requests, you still use `fetch`, Axios, or another networking client.

### Common use cases
- Show an online/offline banner.
- Display Wi-Fi/mobile-data status.
- Decide whether to attempt a network-dependent operation.
- React to network changes and resume synchronization.
- Show useful connectivity messages and debug network issues.

## 2. Install and import

```bash
npx expo install expo-network
```

```js
import * as Network from "expo-network";
```

Expo's installer selects a package version compatible with the project's SDK. The module's Android network/Wi-Fi state permissions are normally handled by Expo; you generally do not need to request them manually.

## 3. The essential distinction: network vs internet vs request success

These are different questions:

| Concept | Meaning |
|---|---|
| `isConnected` | Does the device report an active network connection? |
| `isInternetReachable` | Does the platform report internet reachability? |
| `type` | What type of network is active? |
| API request result | Did this particular server request succeed? |

A device can be connected to Wi-Fi while the router has no internet. Even if internet is reachable, a specific request can still fail because the server is down, DNS fails, the request times out, credentials are invalid, or the server returns an HTTP error.

**Network state is a useful signal, not a guarantee that every API request will succeed.** Always handle errors from your actual `fetch`/Axios request too.

## 4. Main APIs — quick reference

| API | Input | Return/output | Use it for |
|---|---|---|---|
| `useNetworkState()` | None | Current `NetworkState`; React hook updates as network changes | Reactive UI |
| `getNetworkStateAsync()` | None | `Promise<NetworkState>` | One-time snapshot |
| `getIpAddressAsync()` | None | `Promise<string>` | Read current IP address |
| `isAirplaneModeEnabledAsync()` | None | `Promise<boolean>` | Check airplane mode where supported |
| `addNetworkStateListener(listener)` | Callback receiving network state | Subscription object with `.remove()` | Run custom logic when state changes |

Most APIs in this table are asynchronous, so use `await` inside an `async` function. `useNetworkState()` is a React hook, not a Promise-returning snapshot method.

## 5. Understand the `NetworkState` object

`useNetworkState()`, `getNetworkStateAsync()`, and the network-state listener provide network-state information.

Important properties:

| Property | Meaning | Typical type |
|---|---|---|
| `type` | Active network type | `NetworkStateType` |
| `isConnected` | Whether an active network connection exists; Expo docs describe it as false for `NONE` or `UNKNOWN` and true otherwise | `boolean` |
| `isInternetReachable` | Whether internet reachability is reported | `boolean` |

Example:

```js
const state = await Network.getNetworkStateAsync();
console.log(state);
```

Example output (values vary by device):

```js
{
  type: "WIFI",
  isConnected: true,
  isInternetReachable: true
}
```

Read individual values:

```js
console.log(state.type);                 // e.g. "WIFI"
console.log(state.isConnected);          // e.g. true
console.log(state.isInternetReachable);  // e.g. true
```

For UI, render booleans as strings because React Native text children should be text:

```jsx
<Text>Connected: {String(state.isConnected)}</Text>
<Text>Internet reachable: {String(state.isInternetReachable)}</Text>
<Text>Type: {state.type ?? "Unknown"}</Text>
```

## 6. API 1 — `useNetworkState()`

### What it does
A React hook that returns current network state and subscribes to network changes, updating the component as the state changes. Expo manages the hook's subscription lifecycle.

### Example

```jsx
import { View, Text } from "react-native";
import * as Network from "expo-network";

export default function NetworkStatus() {
  const state = Network.useNetworkState();

  return (
    <View>
      <Text>Type: {state.type ?? "Unknown"}</Text>
      <Text>Connected: {String(state.isConnected)}</Text>
      <Text>Internet reachable: {String(state.isInternetReachable)}</Text>
    </View>
  );
}
```

### Output
The UI shows the current network type and the connected/reachable values. These values update when network state changes.

### Simple online/offline display

```js
const online =
  state.isConnected === true &&
  state.isInternetReachable === true;
```

```jsx
<Text>
  {online ? "You are online" : "You may be offline"}
</Text>
```

For more informative status text:

```js
let message = "Checking network...";

if (
  state.isConnected === true &&
  state.isInternetReachable === true
) {
  message = "Internet reachable";
} else if (state.isConnected === true) {
  message = "Connected, but internet may be unavailable";
} else if (state.isConnected === false) {
  message = "No active network connection";
}
```

**Use this hook when the React UI should respond automatically to network changes.** Call it at the top level of a React component or custom hook, following the Rules of Hooks.

## 7. API 2 — `getNetworkStateAsync()`

### What it does
Gets a one-time snapshot of the current network state.

**Return:** `Promise<NetworkState>`

```js
const state = await Network.getNetworkStateAsync();
console.log(state);
```

Example output:

```js
{
  type: "CELLULAR",
  isConnected: true,
  isInternetReachable: true
}
```

The example is illustrative; actual values depend on the current connection.

### When to use it
Use it when you need a snapshot at a particular moment, such as after a button press. Use `useNetworkState()` when the UI needs to keep updating automatically.

## 8. API 3 — `getIpAddressAsync()`

### What it does
Returns the current IP address as a string.

**Return:** `Promise<string>`

```js
const ip = await Network.getIpAddressAsync();
console.log(ip);
```

Example output:

```text
192.168.1.20
```

If the IP address cannot be retrieved, the supplied material documents `"0.0.0.0"` as the fallback value.

```js
const ip = await Network.getIpAddressAsync();

if (ip === "0.0.0.0") {
  console.log("Could not retrieve a useful IP address");
} else {
  console.log("IP address:", ip);
}
```

### Platform note
The supplied material states that on web this method uses the third-party `ipify` service to obtain the public IP address. Native and web behavior can therefore differ. An IP address does not prove that a particular API is reachable.

## 9. API 4 — `isAirplaneModeEnabledAsync()`

### What it does
Checks whether airplane mode is enabled on supported platforms.

**Return:** `Promise<boolean>`

```js
const enabled = await Network.isAirplaneModeEnabledAsync();
console.log(enabled);
```

Output:

```text
true
```

or

```text
false
```

- `true`: airplane mode is enabled.
- `false`: airplane mode is not enabled.

The supplied material marks this method as **Android-only**. Check platform support before depending on it in cross-platform code.

## 10. API 5 — `addNetworkStateListener()`

### What it does
Registers a callback that runs when network state changes.

```js
const subscription = Network.addNetworkStateListener((state) => {
  console.log("Type:", state.type);
  console.log("Connected:", state.isConnected);
  console.log("Internet:", state.isInternetReachable);
});
```

The callback receives the updated network-state object.

### Return value and cleanup
The method returns a subscription object. Call `.remove()` when you no longer need the listener:

```js
subscription.remove();
```

### Correct use in React

```jsx
import { useEffect } from "react";
import * as Network from "expo-network";

export default function NetworkLogger() {
  useEffect(() => {
    const subscription = Network.addNetworkStateListener((state) => {
      console.log("Network changed:", state);
    });

    return () => {
      subscription.remove();
    };
  }, []);

  return null;
}
```

**Why cleanup matters:** without removing a listener, old subscriptions may remain active longer than intended and can cause duplicate callbacks or unnecessary work.

### Listener vs hook

| `useNetworkState()` | `addNetworkStateListener()` |
|---|---|
| Best for reactive React UI | Best for custom callback logic |
| Expo manages hook subscription lifecycle | You must call `subscription.remove()` |
| Returns state to render | Calls your callback when state changes |

Use the hook for an online/offline banner. Use a listener when you need custom work such as logging or triggering a synchronization attempt.

## 11. `NetworkStateEvent` — listener callback data

`NetworkStateEvent` is the network-state information delivered to the callback when a network change is reported. In Expo's documented typing, it corresponds to `NetworkState` rather than a separate object with different fields.

```js
const subscription = Network.addNetworkStateListener((event) => {
  console.log(event.type);
  console.log(event.isConnected);
  console.log(event.isInternetReachable);
});

// Remove the listener when it is no longer needed:
subscription.remove();
```

You can also destructure the fields:

```js
Network.addNetworkStateListener(({ type, isConnected }) => {
  console.log(type, isConnected);
});
```

**Remember:** `NetworkState` describes the current state; `NetworkStateEvent` is that state delivered through a change notification. The fields are the same.

## 12. Extra `NetworkState` details

- `isConnected` is a boolean describing whether an active network connection is reported. Expo documentation notes that it is `false` when the type is `NONE` or `UNKNOWN`, and `true` otherwise; verify this against the installed SDK if your logic depends on it.
- On Android, internet reachability is based on platform network capabilities and validation rather than merely detecting a Wi-Fi or cellular connection. VPN connections may have additional usability checks. In normal app code, use the returned value instead of trying to reproduce these platform checks.
- Some values may be unavailable or platform-limited. Handle unknown values safely, for example `state.type ?? "Unknown"`.

## 13. Network type availability by platform

The original reference lists the following platform availability. Treat it as version-sensitive: check the documentation for your installed Expo SDK before relying on a particular type.

| Network type | Android | iOS | tvOS | Web |
|---|---|---|---|---|
| `BLUETOOTH` | Yes | No | No | No |
| `CELLULAR` | Yes | Yes | No | No |
| `ETHERNET` | Yes | Yes | No | No |
| `NONE` | Yes | Yes | Yes | Yes |
| `OTHER` | Yes | No | No | No |
| `UNKNOWN` | Yes | Yes | Yes | Yes |
| `VPN` | Yes | No | No | No |
| `WIFI` | Yes | Yes | No | No |
| `WIMAX` | Yes | No | No | No |

`UNKNOWN` is a valid result, especially on Web, where browsers do not expose detailed network types consistently. Do not assume every platform can identify Wi-Fi or cellular precisely.

## 14. Practical design notes: offline, retry, sync

`expo-network` helps your app decide when to show a message or attempt an operation, but it does not implement offline storage, retries, or synchronization for you.

- **Offline banner:** use `useNetworkState()` to update the UI when network state changes.
- **Retry:** after a failed `fetch`/Axios request, handle the actual request error; network state alone is not proof that retry will work.
- **Offline-first:** show previously cached data when available and queue changes only if your app has implemented storage/queue logic.
- **Synchronization:** a listener can trigger a sync attempt when connectivity changes, but make the sync safe to retry and avoid duplicate operations.
- **Large downloads:** network type can inform a decision, but detailed type reporting differs across platforms.

## 15. `NetworkStateType` values

The supplied material lists these network-type constants:

| Constant | String value | Meaning / note |
|---|---|---|
| `BLUETOOTH` | `"BLUETOOTH"` | Bluetooth network; platform support is limited |
| `CELLULAR` | `"CELLULAR"` | Mobile/cellular network |
| `ETHERNET` | `"ETHERNET"` | Ethernet connection |
| `NONE` | `"NONE"` | No active network connection |
| `OTHER` | `"OTHER"` | Another network type |
| `UNKNOWN` | `"UNKNOWN"` | Type cannot be determined |
| `VPN` | `"VPN"` | VPN connection; platform support is limited |
| `WIFI` | `"WIFI"` | Wi-Fi connection |
| `WIMAX` | `"WIMAX"` | WiMAX connection; platform support is limited |

Prefer the exported constants when comparing values:

```js
if (state.type === Network.NetworkStateType.WIFI) {
  console.log("Using Wi-Fi");
}
```

A switch example:

```js
switch (state.type) {
  case Network.NetworkStateType.WIFI:
    console.log("Wi-Fi");
    break;
  case Network.NetworkStateType.CELLULAR:
    console.log("Mobile data");
    break;
  case Network.NetworkStateType.NONE:
    console.log("No active network");
    break;
  default:
    console.log("Other or unknown network type");
}
```

Not every network type is available on every platform. Treat `"UNKNOWN"` as a valid value rather than assuming a detailed type is always reported.

## 16. Platform differences to remember

The supplied material highlights these caveats:

- **Android:** network and Wi-Fi state permissions are handled automatically in a normal Expo setup.
- **Android:** `isAirplaneModeEnabledAsync()` is marked Android-only.
- **iOS:** the supplied material states `isInternetReachable` is always the same as `isConnected`; do not assume these values can be distinguished on iOS.
- **Web:** detailed network type reporting is limited; `getNetworkStateAsync()` may report `UNKNOWN` for an active connection and `NONE` when no active connection is detected.
- **Web:** `getIpAddressAsync()` uses the third-party `ipify` service according to the supplied material.
- **Network type constants:** support differs by platform, so verify the platform table for your Expo SDK if your logic depends on a particular type.

## 17. Complete practical example — Network Monitor

This example displays reactive network state and lets the user refresh the IP address. Airplane-mode checking is omitted from the universal UI because the supplied material marks that method Android-only.

```jsx
import { useState } from "react";
import { View, Text, Button, StyleSheet } from "react-native";
import * as Network from "expo-network";

export default function NetworkMonitor() {
  const state = Network.useNetworkState();
  const [ipAddress, setIpAddress] = useState("");
  const [ipError, setIpError] = useState("");

  const loadIpAddress = async () => {
    try {
      setIpError("");
      const ip = await Network.getIpAddressAsync();
      setIpAddress(ip);
    } catch (error) {
      console.error("IP lookup failed:", error);
      setIpError("Could not retrieve the IP address.");
    }
  };

  let status = "Network status unknown";

  if (
    state.isConnected === true &&
    state.isInternetReachable === true
  ) {
    status = "Internet reachable";
  } else if (state.isConnected === true) {
    status = "Connected, but internet may be unavailable";
  } else if (state.isConnected === false) {
    status = "No active network connection";
  }

  return (
    <View style={styles.container}>
      <Text style={styles.title}>Network Monitor</Text>

      <Text>Overall status: {status}</Text>
      <Text>Type: {state.type ?? "Unknown"}</Text>
      <Text>Connected: {String(state.isConnected)}</Text>
      <Text>Internet reachable: {String(state.isInternetReachable)}</Text>
      <Text>IP address: {ipAddress || "Not loaded"}</Text>

      {ipError ? <Text>{ipError}</Text> : null}

      <Button title="Get / Refresh IP" onPress={loadIpAddress} />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: "center",
    gap: 12,
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: "bold",
  },
});
```

### What this example demonstrates
- `useNetworkState()` keeps displayed network values up to date.
- `getIpAddressAsync()` reads the IP address when the button is pressed.
- `useState()` stores the IP and any UI error message.
- `try/catch` handles a failed IP lookup.
- Network state and the success of a real API request are separate concerns.

## 18. Common mistakes to avoid

1. Assuming Wi-Fi connection always means internet access.
2. Treating `isConnected === true` as proof that every API request will succeed.
3. Using `getNetworkStateAsync()` when you actually need UI to react continuously to changes.
4. Calling `useNetworkState()` outside a React component or custom hook.
5. Forgetting to remove an `addNetworkStateListener()` subscription.
6. Assuming every platform reports detailed network types.
7. Assuming `getIpAddressAsync()` always returns a useful IP; `"0.0.0.0"` can indicate retrieval failure in the supplied material.
8. Using `isAirplaneModeEnabledAsync()` without checking platform support.
9. Treating an IP address as proof that a server or API is reachable.
10. Forgetting to handle errors from the actual `fetch`/Axios request.
11. Assuming `expo-network` automatically caches data, retries failed requests, or synchronizes changes; these need separate app logic.

## 19. Final revision checklist

- **Reactive network UI:** `useNetworkState()`
- **One-time network snapshot:** `getNetworkStateAsync()`
- **IP address:** `getIpAddressAsync()`
- **Airplane mode:** `isAirplaneModeEnabledAsync()` (Android-only according to the supplied material)
- **Listen for changes:** `addNetworkStateListener(callback)`
- **Remove a listener:** `subscription.remove()`
- **State fields:** `type`, `isConnected`, `isInternetReachable`
- **Network types:** `BLUETOOTH`, `CELLULAR`, `ETHERNET`, `NONE`, `OTHER`, `UNKNOWN`, `VPN`, `WIFI`, `WIMAX`
- **Key distinction:** connected to a network ≠ internet reachable ≠ API request succeeded.
