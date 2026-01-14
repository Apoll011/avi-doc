# `settings` Namespace

**Namespace:** `settings`  
**Import:** Built-in (globally available)

The `settings` namespace provides access to user-configurable skill settings. Unlike constants which are defined at skill development time, settings can be modified by users through Avi's interface, allowing for runtime customization of skill behavior.

> **Note:** Settings are scoped to the current skill and can be configured by users through the skill settings UI.

---

## Functions

### `settings::get()`

Gets a setting value for the current skill.

#### Signature

```rhai
fn get(name: ImutableString) -> ?
```

#### Parameters

| Parameter | Type     | Description                  |
|-----------|----------|------------------------------|
| `name`    | `ImutableString` | The name of the setting      |

#### Returns

- The setting value (type varies based on setting definition)
- Default value if not set by user
- `UNIT` if setting doesn't exist

#### Examples

```js
// Get a simple setting
let volume = settings::get("volume_level");
set_audio_volume(volume);

// Get setting with fallback
let theme = settings::get("ui_theme");
if theme == UNIT {
    theme = "dark"; // Fallback default
}

// Use in conditional logic
let notifications_enabled = settings::get("enable_notifications");
if notifications_enabled {
    send_notification("Task completed");
}
```

---

### `settings::has()`

Checks if a setting exists for the current skill.

#### Signature

```rhai
fn has(name: ImutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description                  |
|-----------|----------|------------------------------|
| `name`    | `ImutableString` | The name of the setting      |

#### Returns

- `bool` - `true` if the setting exists, `false` otherwise

#### Examples

```js
// Check before accessing
if settings::has("api_endpoint") {
    let endpoint = settings::get("api_endpoint");
    make_api_call(endpoint);
} else {
    log::warn("API endpoint not configured");
}

// Conditional feature enablement
if settings::has("experimental_features") {
    let enabled = settings::get("experimental_features");
    if enabled {
        enable_experimental_mode();
    }
}
```

---

### `settings::full()`

Gets the full setting object including metadata.

#### Signature

```rhai
fn full(name: ImutableString) -> SettingNamed
```

#### Parameters

| Parameter | Type     | Description                  |
|-----------|----------|------------------------------|
| `name`    | `ImutableString` | The name of the setting      |

#### Returns

- `SettingNamed` - The full setting object with metadata (value, type, description, etc.)
- `UNIT` if not found

#### Examples

```js
// Access setting metadata
let setting = settings::full("volume_level");
if setting != UNIT {
    log::info(`Setting: ${setting.name}`);
    log::info(`Type: ${setting.type}`);
    log::info(`Value: ${setting.value}`);
    log::info(`Description: ${setting.description}`);
}

// Validate setting type
let setting = settings::full("timeout");
if setting != UNIT && setting.type == "number" {
    let timeout = setting.value;
    configure_timeout(timeout);
}
```

---

### `settings::list()`

Lists all settings available for the current skill.

#### Signature

```rhai
fn list() -> Map
```

#### Parameters

None

#### Returns

- `Map` - A map of setting names to setting objects
- Empty map if no settings are defined

#### Examples

```js
// List all settings
let all_settings = settings::list();
log::info(`Total settings: ${all_settings.len()}`);

foreach (name, setting in all_settings) {
    log::info(`${name}: ${setting.value}`);
}

// Find settings by type
let all_settings = settings::list();
foreach (name, setting in all_settings) {
    if setting.type == "boolean" {
        log::info(`Toggle: ${name} = ${setting.value}`);
    }
}
```

---

## Settings vs Constants

| Feature           | Settings                          | Constants                     |
|-------------------|-----------------------------------|-------------------------------|
| **Modifiable**    | Yes, by users at runtime          | No, fixed at development time |
| **Scope**         | Per-skill, user-configurable      | Per-skill, developer-defined  |
| **Use Case**      | User preferences, feature toggles | API keys, configuration values|
| **Access**        | `settings::get()`                 | `constant::get()`             |

---

## Best Practices

### 1. Always Check for Existence

```js
// ✅ Good - defensive check
if settings::has("theme") {
    let theme = settings::get("theme");
    apply_theme(theme);
}

// ❌ Bad - assumes setting exists
let theme = settings::get("theme"); // May be UNIT
```

### 2. Provide Sensible Defaults

```js
// ✅ Good - fallback to default
let timeout = settings::get("timeout");
if timeout == UNIT {
    timeout = 30; // Default 30 seconds
}

// ❌ Bad - no fallback
let timeout = settings::get("timeout");
configure_timeout(timeout); // May fail if UNIT
```

### 3. Validate Setting Values

```js
// ✅ Good - validation
let volume = settings::get("volume_level");
if volume != UNIT {
    if volume < 0 || volume > 100 {
        log::warn("Volume out of range, using default");
        volume = 50;
    }
    set_volume(volume);
}
```

### 4. Document Setting Purpose

```js
// ✅ Good - clear documentation
// Setting: "notification_sound" (boolean)
// Purpose: Enable/disable notification sounds
// Default: true
let play_sound = settings::get("notification_sound");
```

---

## Complete Example: Configurable Timer Skill

```js
on_intent "set_timer" {
    let duration = slots::get(intent, "duration");
    
    // Get user preferences
    let notification_enabled = settings::get("enable_notifications");
    if notification_enabled == UNIT {
        notification_enabled = true; // Default
    }
    
    let sound_enabled = settings::get("enable_sound");
    if sound_enabled == UNIT {
        sound_enabled = true; // Default
    }
    
    let volume = settings::get("volume_level");
    if volume == UNIT || volume < 0 || volume > 100 {
        volume = 50; // Default
    }
    
    // Validate required settings
    if !settings::has("timer_sound_file") {
        log::error("Timer sound file not configured");
        dialogue::say("Timer is not properly configured");
        return;
    }
    
    let sound_file = settings::get("timer_sound_file");
    
    log::info(`Setting timer for ${duration} minutes`);
    log::info(`Notifications: ${notification_enabled}, Sound: ${sound_enabled}, Volume: ${volume}`);
    
    // Set up timer with user preferences
    timer::schedule(duration * 60, "timer_expired", #{
        notification: notification_enabled,
        sound: sound_enabled,
        volume: volume,
        sound_file: sound_file
    });
    
    dialogue::say(`Timer set for ${duration} minutes`);
}

fn timer_expired(config) {
    if config.notification {
        send_notification("Timer expired!");
    }
    
    if config.sound {
        play_sound(config.sound_file, config.volume);
    }
    
    dialogue::say("Your timer has expired!");
}
```

---

## Common Setting Types

```js
// Boolean settings (toggles)
let enabled = settings::get("feature_enabled"); // true/false

// Numeric settings (volume, timeout, etc.)
let volume = settings::get("volume_level"); // 0-100

// ImutableString settings (themes, paths, etc.)
let theme = settings::get("ui_theme"); // "dark", "light", etc.

// List/Array settings (allowed values)
let mode = settings::get("operation_mode"); // ["auto", "manual", "hybrid"]
```

---

## See Also

- [`constant` namespace](./constant.md) - Immutable skill-level constants
- [Skill Configuration](../guides/skill-config.md) - Defining settings in skill manifest
- [User Preferences](../guides/user-prefs.md) - Best practices for user-configurable options