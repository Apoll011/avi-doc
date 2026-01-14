# `constant` Namespace

**Namespace:** `constant`  
**Import:** `import "constant";`

The `constant` namespace provides access to skill-level constants defined in your module's configuration. Constants are immutable values that remain fixed throughout the skill's lifetime, perfect for configuration values, thresholds, API keys, and other unchanging data.

> **Note:** Constants are scoped to the current skill and are defined in the skill's manifest or configuration file.

---

## Functions

### `constant::get()`

Retrieves the value of a named constant defined for the current skill.

#### Signature

```rust
fn get(name: ImmutableString) -> Any
```

#### Parameters

| Parameter | Type     | Description                        |
|-----------|----------|------------------------------------|
| `name`    | `ImmutableString` | The name of the constant to retrieve |

#### Returns

- The constant value (type varies based on definition)
- `UNIT` (empty/null) if the constant is not found

#### Behavior

- Returns the exact value as defined in the skill configuration
- Case-sensitive constant name matching
- Does not throw errors for missing constants (returns `UNIT` instead)
- Constants are resolved at skill load time

#### Examples

**Basic Usage**

```js
// Retrieve a simple constant
let max_retries = constant::get("MAX_RETRIES");
log::info(`Maximum retries allowed: ${max_retries}`);
```

**With Default Values**

```js
// Safely handle missing constants
let timeout = constant::get("API_TIMEOUT");
if timeout == UNIT {
    timeout = 30;  // Default fallback
}
log::info(`Using timeout: ${timeout} seconds`);
```

**Practical Example - API Configuration**

```js
on_intent "get_weather" {
    let api_key = constant::get("WEATHER_API_KEY");
    let base_url = constant::get("WEATHER_API_URL");
        
    if api_key == UNIT {
        log::error("Weather API key not configured");
        say("I'm not configured to check the weather yet.");
        return;
    }
            
    let response = http::get(`${base_url}?key=${api_key}&location=${params.location}`);
    // Process weather data...
}

```

**Advanced Example - Feature Flags**

```js
// Check if premium features are enabled
let premium_enabled = constant::get("ENABLE_PREMIUM_FEATURES");

if premium_enabled == true {
    // Unlock advanced functionality
    invoke_premium_service();
} else {
    say("This feature requires a premium subscription.");
}
```

---

### `constant::has()`

Checks whether a constant with the given name exists for the current skill.

#### Signature

```rust
fn has(name: ImmutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description                      |
|-----------|----------|----------------------------------|
| `name`    | `ImmutableString` | The name of the constant to check |

#### Returns

- `true` if the constant exists
- `false` if the constant does not exist

#### Behavior

- Performs a case-sensitive lookup
- Does not evaluate or retrieve the constant value
- Useful for defensive programming and configuration validation

#### Examples

**Basic Usage**

```js
if constant::has("DEBUG_MODE") {
    log::info("Debug mode constant is defined");
}
```

**Configuration Validation**

```js
on_start {
    // Validate required configuration
    let required = ["HOME_LOCATION", "DEVICE_NETWORK", "OWNER_NAME"];
        
    foreach (const_name in required) {
        if !constant::has(const_name) {
            log::error(`Missing required constant: ${const_name}`);
            return false;
        }
    }
        
    log::info("All required constants are configured");
    return true;
}
```

**Conditional Feature Initialization**

```js
// Only initialize optional features if configured
if constant::has("SPOTIFY_CLIENT_ID") && constant::has("SPOTIFY_CLIENT_SECRET") {
    initialize_spotify_integration();
} else {
    log::warn("Spotify integration not configured, skipping");
}
```

---

### `constant::list()`

Returns all constants available for the current skill as a key-value map.

#### Signature

```rust
fn list() -> HashMap<ImmutableString, Any>
```

#### Parameters

None

#### Returns

- `HashMap<ImmutableString, Any>` - A map where keys are constant names and values are their corresponding values
- Empty map if no constants are defined

#### Behavior

- Returns a snapshot of all defined constants
- Values can be of any type (ImmutableString, number, boolean, object, etc.)
- The returned map is a copy; modifying it does not affect the actual constants

#### Examples

**Basic Usage**

```js
let all_constants = constant::list();
log::info(`Total constants defined: ${all_constants.len()}`);
```

**Debugging Configuration**

```js
on_intent "show_config" {
    let constants = constant::list();
            
    say("Here are the current skill constants:");
    foreach (key, value in constants) {
            log::info(`${key} = ${value}`);
        }
    
}
```

**Configuration Report**

```js
// Generate a configuration summary
let constants = constant::list();
let report = [];

foreach (name, value in constants) {
    report.push(`- ${name}: ${value}`);
}

log::info("Skill Configuration:\n" + report.join("\n"));
```

**Selective Constant Access**

```js
// Filter constants by prefix
let constants = constant::list();
let api_keys = [];

foreach (name, value in constants) {
    if name.starts_with("API_") {
        api_keys.push(name);
    }
}

log::info(`Found ${api_keys.len()} API key constants`);
```

---

## Complete Example: Multi-Environment Configuration

This example demonstrates using constants for environment-specific configuration:

```js
on_start {
    // List all constants for debugging
    let all_constants = constant::list();
    log::info(`Loaded ${all_constants.len()} constants`);
        
    // Check for environment indicator
    if !constant::has("ENVIRONMENT") {
        log::error("ENVIRONMENT constant not set");
        return false;
    }
        
    let env = constant::get("ENVIRONMENT");
    log::info(`Running in ${env} environment`);
        
    // Validate required constants for this environment
    let required = ["API_URL", "API_KEY", "LOG_LEVEL"];
    foreach (const_name in required) {
        if !constant::has(const_name) {
            log::error(`Missing required constant: ${const_name}`);
            return false;
        }
    }
        
        // Environment-specific initialization
    if env == "production" {
        let log_level = constant::get("LOG_LEVEL");
        if log_level != "error" {
            log::warn("Production environment should use LOG_LEVEL=error");
        }
    }
    
    return true;
}

```

---

## Best Practices

### 1. Always Check for Existence

```js
// ✅ Good - defensive programming
if constant::has("API_KEY") {
    let key = constant::get("API_KEY");
} else {
    log::error("API_KEY not configured");
}

// ❌ Bad - assumes constant exists
let key = constant::get("API_KEY");
// key might be UNIT
```

### 2. Use Descriptive Names

```js
// ✅ Good - clear and descriptive
constant::get("MAX_RETRY_ATTEMPTS")
constant::get("DEFAULT_VOLUME_LEVEL")

// ❌ Bad - ambiguous
constant::get("MAX")
constant::get("VOL")
```

### 3. Group Related Constants

```js
// ✅ Good - organized with prefixes
constant::get("SPOTIFY_CLIENT_ID")
constant::get("SPOTIFY_CLIENT_SECRET")
constant::get("WEATHER_API_KEY")
constant::get("WEATHER_API_URL")
```

### 4. Validate on Startup

```js
// ✅ Good - validate during initialization
on_start {
    if !constant::has("REQUIRED_SETTING") {
        log::error("Skill cannot start without REQUIRED_SETTING");
        return false;
    }
}
```

---

## Common Use Cases

- **API Keys and Secrets** - Store sensitive authentication tokens
- **Service URLs** - Configure endpoint locations per environment
- **Feature Flags** - Enable/disable functionality without code changes
- **Thresholds** - Define limits (max retries, timeouts, etc.)
- **User Preferences** - Store skill-wide default settings
- **Device IDs** - Reference specific smart home devices
- **Localization** - Language codes, regional settings

---

## See Also

- [`settings` namespace](./settings.md) - Dynamic runtime settings
- [`config` block](../language/config.md) - Defining module configuration
- [Skill Development Guide](../guides/skills.md) - Building skills with constants