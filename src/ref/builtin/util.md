# `util` Namespace

**Namespace:** `util`  
**Import:** Built-in (globally available)

The `util` namespace provides general-purpose utility functions for system operations, assertions, environment access, and data conversion.

---

## System Functions

### `util::os()`

Gets the current operating system name.

#### Signature

```rhai
fn os() -> String
```

#### Parameters

None

#### Returns

- `String` - The OS name (e.g., "linux", "windows", "macos")

#### Examples

```js
let os = util::os();
log::info(`Running on ${os}`);

if os == "windows" {
    // Windows-specific logic
} else if os == "linux" {
    // Linux-specific logic
}
```

---

### `util::env()`

Gets an environment variable with a default fallback.

#### Signature

```rhai
fn env(name: String, default: String) -> String
```

#### Parameters

| Parameter | Type     | Description                                      |
|-----------|----------|--------------------------------------------------|
| `name`    | `String` | The name of the environment variable             |
| `default` | `String` | The default value if the variable is not set     |

#### Returns

- `String` - The environment variable value, or the default if not found

#### Examples

```js
let home = util::env("HOME", "/default/path");
log::info(`Home directory: ${home}`);

// Configuration with fallback
let api_url = util::env("API_URL", "https://api.example.com");
```

---

### `util::cmd()`

Executes a shell command and returns the exit code.

#### Signature

```rhai
fn cmd(command: String) -> int
```

#### Parameters

| Parameter | Type     | Description                  |
|-----------|----------|------------------------------|
| `command` | `String` | The command to execute       |

#### Returns

- `int` - The exit code of the command, or -1 if the code couldn't be determined

#### Behavior

- Uses `cmd /C` on Windows
- Uses `sh -c` on Unix-like systems
- Returns 0 for successful execution
- Returns non-zero for errors

#### Examples

```js
// Execute a system command
let exit_code = util::cmd("ls -la");
if exit_code == 0 {
    log::info("Command executed successfully");
} else {
    log::error(`Command failed with code ${exit_code}`);
}

// Platform-specific commands
if util::os() == "windows" {
    util::cmd("dir");
} else {
    util::cmd("ls");
}
```

---

## Assertion Functions

### `util::assert()`

Asserts a condition, throwing an error if the condition is true.

#### Signature

```rhai
fn assert(condition: bool, msg: String)
```

#### Parameters

| Parameter   | Type     | Description                            |
|-------------|----------|----------------------------------------|
| `condition` | `bool`   | If true, an error will be thrown       |
| `msg`       | `String` | The error message to use               |

#### Returns

Nothing, or throws an error if condition is true

#### Behavior

> **Note:** This has inverted logic compared to typical assertions - it errors when the condition is **true**.

#### Examples

```js
// Validate input
let age = slots::get(intent, "age");
util::assert(age < 0, "Age cannot be negative");
util::assert(age > 150, "Age seems unrealistic");

// Check required conditions
let api_key = constant::get("API_KEY");
util::assert(api_key == UNIT, "API_KEY must be configured");
```

---

## Data Conversion Functions

### `util::uuid()`

Generates a new UUID v4.

#### Signature

```rhai
fn uuid() -> String
```

#### Parameters

None

#### Returns

- `String` - A UUID string in the format "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

#### Examples

```js
// Generate unique identifiers
let session_id = util::uuid();
log::info(`Session ID: ${session_id}`);

// Create unique keys
let request_id = util::uuid();
storage::set(`request:${request_id}`, data);
```

---

### `util::get_string()`

Converts a byte array to a string. Commonly used when receiving data from message subscriptions.

#### Signature

```rhai
fn get_string(bytes: Array) -> String
```

#### Parameters

| Parameter | Type    | Description                      |
|-----------|---------|----------------------------------|
| `bytes`   | `Array` | An array of u8 bytes to convert  |

#### Returns

- `String` - The decoded string

#### Examples

```js
// Convert subscription payload to string
subscribe topic "intent/execute/text" as <input> {
    let text = util::get_string(input);
    log::info(`Received: ${text}`);
    process_text(text);
}

// Process binary data
let message_bytes = receive_data();
let message = util::get_string(message_bytes);
```

---

## Best Practices

### 1. Use Environment Variables for Configuration

```js
// ✅ Good - flexible configuration
let config_path = util::env("AVI_CONFIG_PATH", "/etc/avi/config");

// ❌ Bad - hardcoded paths
let config_path = "/etc/avi/config";
```

### 2. Check Command Exit Codes

```js
// ✅ Good - error handling
let result = util::cmd("backup.sh");
if result != 0 {
    log::error("Backup failed");
    dialogue::say("The backup encountered an error");
}

// ❌ Bad - ignoring errors
util::cmd("backup.sh");
```

### 3. Use Assertions for Validation

```js
// ✅ Good - early validation
fn process_age(age) {
    util::assert(age < 0, "Age must be positive");
    util::assert(age > 120, "Age must be reasonable");
    // Process valid age...
}
```

### 4. Platform-Specific Logic

```js
// ✅ Good - cross-platform support
let os = util::os();
let separator = os == "windows" ? "\\" : "/";
let path = `${base_dir}${separator}${filename}`;
```

---

## Complete Example: Cross-Platform Script Execution

```js
on_intent "run_backup" {
    log::info("Starting backup process");
    
    // Get OS-specific paths
    let os = util::os();
    let backup_dir = util::env("BACKUP_DIR", "/var/backups");
    
    // Validate configuration
    util::assert(backup_dir == "", "BACKUP_DIR not configured");
    
    // Generate unique backup ID
    let backup_id = util::uuid();
    log::info(`Backup ID: ${backup_id}`);
    
    // Execute platform-specific backup command
    let exit_code;
    if os == "windows" {
        exit_code = util::cmd(`backup.bat ${backup_dir} ${backup_id}`);
    } else {
        exit_code = util::cmd(`./backup.sh ${backup_dir} ${backup_id}`);
    }
    
    // Handle result
    if exit_code == 0 {
        log::info("Backup completed successfully");
        dialogue::say("Backup complete");
    } else {
        log::error(`Backup failed with exit code ${exit_code}`);
        dialogue::say("Backup failed");
    }
}
```

---

## See Also

- [Message Subscriptions](../language/subscribe.md) - Using `get_string()` with subscriptions
- [Error Handling](../language/error-handling.md) - Using assertions effectively