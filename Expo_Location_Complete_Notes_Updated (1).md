# Expo Location — Complete Beginner-Friendly Notes

> These notes keep the important concepts and methods from the original material while removing repeated definitions, duplicate code samples, and repeated diagrams. Each major method includes its purpose, syntax, an example, and expected output where applicable.

## Contents
1. What is `expo-location`?
2. Install and import
3. Permissions: ask before accessing location
4. Background permission — `requestBackgroundPermissionsAsync()`
5. Get current location — `getCurrentPositionAsync()`
6. Get last known location — `getLastKnownPositionAsync()`
7. Live tracking — `watchPositionAsync()`
8. Location accuracy options
9. Reverse geocoding — coordinates to address (includes building a full address)
10. Geocoding — address to coordinates
11. Device heading / compass (includes mini compass and calibration)
12. Check Location Services
13. Understand the Location Object
14. Common mistakes
15. Quick revision method cheat sheet

---

## 1. What is `expo-location`?

`expo-location` is an Expo SDK module that lets a React Native app request and work with a device's location.

Common uses:
- Show the user's current location on a map.
- Find a nearby address or place.
- Track movement while the app is in use.
- Convert coordinates into an address and an address into coordinates.
- Read compass heading.
- Check whether Location Services are enabled.

### How can a phone estimate location?

| Source | Simple explanation | Typical trade-off |
|---|---|---|
| GPS / satellites | Estimates position using satellite signals | Often accurate outdoors; can be slower indoors and use more battery |
| Wi-Fi | Uses nearby Wi-Fi network information | Can help indoors; accuracy varies |
| Cellular network | Estimates position using nearby cell towers | Often less precise but can use less power |

The operating system chooses available location sources based on the device and requested accuracy. Do not assume every request uses GPS alone.

### Latitude and longitude

- **Latitude**: north/south position relative to the Equator; range `-90` to `+90`.
- **Longitude**: east/west position relative to the Prime Meridian; range `-180` to `+180`.
- Together, they identify a geographic position.

Example (New Delhi):
```text
Latitude : 28.6139
Longitude: 77.2090
```

### What does accuracy mean?

`accuracy` is an estimated horizontal uncertainty in meters. A smaller value generally means a more precise estimate.

```text
accuracy = 5    // estimated uncertainty about 5 meters
accuracy = 100  // less precise than 5 meters
```

It is an estimate, not a guarantee that the reported point is exactly that many meters from the true position.

### Foreground vs background location

- **Foreground location**: access while the app is in use.
- **Background location**: updates while the app is in the background, subject to extra platform permissions and configuration.

Use background location only when the app genuinely needs it, such as an active fitness or delivery-tracking feature. This guide focuses on foreground access and `watchPositionAsync()` while the app is running.

---

## 2. Install and import

Install the version compatible with your Expo SDK:

```bash
npx expo install expo-location
```

Import it:

```js
import * as Location from "expo-location";
```

The `Location` namespace contains methods such as `requestForegroundPermissionsAsync`, `getCurrentPositionAsync`, `watchPositionAsync`, `reverseGeocodeAsync`, `geocodeAsync`, and `getHeadingAsync`.

---

## 3. Permissions: ask before accessing location

A package installation does **not** automatically grant location access. The user must grant permission.

### Main permission method

```js
const permission = await Location.requestForegroundPermissionsAsync();
console.log(permission);
```

Typical result (exact fields can vary by platform/version):

```js
{
  status: "granted",
  granted: true,
  canAskAgain: true,
  expires: "never"
}
```

### Permission fields

| Field | Meaning |
|---|---|
| `status` | Usually `granted`, `denied`, or `undetermined` |
| `granted` | Boolean: whether access is currently granted |
| `canAskAgain` | Whether the app can ask again through the permission prompt |
| `expires` | Indicates permission expiry behavior; commonly `"never"` |

- `granted`: user allowed access.
- `denied`: access was refused.
- `undetermined`: the user has not made a decision yet.
- If permission cannot be requested again, guide the user to the device's Settings.

### Minimal permission example

```jsx
import { Alert, Button, View } from "react-native";
import * as Location from "expo-location";

export default function App() {
  const requestPermission = async () => {
    try {
      const permission =
        await Location.requestForegroundPermissionsAsync();

      if (permission.granted) {
        Alert.alert("Success", "Location permission granted");
      } else {
        Alert.alert(
          "Permission denied",
          "Allow location access in Settings to use this feature."
        );
      }
    } catch (error) {
      console.error("Permission error:", error);
      Alert.alert("Error", "Could not request location permission.");
    }
  };

  return (
    <View>
      <Button title="Request Location Permission" onPress={requestPermission} />
    </View>
  );
}
```

Expected behavior:
- If allowed: `Success — Location permission granted`.
- If denied: `Permission denied` alert.
- If the request throws an error: error alert and error in the console.

**Important:** In real apps, check permission before requesting a location. Do not repeatedly show a permission popup without a user action or a clear reason.

---


---

## 4. Background permission — `requestBackgroundPermissionsAsync()`

### What it does

Requests permission for the app to access location while it is running in the background, subject to Android/iOS rules and app configuration.

```js
const permission = await Location.requestBackgroundPermissionsAsync();
console.log(permission);
```

Example response when permission is granted (the exact fields and behavior can vary by platform/version):

```js
{
  status: "granted",
  granted: true,
  canAskAgain: true,
  expires: "never"
}
```

If permission is denied, a typical response includes:

```js
{
  status: "denied",
  granted: false,
  canAskAgain: true,
  expires: "never"
}
```

### Safe example

```js
import { Alert } from "react-native";
import * as Location from "expo-location";

async function requestBackgroundPermission() {
  try {
    // Ask for foreground permission first.
    const foreground = await Location.requestForegroundPermissionsAsync();

    if (!foreground.granted) {
      Alert.alert("Permission required", "Allow foreground location first.");
      return;
    }

    const background = await Location.requestBackgroundPermissionsAsync();

    if (background.granted) {
      console.log("Background location permission granted");
    } else {
      console.log("Background location permission denied");
      if (!background.canAskAgain) {
        Alert.alert(
          "Permission needed in Settings",
          "Enable background location permission in your device settings."
        );
      }
    }
  } catch (error) {
    console.error("Background permission error:", error);
  }
}
```

Expected console output:

```text
Background location permission granted
```

or, if permission is refused:

```text
Background location permission denied
```

### Important distinction

- `requestForegroundPermissionsAsync()` asks for access while the app is in use.
- `requestBackgroundPermissionsAsync()` asks for background access.
- **Requesting background permission does not, by itself, start background tracking.** Background tracking needs additional setup and a background location task, and platform rules/configuration apply. For ordinary foreground tracking, use `watchPositionAsync()` and stop its subscription when finished.

Only request background access when the app genuinely needs it and clearly explain why to the user.


## 5. Get current location — `getCurrentPositionAsync()`

### What it does

Gets one location reading for the device. It returns a Promise, so use `await` inside an `async` function.

```js
const location = await Location.getCurrentPositionAsync();
console.log(location);
```

You can request an accuracy level:

```js
const location = await Location.getCurrentPositionAsync({
  accuracy: Location.Accuracy.High,
});
```

### Complete example with output

```jsx
import { useState } from "react";
import { Alert, Button, Text, View } from "react-native";
import * as Location from "expo-location";

export default function App() {
  const [location, setLocation] = useState(null);

  const getLocation = async () => {
    try {
      const permission =
        await Location.requestForegroundPermissionsAsync();

      if (!permission.granted) {
        Alert.alert("Permission required", "Please allow location access.");
        return;
      }

      const current = await Location.getCurrentPositionAsync({
        accuracy: Location.Accuracy.Balanced,
      });

      setLocation(current);
    } catch (error) {
      console.error("Location error:", error);
      Alert.alert("Error", "Could not get the current location.");
    }
  };

  return (
    <View style={{ padding: 20, gap: 10 }}>
      <Button title="Get Current Location" onPress={getLocation} />
      {location && (
        <>
          <Text>Latitude: {location.coords.latitude}</Text>
          <Text>Longitude: {location.coords.longitude}</Text>
          <Text>Accuracy: {location.coords.accuracy} m</Text>
        </>
      )}
    </View>
  );
}
```

Example output (your values will differ):
```text
Latitude: 28.6139
Longitude: 77.2090
Accuracy: 8 m
```

### Important result fields

The result usually has this shape:

```js
{
  coords: {
    latitude: 28.6139,
    longitude: 77.2090,
    altitude: 215,
    accuracy: 8,
    altitudeAccuracy: 5,
    heading: 90,
    speed: 1.4
  },
  timestamp: 1754037892000
}
```

Values may be `null`, may vary by device, and the example numbers are illustrative.

---

## 6. Get last known location — `getLastKnownPositionAsync()`

### What it does

Returns the most recently available cached location, if one exists. It can be faster than obtaining a fresh location, but it may be old or unavailable.

```js
const lastLocation = await Location.getLastKnownPositionAsync();
console.log(lastLocation);
```

Possible output:
```js
{
  coords: {
    latitude: 28.6139,
    longitude: 77.2090,
    accuracy: 15
  },
  timestamp: 1754037892000
}
```

If no cached location is available:
```js
null
```

### Safe example: cached location, then fresh location if needed

```js
const permission = await Location.requestForegroundPermissionsAsync();

if (!permission.granted) {
  console.log("Permission denied");
  return;
}

let location = await Location.getLastKnownPositionAsync();

if (location === null) {
  console.log("No cached location; requesting a fresh location...");
  location = await Location.getCurrentPositionAsync({
    accuracy: Location.Accuracy.Balanced,
  });
}

console.log("Location:", location);
```

Wrap the above in `try...catch` in production code.

### Current vs last known

| Current location | Last known location |
|---|---|
| Requests a fresh reading | Uses an available cached reading |
| Can take longer | Often faster |
| Usually better for a fresh position | May be outdated or `null` |
| Use when freshness matters | Useful for showing an initial position quickly |

---

## 7. Live location tracking — `watchPositionAsync()`

### What it does

Starts listening for location updates. The callback runs whenever a new reading is available. It returns a **subscription**; remove that subscription when tracking stops.

```js
const subscription = await Location.watchPositionAsync(
  { accuracy: Location.Accuracy.High, distanceInterval: 10 },
  (location) => {
    console.log(location.coords.latitude, location.coords.longitude);
  }
);

// When finished:
subscription.remove();
```

### Important options

| Option | Meaning |
|---|---|
| `accuracy` | Desired location accuracy |
| `distanceInterval` | Minimum movement in meters before an update is reported |
| `timeInterval` | Minimum update interval in milliseconds; platform behavior varies (especially on iOS) |

Examples:
```js
{ distanceInterval: 10 } // roughly after 10 m of movement
{ timeInterval: 5000 }   // request a minimum interval of 5 seconds where supported
```

These options do not guarantee perfectly exact timing or distance; the operating system and device affect updates.

### Start/stop example

```jsx
import { useRef, useState } from "react";
import { Alert, Button, Text, View } from "react-native";
import * as Location from "expo-location";

export default function App() {
  const [location, setLocation] = useState(null);
  const subscriptionRef = useRef(null);

  const startTracking = async () => {
    try {
      if (subscriptionRef.current) return; // avoid duplicate subscriptions

      const permission =
        await Location.requestForegroundPermissionsAsync();

      if (!permission.granted) {
        Alert.alert("Permission required", "Allow location access first.");
        return;
      }

      subscriptionRef.current = await Location.watchPositionAsync(
        {
          accuracy: Location.Accuracy.High,
          distanceInterval: 5,
        },
        (nextLocation) => setLocation(nextLocation)
      );
    } catch (error) {
      console.error("Tracking error:", error);
      Alert.alert("Error", "Could not start tracking.");
    }
  };

  const stopTracking = () => {
    subscriptionRef.current?.remove();
    subscriptionRef.current = null;
  };

  return (
    <View style={{ padding: 20, gap: 10 }}>
      <Button title="Start Tracking" onPress={startTracking} />
      <Button title="Stop Tracking" onPress={stopTracking} />
      {location && (
        <>
          <Text>Latitude: {location.coords.latitude}</Text>
          <Text>Longitude: {location.coords.longitude}</Text>
          <Text>Speed: {location.coords.speed ?? "Unavailable"} m/s</Text>
        </>
      )}
    </View>
  );
}
```

Example screen output while moving:
```text
Latitude: 28.61390
Longitude: 77.20902
Speed: 1.4 m/s
```

When the device moves, later readings may show updated coordinates. Press **Stop Tracking** to call `subscription.remove()`.

**Why remove the subscription?** To stop receiving updates and avoid unnecessary location work and battery use. In a full app, also stop tracking when the relevant screen or activity ends.

### One-time location vs tracking

- `getCurrentPositionAsync()` → one reading.
- `watchPositionAsync()` → repeated updates until the subscription is removed.

---

## 8. Location accuracy — `Location.Accuracy`

Choose the lowest accuracy level that satisfies the app's needs.

| Option | General idea | Example use |
|---|---|---|
| `Location.Accuracy.Lowest` | Lowest requested accuracy | Coarse location |
| `Location.Accuracy.Low` | Low accuracy | Approximate area / simple weather use |
| `Location.Accuracy.Balanced` | Balance of accuracy and power | General app / many delivery use cases |
| `Location.Accuracy.High` | Higher accuracy | Pickup location / active tracking |
| `Location.Accuracy.Highest` | Very high requested accuracy | Precision-sensitive field work |
| `Location.Accuracy.BestForNavigation` | Intended for navigation use | Turn-by-turn navigation |

These are requested accuracy priorities, not guarantees. Actual precision depends on signal, device, environment, and OS.

Usage:
```js
const location = await Location.getCurrentPositionAsync({
  accuracy: Location.Accuracy.High,
});
```

For tracking:
```js
const subscription = await Location.watchPositionAsync(
  { accuracy: Location.Accuracy.High },
  (location) => console.log(location.coords)
);
```

Higher accuracy can increase battery use. A weather app usually does not need navigation-level accuracy.

---

## 9. Reverse geocoding — coordinates to address

### Method: `reverseGeocodeAsync()`

Converts latitude and longitude into a human-readable address.

```js
const addresses = await Location.reverseGeocodeAsync({
  latitude: 28.6139,
  longitude: 77.2090,
});

console.log(addresses);
```

The result is an **array** of address objects. Example (fields and values vary by provider/platform):
```js
[
  {
    name: "India Gate",
    street: "Rajpath",
    city: "New Delhi",
    district: "New Delhi",
    region: "Delhi",
    country: "India",
    isoCountryCode: "IN",
    postalCode: "110001"
  }
]
```

Read the first result only after checking that the array is not empty:
```js
const place = addresses[0];

if (place) {
  console.log(place.city);
  console.log(place.country);
}
```

### Common address fields

| Field | Meaning |
|---|---|
| `name` | Place or building name, if available |
| `street` | Street / road, if available |
| `city` | City or locality, if available |
| `district` | District, if available |
| `region` | State / region, if available |
| `country` | Country name |
| `isoCountryCode` | Country code such as `IN` |
| `postalCode` | PIN / postal code |

Not every field is guaranteed to exist. Build the displayed address from whichever fields are present.

### Complete flow

```js
const permission = await Location.requestForegroundPermissionsAsync();
if (!permission.granted) return;

const location = await Location.getCurrentPositionAsync();

const addresses = await Location.reverseGeocodeAsync({
  latitude: location.coords.latitude,
  longitude: location.coords.longitude,
});

const address = addresses[0];
console.log(address ?? "No address found");
```

Expected output:
```text
An address object, for example:
New Delhi, Delhi, India, 110001
```


### Build a full readable address

Address providers may omit some fields, so avoid blindly joining every property. Use `filter(Boolean)` to skip missing values.

```js
const fullAddress = [
  address.name,
  address.street,
  address.city,
  address.district,
  address.region,
  address.postalCode,
  address.country,
]
  .filter(Boolean)
  .join(", ");

console.log(fullAddress);
```

Example output (actual fields depend on the location and provider):

```text
India Gate, Rajpath, New Delhi, Delhi, 110001, India
```

---

## 10. Geocoding — address to coordinates

### Method: `geocodeAsync()`

Converts an address string into an array of coordinate results.

```js
const results = await Location.geocodeAsync("New Delhi");
console.log(results);
```

Example output (illustrative):
```js
[
  {
    latitude: 28.6139,
    longitude: 77.2090,
    altitude: 0,
    accuracy: 0
  }
]
```

Use the first result only if one exists:
```js
if (results.length > 0) {
  console.log(results[0].latitude);
  console.log(results[0].longitude);
} else {
  console.log("No coordinates found");
}
```

### Dynamic address search

```jsx
import { useState } from "react";
import { Button, Text, TextInput, View } from "react-native";
import * as Location from "expo-location";

export default function AddressSearch() {
  const [address, setAddress] = useState("");
  const [coordinates, setCoordinates] = useState(null);

  const search = async () => {
    try {
      if (!address.trim()) return;

      const results = await Location.geocodeAsync(address.trim());
      setCoordinates(results[0] ?? null);
    } catch (error) {
      console.error("Geocoding error:", error);
    }
  };

  return (
    <View style={{ padding: 20, gap: 10 }}>
      <TextInput
        placeholder="Enter a city or address"
        value={address}
        onChangeText={setAddress}
        style={{ borderWidth: 1, padding: 10 }}
      />
      <Button title="Find Coordinates" onPress={search} />
      {coordinates ? (
        <>
          <Text>Latitude: {coordinates.latitude}</Text>
          <Text>Longitude: {coordinates.longitude}</Text>
        </>
      ) : null}
    </View>
  );
}
```

If the user types `Mumbai`, the result may be coordinates for a matching location. Results depend on the geocoding provider and query; they are not guaranteed to be identical every time.

### Geocoding vs reverse geocoding

| Geocoding | Reverse geocoding |
|---|---|
| Address → coordinates | Coordinates → address |
| `Location.geocodeAsync(address)` | `Location.reverseGeocodeAsync({ latitude, longitude })` |
| Result array contains coordinate objects | Result array contains address objects |

---

## 11. Device heading / compass — `getHeadingAsync()`

### What is heading?

Heading tells the direction the device is pointing, measured in degrees. It is different from GPS location: your coordinates can remain the same while you rotate the phone.

| Degrees | Direction |
|---:|---|
| `0°` | North |
| `45°` | North-East |
| `90°` | East |
| `135°` | South-East |
| `180°` | South |
| `225°` | South-West |
| `270°` | West |
| `315°` | North-West |

### Method and output

```js
const heading = await Location.getHeadingAsync();
console.log(heading);
```

Example output:
```js
{
  trueHeading: 87.5,
  magHeading: 86.9,
  accuracy: 3
}
```

- `trueHeading`: direction relative to geographic true north, when available.
- `magHeading`: direction relative to magnetic north.
- `accuracy`: heading measurement's reported accuracy.

Values can vary; sensor availability and calibration matter.

### Display heading

```jsx
import { useState } from "react";
import { Button, Text, View } from "react-native";
import * as Location from "expo-location";

export default function CompassExample() {
  const [heading, setHeading] = useState(null);

  const readHeading = async () => {
    try {
      const result = await Location.getHeadingAsync();
      setHeading(result);
    } catch (error) {
      console.error("Heading error:", error);
    }
  };

  return (
    <View style={{ padding: 20, gap: 10 }}>
      <Button title="Get Heading" onPress={readHeading} />
      {heading && (
        <>
          <Text>True heading: {heading.trueHeading}°</Text>
          <Text>Magnetic heading: {heading.magHeading}°</Text>
          <Text>Accuracy: {heading.accuracy}</Text>
        </>
      )}
    </View>
  );
}
```

Example screen output:
```text
True heading: 87.5°
Magnetic heading: 86.9°
Accuracy: 3
```

### Convert degrees to a direction label

```js
function getDirection(degree) {
  if (degree >= 337.5 || degree < 22.5) return "North";
  if (degree < 67.5) return "North-East";
  if (degree < 112.5) return "East";
  if (degree < 157.5) return "South-East";
  if (degree < 202.5) return "South";
  if (degree < 247.5) return "South-West";
  if (degree < 292.5) return "West";
  return "North-West";
}

console.log(getDirection(90));  // East
console.log(getDirection(180)); // South
console.log(getDirection(315)); // North-West
```

Expected output:
```text
East
South
North-West
```

**Device note:** Heading depends on compass/magnetometer sensor support and calibration. Test on a physical device; simulators/emulators may not provide real compass readings.


### Mini compass app: live heading updates

`getHeadingAsync()` gives one heading reading. For a compass that updates while the phone rotates, use `watchHeadingAsync()` and remove its subscription when the screen/activity is finished.

```jsx
import { useEffect, useRef, useState } from "react";
import { Button, Text, View } from "react-native";
import * as Location from "expo-location";

function getDirection(degree) {
  if (degree >= 337.5 || degree < 22.5) return "North";
  if (degree < 67.5) return "North-East";
  if (degree < 112.5) return "East";
  if (degree < 157.5) return "South-East";
  if (degree < 202.5) return "South";
  if (degree < 247.5) return "South-West";
  if (degree < 292.5) return "West";
  return "North-West";
}

export default function MiniCompass() {
  const [heading, setHeading] = useState(null);
  const subscriptionRef = useRef(null);

  const startCompass = async () => {
    try {
      if (subscriptionRef.current) return;

      subscriptionRef.current = await Location.watchHeadingAsync((result) => {
        setHeading(result);
      });
    } catch (error) {
      console.error("Compass error:", error);
    }
  };

  const stopCompass = () => {
    subscriptionRef.current?.remove();
    subscriptionRef.current = null;
  };

  useEffect(() => {
    return () => subscriptionRef.current?.remove();
  }, []);

  return (
    <View style={{ padding: 20, gap: 10 }}>
      <Button title="Start Compass" onPress={startCompass} />
      <Button title="Stop Compass" onPress={stopCompass} />
      {heading && (
        <>
          <Text>Heading: {heading.trueHeading}°</Text>
          <Text>Direction: {getDirection(heading.trueHeading)}</Text>
          <Text>Magnetic heading: {heading.magHeading}°</Text>
        </>
      )}
    </View>
  );
}
```

Example screen output:

```text
Heading: 87.5°
Direction: East
Magnetic heading: 86.9°
```

These are sample values. As you rotate the phone, the heading and direction can change. Heading values depend on sensor support and calibration; test on a physical device.

### Compass calibration

Sometimes a compass reading is unstable or inaccurate. Follow any calibration instructions shown by the operating system. The device may ask you to move the phone in a figure-eight (`∞`) pattern. This can help the device recalibrate its compass sensors, but the exact process depends on the phone and operating system.

---

## 12. Check Location Services

Permission and Location Services are different:

- **Permission granted** means the app is allowed to request location.
- **Location Services enabled** means the device's location setting is turned on.

### Check whether Location Services are enabled

```js
const enabled = await Location.hasServicesEnabledAsync();
console.log(enabled);
```

Possible output:
```text
true
```
or
```text
false
```

### Example flow

```jsx
import { Alert } from "react-native";
import * as Location from "expo-location";

async function getLocationSafely() {
  try {
    const permission =
      await Location.requestForegroundPermissionsAsync();

    if (!permission.granted) {
      Alert.alert("Permission required", "Allow location access.");
      return;
    }

    const enabled = await Location.hasServicesEnabledAsync();

    if (!enabled) {
      Alert.alert(
        "Location is off",
        "Turn on Location Services in your device Settings."
      );
      return;
    }

    const location = await Location.getCurrentPositionAsync();
    console.log(location);
  } catch (error) {
    console.error("Could not get location:", error);
    Alert.alert("Error", "Unable to get location.");
  }
}
```

Expected behavior:
- Permission denied → permission alert.
- Services disabled → ask user to turn Location Services on.
- Both okay → logs a location object.

### Android high-accuracy prompt

```js
await Location.enableNetworkProviderAsync();
```

This can prompt Android users to enable a higher-accuracy location mode where supported. It is Android-specific; do not rely on it as a cross-platform method. For a simple city/weather app, it is often unnecessary.

---

## 13. Understand the complete Location Object

A location object commonly looks like this:

```js
{
  coords: {
    latitude: 28.6139,
    longitude: 77.2090,
    altitude: 215.3,
    accuracy: 5,
    altitudeAccuracy: 4,
    heading: 90,
    speed: 1.8
  },
  timestamp: 1754037892000
}
```

| Property | Meaning | Notes |
|---|---|---|
| `coords` | Groups the coordinate and movement values | Most properties are inside this object |
| `coords.latitude` | North/south coordinate | Range `-90` to `+90` |
| `coords.longitude` | East/west coordinate | Range `-180` to `+180` |
| `coords.accuracy` | Estimated horizontal uncertainty in meters | Smaller generally means more precise |
| `coords.altitude` | Estimated altitude in meters | May be `null` |
| `coords.altitudeAccuracy` | Estimated altitude uncertainty in meters | May be `null` |
| `coords.heading` | Direction of travel in degrees, if available | May be `null` if not moving / unavailable |
| `coords.speed` | Speed in meters per second | May be `null` |
| `timestamp` | Time the reading was captured, in milliseconds since Unix epoch | Convert with `new Date(timestamp)` |

### Read individual values

```js
console.log(location.coords.latitude);
console.log(location.coords.longitude);
console.log(location.coords.accuracy);
console.log(location.coords.altitude);
console.log(location.coords.altitudeAccuracy);
console.log(location.coords.heading);
console.log(location.coords.speed);
console.log(location.timestamp);
```

Example output:
```text
28.6139
77.2090
5
215.3
4
90
1.8
1754037892000
```

### Convert timestamp to a readable date

```js
console.log(new Date(location.timestamp).toString());
```

Example output (will depend on timestamp and device time zone):
```text
Fri Aug 01 2025 11:24:52 GMT+0530
```

### Destructure the object

Instead of repeatedly writing `location.coords`:

```js
const { latitude, longitude, accuracy, speed, heading } = location.coords;

console.log(latitude, longitude, accuracy, speed, heading);
```

Example output:
```text
28.6139 77.2090 5 1.8 90
```

Some properties can be `null`; handle missing values in the UI:
```jsx
<Text>Speed: {location.coords.speed ?? "Unavailable"}</Text>
```

---

## 14. Common mistakes to avoid

1. **Not asking permission** before requesting the user's location.
2. **Assuming permission means GPS is on** — check Location Services separately if needed.
3. **Assuming every field has a value** — `speed`, `heading`, altitude fields, or cached location may be unavailable.
4. **Forgetting `await`** for asynchronous methods.
5. **Not using `try...catch`** around location requests that may fail.
6. **Not checking an array before using `[0]`** after geocoding or reverse geocoding.
7. **Not stopping tracking** — call `subscription.remove()`.
8. **Using maximum accuracy everywhere** — it may waste battery.
9. **Confusing heading and location** — heading is direction; latitude/longitude describe position.
10. **Trusting sample output as fixed** — coordinates, accuracy, address fields, and sensor values vary by device and location.
11. **Assuming foreground tracking continues after the app is closed** — background tracking requires additional permissions and setup.
12. **Testing compass only in an emulator** — use a real device with a supported sensor.
13. **Assuming background permission starts tracking** — a background task and platform configuration are also required.
14. **Assuming all address fields exist** — combine available fields with `filter(Boolean)`.
15. **Leaving a compass listener running** — remove its subscription when it is no longer needed.

---

## 15. Quick revision: method cheat sheet

| Method | Purpose | Returns / output |
|---|---|---|
| `Location.requestForegroundPermissionsAsync()` | Ask for foreground permission | Permission object |
| `Location.requestBackgroundPermissionsAsync()` | Ask for background location permission | Permission object; does not start tracking by itself |
| `Location.getCurrentPositionAsync(options?)` | Get one fresh location reading | Location object |
| `Location.getLastKnownPositionAsync(options?)` | Get cached location if available | Location object or `null` |
| `Location.watchPositionAsync(options, callback)` | Receive repeated location updates | Subscription; call `.remove()` to stop |
| `Location.reverseGeocodeAsync({ latitude, longitude })` | Coordinates → address | Array of address objects |
| `Location.geocodeAsync(address)` | Address → coordinates | Array of coordinate objects |
| `Location.getHeadingAsync()` | Read one compass heading | Heading object |
| `Location.watchHeadingAsync(callback)` | Receive compass heading updates | Subscription; call `.remove()` to stop |
| `Location.hasServicesEnabledAsync()` | Check whether Location Services are enabled | Boolean (`true` / `false`) |
| `Location.enableNetworkProviderAsync()` | Prompt for Android high-accuracy mode where supported | Resolves when handled; Android-specific |

### Remember these four flows

```text
GET CURRENT LOCATION
Permission → getCurrentPositionAsync() → Location Object

LAST KNOWN LOCATION
Permission → getLastKnownPositionAsync() → Object or null

LIVE TRACKING
Permission → watchPositionAsync() → repeated callbacks → subscription.remove()

ADDRESS CONVERSION
Address → geocodeAsync() → coordinates
Coordinates → reverseGeocodeAsync() → address
```

These notes retain the original core concepts while removing repeated explanations and adding the missing background-permission, full-address, and compass-practical sections.
