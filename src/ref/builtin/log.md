# `log` Namespace

**Namespace:** `log`  
**Import:** Built-in (globally available)

The `log` namespace provides structured logging functions for debugging and monitoring skill execution. Logs are written to Avi's system logs and can be viewed for troubleshooting and performance analysis.

---

## Log Levels

AviScript supports five log levels, ordered from most to least verbose:

1. **TRACE** - Very detailed diagnostic information
2. **DEBUG** - Diagnostic information useful for debugging
3. **INFO** - Informational messages about normal operation
4. **WARN** - Warning messages for potentially problematic situations
5. **ERROR** - Error messages for serious problems

---

## Functions

### `log::trace()`

Logs a trace-level message for detailed debugging.

#### Signature

```rhai
fn trace(text: ImutableString)
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `text`    | `ImutableString` | The message to log    |

#### Returns

Nothing

#### Examples

```js
log::trace("Entering parse_response function");
log::trace(`Response data: ${response}`);
```

---

### `log::debug()`

Logs a debug-level message.

#### Signature

```rhai
fn debug(text: ImutableString)
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `text`    | `ImutableString` | The message to log    |

#### Returns

Nothing

#### Examples

```js
log::debug("Processing user intent");
log::debug(`Slot values: ${slots}`);
```

---

### `log::info()`

Logs an informational message.

#### Signature

```rhai
fn info(text: ImutableString)
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `text`    | `ImutableString` | The message to log    |

#### Returns

Nothing

#### Examples

```js
log::info("Skill started successfully");
log::info(`User ${user::name()} triggered timer skill`);
```

---

### `log::warn()`

Logs a warning message.

#### Signature

```rhai
fn warn(text: ImutableString)
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `text`    | `ImutableString` | The message to log    |

#### Returns

Nothing

#### Examples

```js
log::warn("API response time exceeded threshold");
log::warn("User location not set, using default");
```

---

### `log::error()`

Logs an error message.

#### Signature

```rhai
fn error(text: ImutableString)
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `text`    | `ImutableString` | The message to log    |

#### Returns

Nothing

#### Examples

```js
log::error("Failed to connect to external service");
log::error(`Invalid slot value: ${value}`);
```

---

## Usage Guidelines

### When to Use Each Level

```js
// TRACE - Very detailed execution flow
log::trace("Entering function calculate_weather");
log::trace("Loop iteration 5 of 10");

// DEBUG - Development and debugging information
log::debug(`API request payload: ${json::stringify(payload)}`);
log::debug("Cache hit for key: weather_london");

// INFO - Normal operational messages
log::info("Weather skill initialized");
log::info("Successfully fetched weather data");

// WARN - Potential issues that don't prevent operation
log::warn("Slow API response: 3.5 seconds");
log::warn("Fallback to cached data due to timeout");

// ERROR - Serious problems requiring attention
log::error("Database connection failed");
log::error("Required configuration missing");
```

---

## Best Practices

### 1. Use Appropriate Log Levels

```js
// ✅ Good - matches severity
if !api_key {
    log::error("API key not configured");
}

// ❌ Bad - wrong severity
if !api_key {
    log::info("API key not configured"); // Should be error
}
```

### 2. Include Context

```js
// ✅ Good - provides context
log::error(`Failed to fetch weather for ${city}: ${error_msg}`);

// ❌ Bad - lacks context
log::error("Request failed");
```

### 3. Avoid Logging Sensitive Data

```js
// ✅ Good - masks sensitive data
log::info(`API key configured: ${api_key.substring(0, 4)}...`);

// ❌ Bad - logs full credentials
log::info(`API key: ${api_key}`);
```

### 4. Use Conditional Debug Logging

```js
// ✅ Good - conditional verbose logging
if constant::get("DEBUG_MODE") {
    log::debug(`Full request: ${http_request}`);
}

// Production code without debug bloat
log::info("Request sent");
```

---

## Complete Example: Error Handling with Logging

```js
on_intent "fetch_data" {
    log::info("Starting data fetch");
    
    try {
        log::debug("Validating input parameters");
        if !slots::exists(intent, "query") {
            log::warn("Missing required slot: query");
            dialogue::say("Please specify what to search for");
            return;
        }
        
        let query = slots::get(intent, "query");
        log::info(`Fetching data for query: ${query}`);
        
        let result = http::get(`https://api.example.com/search?q=${query}`);
        
        if result.status == 200 {
            log::info("Data fetch successful");
            process_result(result.body);
        } else {
            log::error(`API returned status ${result.status}`);
            dialogue::say("Sorry, I couldn't fetch that data");
        }
        
    } catch (e) {
        log::error(`Unexpected error: ${e}`);
        dialogue::say("Something went wrong");
    }
}
```

---

## See Also

- [Error Handling](../language/error-handling.md) - Exception handling patterns
- [Debugging Skills](../guides/debugging.md) - Troubleshooting guide