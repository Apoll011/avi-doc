# `fs` Namespace

**Namespace:** `fs`  
**Import:** Built-in (globally available)

The `fs` namespace provides file system operations for reading, writing, and managing files and directories. Use these functions to persist data, load configuration, and interact with the file system.

---

## File Reading and Writing

### `fs::read()`

Reads the entire contents of a file as a string.

#### Signature

```rhai
fn read(path: ImutableString) -> ImutableString
```

#### Parameters

| Parameter | Type     | Description                  |
|-----------|----------|------------------------------|
| `path`    | `ImutableString` | The path to the file to read |

#### Returns

- `ImutableString` - The file contents
- `UNIT` if the file could not be read

#### Examples

```js
// Read a configuration file
let config_path = `${skill::dir()}/config.txt`;
let content = fs::read(config_path);

if content != UNIT {
    log::info(`Config loaded: ${content}`);
} else {
    log::error("Failed to read config file");
}

// Read and parse JSON
let json_path = `${skill::dir()}/data.json`;
let json_str = fs::read(json_path);
let data = json::parse(json_str);
```

---

### `fs::write()`

Writes a string to a file, overwriting its contents.

#### Signature

```rhai
fn write(path: ImutableString, content: ImutableString)
```

#### Parameters

| Parameter | Type     | Description                   |
|-----------|----------|-------------------------------|
| `path`    | `ImutableString` | The path to the file to write |
| `content` | `ImutableString` | The string to write           |

#### Returns

Nothing (throws error on failure)

#### Examples

```js
// Write configuration
let config = "volume=50\ntheme=dark";
fs::write(`${skill::dir()}/config.txt`, config);

// Save JSON data
let data = #{ name: "Alice", score: 100 };
fs::write(`${skill::dir()}/data.json`, json::to_json(data));
```

---

### `fs::append()`

Appends a string to the end of a file.

#### Signature

```rhai
fn append(path: ImutableString, content: ImutableString)
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `path`    | `ImutableString` | The path to the file           |
| `content` | `ImutableString` | The string to append           |

#### Returns

Nothing (throws error on failure)

#### Examples

```js
// Append to log file
let log_path = `${skill::dir()}/activity.log`;
let timestamp = time::now();
fs::append(log_path, `${timestamp}: User logged in\n`);

// Add to journal
fs::append("journal.txt", "Today was a great day!\n");
```

---

## File and Directory Management

### `fs::exists()`

Checks if a path exists.

#### Signature

```rhai
fn exists(path: ImutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `path`    | `ImutableString` | The path to check     |

#### Returns

- `bool` - `true` if the path exists, `false` otherwise

#### Examples

```js
// Check before reading
let file_path = `${skill::dir()}/data.txt`;
if fs::exists(file_path) {
    let content = fs::read(file_path);
} else {
    log::warn("File does not exist, using defaults");
}
```

---

### `fs::copy()`

Copies a file from one path to another.

#### Signature

```rhai
fn copy(src: ImutableString, dest: ImutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description               |
|-----------|----------|---------------------------|
| `src`     | `ImutableString` | The source path           |
| `dest`    | `ImutableString` | The destination path      |

#### Returns

- `bool` - `true` if successful, `false` otherwise

#### Examples

```js
// Create backup
let success = fs::copy("data.json", "data.backup.json");
if success {
    log::info("Backup created");
} else {
    log::error("Backup failed");
}
```

---

### `fs::move()`

Moves or renames a file or directory.

#### Signature

```rhai
fn move(src: ImutableString, dest: ImutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description               |
|-----------|----------|---------------------------|
| `src`     | `ImutableString` | The source path           |
| `dest`    | `ImutableString` | The destination path      |

#### Returns

- `bool` - `true` if successful, `false` otherwise

#### Examples

```js
// Rename file
fs::move("old_name.txt", "new_name.txt");

// Move to different directory
fs::move("temp/file.txt", "archive/file.txt");
```

---

### `fs::delete()`

Deletes a file or an empty directory.

#### Signature

```rhai
fn delete(path: ImutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `path`    | `ImutableString` | The path to delete    |

#### Returns

- `bool` - `true` if successful, `false` otherwise

#### Examples

```js
// Delete temporary file
if fs::delete("temp.txt") {
    log::info("Temporary file removed");
} else {
    log::warn("Could not delete file");
}
```

---

### `fs::mkdir()`

Creates a directory and all its parent directories if they don't exist.

#### Signature

```rhai
fn mkdir(path: ImutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description                       |
|-----------|----------|-----------------------------------|
| `path`    | `ImutableString` | The path of the directory to create |

#### Returns

- `bool` - `true` if successful, `false` otherwise

#### Examples

```js
// Create nested directories
if fs::mkdir("data/backups/2024") {
    log::info("Directory structure created");
}

// Ensure directory exists before writing
let dir = `${skill::dir()}/cache`;
if !fs::exists(dir) {
    fs::mkdir(dir);
}
fs::write(`${dir}/data.txt`, "content");
```

---

### `fs::list_files()`

Lists the names of files and directories in a given path.

#### Signature

```rhai
fn list_files(path: ImutableString) -> Array
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `path`    | `ImutableString` | The path to list      |

#### Returns

- `Array` - A list of file and directory names

#### Examples

```js
// List all files in directory
let files = fs::list_files(`${skill::dir()}/data`);
foreach (file in files) {
    log::info(`Found: ${file}`);
}

// Process all JSON files
let files = fs::list_files("configs");
foreach (file in files) {
    if file.ends_with(".json") {
        let path = `configs/${file}`;
        let data = json::parse(fs::read(path));
        process_config(data);
    }
}
```

---

## Path Manipulation

### `fs::basename()`

Gets the last component of a path (filename).

#### Signature

```rhai
fn basename(path: ImutableString) -> ImutableString
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `path`    | `ImutableString` | The path to process   |

#### Returns

- `ImutableString` - The last component of the path

#### Examples

```js
let filename = fs::basename("/home/user/documents/file.txt");
// Result: "file.txt"

let name = fs::basename("C:\\Users\\Alice\\photo.jpg");
// Result: "photo.jpg"
```

---

### `fs::dirname()`

Gets the parent directory of a path.

#### Signature

```rhai
fn dirname(path: ImutableString) -> ImutableString
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `path`    | `ImutableString` | The path to process   |

#### Returns

- `ImutableString` - The parent directory of the path

#### Examples

```js
let dir = fs::dirname("/home/user/documents/file.txt");
// Result: "/home/user/documents"

let parent = fs::dirname("data/cache/temp.txt");
// Result: "data/cache"
```

---

## Best Practices

### 1. Always Check if Files Exist

```js
// ✅ Good - defensive check
if fs::exists(file_path) {
    let content = fs::read(file_path);
} else {
    log::warn("File not found, creating with defaults");
    fs::write(file_path, default_content);
}

// ❌ Bad - assumes file exists
let content = fs::read(file_path); // May return UNIT
```

### 2. Use Skill Directory for Storage

```js
// ✅ Good - use skill directory
let data_path = `${skill::dir()}/user_data.json`;
fs::write(data_path, json::to_json(data));

// ❌ Bad - hardcoded absolute paths
fs::write("/tmp/data.json", content);
```

### 3. Handle File Operation Errors

```js
// ✅ Good - error handling
try {
    fs::write(path, content);
    log::info("File saved successfully");
} catch (e) {
    log::error(`Failed to save file: ${e}`);
    dialogue::say("Could not save your data");
}
```

### 4. Create Directories Before Writing

```js
// ✅ Good - ensure directory exists
let dir = fs::dirname(file_path);
if !fs::exists(dir) {
    fs::mkdir(dir);
}
fs::write(file_path, content);
```

---

## Complete Example: Simple Database

```js
on_intent "save_note" {
    let note = slots::get(intent, "note_text");
    if note == UNIT {
        dialogue::say("What would you like to save?");
        return;
    }
    
    // Ensure notes directory exists
    let notes_dir = `${skill::dir()}/notes`;
    if !fs::exists(notes_dir) {
        fs::mkdir(notes_dir);
    }
    
    // Generate filename with timestamp
    let timestamp = time::now();
    let filename = `note_${timestamp}.txt`;
    let file_path = `${notes_dir}/${filename}`;
    
    // Save note
    try {
        fs::write(file_path, note);
        dialogue::say("Note saved successfully");
        log::info(`Saved note: ${filename}`);
    } catch (e) {
        log::error(`Failed to save note: ${e}`);
        dialogue::say("Sorry, I couldn't save your note");
    }
}

on_intent "list_notes" {
    let notes_dir = `${skill::dir()}/notes`;
    
    if !fs::exists(notes_dir) {
        dialogue::say("You don't have any notes yet");
        return;
    }
    
    let files = fs::list_files(notes_dir);
    
    if files.len() == 0 {
        dialogue::say("You don't have any notes");
    } else {
        dialogue::say(`You have ${files.len()} notes`);
        foreach (file in files) {
            log::info(`Note: ${file}`);
        }
    }
}

on_intent "read_note" {
    let note_id = slots::get(intent, "note_number");
    let notes_dir = `${skill::dir()}/notes`;
    
    if !fs::exists(notes_dir) {
        dialogue::say("You don't have any notes");
        return;
    }
    
    let files = fs::list_files(notes_dir);
    
    if note_id > files.len() || note_id < 1 {
        dialogue::say("That note doesn't exist");
        return;
    }
    
    let filename = files[note_id - 1];
    let file_path = `${notes_dir}/${filename}`;
    let content = fs::read(file_path);
    
    if content != UNIT {
        dialogue::say(content);
    } else {
        dialogue::say("Could not read the note");
    }
}
```

---

## See Also

- [`json` namespace](./json.md) - Reading/writing JSON files
- [`skill` namespace](./skill.md) - Getting skill directory path
- [Data Persistence](../guides/data-persistence.md) - Best practices for storing data