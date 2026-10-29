# Expo File System (`expo-file-system`) — Compared & Updated Complete Notes

> Easy revision notes for React Native + Expo, compared against the supplied original. Repetition removed; important concepts, APIs, inputs/options, outputs, examples, and cautions retained and gaps patched.
>
> **Version note:** These notes summarize the supplied source. Expo File System APIs and platform support can change by SDK. Check the documentation for your installed Expo SDK before production use.

## 1. What is `expo-file-system`?

`expo-file-system` lets an Expo app manage files and folders on the device: create, read, write, copy, move, rename, delete, list, watch, download, and upload.

```bash
npx expo install expo-file-system
```

```tsx
import { File, Directory, Paths } from "expo-file-system";
```

It does **not** replace:
- **React state:** current UI state.
- **AsyncStorage:** simple key-value data.
- **SecureStore:** small sensitive values such as tokens.
- **Database:** structured records, queries, and relationships.

Use the file system for actual files such as PDFs, images, videos, text, and JSON files.

## 2. Core concepts

| Term | Simple meaning |
|---|---|
| File | One data item, e.g. `notes.txt` |
| Directory | Folder containing files/directories |
| Path | Describes a location |
| URI | Resource reference, often like `file:///...` |
| Metadata | Information about a file: name, size, type, existence, timestamps |
| MIME type | Describes content format, e.g. `image/jpeg` |
| Persistent file | File the app wants to keep |
| Cache file | Temporary/re-creatable file |

**Important:** A `new File(...)` or `new Directory(...)` creates a JavaScript reference to a location. It does **not** necessarily create the physical file/folder. Use `.create()` when you need the actual resource.

Avoid hard-coded Android/iOS storage paths. Use `Paths`.

## 3. `Paths` — choose a location

`Paths` provides app locations, storage information, and path utilities.

```tsx
import { Paths } from "expo-file-system";
```

### Important locations

| Property | Purpose | Use for |
|---|---|---|
| `Paths.document` | App document directory; files are safe from automatic system deletion | Saved notes, important downloads, user-created files |
| `Paths.cache` | Temporary/re-creatable storage; system may clear it | Previews, temporary downloads, cached files |
| `Paths.bundle` | Resources bundled with the app | App-bundled assets |
| `Paths.availableDiskSpace` | Available internal storage, in bytes | Check free space |
| `Paths.totalDiskSpace` | Total internal storage, in bytes | Check total capacity |
| `Paths.appleSharedContainers` | Apple shared-container information | Platform-specific/advanced use |

`Paths.document`, `Paths.cache`, and `Paths.bundle` are `Directory` objects.

```tsx
console.log(Paths.document.uri);
console.log(Paths.cache.uri);
console.log(Paths.bundle.uri);
console.log(Paths.availableDiskSpace);
console.log(Paths.totalDiskSpace);
```

Example output (device-dependent):
```text
file:///.../Documents/
file:///.../Caches/
<platform-specific bundle URI>
1234567890
64000000000
```

Convert bytes to GiB:
```tsx
const availableGB = Paths.availableDiskSpace / 1024 ** 3;
console.log(availableGB.toFixed(2));
```

### Path utility methods

These help inspect or combine path strings; they do not read the file content.

| Method | Input / purpose | Example output |
|---|---|---|
| `Paths.basename(path)` | Full path | `react-native.txt` |
| `Paths.dirname(path)` | Full path | `/documents/notes` |
| `Paths.extname(path)` | Full path | `.txt` |
| `Paths.join(...parts)` | Path parts | `Documents/notes/note.txt` |
| `Paths.normalize(path)` | Normalize path format | Normalized path string |
| `Paths.parse(path)` | Parse path into parts | Parsed path information |
| `Paths.relative(from, to)` | Relative path between locations | Relative path string |
| `Paths.isAbsolute(path)` | Test whether path is absolute | `true` / `false` |
| `Paths.info(path)` | Inspect path information | Information object |

Example:
```tsx
Paths.basename("/documents/notes/react-native.txt"); // "react-native.txt"
Paths.extname("/documents/notes/react-native.txt");  // ".txt"
Paths.isAbsolute("/documents/notes.txt");            // true (absolute path example)
```

## 4. `Directory` — work with folders

```tsx
const notesDir = new Directory(Paths.document, "notes");
```

This creates a reference to `Documents/notes`, not necessarily the folder itself.

### Properties

| Property | Meaning / output |
|---|---|
| `directory.name` | Folder name, e.g. `"notes"` |
| `directory.uri` | Folder URI string |
| `directory.exists` | `true` / `false` |
| `directory.parentDirectory` | Parent `Directory` object |
| `directory.size` | Size in bytes; may be `null` if unavailable |

### Main methods

| Method | Input / options | Return/output and purpose |
|---|---|---|
| `create(options?)` | `{ idempotent?, intermediates?, overwrite? }` | Creates the directory |
| `createDirectory(name)` | Child folder name | Returns a child `Directory` object and creates the child |
| `createFile(name, mimeType)` | File name and MIME type | Returns a `File` object for the created file |
| `list()` | None | Array of `File` and `Directory` objects inside folder |
| `info()` | None | Metadata object; source lists fields such as `exists`, `size`, `creationTime`, `files` |
| `copy(destination)` | Destination `Directory` | Copies folder; original remains |
| `copySync(destination)` | Destination `Directory` | Synchronous copy |
| `move(destination)` | Destination `Directory` | Moves folder to another location |
| `moveSync(destination)` | Destination `Directory` | Synchronous move |
| `rename(name)` | New name | Renames folder |
| `delete()` | None | Deletes folder; destructive |
| `watch(callback)` | Callback `(event) => {}` | Subscription for folder changes; call `.remove()` to stop |

### `create()` options

- `idempotent: true` — do not fail just because the directory already exists.
- `intermediates: true` — create missing parent folders.
- `overwrite` — controls behavior if the target exists; use carefully because overwriting can destroy data.

```tsx
const notesDir = new Directory(Paths.document, "notes", "college");
notesDir.create({ idempotent: true, intermediates: true });
```

### Create a child folder and file

```tsx
const notesDir = new Directory(Paths.document, "notes");
notesDir.create({ idempotent: true });

const collegeDir = notesDir.createDirectory("college");
const noteFile = collegeDir.createFile("note.txt", "text/plain");
```

Common MIME types: `.txt` → `text/plain`, `.json` → `application/json`, `.pdf` → `application/pdf`, `.png` → `image/png`, `.jpg` → `image/jpeg`.

### List folder contents

```tsx
const items = notesDir.list();

for (const item of items) {
  if (item instanceof Directory) {
    console.log("Folder:", item.name);
  } else {
    console.log("File:", item.name);
  }
}
```

Output example:
```text
File: note1.txt
File: note2.txt
Folder: college
```

`list()` can throw if the folder does not exist or cannot be read. Check `.exists` or use `try/catch`.

### Watch a folder

```tsx
const subscription = notesDir.watch((event) => {
  console.log(event.type);
  console.log(event.target.uri);
});

// Stop watching when finished:
subscription.remove();
```

Possible event types listed in the source: `created`, `modified`, `deleted`, `renamed`.

In React, remove the subscription in `useEffect` cleanup. Platform behavior differs: on iOS, changes inside children may appear as a coarse `modified` event on the directory itself.

### Copy vs move vs rename vs delete

- `copy(destination)` — creates a duplicate; original remains.
- `move(destination)` — changes location.
- `rename(newName)` — changes the name.
- `delete()` — removes the resource. Confirm with the user before deleting important files.

## 5. `File` — work with one file

```tsx
const file = new File(Paths.document, "note.txt");
```

Again, this is only a reference until created or otherwise written/downloaded.

### Properties

| Property | Meaning / example |
|---|---|
| `file.name` | `"note.txt"` |
| `file.extension` | `".txt"` |
| `file.uri` | URI string such as `file:///.../note.txt` |
| `file.parentDirectory` | Parent `Directory` object |
| `file.size` | Size in bytes |
| `file.type` | MIME type, e.g. `"text/plain"` |
| `file.exists` | `true` / `false` |

### Main methods

| Method | Input / options | Return/output and purpose |
|---|---|---|
| `create(options?)` | Source documents show `{ overwrite: true }` as an option | Creates the actual file |
| `write(content)` | String or supported file data | Writes content; replaces existing content in the source examples |
| `text()` | None | `Promise<string>`: reads text |
| `textSync()` | None | String: synchronous text read; can block execution |
| `json()` | None | `Promise` resolving to parsed JSON value |
| `bytes()` | None | `Promise<Uint8Array>`: reads raw bytes |
| `bytesSync()` | None | `Uint8Array`: synchronous byte read |
| `info()` | None | Metadata object; source lists `exists`, `size`, `creationTime`, `modificationTime` |
| `copy(destination)` | Destination `File` or `Directory` | Copies file; original remains |
| `move(destination)` | Destination `File` or `Directory` | Moves file |
| `rename(name)` | New name | Renames file |
| `delete()` | None | Deletes file |
| `open(mode?)` | Optional `FileMode` | Returns `FileHandle` for lower-level access |
| `upload(url, options?)` | Server URL and optional settings | `Promise` with response such as status/body |
| `createUploadTask(url, options?)` | URL and upload options | Upload task for progress/control |
| `downloadFileAsync(url, destination, options?)` | Remote URL, destination `File` or `Directory`, options | `Promise<File>` for downloaded file |
| `createDownloadTask(url, destination, options?)` | URL, destination, optional options | `DownloadTask`; does not start until `downloadAsync()` |
| `pickFileAsync(options?)` | Picker options | Opens the supported system file picker; platform support can vary |

### Create, write, and read a text file

```tsx
import { File, Paths } from "expo-file-system";

const file = new File(Paths.document, "note.txt");
file.create({ overwrite: true });
file.write("Learn Expo File System");

const text = await file.text();
console.log(text);
```

Output:
```text
Learn Expo File System
```

`write()` replaces existing content in the supplied examples. To append, read the old text and write the combined text:

```tsx
const oldText = await file.text();
file.write(oldText + "\nAnother line");
```

### JSON file

```tsx
const file = new File(Paths.document, "user.json");
file.create({ overwrite: true });

file.write(JSON.stringify({ name: "Raj", age: 20 }));

const user = await file.json();
console.log(user.name, user.age);
```

Output:
```text
Raj 20
```

- `JSON.stringify(object)` converts a JS object to a JSON string before writing.
- `file.json()` reads and parses JSON into a JavaScript value.
- Invalid JSON can cause parsing to fail; use `try/catch`.

### `exists` vs `info()`

- `file.exists` → quick boolean.
- `file.info()` → more metadata.

Always check existence where appropriate, handle errors, and be careful with `delete()` or overwrite options.

## 6. `FileHandle` — low-level byte access

Use a handle when you need byte-level reads/writes, offsets, random access, or chunking large files.

```tsx
import { File, FileMode, Paths } from "expo-file-system";

const file = new File(Paths.document, "data.bin");
const handle = file.open(FileMode.ReadOnly);
```

### `FileMode` values listed in the source

| Mode | Meaning |
|---|---|
| `FileMode.ReadOnly` | Read only |
| `FileMode.ReadWrite` | Read and write |
| `FileMode.WriteOnly` | Write only |
| `FileMode.Append` | Write at the end |
| `FileMode.Truncate` | Open for writing and clear existing content |

### Handle properties/methods

| Member | Meaning / output |
|---|---|
| `handle.offset` | Current byte position; can be changed for random access |
| `handle.size` | File size used in the source's chunk-reading example |
| `handle.readBytes(count)` | Reads bytes and returns `Uint8Array` (actual count may be less at end of file) |
| `handle.writeBytes(bytes)` | Writes a `Uint8Array` |
| `handle.close()` | Releases the file handle |

```tsx
const handle = file.open(FileMode.ReadOnly);
try {
  const chunk = await handle.readBytes(100);
  console.log(chunk.length); // number of bytes actually read
  console.log(handle.offset); // current offset
} finally {
  handle.close();
}
```

Set the offset for random access:
```tsx
handle.offset = 1000;
const chunk = await handle.readBytes(500);
```

For large files, process chunks instead of loading everything into memory. Use `await` when performing asynchronous operations on the same handle so offset-sensitive operations happen in the expected order.

**Critical rule:** `open()` → use the handle → `close()` (prefer `try/finally`). An open handle may prevent the file from being moved, deleted, or opened elsewhere.

## 7. Downloading files

Download means copying a remote file onto the device. Reading means opening a file that is already available locally.

### Simple download: `File.downloadFileAsync()`

```tsx
import { File, Paths } from "expo-file-system";

const destination = new File(Paths.document, "course.pdf");

const downloadedFile = await File.downloadFileAsync(
  "https://your-server.com/course.pdf",
  destination,
  {
    idempotent: true,
    headers: { Authorization: `Bearer ${token}` },
    onProgress: ({ bytesWritten, totalBytes }) => {
      if (totalBytes > 0) {
        console.log(`${Math.round(bytesWritten / totalBytes * 100)}%`);
      }
    },
  }
);

console.log(downloadedFile.uri);
console.log(downloadedFile.exists);
console.log(downloadedFile.size);
```

**Inputs**
1. `url`: remote file URL.
2. `destination`: destination `File` or `Directory`.
3. Optional options shown in the source:
   - `idempotent`: controls behavior if the target already exists.
   - `intermediates`: relevant when creating destination directories with missing parents.
   - `headers`: request headers, e.g. `Authorization`.
   - `onProgress({ bytesWritten, totalBytes })`: callback during transfer.
   - `signal`: `AbortSignal` for cancellation.

**Return:** `Promise<File>` for the downloaded file. `bytesWritten` is bytes downloaded so far. `totalBytes` is expected total bytes and may be `-1` if unknown; do not calculate a percentage unless `totalBytes > 0`.

### Cancel a simple download

```tsx
const controller = new AbortController();

const promise = File.downloadFileAsync(url, destination, {
  signal: controller.signal,
});

controller.abort(); // cancels; handle the rejected promise
```

Use `try/catch`; cancellation may reject with an `AbortError`.

### Destination: `File` vs `Directory`

- Pass a `File` when you want a specific filename.
- Pass a `Directory` when you want the downloaded file placed in a folder and the API determines the file name from the download.
- Create destination folders first when needed.
- Use `Paths.document` for files users should keep; use `Paths.cache` for temporary/re-creatable downloads.

### Advanced download: `DownloadTask`

Use a task when you need pause/resume, state, cancellation, or resumable downloads.

```tsx
import { File, Paths } from "expo-file-system";

const destination = new File(Paths.document, "video.mp4");
const task = File.createDownloadTask(
  "https://your-server.com/video.mp4",
  destination,
  {
    onProgress: ({ bytesWritten, totalBytes }) => {
      console.log(bytesWritten, totalBytes);
    },
  }
);

const resultFile = await task.downloadAsync();
console.log(resultFile?.uri);
console.log(task.state);
```

Creating the task does not start downloading; `downloadAsync()` starts it.

| Task member | Meaning/output |
|---|---|
| `task.state` | Current state: `idle`, `active`, `paused`, `completed`, `cancelled`, or `error` |
| `downloadAsync()` | Starts download; resolves with file when completed (source notes it may not return a completed file if paused) |
| `pause()` | Requests pause without waiting for paused state |
| `pauseAsync()` | Requests pause and waits for paused state |
| `resumeAsync()` | Resumes a paused task; may resolve without a completed file if paused again |
| `cancel()` | Cancels the task; create a new task to start again later |
| `addListener("progress", callback)` | Subscribes to progress; returns subscription |
| `savable()` | Returns a savable representation of paused-download state |
| `DownloadTask.fromSavable(savedState)` | Restores a task from saved state |

Remove progress listener:
```tsx
const subscription = task.addListener("progress", ({ bytesWritten, totalBytes }) => {
  console.log(bytesWritten, totalBytes);
});
subscription.remove();
```

Save/restore idea:
```tsx
await task.pauseAsync();
const savedState = task.savable();

// Later:
const restoredTask = DownloadTask.fromSavable(savedState);
const file = await restoredTask.resumeAsync();
```

### Download errors and HTTP status

Use `try/catch` for network/file errors. A server may return an HTTP error (e.g. 401/404/500) even when a transfer request itself completed; inspect the result/status where the API exposes it. Never assume network availability guarantees a successful download.

### Sequential vs parallel downloads

```tsx
for (const item of files) {
  await File.downloadFileAsync(item.url, item.destination);
}
```

This is sequential. `Promise.all()` can start several downloads in parallel, but consider bandwidth, device resources, server limits, and user experience.

## 8. Uploading files

Upload sends a local file to a server. The endpoint and request format must match what your backend accepts.

### Simple upload: `file.upload(url)`

```tsx
const file = new File(Paths.document, "resume.pdf");

if (file.exists) {
  const response = await file.upload("https://your-server.com/upload");
  console.log(response.status);
  console.log(response.body);
}
```

**Return:** Promise resolving to a response object; the supplied source uses `status` and `body`. Check for success status (typically 2xx) and handle errors.

### Controlled upload: `file.createUploadTask(url, options)`

```tsx
import { File, Paths, UploadType } from "expo-file-system";

const file = new File(Paths.document, "resume.pdf");
const task = file.createUploadTask("https://your-server.com/upload", {
  uploadType: UploadType.MULTIPART,
  fieldName: "file",
  onProgress: ({ bytesSent, totalBytes }) => {
    if (totalBytes > 0) {
      console.log(`${Math.round(bytesSent / totalBytes * 100)}% uploaded`);
    }
  },
});

const response = await task.uploadAsync();
console.log(response.status);
```

Important options shown in the source:
- `uploadType: UploadType.MULTIPART` — send as multipart form data.
- `fieldName: "file"` — field name expected by the backend.
- `onProgress({ bytesSent, totalBytes })` — upload progress callback.

Use the task form when progress or more control is required. `expo/fetch` is another option when a normal Fetch-style request is needed.

## 9. File selection / document picker

File selection is separate from uploading: selecting a file does not automatically upload it.

### Option A: `expo-document-picker`

Install:
```bash
npx expo install expo-document-picker
```

```tsx
import * as DocumentPicker from "expo-document-picker";

const result = await DocumentPicker.getDocumentAsync({
  copyToCacheDirectory: true,
  multiple: false,
});

if (!result.canceled) {
  const asset = result.assets[0];
  console.log(asset.name);
  console.log(asset.size);    // bytes, may be undefined
  console.log(asset.uri);
  console.log(asset.mimeType);
}
```

| Option/property | Meaning |
|---|---|
| `copyToCacheDirectory: true` | Requests a cache copy for easier immediate access; cache is temporary |
| `multiple: false` | Select one file |
| `multiple: true` | Allow multiple files; use `result.assets` |
| `result.canceled` | User cancelled the picker |
| `result.assets` | Selected file asset array |
| `asset.name` | File name |
| `asset.size` | Size in bytes, if available |
| `asset.uri` | URI of selected resource |
| `asset.mimeType` | MIME type, if available |

If the app needs a persistent copy, copy the selected file to `Paths.document`. Use a safe/unique destination name and handle existing-file behavior.

### Option B: `File.pickFileAsync()`

The modern File System API also lists `File.pickFileAsync()` for opening a system file picker. Its exact options and platform support depend on the installed Expo SDK; check the matching docs before relying on it. The source recommends `expo-document-picker` for its beginner example.



## 9.1 Important comparison additions — persistent picker copies

The original material includes a practical step that is easy to miss: **selecting a file is not the same as saving a permanent app-owned copy**. `copyToCacheDirectory: true` makes immediate file access easier, but the cache copy is temporary.

If you need to keep the selected file in app storage, copy it to `Paths.document`:

```tsx
import { Directory, File, Paths } from "expo-file-system";

// selectedUri and selectedName come from the successful picker result.
const destinationDirectory = new Directory(Paths.document, "SelectedFiles");
destinationDirectory.create({
  idempotent: true,
  intermediates: true,
});

const sourceFile = new File(selectedUri);
const destinationFile = new File(destinationDirectory, selectedName);
sourceFile.copy(destinationFile);
```

**Expected result:** the selected file is copied into the app's `Documents/SelectedFiles` location. The source file is not intentionally deleted by this copy operation.

Important:
- Use a unique or sanitized filename if users can select files with the same name.
- Check the installed SDK's overwrite behavior before copying to a destination that already exists.
- Catch errors around picker and copy operations in production code.

## 9.2 `expo/fetch` — another file-transfer option

The original material also mentions `expo/fetch`. It provides a Fetch-style approach for network requests that involve files. Choose it when you need a normal HTTP request pattern; use the dedicated `File.upload()` or download/task APIs when their file-specific behavior better matches the job.

| Need | API to consider |
|---|---|
| Simple file upload and response | `file.upload(url)` |
| Upload progress / task control | `file.createUploadTask(url, options)` |
| Fetch-style HTTP request involving a file | `expo/fetch` |
| Simple remote download to local storage | `File.downloadFileAsync(url, destination, options?)` |
| Pause/resume or saved download state | `File.createDownloadTask(...)` |

The exact supported request/body options depend on the installed Expo SDK. Follow the documentation for that SDK rather than assuming browser `fetch` examples work unchanged.

## 9.3 Streams — advanced concept

The original material names **Streams** as an advanced option alongside `FileHandle` and transfer tasks, but does not explain a complete Streams API or its exact methods. Keep this as an awareness-level concept until you consult the docs matching your Expo SDK.

A stream processes data progressively instead of requiring the whole file to be loaded into memory at once. This can be useful for large files or ongoing data flows.

| Approach | Mental model |
|---|---|
| `file.text()` / `file.bytes()` | Read the file content as a whole |
| `FileHandle` with chunks | Read/write controlled byte ranges using an offset |
| Streams | Process data progressively as chunks become available |

Do not invent or rely on stream method names from these notes; verify the exact API and platform support in your installed SDK's documentation.

## 9.4 Android directory selection

The original material mentions a supported **Android directory-picker API** as another option. This is separate from selecting an individual document.

- Choose `expo-document-picker` when the user needs to select a document/file.
- Choose `File.pickFileAsync()` when its supported platform behavior fits the project.
- Consider the Android directory picker only when the user needs to choose a folder and the installed SDK supports that API.

The source material does not provide the directory-picker method signature or options, so check the SDK-specific docs before implementing it.

## 9.5 Final comparison checklist

After comparing the original source with these cleaned notes:

- Core concepts and the main `Paths`, `Directory`, `File`, `FileHandle`, download, upload, and document-picker APIs are covered in the main sections.
- The extra sections above preserve the useful source details about keeping a picked file permanently, `expo/fetch`, Streams as an advanced concept, and Android directory selection.
- Repetitive explanations, duplicate examples, and long classroom/demo flows remain intentionally removed.
- Some API behavior is SDK/platform-dependent. These notes are a learning reference, not a substitute for the documentation matching your installed Expo SDK.


## 10. Important platform and storage cautions

- Actual physical paths and URIs vary by platform; use `Paths`.
- `Paths.cache` may be cleared by the system. Do not keep the only copy of an important file there.
- Platform support differs for file pickers and directory pickers; the source mentions an Android directory-picker API.
- Directory watching differs across platforms, especially on iOS.
- File and network operations can fail: use `try/catch`.
- A file's extension alone does not prove its content type.
- File size values are in bytes.
- Avoid synchronous operations for potentially large files because they can block execution.
- Always close `FileHandle` instances.
- Check before overwriting/deleting important files.
- Use modern `File`, `Directory`, and `Paths` APIs from `expo-file-system`. The source mentions older APIs such as `FileSystem.downloadAsync()` and `FileSystem.createDownloadResumable()` as legacy; do not mix legacy examples with modern API examples without checking your SDK documentation.

## 11. Quick revision sheet

| Need | Use |
|---|---|
| App's persistent files | `Paths.document` |
| Temporary/cache files | `Paths.cache` |
| Bundled app resources | `Paths.bundle` |
| Create folder | `new Directory(...).create()` |
| Create file | `new File(...).create()` or `directory.createFile()` |
| List folder | `directory.list()` |
| Check existence | `.exists` |
| Read text | `await file.text()` |
| Read JSON | `await file.json()` |
| Read raw bytes | `await file.bytes()` |
| Write/replace content | `file.write(content)` |
| File metadata | `file.info()` |
| Copy | `.copy(destination)` |
| Move | `.move(destination)` |
| Rename | `.rename(name)` |
| Delete | `.delete()` |
| Watch folder | `.watch(callback)` then `subscription.remove()` |
| Low-level bytes | `file.open(FileMode...)` → read/write → `close()` |
| Simple download | `File.downloadFileAsync(url, destination, options?)` |
| Pause/resume download | `File.createDownloadTask(...)` |
| Simple upload | `file.upload(url)` |
| Upload with progress | `file.createUploadTask(url, options)` |
| Pick a document | `DocumentPicker.getDocumentAsync(...)` or supported `File.pickFileAsync()` |

### Final mental model

```text
Paths = WHERE?
Directory = WHICH FOLDER?
File = WHICH FILE?
FileHandle = LOW-LEVEL BYTES / OFFSET
download = REMOTE → DEVICE
upload = DEVICE → SERVER
document = KEEP
cache = TEMPORARY
bundle = INCLUDED WITH APP
```
