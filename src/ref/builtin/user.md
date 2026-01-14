# `user` Namespace

**Namespace:** `user`  
**Import:** Built-in (globally available)

The `user` namespace provides access to user profile information and preferences. These functions allow skills to personalize responses and behavior based on user data configured in Avi's settings.

---

## Functions

### `user::name()`

Gets the user's name.

#### Signature

```rhai
fn name() -> ImutableString
```

#### Parameters

None

#### Returns

- `ImutableString` - The user's name, or an empty string if not available

#### Examples

```js
on_intent "greeting" {
    let name = user::name();
    if name != "" {
        say(`Hello, ${name}! How can I help you?`);
    } else {
        say("Hello! How can I help you?");
    }
}
```

---

### `user::nickname()`

Gets the user's nickname.

#### Signature

```rhai
fn nickname() -> ?
```

#### Parameters

None

#### Returns

- `ImutableString` - The user's nickname
- `()` if not set

#### Examples

```js
on_intent "casual_greeting" {
    let nickname = user::nickname();
    if nickname != () {
        say(`Hey ${nickname}!`);
    } else {
        say(`Hey there!`);
    }
}
```

---

### `user::id()`

Gets the user's unique identifier.

#### Signature

```rhai
fn id() -> ?
```

#### Parameters

None

#### Returns

- `ImutableString` - The user's ID
- Empty string if not available

#### Examples

```js
// Use user ID for personalized storage
let user_id = user::id();
let prefs_key = `preferences:${user_id}`;
let prefs = storage::get(prefs_key);
```

---

### `user::birthday()`

Gets the user's birthday as a Unix timestamp.

#### Signature

```rhai
fn birthday() -> ?
```

#### Parameters

None

#### Returns

- `Timestamp` - The birthday as a Unix timestamp
- `()` if not set

#### Examples

```js
on_intent "check_birthday" {
    let birthday = user::birthday();
    if birthday != () {
        let today = time::now();
        // Check if it's the user's birthday
        if is_same_day(birthday, today) {
            say("Happy birthday!");
        }
    }
}
```

---

### `user::language()`

Gets the user's language preference.

#### Signature

```rhai
fn language() -> ImutableString
```

#### Parameters

None

#### Returns

- `ImutableString` - The language code (e.g., "en", "es", "fr"), defaults to "en" if not set

#### Examples

```js
// Provide localized responses
let lang = user::language();
if lang == "es" {
    say("¡Hola!");
} else if lang == "fr" {
    say("Bonjour!");
} else {
    say("Hello!");
}
```

---

### `user::location()`

Gets the user's location.

#### Signature

```rhai
fn location() -> Location
```

#### Parameters

None

#### Returns

- `Location` - A map with `city` and `country` fields
- `()` if not set

#### Examples

```js
on_intent "local_weather" {
    let location = user::location();
    if location != () {
        say(`Checking weather for ${location.city}, ${location.country}`);
        // Fetch weather data...
    } else {
        say("Please set your location in settings first");
    }
}
```

---

### `user::quiet_hours()`

Gets the user's quiet hours configuration.

#### Signature

```rhai
fn quiet_hours() -> QuietHours
```

#### Parameters

None

#### Returns

- `QuietHours` - A map with `start` and `end` time fields
- `()` if not set

#### Examples

```js
// Respect quiet hours for notifications
let quiet_hours = user::quiet_hours();
if quiet_hours != () {
    let current_time = time::now();
    if is_in_quiet_hours(current_time, quiet_hours) {
        log::info("Suppressing notification during quiet hours");
        return;
    }
}
send_notification("Your reminder is ready!");
```

---

### `user::voice_profile_id()`

Gets the user's voice profile identifier.

#### Signature

```rhai
fn voice_profile_id() -> ?
```

#### Parameters

None

#### Returns

- `ImutableString` - The voice profile ID
- `()` if not set

#### Examples

```js
// Use for voice authentication or personalization
let profile_id = user::voice_profile_id();
if profile_id != () {
    log::info(`Using voice profile: ${profile_id}`);
}
```

---

## Best Practices

### 1. Always Handle Missing Data

```js
// ✅ Good - check before using
let location = user::location();
if location != () {
    use_location(location.city);
} else {
    prompt_for_location();
}

// ❌ Bad - assumes data exists
let location = user::location();
say(`Weather in ${location.city}`); // May fail
```

### 2. Provide Fallbacks

```js
// ✅ Good - graceful degradation
let name = user::name();
let greeting = name != "" ? `Hello, ${name}` : "Hello";
say(greeting);
```

### 3. Respect User Privacy

```js
// ✅ Good - only log when necessary
if constant::get("DEBUG_MODE") {
    log::debug(`User ID: ${user::id()}`);
}

// ❌ Bad - unnecessary logging of personal data
log::info(`User ${user::name()} with ID ${user::id()}`);
```

---

## See Also

- [`settings` namespace](./settings.md) - Skill-specific settings
- [User Profile Configuration](../guides/user-profile.md) - Setting up user data