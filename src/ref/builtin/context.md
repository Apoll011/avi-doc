# `context` Namespace

**Namespace:** `context`  
**Import:** `import "context";`

The `context` namespace provides persistent storage for skill state and data that needs to survive across multiple invocations. Context data is skill-scoped and can be configured with time-to-live (TTL) and persistence options, making it ideal for conversation state, user preferences, and temporary caches.

> **Note:** Context storage is persistent by default and survives skill restarts. Data can be scoped to sessions or made permanent across all interactions.

---

## Functions

### `context::get()`

Retrieves a value from the skill's persistent context storage.

#### Signature

```rust
fn get(key: ImmutableString) -> Any
```

#### Parameters

| Parameter | Type     | Description                           |
|-----------|----------|---------------------------------------|
| `key`     | `ImmutableString` | The key of the value to retrieve      |

#### Returns

- The value associated with the key (type varies based on what was stored)
- `UNIT` (empty/null) if the key is not found

#### Behavior

- Case-sensitive key lookup
- Returns `UNIT` for missing or expired keys
- Does not throw errors for non-existent keys
- Automatically handles TTL expiration

#### Examples

**Basic Usage**

```js
// Retrieve stored value
let user_name = context::get("last_user");
if user_name != UNIT {
    log::info(`Welcome back, ${user_name}!`);
}
```

**Conversation State**

```js
on_intent "continue_conversation" {
    let topic = context::get("current_topic");
    
    if topic == UNIT {
        say("What would you like to talk about?");
    } else {
        say(`Let's continue talking about ${topic}.`);
    }
}
```

**Counter Example**

```js
on_intent "increment_score" {
    let score = context::get("user_score");
    if score == UNIT {
        score = 0;
    }
    
    score += 1;
    context::set("user_score", score, 0, true);
    say(`Your score is now ${score}!`);
}
```

**Multi-turn Workflow**

```js
on_intent "book_flight" {
    // Check if we already have departure city
    let departure = context::get("flight_departure");
    
    if departure == UNIT {
        say("Where are you flying from?");
        context::set("workflow_step", "awaiting_departure", 300, false);
    } else {
        let destination = context::get("flight_destination");
        if destination == UNIT {
            say(`Flying from ${departure}. Where to?`);
        } else {
            say(`Booking flight from ${departure} to ${destination}.`);
        }
    }
}
```

---

### `context::has()`

Checks whether a key exists in the skill's persistent context.

#### Signature

```rust
fn has(key: ImmutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description              |
|-----------|----------|--------------------------|
| `key`     | `ImmutableString` | The key to check         |

#### Returns

- `true` if the key exists and has not expired
- `false` if the key does not exist or has expired

#### Behavior

- Performs case-sensitive lookup
- Respects TTL expiration (returns `false` for expired keys)
- More efficient than retrieving and checking for `UNIT`

#### Examples

**Basic Usage**

```js
if context::has("user_preferences") {
    log::info("User preferences are configured");
}
```

**Conditional Logic**

```js
on_intent "set_reminder" {
    if context::has("reminder_list") {
        let reminders = context::get("reminder_list");
        reminders.push(params.reminder);
        context::set("reminder_list", reminders, 0, true);
    } else {
        context::set("reminder_list", [params.reminder], 0, true);
    }
    
    say("Reminder saved!");
}
```

**Session Detection**

```js
on_start {
    if context::has("session_start_time") {
        log::info("Resuming existing session");
    } else {
        log::info("Starting new session");
        context::set("session_start_time", now(), 3600, false);
    }
}
```

**Cache Validation**

```js
// Check if cached data is still valid
if context::has("weather_cache") {
    let cached_weather = context::get("weather_cache");
    say(`The weather is ${cached_weather.condition}.`);
} else {
    // Cache expired or doesn't exist, fetch fresh data
    let weather = fetch_weather_data();
    context::set("weather_cache", weather, 600, false); // 10 min cache
    say(`The weather is ${weather.condition}.`);
}
```

---

### `context::set()`

Stores a value in the skill's persistent context with optional TTL and persistence settings.

#### Signature

```rust
fn set(key: ImmutableString, value: Any, ttl: u64, persist: bool)
```

#### Parameters

| Parameter | Type     | Description                                          |
|-----------|----------|------------------------------------------------------|
| `key`     | `ImmutableString` | The key to store the value under            |
| `value`   | `Any`    | The value to store (any type)                        |
| `ttl`     | `u64`    | Time-to-live in seconds (0 = no expiration)          |
| `persist` | `bool`   | Whether to persist across sessions and restarts      |

#### Returns

- `void` (nothing)

#### Behavior

- **TTL = 0**: Value never expires (permanent until explicitly removed)
- **TTL > 0**: Value expires after specified seconds
- **persist = true**: Value survives skill restarts and system reboots
- **persist = false**: Value exists only in memory (cleared on restart)
- Overwrites existing values with the same key

#### Examples

**Basic Usage**

```js
// Store permanent user preference
context::set("preferred_language", "en-US", 0, true);

// Store temporary session data (5 minutes)
context::set("temp_token", "abc123", 300, false);
```

**User Preferences**

```js
on_intent "set_temperature_preference" {
    let preferred_temp = params.temperature;
    
    // Store permanently, no expiration
    context::set("home_temp_preference", preferred_temp, 0, true);
    
    say(`I'll remember you prefer ${preferred_temp} degrees.`);
}
```

**Conversation Memory**

```js
on_intent "remember_name" {
    let name = params.user_name;
    
    // Remember for 30 days
    context::set("user_name", name, 2592000, true);
    
    say(`Nice to meet you, ${name}! I'll remember that.`);
}
```

**Session Tracking**

```js
on_start {
    // Track session start (expires in 1 hour)
    context::set("session_id", util::uuid(), 3600, false);
    context::set("session_start", now(), 3600, false);
    
    log::info("New session started");
}
```

**Caching API Responses**

```js
on_intent "get_stock_price" {
    let symbol = params.stock_symbol;
    let cache_key = `stock_price_${symbol}`;
    
    if !context::has(cache_key) {
        // Fetch fresh data
        let price = fetch_stock_price(symbol);
        
        // Cache for 60 seconds
        context::set(cache_key, price, 60, false);
        
        say(`${symbol} is trading at $${price}.`);
    } else {
        let cached_price = context::get(cache_key);
        say(`${symbol} is at $${cached_price} (cached).`);
    }
}
```

**Complex Data Storage**

```js
on_intent "save_shopping_list" {
    let shopping_list = {
        items: ["milk", "bread", "eggs"],
        created: now(),
        completed: false
    };
    
    // Store object permanently
    context::set("shopping_list", shopping_list, 0, true);
    
    say("Shopping list saved!");
}
```

---

### `context::remove()`

Removes a value from the skill's persistent context storage.

#### Signature

```rust
fn remove(key: ImmutableString)
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `key`     | `ImmutableString` | The key of the value to remove |

#### Returns

- `void` (nothing)

#### Behavior

- Removes both memory and persistent storage
- Silently succeeds even if key doesn't exist
- Cannot be undone
- Frees up storage space

#### Examples

**Basic Usage**

```js
// Remove a value
context::remove("temporary_data");
```

**Clear Session Data**

```js
on_end {
    // Clean up session data
    context::remove("session_id");
    context::remove("session_start");
    context::remove("temp_cache");
    
    log::info("Session cleaned up");
}
```

**Reset User Preferences**

```js
on_intent "reset_preferences" {
    context::remove("preferred_language");
    context::remove("home_temp_preference");
    context::remove("notification_settings");
    
    say("All preferences have been reset to defaults.");
}
```

**Workflow Cleanup**

```js
on_intent "cancel_booking" {
    // Remove all booking-related context
    context::remove("flight_departure");
    context::remove("flight_destination");
    context::remove("flight_date");
    context::remove("workflow_step");
    
    say("Booking cancelled. All details cleared.");
}
```

**Expired Data Cleanup**

```js
// Manual cleanup of old data
if context::has("old_cache_v1") {
    context::remove("old_cache_v1");
    log::info("Removed deprecated cache");
}
```

---

## Complete Example: Multi-Turn Conversation

This example demonstrates using context for a multi-step conversation:

```js
on_intent "order_pizza" {
    // Step 1: Get pizza size
    if !context::has("pizza_size") {
        say("What size pizza would you like? Small, medium, or large?");
        context::set("workflow", "awaiting_size", 300, false);
        return;
    }
    
    let size = context::get("pizza_size");
    
    // Step 2: Get toppings
    if !context::has("pizza_toppings") {
        say(`Great! ${size} pizza. What toppings?`);
        context::set("workflow", "awaiting_toppings", 300, false);
        return;
    }
    
    let toppings = context::get("pizza_toppings");
    
    // Step 3: Confirm and place order
    say(`Ordering ${size} pizza with ${toppings}. Confirm?`);
    context::set("workflow", "awaiting_confirmation", 300, false);
}

on_intent "confirm" {
    let workflow = context::get("workflow");
    
    if workflow == "awaiting_confirmation" {
        let size = context::get("pizza_size");
        let toppings = context::get("pizza_toppings");
        
        // Place order
        place_order(size, toppings);
        
        // Clear context
        context::remove("pizza_size");
        context::remove("pizza_toppings");
        context::remove("workflow");
        
        say("Your pizza order is placed! It'll arrive in 30 minutes.");
    }
}

on_intent "cancel" {
    // Clear any ongoing workflow
    context::remove("pizza_size");
    context::remove("pizza_toppings");
    context::remove("workflow");
    
    say("Order cancelled.");
}
```

---

## Best Practices

### 1. Always Check Before Getting

```js
// ✅ Good - defensive programming
if context::has("user_data") {
    let data = context::get("user_data");
    process_data(data);
}

// ❌ Bad - may process UNIT
let data = context::get("user_data");
process_data(data);  // Could fail if data is UNIT
```

### 2. Use Appropriate TTL

```js
// ✅ Good - appropriate TTLs
context::set("session_token", token, 3600, false);     // 1 hour
context::set("weather_cache", data, 600, false);        // 10 minutes
context::set("user_preference", pref, 0, true);         // Permanent

// ❌ Bad - unnecessarily long or short
context::set("session_token", token, 0, true);          // Token never expires
context::set("user_preference", pref, 5, true);         // Preference expires too fast
```

### 3. Choose Persistence Wisely

```js
// ✅ Good - appropriate persistence
context::set("user_name", name, 0, true);               // Persist across restarts
context::set("cache_data", data, 300, false);           // Temporary, no persistence

// ❌ Bad - wrong persistence settings
context::set("temp_token", token, 60, true);            // Temporary data persisted
context::set("important_pref", pref, 0, false);         // Lost on restart
```

### 4. Clean Up When Done

```js
// ✅ Good - cleanup workflow data
on_intent "complete_task" {
    // Process task...
    
    // Clean up
    context::remove("task_step");
    context::remove("task_data");
}

// ❌ Bad - leaves orphaned data
on_intent "complete_task" {
    // Process task...
    // Context data left behind
}
```

---

## Common Use Cases

| Use Case | TTL | Persist | Example |
|----------|-----|---------|---------|
| **User Preferences** | 0 (permanent) | true | Language, theme, favorite devices |
| **Session Data** | 1800-3600s | false | Active conversation, temporary state |
| **API Caches** | 300-600s | false | Weather data, stock prices |
| **Conversation Memory** | 86400s+ | true | User's name, past interactions |
| **Workflow State** | 300-900s | false | Multi-step task progress |
| **Authentication** | 3600-7200s | false | Session tokens, credentials |
| **Feature Flags** | 0 (permanent) | true | Enabled features per user |

---

## Context vs Settings

| Feature | `context` | `settings` |
|---------|-----------|------------|
| Scope | Skill-level | System-wide |
| Lifetime | Configurable TTL | Permanent |
| Modified by | Skill code | User/Admin UI |
| Use for | Runtime state | Configuration |

> **Tip:** Use `context` for dynamic data that changes during skill execution. Use `settings` for static configuration values.

---

## See Also

- [`settings` namespace](./settings.md) - System-wide settings
- [`constant` namespace](./constant.md) - Immutable configuration
- [State Management Guide](../guides/state-management.md) - Best practices