# Expo Notifications — Clean & Complete Notes

> Easy revision notes for React Native + Expo, based on the supplied `expo-notifications` material targeting **Expo SDK 54**. Repeated examples and long duplicate explanations are removed. Important concepts, APIs, outputs, options, platform notes, and workflows are retained.
>
> **Version note:** Notification APIs and trigger shapes can vary by Expo SDK and platform. Match examples to the SDK installed in your project and test on a real device/development build where required.

## 1. Core mental model

```text
PERMISSION → Is the app allowed to notify?
CONTENT    → What should the notification say?
DATA       → What app-specific context is attached?
TRIGGER    → When should it fire?
SCHEDULE   → Register the notification request
ID         → Identify it for later management
LISTENER   → React when a notification arrives
RESPONSE   → React when the user interacts
CHANNEL    → Android notification behavior/category
CATEGORY   → Interactive actions such as Open / Reply
CANCEL     → Stop a scheduled request
```

### Local vs push notifications

| Local notification | Push notification |
|---|---|
| Created/scheduled on the device | Usually originates from a backend/remote service |
| Good for study, water, habit, and task reminders | Good for new messages, order updates, server events |
| Basic scheduling does not need a backend or internet | Remote delivery normally needs network connectivity and push setup |

A scheduled local notification can be delivered by the OS even when the app is not the visible screen, subject to permissions, OS settings, battery restrictions, and platform behavior. A JavaScript `setTimeout()` is **not** a replacement for OS notifications.

## 2. Install and import

```bash
npx expo install expo-notifications
```

```js
import * as Notifications from "expo-notifications";
```

Use `npx expo install` so Expo can select a compatible package version. Installing the package does **not** grant permission automatically.

## 3. Notification content

The notification request generally contains two main parts:

```js
{
  content: {
    title: "Study Reminder",
    body: "Time to study React Native!",
    data: { screen: "Study", lessonId: "lesson-101" },
  },
  trigger: {
    // when it should fire
  },
}
```

- `content.title`: short heading.
- `content.body`: explanatory text.
- `content.data`: custom app data, often IDs or a destination key. It is not necessarily displayed to the user.
- `trigger`: determines when the request should fire.

Example visible notification:

```text
Study Reminder
Time to study React Native!
```

Example attached data:

```js
{ screen: "Study", lessonId: "lesson-101" }
```

**Privacy rule:** Do not put passwords, authentication tokens, or sensitive personal details in notification text or data.

## 4. Configure foreground presentation

Configure the handler once, early in the app lifecycle (not on every screen render).

```js
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowBanner: true,
    shouldShowList: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});
```

| Handler option | Meaning when `true` |
|---|---|
| `shouldShowBanner` | Request banner presentation where supported |
| `shouldShowList` | Request notification-list presentation where supported |
| `shouldPlaySound` | Request sound where supported |
| `shouldSetBadge` | Allow badge-related behavior where supported |

The handler mainly controls presentation while the app is **foregrounded**. It does not schedule notifications and does not replace a listener. Actual presentation still depends on OS permissions/settings.

## 5. Permissions

### Check existing permission

```js
const permission = await Notifications.getPermissionsAsync();
console.log(permission);
console.log(permission.status);
```

The result is an object with a `status` field and potentially other platform-specific fields. Common status values include:

- `"granted"` — permission granted.
- `"denied"` — permission denied.
- `"undetermined"` — user has not decided yet (exact states can vary).

### Request permission

```js
const result = await Notifications.requestPermissionsAsync();
console.log(result);
console.log(result.status);
```

Expected status examples: `"granted"` or `"denied"`.

### Recommended reusable permission flow

```js
import * as Notifications from "expo-notifications";

export async function ensureNotificationPermission() {
  const current = await Notifications.getPermissionsAsync();

  if (current.status === "granted") {
    return true;
  }

  const requested = await Notifications.requestPermissionsAsync();
  return requested.status === "granted";
}
```

Use it:

```js
const allowed = await ensureNotificationPermission();

if (!allowed) {
  console.log("Notification permission was not granted");
  return;
}
```

For a real app, explain why reminders are useful before showing the system prompt. If permission is denied, you may guide the user to device settings with React Native's `Linking.openSettings()`, but do not repeatedly prompt without a reason.

**Important distinction**
- Package installation = API is installed.
- Permission = user/OS allows notifications.
- Scheduling = create a notification request.
- Android channel = configure a notification category's behavior on Android.

## 6. Schedule a local notification

### Main method: `scheduleNotificationAsync()`

```js
const notificationId =
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "Hello!",
      body: "Your local notification is working.",
    },
    trigger: {
      type: "timeInterval",
      seconds: 10,
      repeats: false,
    },
  });

console.log(notificationId);
```

**Output:** a generated notification identifier, commonly a string. It identifies the scheduled request; it does **not** mean the notification has already appeared.

Flow:

```text
Permission → content + trigger → scheduleNotificationAsync()
           → notification ID → wait for trigger → OS presents notification
```

Scheduling success and visible presentation are different stages. Do not assume the notification is displayed immediately after the scheduling Promise resolves.

### Dynamic content + data

```js
const task = {
  id: "task-101",
  title: "Complete React Native assignment",
};

const id = await Notifications.scheduleNotificationAsync({
  content: {
    title: "Task Reminder",
    body: task.title,
    data: {
      type: "task",
      screen: "TaskDetails",
      taskId: task.id,
    },
  },
  trigger: {
    type: "timeInterval",
    seconds: 10,
    repeats: false,
  },
});

console.log("Scheduled ID:", id);
```

The visible text is “Task Reminder” and the task title; `data` is available to app code when handling the notification.

### Error handling

```js
try {
  const id = await Notifications.scheduleNotificationAsync({
    content: {
      title: "Study Reminder",
      body: "Time to study.",
    },
    trigger: {
      type: "timeInterval",
      seconds: 10,
      repeats: false,
    },
  });

  console.log("Scheduled:", id);
} catch (error) {
  console.error("Could not schedule notification:", error);
}
```

Use `try/catch` because scheduling is asynchronous and can fail due to invalid input, platform limits, or configuration problems.

## 7. Trigger types and examples

A trigger answers **WHEN**. A content object answers **WHAT**.

| Trigger type | Use |
|---|---|
| `TIME_INTERVAL` / `"timeInterval"` | After a specified duration |
| `DATE` / `"date"` | One specific future date/time |
| `DAILY` / `"daily"` | Repeating daily time |
| `WEEKLY` / `"weekly"` | Repeating weekday/time |
| `MONTHLY` / `"monthly"` | Repeating day-of-month/time |
| `YEARLY` / `"yearly"` | Repeating date/time yearly |
| `CALENDAR` / calendar trigger | Match calendar components; availability is platform-specific |
| `null` | Immediate/no-delay trigger behavior as supported by the installed SDK |

Expo SDK 54 exposes trigger types through `Notifications.SchedulableTriggerInputTypes`. The exact shape and availability of triggers differs by platform. Verify the installed SDK documentation rather than mixing trigger syntaxes from different versions.

### A. One-time delay

```js
trigger: {
  type: "timeInterval",
  seconds: 30,
  repeats: false,
}
```

**Meaning:** request delivery after about 30 seconds, subject to OS scheduling behavior.

### B. Repeating interval

```js
trigger: {
  type: "timeInterval",
  seconds: 60,
  repeats: true,
}
```

**Meaning:** repeat at the specified interval where supported. The supplied material notes that iOS repeating time intervals must be at least **60 seconds**; do not use a 10-second repeating interval as a cross-platform example.

### C. One specific date/time

```js
const reminderDate = new Date("2027-01-24T09:00:00");

const id = await Notifications.scheduleNotificationAsync({
  content: {
    title: "Reminder",
    body: "Your event starts today.",
  },
  trigger: {
    type: "date",
    date: reminderDate,
  },
});
```

This is an illustrative date-trigger pattern; use the exact shape supported by your installed SDK. Validate the date and do not schedule a date in the past.

### D. Daily reminder

A daily trigger represents a local-time rule such as 9:00 AM every day. SDK trigger shapes can differ by platform; the source describes dedicated daily triggers on Android and calendar-based equivalents on iOS.

Conceptual fields:

```js
{
  hour: 9,
  minute: 0,
}
```

### E. Weekly reminder

A weekly reminder needs a weekday plus hour/minute. Check the SDK's weekday numbering before using it; a wrong weekday index is a common bug.

```js
{
  weekday: 2, // verify numbering for your API/version
  hour: 9,
  minute: 0,
}
```

### F. Monthly reminder

A monthly reminder uses a day-of-month and time. Validate the day: not every month has 29, 30, or 31 days.

```js
{
  day: 1,
  hour: 9,
  minute: 0,
}
```

### G. Yearly reminder

A yearly reminder represents a month/day/time rule. Validate leap-day and date edge cases.

```js
{
  month: 1, // verify API's month numbering
  day: 24,
  hour: 9,
  minute: 0,
}
```

### Trigger selection rules

- “In 30 seconds” → time interval.
- “At this exact date/time” → date trigger.
- “Every day at 9 AM” → daily/calendar rule.
- “Every Monday” → weekly/calendar rule.
- “On the first of each month” → monthly/calendar rule.
- “Every year on this date” → yearly/calendar rule.

Do not assume exact timing is a stopwatch. Android exact-alarm behavior, battery optimization, time zones, daylight-saving changes, and OS policies can affect delivery. For reminders that follow the user's local clock, plan time-zone behavior deliberately.

### `getNextTriggerDateAsync()`

Checks when a trigger would fire next.

```js
const nextTimestamp =
  await Notifications.getNextTriggerDateAsync({
    hour: 9,
    minute: 0,
  });

console.log(nextTimestamp);
```

**Output:** a Unix timestamp in milliseconds or `null` when the notification will not trigger, as described in the supplied SDK-54 material. Convert a non-null timestamp to a date for display:

```js
if (nextTimestamp !== null) {
  console.log(new Date(nextTimestamp).toString());
}
```

The trigger object passed must match the trigger type/API supported by the installed SDK.

## 8. Manage scheduled notifications

### `cancelScheduledNotificationAsync(id)`

Cancels one scheduled notification.

```js
await Notifications.cancelScheduledNotificationAsync(notificationId);
```

**Output:** asynchronous completion; the important effect is that the identified scheduled request is cancelled. Use the ID returned by `scheduleNotificationAsync()`.

### `cancelAllScheduledNotificationsAsync()`

Cancels all scheduled notification requests.

```js
await Notifications.cancelAllScheduledNotificationsAsync();
```

**Output:** asynchronous completion. Use carefully: this can remove every scheduled reminder, not just one feature's reminder.

### `getAllScheduledNotificationsAsync()`

Reads the currently scheduled notification requests.

```js
const scheduled =
  await Notifications.getAllScheduledNotificationsAsync();

console.log(scheduled);
console.log("Scheduled count:", scheduled.length);
```

**Output:** an array of scheduled notification request objects. Each item includes request information such as its identifier, content, and trigger.

### Important reminder-management rule

Keep your app's reminder record and the OS notification request synchronized.

```text
CREATE → validate → schedule → save returned notification ID
EDIT   → cancel old request → schedule replacement → save new ID
DELETE → cancel scheduled request → delete app reminder
DISABLE→ cancel request, retain reminder record if needed
ENABLE → schedule again and save new ID
```

Your app's reminder ID and the notification request ID are **not the same thing**. Store both if your application needs both. A notification ID may change when a reminder is rescheduled.

## 9. Notification received listener

### `addNotificationReceivedListener(callback)`

Listens for a notification received while the app is running.

```js
import { useEffect } from "react";
import * as Notifications from "expo-notifications";

useEffect(() => {
  const subscription =
    Notifications.addNotificationReceivedListener((notification) => {
      console.log("Received notification:", notification);
      console.log("Content:", notification.request.content);
      console.log("Data:", notification.request.content.data);
    });

  return () => subscription.remove();
}, []);
```

**Callback input:** a notification object. Common access path:

```js
notification.request.content.title
notification.request.content.body
notification.request.content.data
```

**Output:** the listener registration returns a subscription object. Call `subscription.remove()` during cleanup to avoid duplicate listeners.

Use this listener to update in-app state, counters, or an inbox while the app is running. It does not schedule notifications and does not permanently store notification history.

## 10. Notification response listener (user interaction)

### `addNotificationResponseReceivedListener(callback)`

Listens for a user response to a notification, such as tapping it or selecting an action.

```js
useEffect(() => {
  const subscription =
    Notifications.addNotificationResponseReceivedListener((response) => {
      const notification = response.notification;
      const data = notification.request.content.data;

      console.log("Response:", response);
      console.log("Notification data:", data);
      console.log("Action:", response.actionIdentifier);
    });

  return () => subscription.remove();
}, []);
```

**Callback input:** a notification response object. It contains the notification and interaction information, including an action identifier.

Use it to decide whether to open a task, order, lesson, or chat. The listener does **not** automatically navigate; your app must validate the data and call the navigation API when navigation is ready.

### Received listener vs response listener

| Listener | Meaning |
|---|---|
| `addNotificationReceivedListener` | Notification was received while app code is running |
| `addNotificationResponseReceivedListener` | User interacted with a notification |
| `setNotificationHandler` | Controls foreground presentation behavior |

These three APIs solve different problems.

### App opened from a notification

```js
const lastResponse =
  await Notifications.getLastNotificationResponseAsync();

console.log(lastResponse);
```

**Output:** a notification response object or `null` if there is no relevant last response. This helps handle an app launch caused by a notification interaction. Combine startup handling with a response listener when needed, and prevent duplicate navigation.

### Listener best practices

- Register app-level listeners early enough to catch relevant events.
- Use `useEffect()` for screen/component listeners.
- Always call `subscription.remove()` during cleanup.
- Avoid registering the same listener repeatedly on every render.
- Validate `data.type`, IDs, and expected fields before navigating.
- Use persistent storage if notification history must survive app restarts; React state alone is temporary.

## 11. Android notification channels

Android uses **notification channels** to group notification types and control their behavior. Channels are an Android concept; iOS uses a different notification model.

### Create a channel

```js
import { Platform } from "react-native";
import * as Notifications from "expo-notifications";

if (Platform.OS === "android") {
  await Notifications.setNotificationChannelAsync("reminders", {
    name: "Reminders",
    importance: Notifications.AndroidImportance.HIGH,
    vibrationPattern: [0, 250, 250, 250],
    enableVibrate: true,
    sound: "default",
    description: "Study and task reminders",
  });
}
```

**Arguments:**
1. `"reminders"` — stable channel ID.
2. Configuration object — channel name and behavior.

**Output:** an asynchronous result representing the configured channel (or platform-specific result). The supplied notes do not require a specific printed output for normal use.

Important channel fields:
- `name`: user-visible channel name.
- `description`: explains the channel.
- `importance`: importance level, e.g. `Notifications.AndroidImportance.HIGH`.
- `sound`: sound setting where supported.
- `vibrationPattern`: vibration timing pattern, e.g. `[0, 250, 250, 250]`.
- `enableVibrate`: enable vibration where supported.
- `lightColor`: notification light color on supported devices.
- `groupId`: optional channel-group association where supported.

### Channel importance (conceptual)

| Importance | General intent |
|---|---|
| `MIN` | Very low visibility |
| `LOW` | Low priority |
| `DEFAULT` | Normal priority |
| `HIGH` | Higher visibility/attention |
| `MAX` | Highest level where supported |

Use the actual constants exposed by the installed Expo SDK. Do not assume importance guarantees sound/banner behavior; user and OS settings can override it.

### Associate a scheduled notification with a channel

For Android channel-aware scheduling, include the channel ID in the notification content/request where supported by the installed SDK:

```js
await Notifications.scheduleNotificationAsync({
  content: {
    title: "Study Reminder",
    body: "Time to study.",
    data: { type: "study" },
    // channelId: "reminders", // use the field/location required by your SDK
  },
  trigger: {
    type: "timeInterval",
    seconds: 10,
    repeats: false,
  },
});
```

Create the channel **before** scheduling notifications that use it. Confirm the exact `channelId` placement for the Expo SDK version in your project.

### Read channel information

```js
const channels = await Notifications.getNotificationChannelsAsync();
console.log(channels);
```

**Output:** an array of configured channels (primarily Android; may be unavailable/empty on other platforms).

```js
const channel = await Notifications.getNotificationChannelAsync("reminders");
console.log(channel);
```

**Output:** a channel object or `null` if the channel does not exist.

### Delete a channel

```js
await Notifications.deleteNotificationChannelAsync("reminders");
```

**Output:** asynchronous completion. Delete only when the channel is no longer needed; treat channel removal as a configuration change.

### Channel groups

```js
await Notifications.setNotificationChannelGroupAsync("communication", {
  name: "Communication",
  description: "Communication-related notifications",
});
```

A channel group organizes multiple Android channels. The supplied notes describe `setNotificationChannelGroupAsync()` for creating/configuring a group; verify its exact configuration and association fields for your SDK.

**Important:** Android channel settings can persist, and users have control over them. Do not rely on changing importance/sound in app code to override a channel the user has already customized.

## 12. Interactive notification categories and actions

A **category** defines the actions available on a notification (e.g. Open, Mark as Read, Reply). It is different from an Android channel.

### Create a category

```js
await Notifications.setNotificationCategoryAsync("message", [
  {
    identifier: "open",
    buttonTitle: "Open",
  },
  {
    identifier: "mark_read",
    buttonTitle: "Mark as Read",
  },
]);
```

**Output:** asynchronous result for the category configuration. The category identifier (`"message"`) is later attached to a notification using `categoryIdentifier` in the content, as supported by the installed SDK.

### Associate category with notification

```js
await Notifications.scheduleNotificationAsync({
  content: {
    title: "New Message",
    body: "You have a new message.",
    categoryIdentifier: "message",
    data: { type: "chat", chatId: "chat-123" },
  },
  trigger: {
    type: "timeInterval",
    seconds: 10,
    repeats: false,
  },
});
```

### Handle selected action

The response listener receives `response.actionIdentifier`. Use it to branch:

```js
switch (response.actionIdentifier) {
  case "open":
    console.log("Open the relevant screen");
    break;
  case "mark_read":
    console.log("Mark message as read");
    break;
  default:
    console.log("Default notification tap");
}
```

A text-input action can allow a reply, but action support and exact option fields are platform-sensitive. Test on the target OS.

### Inspect/delete categories

```js
const categories = await Notifications.getNotificationCategoriesAsync();
console.log(categories);
```

**Output:** an array of configured notification categories/actions.

```js
await Notifications.deleteNotificationCategoryAsync("message");
```

**Output:** asynchronous completion; removes that category configuration. Only delete categories no longer required.

### Category vs channel

| Channel | Category |
|---|---|
| Android notification behavior/grouping | Interactive action layout/behavior |
| Examples: Reminders, Messages | Examples: Open, Reply, Mark as Read |
| Android-specific | Interaction feature with platform-specific support |

## 13. Badges, sound, vibration, and presentation

### Badge count

```js
const success = await Notifications.setBadgeCountAsync(3);
console.log(success);
```

This asks the platform to set the app badge to `3`. The method resolves to a boolean success indicator in the supplied SDK-54 material.

Clear badge:

```js
await Notifications.setBadgeCountAsync(0);
```

Read badge:

```js
const count = await Notifications.getBadgeCountAsync();
console.log(count);
```

**Output:** a number representing the current badge count (commonly `0` if cleared). Badge support/appearance varies by platform and launcher. A badge is not automatically the number of unread notifications; your app should define what the count means.

### Sound and vibration

- `shouldPlaySound` in the foreground handler controls requested foreground sound behavior.
- `sound` can configure a notification/channel sound where supported.
- Android channel sound settings can take precedence for Android notifications.
- `vibrationPattern`, for example `[0, 250, 250, 250]`, describes vibration timing where supported.
- Device settings, channel settings, OS policy, and sound/vibration modes may affect actual behavior.

Do not overuse sound or vibration. They should match the importance of the notification.

## 14. Method reference: inputs and outputs

| Method | Important input | Output / effect |
|---|---|---|
| `setNotificationHandler(handler)` | Object with `handleNotification` returning presentation options | Configures foreground presentation; no useful value normally needed |
| `getPermissionsAsync()` | None | Promise resolving to permission object with `status` and possibly extra fields |
| `requestPermissionsAsync()` | Optional platform-specific permission options | Promise resolving to permission object; inspect `.status` |
| `scheduleNotificationAsync(request)` | `content`, `trigger` | Promise resolving to notification ID |
| `getNextTriggerDateAsync(trigger)` | Trigger rule | Timestamp in milliseconds or `null` according to supplied notes |
| `cancelScheduledNotificationAsync(id)` | Scheduled request ID | Cancels one scheduled request |
| `cancelAllScheduledNotificationsAsync()` | None | Cancels all scheduled requests |
| `getAllScheduledNotificationsAsync()` | None | Array of scheduled request objects |
| `addNotificationReceivedListener(callback)` | Callback receiving notification | Subscription; clean up with `.remove()` |
| `addNotificationResponseReceivedListener(callback)` | Callback receiving response | Subscription; clean up with `.remove()` |
| `getLastNotificationResponseAsync()` | None | Response object or `null` |
| `setNotificationChannelAsync(id, config)` | Channel ID + configuration | Creates/configures Android channel |
| `getNotificationChannelAsync(id)` | Channel ID | Channel object or `null` |
| `getNotificationChannelsAsync()` | None | Array of channels where supported |
| `deleteNotificationChannelAsync(id)` | Channel ID | Deletes that Android channel |
| `setNotificationChannelGroupAsync(id, config)` | Group ID + configuration | Creates/configures Android channel group |
| `setNotificationCategoryAsync(id, actions)` | Category ID + action array | Creates/configures interactive category |
| `getNotificationCategoriesAsync()` | None | Array of categories and actions |
| `deleteNotificationCategoryAsync(id)` | Category ID | Deletes that category |
| `setBadgeCountAsync(count)` | Non-negative badge count | Boolean success indicator as described in source |
| `getBadgeCountAsync()` | None | Current badge count number |

For channel/category methods, exact returned object details and platform availability can vary. Use return values for debugging if needed, but do not assume all methods work identically on Android and iOS.

## 15. Common React Native props used in examples

These are **React Native UI props**, not `expo-notifications` options.

| Prop | Common component | Meaning |
|---|---|---|
| `onPress` | `Button`, `Pressable` | Function invoked when pressed |
| `title` | `Button` | Button label |
| `value` | `TextInput` | Current input value |
| `onChangeText` | `TextInput` | Receives new text |
| `placeholder` | `TextInput` | Hint when input is empty |
| `style` | UI components | Layout/appearance |
| `useEffect(..., [])` | React Hook | Register on mount; cleanup on unmount |

A notification's `content.title` is different from a Button's `title` prop.

## 16. Complete minimal example

This example checks permission and schedules a notification 10 seconds later. The trigger shape must match your installed SDK.

```jsx
import { useEffect } from "react";
import { Alert, Pressable, Text, View } from "react-native";
import * as Notifications from "expo-notifications";

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowBanner: true,
    shouldShowList: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

export default function NotificationScreen() {
  const scheduleReminder = async () => {
    try {
      let permission = await Notifications.getPermissionsAsync();

      if (permission.status !== "granted") {
        permission = await Notifications.requestPermissionsAsync();
      }

      if (permission.status !== "granted") {
        Alert.alert("Permission required", "Enable notifications to receive reminders.");
        return;
      }

      const id = await Notifications.scheduleNotificationAsync({
        content: {
          title: "Study Reminder",
          body: "Time to study React Native!",
          data: { type: "study", screen: "Study" },
        },
        trigger: {
          type: "timeInterval",
          seconds: 10,
          repeats: false,
        },
      });

      console.log("Notification ID:", id);
      Alert.alert("Scheduled", "The reminder is scheduled for about 10 seconds from now.");
    } catch (error) {
      console.error("Notification error:", error);
      Alert.alert("Error", "Could not schedule the notification.");
    }
  };

  useEffect(() => {
    const received = Notifications.addNotificationReceivedListener((notification) => {
      console.log("Received:", notification.request.content);
    });

    const response = Notifications.addNotificationResponseReceivedListener((event) => {
      console.log("User interaction:", event.actionIdentifier);
      console.log("Data:", event.notification.request.content.data);
    });

    return () => {
      received.remove();
      response.remove();
    };
  }, []);

  return (
    <View style={{ flex: 1, justifyContent: "center", padding: 20 }}>
      <Pressable onPress={scheduleReminder} style={{ padding: 16 }}>
        <Text>Schedule Reminder</Text>
      </Pressable>
    </View>
  );
}
```

## 17. Common mistakes to avoid

1. Assuming package installation automatically grants permission.
2. Assuming scheduling means the notification is already displayed.
3. Confusing `content` (what) with `trigger` (when).
4. Using `setTimeout()` instead of OS scheduling for reminders.
5. Forgetting to save the ID returned by `scheduleNotificationAsync()`.
6. Cancelling all notifications when only one should be cancelled.
7. Forgetting to cancel the old request when editing a reminder.
8. Forgetting `subscription.remove()` and registering duplicate listeners.
9. Treating a received listener as a notification-response listener.
10. Assuming response listeners automatically navigate to a screen.
11. Failing to validate notification `data` before using its IDs.
12. Assuming Android channels exist on iOS or behave identically.
13. Creating a channel after scheduling a notification that depends on it.
14. Assuming permission granted means sound, banner, and badge must always appear.
15. Using invalid date components, weekday values, or a date in the past.
16. Using a repeating iOS time interval shorter than 60 seconds.
17. Assuming notification timing is exact regardless of OS/battery/time-zone behavior.
18. Assuming in-memory React state is permanent notification history.
19. Putting private data into notification title/body/data.
20. Mixing API examples from different Expo SDK versions without checking compatibility.

## 18. Debugging checklist

- Confirm `expo-notifications` is installed with `npx expo install`.
- Inspect `await Notifications.getPermissionsAsync()` and `.status`.
- Check Android channel configuration and user settings on Android.
- Inspect `await Notifications.getAllScheduledNotificationsAsync()`.
- Log the returned notification ID.
- Use `getNextTriggerDateAsync()` where appropriate to inspect the next trigger.
- Confirm the trigger is in the future and uses the correct type/fields.
- Test foreground, background, and terminated-app behavior separately.
- Check listener registration and cleanup.
- Test notification response data and action identifiers.
- Test on the target physical device/development build; Expo Go and production builds may not represent every native behavior identically.
- Remember that OS settings, battery restrictions, exact-alarm rules, and time zones can affect delivery.

## 19. Real-world reminder architecture

Keep the app's reminder data separate from the OS notification request.

```text
Reminder record (app data)
  ├── reminderId
  ├── title / body
  ├── due time / recurrence
  ├── enabled
  └── notificationId (OS request ID)

CREATE → validate → schedule → store notificationId
EDIT   → cancel old → schedule replacement → update notificationId
DELETE → cancel request → delete reminder record
DISABLE→ cancel request → keep reminder record if desired
ENABLE → schedule again → store new notificationId
```

Useful architecture concepts:
- A notification service centralizes permission checks, scheduling, cancellation, and trigger construction.
- The application reminder ID identifies your business/domain record.
- The notification ID identifies a scheduled OS request.
- The OS scheduler and app storage are separate systems and can become out of sync; reconcile them when the app starts if the app needs reliable reminder management.
- Register app-level listeners in one suitable place to avoid duplicate handlers.
- Use a central response handler to validate `data.type` and IDs, then navigate only when navigation is ready.

## 20. Final quick revision

- **Permission:** `getPermissionsAsync()`, `requestPermissionsAsync()`.
- **Foreground presentation:** `setNotificationHandler()`.
- **Schedule:** `scheduleNotificationAsync()` → returns ID.
- **Inspect schedules:** `getAllScheduledNotificationsAsync()`.
- **Next trigger preview:** `getNextTriggerDateAsync()` → timestamp or `null`.
- **Cancel one:** `cancelScheduledNotificationAsync(id)`.
- **Cancel all:** `cancelAllScheduledNotificationsAsync()`.
- **Received event:** `addNotificationReceivedListener()`.
- **User interaction:** `addNotificationResponseReceivedListener()`.
- **Startup interaction check:** `getLastNotificationResponseAsync()` → response or `null`.
- **Android channel:** `setNotificationChannelAsync()`.
- **Interactive category:** `setNotificationCategoryAsync()`.
- **Badge:** `setBadgeCountAsync()`, `getBadgeCountAsync()`.
- **Cleanup listener:** `subscription.remove()`.

**One-line mental model:** Check permission → configure platform behavior → create content + trigger → schedule and save ID → listen for receipt/response → validate data → update UI/navigation → cancel or reschedule when the reminder changes.


---

## 21. Important topics restored after comparing with the full source

This section adds useful material from the original long notes that was missing or too compressed in the clean version. It avoids repeating the basic concepts above.

### A. App lifecycle: foreground, background, terminated

| State | Meaning | Key point |
|---|---|---|
| Foreground | App is open and active | `setNotificationHandler()` controls foreground presentation; a received listener can react to arrival |
| Background | App is not the active screen | The OS may present a scheduled notification |
| Terminated | App process is closed | The OS may still present a previously scheduled local notification; tapping it can launch the app |

Actual behavior depends on OS settings, permission, notification type, and device restrictions. Test these states separately. A notification response listener does not automatically navigate; validate its data and navigate when your app/router is ready.

### B. Immediate notification vs scheduled notification

- **Immediate** means a request intended to fire without a future delay, using the immediate trigger pattern supported by the installed SDK.
- **Scheduled** means a request with a future time or rule.
- Both use a notification request with `content` and `trigger`; exact trigger syntax is SDK-version-sensitive.

Do not use `setTimeout()` or `setInterval()` as a replacement for OS-managed reminders. JavaScript timers are unreliable when the app is suspended or terminated.

### C. Android 13+ permission reminder

On Android 13 and later, notification permission is runtime-controlled. Check the result of `getPermissionsAsync()` and request permission when needed. Permission granted still does not guarantee a visible banner, sound, or vibration: channel settings and user/device settings can change presentation.

### D. Custom notification sounds

- `shouldPlaySound` controls requested foreground sound behavior in the notification handler.
- `sound` can configure a sound where the platform and API support it.
- A custom sound may require adding a sound asset and native app configuration; it may require a development/production build rather than working in every preview environment.
- Android channel sound settings are persistent and user-controlled. Changing the app's configuration may not override a channel that already exists.
- Test default and custom sounds on the actual target OS/device. Do not assume the sound name or asset path is identical across platforms.

### E. Text-input actions and reply buttons

A notification category can define regular actions (for example, **Open** or **Mark as Read**) and, where supported, a text-input action (for example, **Reply**). The response listener can inspect `response.actionIdentifier` and any supported user-input field on the response.

Conceptual category action shape (confirm exact fields and platform support for the installed SDK):

```js
await Notifications.setNotificationCategoryAsync("message", [
  {
    identifier: "reply",
    buttonTitle: "Reply",
    textInput: {
      submitButtonTitle: "Send",
      placeholder: "Write a reply...",
    },
  },
]);
```

The action's identifier is not the same as the notification's `categoryIdentifier`:
- `categoryIdentifier` selects the set of actions attached to a notification.
- `actionIdentifier` tells your app which action the user selected.

Treat text-input actions as platform-sensitive and test on real devices. Validate the notification data and the input before performing an action.

### F. Category lifecycle and design

- Create categories before scheduling notifications that refer to them.
- Use separate categories when the available actions differ by purpose (for example, `message`, `order`, and `reminder`).
- Keep category IDs and action IDs consistent in one place to avoid spelling mismatches.
- Inspect categories with `getNotificationCategoriesAsync()` and remove obsolete ones with `deleteNotificationCategoryAsync()`.
- A category defines **interaction actions**; an Android channel defines **notification behavior/settings**. They solve different problems.

### G. App data vs OS-scheduled data

The OS's scheduled notification list is not your app's reminder database. Store the reminder's own ID, title/time/rule, enabled state, and the notification request ID when needed.

Recommended CRUD synchronization:

```text
CREATE: validate reminder → schedule request → save notification ID + reminder
READ:   load app reminders → optionally inspect OS scheduled requests
EDIT:   cancel old request → schedule replacement → save new notification ID
DELETE: cancel request if present → delete reminder record
DISABLE: cancel request → retain reminder record as disabled
ENABLE: validate → schedule again → store the new request ID
```

The reminder ID and notification request ID are different identifiers. The request ID can change after rescheduling. Handle missing IDs and scheduling errors without corrupting the saved reminder.

### H. Suggested project structure for a larger app

A small learning app can keep the code together. As the app grows, split responsibilities rather than copying scheduling logic across screens:

```text
src/
  notifications/
    notificationConfig.js       # foreground handler / shared constants
    notificationPermissions.js  # check and request permission
    notificationChannels.js     # Android channels
    notificationScheduler.js    # create and cancel requests
    notificationActions.js      # response/action handling
    notificationService.js      # shared high-level reminder operations
  reminders/
    reminderModel.js            # app-level reminder data
    reminderStorage.js          # persistent storage
```

This is a suggested organization, not a requirement imposed by Expo. Keep UI separate from notification service logic, and use persistent storage if reminders must survive app restarts.

### I. Response handling at startup

When the user taps a notification while the app is closed, the app may start from that interaction. A robust pattern is to:
1. Register the response listener early.
2. Check `getLastNotificationResponseAsync()` during startup if needed.
3. Validate the `data` payload and action identifier.
4. Wait until navigation is ready before navigating.
5. Prevent the same response from being handled twice.

The response object may be `null`; do not assume the app was opened from a notification every time.

### J. Extra testing matrix

| Test case | Verify |
|---|---|
| Permission granted / denied | UI explains the outcome and does not silently fail |
| App foreground | Handler and received listener behave as expected |
| App background / terminated | OS presentation and tap behavior work on target device |
| Create / edit / delete reminder | App record and OS request stay synchronized |
| Repeating date rule | Correct weekday, month/day, time zone, and next trigger |
| Android channel | Correct channel ID, importance, sound, vibration, and user settings |
| Category action / reply | Correct `actionIdentifier`, validated data/input, and no duplicate action |
| Badge | App's defined unread/reminder count stays consistent |
| Reinstall / app restart | App storage and OS-scheduled requests are reconciled as appropriate |
| Development vs production build | Native configuration and custom sound behavior are verified |

### K. Push notifications: scope boundary

The original material explains the **difference** between local and push notifications, but its detailed implementation is mainly for local scheduling. Remote push notifications usually also require a server/push service, device or Expo push tokens, credentials/configuration, and delivery/error handling. Do not treat `scheduleNotificationAsync()` as a way to send a push notification from a backend.

### L. High-value API/output recap

| API | Input | Important result |
|---|---|---|
| `getPermissionsAsync()` | None | Permission object; inspect `.status` |
| `requestPermissionsAsync()` | Optional permission options | Permission object; inspect `.status` |
| `scheduleNotificationAsync()` | Request containing `content` and `trigger` | Notification request ID |
| `getAllScheduledNotificationsAsync()` | None | Array of scheduled request objects |
| `cancelScheduledNotificationAsync(id)` | Request ID | Cancels one scheduled request |
| `cancelAllScheduledNotificationsAsync()` | None | Cancels all scheduled requests |
| `getLastNotificationResponseAsync()` | None | Response object or `null` |
| `getNotificationCategoriesAsync()` | None | Configured categories/actions |
| `setBadgeCountAsync(count)` | Non-negative count | Boolean success indicator in the supplied notes |
| `getBadgeCountAsync()` | None | Current badge count |

For all APIs, use the exact signatures and platform behavior of the Expo SDK installed in the project.
