# Expo Contacts — Clean & Complete Notes

> Beginner-friendly notes for React Native + Expo. Repeated explanations and duplicate examples have been removed. Keep this as a reference for the important concepts, methods, options/props, and typical outputs.
>
> **Important:** Contact data and method results can differ by Android/iOS version, device, permissions, and installed Expo SDK. Outputs below are illustrative, not guaranteed literal results.

## 1. What is `expo-contacts`?

`expo-contacts` lets an Expo React Native app work with the device's contact book.

Important tasks:
- Ask for contact permission.
- Read contacts and selected fields.
- Sort, search, and display contacts.
- Read phone numbers, emails, and profile images.
- Add contacts or open the native contact editor.
- Delete contacts.
- Export a contact as a `.vcf` (vCard) file.

### Basic workflow

```text
Install package → Request/check permission → Read contacts
               → Display/search/sort → Add/edit/delete/export
```

## 2. Install and import

```bash
npx expo install expo-contacts
```

```js
import * as Contacts from "expo-contacts";
```

Use `npx expo install` because it selects a package version compatible with the project's Expo SDK.

**Installing does not grant permission automatically.** Ask permission before reading or modifying contacts.

## 3. Contact permission

### Main methods

```js
const current = await Contacts.getPermissionsAsync();
const permission = await Contacts.requestPermissionsAsync();
```

- `getPermissionsAsync()` checks existing permission; it does **not** show a permission popup.
- `requestPermissionsAsync()` asks the user if permission is needed and returns a permission object.

### Typical permission output

```js
{
  status: "granted",
  granted: true,
  canAskAgain: true,
  expires: "never"
}
```

| Property | Meaning |
|---|---|
| `status` | Common statuses include `"granted"`, `"denied"`, and `"undetermined"` |
| `granted` | `true` if access is granted; otherwise `false` |
| `canAskAgain` | Whether the app can ask again; if false, guide the user to device settings |
| `expires` | Expiry information; commonly `"never"` |

### Recommended pattern

```js
const permission = await Contacts.getPermissionsAsync();

if (!permission.granted) {
  const requested = await Contacts.requestPermissionsAsync();

  if (!requested.granted) {
    console.log("Contact permission denied");
    return;
  }
}

console.log("Permission granted");
```

**Output:**
```text
Permission granted
```
or, if permission is denied:
```text
Contact permission denied
```

Wrap asynchronous operations in `try...catch` so failures can be handled gracefully.

## 4. Read contacts — `getContactsAsync()`

### Basic syntax

```js
const result = await Contacts.getContactsAsync();
console.log(result);
```

The result is an **object**, not a bare array. The contact array is in `result.data`.

### Typical output

```js
{
  data: [
    { id: "1", name: "Amit Sharma" },
    { id: "2", name: "Rahul Patel" }
  ],
  hasNextPage: false,
  hasPreviousPage: false
}
```

Useful access patterns:

```js
const { data } = await Contacts.getContactsAsync();

console.log(data);       // array of contacts
console.log(data[0]);    // first contact, if present
console.log(data.length);// number of returned contacts
```

Always handle an empty result:

```js
if (data.length === 0) {
  console.log("No contacts found");
}
```

### Important options for `getContactsAsync()`

```js
const result = await Contacts.getContactsAsync({
  fields: [
    Contacts.Fields.Name,
    Contacts.Fields.PhoneNumbers,
    Contacts.Fields.Emails,
  ],
  sort: Contacts.SortTypes.FirstName,
});
```

| Option | Purpose |
|---|---|
| `fields` | Array of contact fields to retrieve |
| `sort` | How returned contacts should be sorted |
| `pageSize` / `pageOffset` | Pagination options where supported by the installed Expo SDK |
| `id`, `name`, `phoneNumber`, `email` | Search/filter options where supported by the installed Expo SDK |

Use the option names and supported values for your installed Expo SDK. If a filter/pagination option is not supported in that version, filter the returned `data` array in JavaScript instead.

## 5. Contact object and common fields

A contact can contain many optional properties. Some values may be absent, `null`, or `undefined`.

### Typical contact shape

```js
{
  id: "1",
  name: "Rahul Sharma",
  firstName: "Rahul",
  middleName: null,
  lastName: "Sharma",
  nickname: "Rahul",
  company: "Example Ltd",
  jobTitle: "Developer",
  phoneNumbers: [
    { number: "+91 9876543210", label: "mobile" }
  ],
  emails: [
    { email: "rahul@example.com", label: "work" }
  ],
  birthday: { day: 5, month: 8, year: 2000 },
  imageAvailable: false,
  image: null,
  note: "College friend"
}
```

| Property | Meaning |
|---|---|
| `id` | Contact identifier; useful for React list keys and contact actions |
| `name` | Display/full name |
| `firstName`, `middleName`, `lastName`, `nickname` | Name components; optional |
| `phoneNumbers` | Array of phone-number objects |
| `emails` | Array of email objects |
| `company`, `jobTitle` | Work information |
| `birthday` | Object containing `day`, `month`, and possibly `year` |
| `imageAvailable` | Boolean indicating whether a contact photo is available |
| `image` | May contain an image URI, e.g. `{ uri: "file://..." }` |
| `note` | Optional note; platform/provider support may differ |

### Avoid crashes with optional values

Risky:
```js
contact.phoneNumbers[0].number
```

Safer:
```js
contact.phoneNumbers?.[0]?.number ?? "No phone number"
contact.emails?.[0]?.email ?? "No email"
contact.name?.[0] ?? "?"
```

`?.` is **optional chaining**; `??` supplies a fallback when the value is `null` or `undefined`.

## 6. Choose fields — `Contacts.Fields`

Request only the information your app needs. This is better for performance and privacy.

Common field constants used in the source notes:

```js
Contacts.Fields.Name
Contacts.Fields.FirstName
Contacts.Fields.LastName
Contacts.Fields.PhoneNumbers
Contacts.Fields.Emails
Contacts.Fields.Image
Contacts.Fields.Company
Contacts.Fields.JobTitle
Contacts.Fields.Birthday
Contacts.Fields.Note
```

Example:

```js
const { data } = await Contacts.getContactsAsync({
  fields: [
    Contacts.Fields.Name,
    Contacts.Fields.PhoneNumbers,
    Contacts.Fields.Emails,
    Contacts.Fields.Image,
  ],
});
```

**Typical returned contact:**
```js
{
  id: "1",
  name: "Rahul Sharma",
  phoneNumbers: [{ number: "+91 9876543210", label: "mobile" }],
  emails: [{ email: "rahul@example.com", label: "work" }],
  imageAvailable: false
}
```

The actual returned fields depend on the requested fields and device contact data.

## 7. Sort contacts — `Contacts.SortTypes`

Common sort options:

```js
Contacts.SortTypes.FirstName
Contacts.SortTypes.LastName
Contacts.SortTypes.UserDefault
```

Example:

```js
const { data } = await Contacts.getContactsAsync({
  fields: [Contacts.Fields.Name],
  sort: Contacts.SortTypes.FirstName,
});
```

**Illustrative output order:**
```text
Amit Patel
Ankit Shah
Rahul Sharma
Zoya Khan
```

- `FirstName`: sort by first name.
- `LastName`: sort by last name.
- `UserDefault`: follow the device user's contact sorting preference.

Order can differ by platform and locale.

## 8. Display contacts with `FlatList`

`FlatList` is preferable to mapping a very large contact list because it virtualizes items.

```jsx
import { useState } from "react";
import { View, Text, Button, FlatList, Alert } from "react-native";
import * as Contacts from "expo-contacts";

export default function ContactsScreen() {
  const [contacts, setContacts] = useState([]);

  const loadContacts = async () => {
    try {
      let permission = await Contacts.getPermissionsAsync();

      if (!permission.granted) {
        permission = await Contacts.requestPermissionsAsync();
      }

      if (!permission.granted) {
        Alert.alert("Permission denied", "Allow contacts access to continue.");
        return;
      }

      const { data } = await Contacts.getContactsAsync({
        fields: [
          Contacts.Fields.Name,
          Contacts.Fields.PhoneNumbers,
        ],
        sort: Contacts.SortTypes.FirstName,
      });

      setContacts(data);
    } catch (error) {
      console.error("Could not load contacts:", error);
      Alert.alert("Error", "Unable to load contacts.");
    }
  };

  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Button title="Load contacts" onPress={loadContacts} />

      <FlatList
        data={contacts}
        keyExtractor={(item) => item.id}
        renderItem={({ item }) => (
          <View style={{ paddingVertical: 12 }}>
            <Text>{item.name ?? "Unnamed contact"}</Text>
            <Text>{item.phoneNumbers?.[0]?.number ?? "No phone number"}</Text>
          </View>
        )}
        ListEmptyComponent={<Text>No contacts loaded or no contacts found.</Text>}
        ItemSeparatorComponent={() => (
          <View style={{ height: 1, backgroundColor: "#ddd" }} />
        )}
      />
    </View>
  );
}
```

### Important `FlatList` props

| Prop | Purpose |
|---|---|
| `data` | Array to render |
| `keyExtractor` | Returns a stable unique key for each item |
| `renderItem` | UI for one item; receives `{ item, index, separators }` |
| `ListEmptyComponent` | UI shown when there are no items |
| `ItemSeparatorComponent` | Component between rows |
| `ListHeaderComponent` / `ListFooterComponent` | Optional header/footer |

**Output on screen (example):**
```text
Load contacts

Amit Patel
+91 9000000001
----------------
Rahul Sharma
No phone number
```

## 9. Phone numbers

A contact can have multiple phone numbers, so `phoneNumbers` is an array.

```js
phoneNumbers: [
  { number: "+91 9876543210", label: "mobile", isPrimary: true },
  { number: "+91 1111111111", label: "work", isPrimary: false }
]
```

Common properties:
- `number`: phone number string.
- `label`: e.g. `"mobile"`, `"work"`, `"home"`; labels can vary.
- `isPrimary`: may identify the primary number when available.

Access safely:

```js
const firstPhone = contact.phoneNumbers?.[0]?.number;
const secondPhone = contact.phoneNumbers?.[1]?.number;
```

Display all numbers:

```js
contact.phoneNumbers?.map((phone, index) => (
  <Text key={`${contact.id}-phone-${index}`}>
    {phone.label ?? "Phone"}: {phone.number}
  </Text>
));
```

If no number exists, show a fallback instead of assuming `[0]` exists.

## 10. Email addresses

`emails` is also an array because a contact may have multiple email addresses.

```js
emails: [
  { email: "rahul@example.com", label: "work", isPrimary: true },
  { email: "rahul@personal.com", label: "home", isPrimary: false }
]
```

Common properties:
- `email`: email address.
- `label`: e.g. `"work"` or `"home"`.
- `isPrimary`: primary marker if available.

Safe access:

```js
const firstEmail = contact.emails?.[0]?.email ?? "No email";
```

## 11. Contact photos

Ask for the image field:

```js
const { data } = await Contacts.getContactsAsync({
  fields: [Contacts.Fields.Name, Contacts.Fields.Image],
});
```

Photo information may look like:

```js
{
  imageAvailable: true,
  image: { uri: "file://..." }
}
```

If there is no photo, `imageAvailable` may be `false` and `image` may be missing or `null`.

```jsx
{item.imageAvailable && item.image?.uri ? (
  <Image
    source={{ uri: item.image.uri }}
    style={{ width: 52, height: 52, borderRadius: 26 }}
  />
) : (
  <Text>{item.name?.[0] ?? "?"}</Text>
)}
```

Remember to import `Image` from `react-native`. Photo availability and URI behavior may differ by platform.

## 12. Search and filter contacts

A simple client-side search uses JavaScript `filter()` and `includes()`.

```js
const [query, setQuery] = useState("");

const filteredContacts = contacts.filter((contact) => {
  const name = contact.name?.toLowerCase() ?? "";
  const phone = contact.phoneNumbers?.map((p) => p.number).join(" ") ?? "";
  const email = contact.emails?.map((e) => e.email).join(" ") ?? "";
  const q = query.trim().toLowerCase();

  return name.includes(q) ||
    phone.toLowerCase().includes(q) ||
    email.toLowerCase().includes(q);
});
```

Input:
```jsx
<TextInput
  value={query}
  onChangeText={setQuery}
  placeholder="Search name, phone, or email"
/>
```

Use `filteredContacts` as the `FlatList` `data` prop.

**Example:** query `"rahul"` displays contacts whose name, phone, or email contains `"rahul"` (case-insensitively). An empty query matches all contacts.

Important:
- `filter()` returns a new array; it does not modify the original array.
- `includes()` checks whether a string contains the query.
- `toLowerCase()` makes text matching case-insensitive.
- For very large contact lists, consider debouncing search or using supported native query options.

## 13. Add a contact — `addContactAsync()`

Use this when you want to create a contact programmatically.

```js
const contact = {
  [Contacts.Fields.FirstName]: "Rahul",
  [Contacts.Fields.LastName]: "Sharma",
  phoneNumbers: [
    { label: "mobile", number: "+91 9876543210" },
  ],
  emails: [
    { label: "work", email: "rahul@example.com" },
  ],
  company: "Example Ltd",
  jobTitle: "Developer",
};

const contactId = await Contacts.addContactAsync(contact);
console.log(contactId);
```

**Illustrative output:**
```text
12345
```

The returned value is the new contact's identifier in the source notes. Actual return shape can vary by Expo SDK, so check your installed version's API reference.

### Important input properties
- Name fields: `firstName`, `lastName` or the relevant `Contacts.Fields` constants.
- `phoneNumbers`: array of `{ label, number }`.
- `emails`: array of `{ label, email }`.
- Optional fields include `company`, `jobTitle`, birthday, and note when supported.

**Android note:** direct insertion may be affected by the device's contacts-account configuration. If it fails, opening the native contact form can be a more user-friendly approach.

## 14. Native contact form — `presentFormAsync()`

This opens the device's native contact form. The user can review, save, or cancel.

### Create a new contact

```js
const result = await Contacts.presentFormAsync(
  null,
  {
    [Contacts.Fields.FirstName]: "Rahul",
    [Contacts.Fields.LastName]: "Sharma",
    phoneNumbers: [
      { label: "mobile", number: "+91 9876543210" },
    ],
  },
  { isNew: true }
);

console.log(result);
```

### Edit an existing contact

```js
await Contacts.presentFormAsync(contact.id);
```

The source notes describe the arguments as:
1. `contactId`: existing contact ID, or `null` for a new contact.
2. `contact`: optional initial/default contact data.
3. `options`: additional options such as `{ isNew: true }`.

**Possible result shown in the source notes:**
```js
{ contactId: "12345", action: "saved" }
```
or:
```js
{ action: "canceled" }
```

The exact result fields can depend on the installed Expo SDK/platform. Use the result documented for your SDK version.

## 15. Update a contact

The source file mentions `updateContactAsync()` but does not provide a complete example or output. Its exact signature should be checked against the Expo SDK version installed in the project.

Conceptually, updating requires:
- Permission to access contacts.
- The existing contact's ID.
- The changed contact fields.
- Error handling.
- Refreshing the displayed contact list after the update.

For a simple edit flow, `presentFormAsync(contact.id)` opens the native editor and lets the user save or cancel.

## 16. Delete a contact — `removeContactAsync()`

**Warning:** deletion changes the device's contacts. Ask for confirmation before deleting.

```js
await Contacts.removeContactAsync(contact.id);
```

The source notes describe this method as accepting the contact ID.

### Safe UI update pattern

```js
try {
  await Contacts.removeContactAsync(contact.id);

  setContacts((previous) =>
    previous.filter((item) => item.id !== contact.id)
  );

  console.log("Contact deleted");
} catch (error) {
  console.error("Delete failed:", error);
}
```

**Output on success:**
```text
Contact deleted
```

The `console.log` is our app message; it is not the method's returned value. Refreshing/removing the item from state updates the UI.

## 17. Open phone, SMS, email, or website — `Linking`

`Linking` is a React Native API, not a method from `expo-contacts`.

```js
import { Linking } from "react-native";
```

### Common URL schemes

| Action | Example |
|---|---|
| Website | `https://example.com` |
| Open dialer | `tel:+919876543210` |
| SMS app | `sms:+919876543210` |
| Email app | `mailto:rahul@example.com` |

Open the dialer for a contact:

```js
const phone = contact.phoneNumbers?.[0]?.number;

if (!phone) {
  console.log("No phone number");
  return;
}

try {
  await Linking.openURL(`tel:${phone}`);
} catch (error) {
  console.error("Could not open phone app:", error);
}
```

`Linking.openURL(url)` asks the operating system to open the URL with a compatible app. It does not guarantee a call will be placed; `tel:` generally opens the dialer.

### `canOpenURL()`

```js
const supported = await Linking.canOpenURL(`tel:${phone}`);
console.log(supported);
```

Output:
```js
true
```
or:
```js
false
```

If it returns `false`, show a friendly message. Availability and platform configuration can affect the result.

## 18. Export a contact — `writeContactToFileAsync()`

This creates a VCF/vCard file for a contact.

```js
const fileUri = await Contacts.writeContactToFileAsync(contact.id);
console.log(fileUri);
```

**Illustrative output:**
```text
file:///.../Rahul.vcf
```

The result is a file URI in the source notes. The exact path varies by platform.

A VCF file can contain contact information such as a name, phone number, email, and work details. It can be imported into compatible contacts apps.

### Export and share flow

```text
Select contact → writeContactToFileAsync(contact.id)
               → receive file URI → share/open the VCF file
```

The source notes mention `expo-sharing` as an option for sharing the exported file. File access and sharing behavior vary by platform.

## 19. Main methods — quick reference

| Method | Purpose | Input / options | Typical result |
|---|---|---|---|
| `getPermissionsAsync()` | Check current permission | None | Permission object |
| `requestPermissionsAsync()` | Request permission | None | Permission object |
| `getContactsAsync(options?)` | Read contacts | `fields`, `sort`, and supported filters/pagination options | Object containing `data` and paging flags |
| `addContactAsync(contact)` | Create a contact | Contact object | New contact ID/result, SDK-dependent |
| `presentFormAsync(contactId, contact?, options?)` | Open native create/edit form | ID or `null`, optional initial contact, options | Save/cancel result, SDK-dependent |
| `updateContactAsync(...)` | Update a contact | Verify exact signature for installed SDK | Not specified in source notes |
| `removeContactAsync(contactId)` | Delete a contact | Contact ID | Do not assume a specific return value |
| `writeContactToFileAsync(contactId)` | Export contact | Contact ID | File URI |

## 20. Common mistakes to avoid

1. **Forgetting permission:** check/request permission before accessing contacts.
2. **Assuming the result is an array:** use `const { data } = await Contacts.getContactsAsync()`.
3. **Assuming every contact has a phone/email/photo:** use optional chaining and fallbacks.
4. **Loading every field unnecessarily:** request only needed fields.
5. **Using `map()` for a huge list:** prefer `FlatList`.
6. **Deleting without confirmation:** show a confirmation dialog and handle errors.
7. **Assuming platform parity:** fields, images, native forms, and contact storage can differ.
8. **Hard-coding output paths or IDs:** IDs and file URIs vary by device.
9. **Assuming examples are exact API contracts:** check the API documentation for the Expo SDK installed in your project.
10. **Ignoring privacy:** contacts contain personal data; request only needed access and handle it securely.

## 21. Final revision checklist

- [ ] Install: `npx expo install expo-contacts`
- [ ] Import: `import * as Contacts from "expo-contacts"`
- [ ] Check/request permission and inspect `permission.granted`
- [ ] Read contacts with `getContactsAsync()` and use `result.data`
- [ ] Understand contact properties and optional values
- [ ] Select fields with `Contacts.Fields`
- [ ] Sort with `Contacts.SortTypes`
- [ ] Display with `FlatList`
- [ ] Search using `filter()`, `includes()`, and `toLowerCase()`
- [ ] Read multiple phone numbers/emails safely
- [ ] Handle profile images and fallback avatars
- [ ] Know `addContactAsync()` and `presentFormAsync()`
- [ ] Understand the purpose of `updateContactAsync()` and verify its SDK signature
- [ ] Delete with confirmation using `removeContactAsync()`
- [ ] Use `Linking` for phone/SMS/email/website actions
- [ ] Export a VCF using `writeContactToFileAsync()`
- [ ] Handle errors, empty lists, platform differences, and privacy

---

**Study tip:** Learn the permission flow first, then `getContactsAsync()` and the Contact Object. After that, learn fields/sorting/display/search, and finally add/edit/delete/export operations.

---

# 22. Loading State with `ActivityIndicator`

When contacts are being loaded, show a loading indicator so the user knows the app is working.

### Important state

```jsx
const [contacts, setContacts] = useState([]);
const [loading, setLoading] = useState(false);
```

- `loading`: tells whether the app is currently loading.
- `ActivityIndicator`: displays a spinning loader.
- `finally`: runs whether the operation succeeds or fails, so loading can be switched off.

### Complete loading pattern

```jsx
import { useState } from "react";
import {
  View,
  Text,
  Button,
  FlatList,
  ActivityIndicator,
  Alert,
} from "react-native";
import * as Contacts from "expo-contacts";

export default function ContactsScreen() {
  const [contacts, setContacts] = useState([]);
  const [loading, setLoading] = useState(false);

  const loadContacts = async () => {
    setLoading(true);

    try {
      let permission = await Contacts.getPermissionsAsync();

      if (!permission.granted) {
        permission = await Contacts.requestPermissionsAsync();
      }

      if (!permission.granted) {
        Alert.alert(
          "Permission required",
          "Allow contacts access to load contacts."
        );
        return;
      }

      const result = await Contacts.getContactsAsync({
        fields: [
          Contacts.Fields.Name,
          Contacts.Fields.PhoneNumbers,
          Contacts.Fields.Emails,
        ],
        sort: Contacts.SortTypes.FirstName,
      });

      setContacts(result.data);
    } catch (error) {
      console.error("Load contacts failed:", error);
      Alert.alert("Error", "Could not load contacts.");
    } finally {
      setLoading(false);
    }
  };

  return (
    <View style={{ flex: 1, padding: 16 }}>
      <Button title="Load contacts" onPress={loadContacts} />

      {loading ? (
        <ActivityIndicator size="large" />
      ) : (
        <FlatList
          data={contacts}
          keyExtractor={(item) => item.id}
          renderItem={({ item }) => (
            <View style={{ paddingVertical: 10 }}>
              <Text>{item.name ?? "Unnamed contact"}</Text>
              <Text>
                {item.phoneNumbers?.[0]?.number ?? "No phone number"}
              </Text>
            </View>
          )}
          ListEmptyComponent={<Text>No contacts loaded or no contacts found.</Text>}
        />
      )}
    </View>
  );
}
```

**What happens?**

1. The user taps **Load contacts**.
2. `loading` becomes `true`.
3. The spinner appears.
4. The app checks permission and loads contacts.
5. `finally` sets `loading` to `false`, so the spinner disappears.

**Remember:** `return` inside `try` still runs the `finally` block.

---

# 23. Search Improvements and “No Results Found”

The basic search section already covers `filter()`, `includes()`, and `toLowerCase()`. This section adds UI states and separate search patterns.

### A. Search by name, phone, or email

```jsx
const [query, setQuery] = useState("");

const filteredContacts = contacts.filter((contact) => {
  const q = query.trim().toLowerCase();

  const name = contact.name?.toLowerCase() ?? "";
  const phone =
    contact.phoneNumbers?.map((p) => p.number ?? "").join(" ").toLowerCase() ?? "";
  const email =
    contact.emails?.map((e) => e.email ?? "").join(" ").toLowerCase() ?? "";

  return (
    name.includes(q) ||
    phone.includes(q) ||
    email.includes(q)
  );
});
```

Use it in the UI:

```jsx
<TextInput
  value={query}
  onChangeText={setQuery}
  placeholder="Search name, phone, or email"
  autoCapitalize="none"
/>

<FlatList
  data={filteredContacts}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => (
    <Text>{item.name ?? "Unnamed contact"}</Text>
  )}
  ListEmptyComponent={
    <Text>
      {query.trim()
        ? "No matching contacts found."
        : "No contacts available."}
    </Text>
  }
/>
```

### Expected behavior

| Situation | UI message |
|---|---|
| Contacts have not loaded or list is empty | `No contacts available.` |
| Search text does not match any contact | `No matching contacts found.` |
| Search box is cleared | All loaded contacts appear |

### B. Search by phone only

```js
const phoneResults = contacts.filter((contact) =>
  (contact.phoneNumbers ?? []).some((p) =>
    (p.number ?? "").includes(query.trim())
  )
);
```

### C. Search by email only

```js
const emailResults = contacts.filter((contact) =>
  (contact.emails ?? []).some((e) =>
    (e.email ?? "").toLowerCase().includes(query.trim().toLowerCase())
  )
);
```

**Important concepts**

- `filter()` returns a new array; it does not change the original array.
- `some()` returns `true` if at least one array item matches.
- `trim()` removes spaces from the beginning and end of the query.
- An empty query matches all contacts in the combined search example. If you need a different behavior, handle `query.trim() === ""` separately.
- For very large lists, consider debouncing input to reduce repeated filtering.

---

# 24. Add Contact Using a Form and Validate Input

`addContactAsync()` creates a contact programmatically. A form is useful when the user should enter a name and phone number.

This is a simple example with a name and phone field. It demonstrates validation and then calls `addContactAsync()`. Check the return value and API behavior for your installed Expo SDK.

```jsx
import { useState } from "react";
import {
  View,
  Text,
  TextInput,
  Button,
  Alert,
} from "react-native";
import * as Contacts from "expo-contacts";

export default function AddContactScreen() {
  const [name, setName] = useState("");
  const [phone, setPhone] = useState("");
  const [saving, setSaving] = useState(false);

  const saveContact = async () => {
    const cleanName = name.trim();
    const cleanPhone = phone.trim();

    if (!cleanName) {
      Alert.alert("Name required", "Enter a contact name.");
      return;
    }

    if (!cleanPhone) {
      Alert.alert("Phone required", "Enter a phone number.");
      return;
    }

    setSaving(true);

    try {
      let permission = await Contacts.getPermissionsAsync();

      if (!permission.granted) {
        permission = await Contacts.requestPermissionsAsync();
      }

      if (!permission.granted) {
        Alert.alert("Permission denied", "Contacts access is required.");
        return;
      }

      const contact = {
        [Contacts.Fields.FirstName]: cleanName,
        phoneNumbers: [
          { label: "mobile", number: cleanPhone },
        ],
      };

      const result = await Contacts.addContactAsync(contact);

      Alert.alert("Contact saved", `Result: ${String(result)}`);
      setName("");
      setPhone("");
    } catch (error) {
      console.error("Could not add contact:", error);
      Alert.alert(
        "Could not save",
        "The contact could not be saved. Check the device and Expo SDK behavior."
      );
    } finally {
      setSaving(false);
    }
  };

  return (
    <View style={{ padding: 16, gap: 12 }}>
      <Text>Name</Text>
      <TextInput
        value={name}
        onChangeText={setName}
        placeholder="Enter name"
        autoCapitalize="words"
        style={{ borderWidth: 1, padding: 10, borderRadius: 8 }}
      />

      <Text>Phone number</Text>
      <TextInput
        value={phone}
        onChangeText={setPhone}
        placeholder="Enter phone number"
        keyboardType="phone-pad"
        style={{ borderWidth: 1, padding: 10, borderRadius: 8 }}
      />

      <Button
        title={saving ? "Saving..." : "Save contact"}
        onPress={saveContact}
        disabled={saving}
      />
    </View>
  );
}
```

### Example validation outputs

- If name is blank: `Name required — Enter a contact name.`
- If phone is blank: `Phone required — Enter a phone number.`
- If permission is denied: `Permission denied — Contacts access is required.`
- If saving succeeds: a `Contact saved` alert appears.

The exact result returned by `addContactAsync()` depends on the installed SDK, so the example converts it to text rather than assuming every version returns the same value.

### Native form alternative

If you prefer the device's built-in contact editor, use the `presentFormAsync()` pattern already described in Section 14. It lets the user review the details and save or cancel in the native UI.

---

# 25. Delete Contact with Confirmation

Deleting a contact changes the device's contact book. Always ask the user before deleting.

### Confirmation pattern

```jsx
import { Alert } from "react-native";
import * as Contacts from "expo-contacts";

const confirmDeleteContact = (contact) => {
  Alert.alert(
    "Delete contact?",
    `Are you sure you want to delete ${contact.name ?? "this contact"}?`,
    [
      { text: "Cancel", style: "cancel" },
      {
        text: "Delete",
        style: "destructive",
        onPress: () => deleteContact(contact),
      },
    ]
  );
};

const deleteContact = async (contact) => {
  try {
    await Contacts.removeContactAsync(contact.id);

    setContacts((previous) =>
      previous.filter((item) => item.id !== contact.id)
    );

    Alert.alert("Deleted", "The contact was deleted.");
  } catch (error) {
    console.error("Delete failed:", error);
    Alert.alert("Error", "Could not delete the contact.");
  }
};
```

Add a button to each contact row:

```jsx
<FlatList
  data={contacts}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => (
    <View
      style={{
        flexDirection: "row",
        justifyContent: "space-between",
        alignItems: "center",
        paddingVertical: 10,
      }}
    >
      <Text>{item.name ?? "Unnamed contact"}</Text>
      <Button
        title="Delete"
        color="red"
        onPress={() => confirmDeleteContact(item)}
      />
    </View>
  )}
/>
```

### Output flow

```text
Tap Delete
   ↓
Confirmation dialog
   ↓
Cancel ──→ Nothing changes
   ↓
Delete
   ↓
removeContactAsync(contact.id)
   ↓
Remove item from React state
```

**Important:** `setContacts(...)` updates the visible list in the app. It does not itself delete anything from the device. The actual deletion is performed by `removeContactAsync()`.

---

# 26. Delete Multiple Selected Contacts

For a bulk-delete screen, keep track of selected contact IDs. The following is a simple pattern; use it only after verifying that your installed Expo SDK supports the needed delete operation for each contact.

```jsx
const [selectedIds, setSelectedIds] = useState([]);

const toggleSelection = (contactId) => {
  setSelectedIds((previous) =>
    previous.includes(contactId)
      ? previous.filter((id) => id !== contactId)
      : [...previous, contactId]
  );
};

const deleteSelectedContacts = async () => {
  if (selectedIds.length === 0) {
    Alert.alert("Nothing selected", "Select at least one contact.");
    return;
  }

  Alert.alert(
    "Delete selected contacts?",
    `Delete ${selectedIds.length} selected contact(s)?`,
    [
      { text: "Cancel", style: "cancel" },
      {
        text: "Delete",
        style: "destructive",
        onPress: async () => {
          try {
            for (const id of selectedIds) {
              await Contacts.removeContactAsync(id);
            }

            setContacts((previous) =>
              previous.filter((item) => !selectedIds.includes(item.id))
            );
            setSelectedIds([]);
            Alert.alert("Done", "Selected contacts were deleted.");
          } catch (error) {
            console.error("Bulk delete failed:", error);
            Alert.alert(
              "Some contacts may not have been deleted",
              "Reload the contact list and check which contacts remain."
            );
          }
        },
      },
    ]
  );
};
```

A selectable row can use:

```jsx
<Pressable onPress={() => toggleSelection(item.id)}>
  <Text>
    {selectedIds.includes(item.id) ? "☑" : "□"}{" "}
    {item.name ?? "Unnamed contact"}
  </Text>
</Pressable>
```

Import `Pressable` from `react-native`.

**Bulk-delete caution:** this example deletes sequentially. If one deletion fails, earlier deletions may already have succeeded. Reload the contacts after an error rather than assuming the operation was all-or-nothing.

---

# 27. Call, SMS, and Email Buttons Inside a Contact Row

`Linking` belongs to React Native, not to `expo-contacts`. It asks the operating system to open a compatible app.

```jsx
import { Linking, Alert, View, Text, Button } from "react-native";

const openContactPhone = async (contact) => {
  const phone = contact.phoneNumbers?.[0]?.number;

  if (!phone) {
    Alert.alert("No phone number", "This contact has no saved number.");
    return;
  }

  try {
    await Linking.openURL(`tel:${phone}`);
  } catch (error) {
    Alert.alert("Unable to open dialer", "Try calling from the phone app.");
  }
};

const openContactSMS = async (contact) => {
  const phone = contact.phoneNumbers?.[0]?.number;

  if (!phone) {
    Alert.alert("No phone number", "This contact has no saved number.");
    return;
  }

  try {
    await Linking.openURL(`sms:${phone}`);
  } catch (error) {
    Alert.alert("Unable to open SMS", "No compatible messaging app was opened.");
  }
};

const openContactEmail = async (contact) => {
  const email = contact.emails?.[0]?.email;

  if (!email) {
    Alert.alert("No email", "This contact has no saved email address.");
    return;
  }

  try {
    await Linking.openURL(`mailto:${email}`);
  } catch (error) {
    Alert.alert("Unable to open email", "No compatible email app was opened.");
  }
};
```

Use these handlers in `FlatList`:

```jsx
<FlatList
  data={contacts}
  keyExtractor={(item) => item.id}
  renderItem={({ item }) => (
    <View style={{ padding: 12, gap: 6 }}>
      <Text>{item.name ?? "Unnamed contact"}</Text>
      <Button title="Call" onPress={() => openContactPhone(item)} />
      <Button title="SMS" onPress={() => openContactSMS(item)} />
      <Button title="Email" onPress={() => openContactEmail(item)} />
    </View>
  )}
/>
```

**Expected behavior:** the device tries to open the dialer, messaging app, or email app. `tel:` normally opens the dialer; it does not automatically place a call.

For contacts with multiple phone numbers or emails, let the user choose which one to use instead of always using `[0]`.

---

# 28. Export a Contact and Share Its VCF File

`writeContactToFileAsync(contactId)` creates a VCF/vCard file. To open the system sharing sheet, install `expo-sharing`:

```bash
npx expo install expo-sharing
```

### Single-contact export and share

```jsx
import { Button, Alert } from "react-native";
import * as Contacts from "expo-contacts";
import * as Sharing from "expo-sharing";

const exportAndShareContact = async (contact) => {
  try {
    const fileUri = await Contacts.writeContactToFileAsync(contact.id);

    const canShare = await Sharing.isAvailableAsync();

    if (!canShare) {
      Alert.alert("Sharing unavailable", `VCF file created at:\n${fileUri}`);
      return;
    }

    await Sharing.shareAsync(fileUri, {
      mimeType: "text/vcard",
      dialogTitle: "Share contact",
    });
  } catch (error) {
    console.error("Export/share failed:", error);
    Alert.alert("Export failed", "Could not export or share this contact.");
  }
};
```

Attach it to a row:

```jsx
<Button
  title="Export / Share"
  onPress={() => exportAndShareContact(item)}
/>
```

**Flow:**

```text
Choose contact
   ↓
writeContactToFileAsync(contact.id)
   ↓
Get VCF file URI
   ↓
Check Sharing.isAvailableAsync()
   ↓
Sharing.shareAsync(fileUri)
   ↓
User chooses a compatible sharing destination
```

The share sheet and available destinations depend on the device. This code does not guarantee that WhatsApp, email, or another specific app is installed.

### Export multiple contacts one by one

```js
for (const contact of contacts) {
  try {
    const fileUri = await Contacts.writeContactToFileAsync(contact.id);
    console.log(contact.name, fileUri);
  } catch (error) {
    console.error(`Failed to export ${contact.name ?? contact.id}`, error);
  }
}
```

This creates a separate VCF file for each contact. It does **not** combine every contact into one VCF file or automatically share every file.

### Export a specific contact by name

```js
const contact = contacts.find(
  (item) => item.name === "Rahul Sharma"
);

if (contact) {
  const fileUri = await Contacts.writeContactToFileAsync(contact.id);
  console.log(fileUri);
} else {
  console.log("Contact not found");
}
```

Names may not be unique. In a real app, selecting the contact by its ID is safer than relying on the name alone.

---

# 29. Contact Images: Fallbacks and Layout

A contact photo may be unavailable, so never assume `image.uri` exists.

```jsx
import { Image, Text, View } from "react-native";

const ContactAvatar = ({ contact }) => {
  const photoUri = contact.image?.uri;

  if (contact.imageAvailable && photoUri) {
    return (
      <Image
        source={{ uri: photoUri }}
        style={{
          width: 52,
          height: 52,
          borderRadius: 26,
          backgroundColor: "#eeeeee",
        }}
      />
    );
  }

  return (
    <View
      style={{
        width: 52,
        height: 52,
        borderRadius: 26,
        alignItems: "center",
        justifyContent: "center",
        backgroundColor: "#eeeeee",
      }}
    >
      <Text>{contact.name?.trim()?.[0]?.toUpperCase() ?? "?"}</Text>
    </View>
  );
};
```

Usage:

```jsx
<ContactAvatar contact={item} />
```

**Props used in this example**

| Prop | Meaning |
|---|---|
| `source={{ uri: photoUri }}` | Tells `Image` where the image is located |
| `width`, `height` | Sets avatar dimensions |
| `borderRadius` | Makes a square image circular when radius is half the size |
| `imageAvailable` | Indicates whether a contact image is reported as available |

The `52 × 52` size is just a UI choice, not a required Expo Contacts setting. Photo URI availability and behavior can vary by platform.

---

# 30. If Permission Was Denied

Check both `granted` and `canAskAgain`.

```js
const requestContactAccess = async () => {
  let permission = await Contacts.getPermissionsAsync();

  if (!permission.granted && permission.canAskAgain) {
    permission = await Contacts.requestPermissionsAsync();
  }

  if (permission.granted) {
    return true;
  }

  if (!permission.canAskAgain) {
    Alert.alert(
      "Permission needed",
      "Enable Contacts permission in your device settings.",
      [
        { text: "Cancel", style: "cancel" },
        {
          text: "Open settings",
          onPress: () => Linking.openSettings(),
        },
      ]
    );
  } else {
    Alert.alert(
      "Permission denied",
      "Contacts permission is needed to load contacts."
    );
  }

  return false;
};
```

Required imports:

```js
import { Alert, Linking } from "react-native";
import * as Contacts from "expo-contacts";
```

`Linking.openSettings()` opens the app's settings page when supported. It does not grant permission automatically; the user must change the setting.

---

# 31. Extra Contact Fields and Platform Differences

The original notes also mention the `contactType` property and fields such as addresses, websites, and social profiles. These may not be available for every contact or platform.

### `contactType`

A contact may include a type such as:

```js
{
  id: "22",
  contactType: "person",
  name: "Rajesh Ranjan"
}
```

Treat this as an illustrative example, not a guaranteed object shape. Consult the type definitions for the installed Expo SDK to confirm supported values.

### Other possible contact details

Depending on the platform, contact data, and SDK, contact records may also support information such as:

- Postal addresses
- Websites
- Social profiles
- Other name components

Do not assume that every `Contacts.Fields` constant exists in every Expo SDK version. Use the fields documented for the SDK installed in your project.

---

# 32. Common Practical Mistakes — Expanded

1. **No permission check:** check existing permission and request it when needed.
2. **Wrong result access:** `getContactsAsync()` returns a result object; the contacts are in `result.data`.
3. **Missing optional values:** use `?.` and `??` for optional names, numbers, emails, and images.
4. **No loading state:** use `loading` and `ActivityIndicator` during async work.
5. **Spinner never stops:** use `finally` to reset loading state after success, failure, or an early return.
6. **No empty state:** distinguish an empty contact list from a search with no matches.
7. **Unsafe deletion:** confirm before deleting and catch errors.
8. **Stale UI after deletion:** update React state after a successful deletion or reload contacts.
9. **Assuming bulk deletion is atomic:** earlier deletions may succeed before a later one fails.
10. **Assuming a phone call is placed:** `Linking.openURL("tel:...")` normally opens the dialer.
11. **Assuming a sharing app exists:** check `Sharing.isAvailableAsync()` and let the user choose a destination.
12. **Assuming output IDs or file URIs are fixed:** they vary by device and SDK.
13. **Assuming every field exists on Android and iOS:** platform and provider differences matter.
14. **Ignoring privacy:** request only the fields you need and do not expose contact information unnecessarily.
15. **Ignoring the installed SDK version:** verify method signatures and return values against the SDK in your project.

---

# 33. Updated Final Revision Checklist

- [ ] Install `expo-contacts` using `npx expo install expo-contacts`.
- [ ] Import `* as Contacts`.
- [ ] Check/request permission and handle denied access.
- [ ] Read contacts with `getContactsAsync()` and use `result.data`.
- [ ] Understand the contact object and optional properties.
- [ ] Select fields with `Contacts.Fields`.
- [ ] Sort using supported `Contacts.SortTypes`.
- [ ] Display contacts with `FlatList`.
- [ ] Show loading state with `ActivityIndicator`.
- [ ] Handle empty contact lists and no-search-results states.
- [ ] Search by name, phone, and email.
- [ ] Display multiple phone numbers and email addresses safely.
- [ ] Display contact photos with a fallback avatar.
- [ ] Add contacts with input validation.
- [ ] Use `presentFormAsync()` for native create/edit forms.
- [ ] Verify the exact `updateContactAsync()` signature for your SDK.
- [ ] Confirm before deleting contacts.
- [ ] Understand the risks of sequential bulk deletion.
- [ ] Use `Linking` to open the dialer, SMS, email, or settings.
- [ ] Export a VCF with `writeContactToFileAsync()`.
- [ ] Share VCF files using `expo-sharing` when available.
- [ ] Handle errors, platform differences, and privacy.

## Study Order

1. Permissions and `getContactsAsync()`
2. Contact object and `Contacts.Fields`
3. `FlatList`, loading, empty states, and images
4. Search and filtering
5. Add and edit contacts
6. Delete and confirm deletion
7. `Linking` for calls/SMS/email
8. Export and share VCF files
9. Error handling, platform differences, and privacy

**Final note:** These examples are practical learning patterns. Check the API documentation for your installed Expo SDK before relying on exact method signatures, return values, platform behavior, or optional fields.

