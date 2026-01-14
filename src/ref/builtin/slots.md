# `slots` Namespace

**Namespace:** `slots`  
**Import:** Built-in (globally available)

The `slots` namespace provides functions for working with intent slots - the extracted parameters from user utterances. Slots contain the structured data parsed from natural language input, such as locations, times, numbers, and custom entities.

> **Note:** Slots are extracted by the natural language understanding (NLU) engine and represent the structured parameters of an intent.

---

## Functions

### `slots::get()`

Gets the value of a slot from the current intent.

#### Signature

```rhai
fn get(intent: Intent, name: ImutableString) -> ?
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `intent`  | `Intent` | The intent to check            |
| `name`    | `ImutableString` | The name of the slot           |

#### Returns

- The value of the slot as a Rhai object
- `UNIT` if the slot is missing

#### Examples

```js
on_intent "set_timer" {
    let duration = slots::get(intent, "duration");
    if duration != UNIT {
        say(`Setting timer for ${duration} minutes`);
    }
}
```

---

### `slots::get_raw()`

Gets the raw text value of a slot from the current intent.

#### Signature

```rhai
fn get_raw(intent: Intent, name: ImutableString) -> ?
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `intent`  | `Intent` | The intent to check            |
| `name`    | `ImutableString` | The name of the slot           |

#### Returns

- The raw text value of the slot
- `UNIT` if the slot is missing

#### Examples

```js
on_intent "search" {
    let query_text = slots::get_raw(intent, "query");
    log::info(`User said: "${query_text}"`);
}
```

---

### `slots::full()`

Gets the full slot object from the current intent.

#### Signature

```rhai
fn full(intent: Intent, name: ImutableString) -> ?
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `intent`  | `Intent` | The intent to check            |
| `name`    | `ImutableString` | The name of the slot           |

#### Returns

- The full slot object containing metadata (value, raw text, confidence, etc.)
- `UNIT` if the slot is missing

#### Examples

```js
on_intent "book_flight" {
    let destination = slots::full(intent, "destination");
    if destination != UNIT {
        log::info(`Value: ${destination.value}, Confidence: ${destination.confidence}`);
    }
}
```

---

### `slots::exists()`

Checks if a slot exists in the current intent.

#### Signature

```rhai
fn exists(intent: Intent, name: ImutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `intent`  | `Intent` | The intent to check            |
| `name`    | `ImutableString` | The name of the slot           |

#### Returns

- `bool` - `true` if the slot exists, `false` otherwise

#### Examples

```js
on_intent "play_music" {
    if slots::exists(intent, "artist") {
        let artist = slots::get(intent, "artist");
        say(`Playing music by ${artist}`);
    } else {
        say("Playing your default playlist");
    }
}
```

---

### `slots::require()`

Asserts that a slot exists in the current intent. Throws an error if the slot is missing.

#### Signature

```rhai
fn require(intent: Intent, name: ImutableString)
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `intent`  | `Intent` | The intent to check            |
| `name`    | `ImutableString` | The name of the slot           |

#### Returns

Nothing, or throws an error if the slot is missing

#### Examples

```js
on_intent "send_message" {
    // Will throw error if recipient is missing
    slots::require(intent, "recipient");
    
    let recipient = slots::get(intent, "recipient");
    let message = slots::get(intent, "message");
    send_message(recipient, message);
}
```

---

### `slots::assert_equal()`

Asserts that a slot's value is equal to a given value.

#### Signature

```rhai
fn assert_equal(intent: Intent, name: ImutableString, val: ?) -> bool
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `intent`  | `Intent` | The intent to check            |
| `name`    | `ImutableString` | The name of the slot           |
| `val`     | `?`      | The value to compare against   |

#### Returns

- `bool` - `true` if the slot value is equal, `false` otherwise

#### Examples

```js
on_intent "control_device" {
    if slots::assert_equal(intent, "action", "on") {
        turn_device_on();
    } else if slots::assert_equal(intent, "action", "off") {
        turn_device_off();
    }
}
```

---

### `slots::assert_in()`

Asserts that a slot's value is in a given list or matches a value.

#### Signature

```rhai
fn assert_in(intent: Intent, name: ImutableString, list: ?) -> bool
```

#### Parameters

| Parameter | Type     | Description                                      |
|-----------|----------|--------------------------------------------------|
| `intent`  | `Intent` | The intent to check                              |
| `name`    | `ImutableString` | The name of the slot                             |
| `list`    | `?`      | The list of values or a single value to check    |

#### Returns

- `bool` - `true` if the slot value is in the list or matches the value, `false` otherwise

#### Examples

```js
on_intent "set_temperature" {
    let valid_units = ["celsius", "fahrenheit", "kelvin"];
    
    if slots::assert_in(intent, "unit", valid_units) {
        let temp = slots::get(intent, "temperature");
        let unit = slots::get(intent, "unit");
        say(`Setting temperature to ${temp} degrees ${unit}`);
    } else {
        say("Please specify a valid temperature unit");
    }
}
```

---

### `slots::assert_in_dict()`

Asserts that a slot's string value is a key in a given map.

#### Signature

```rhai
fn assert_in_dict(intent: Intent, name: ImutableString, dict: ?) -> bool
```

#### Parameters

| Parameter | Type     | Description                    |
|-----------|----------|--------------------------------|
| `intent`  | `Intent` | The intent to check            |
| `name`    | `ImutableString` | The name of the slot           |
| `dict`    | `?`      | The map to check for the key   |

#### Returns

- `bool` - `true` if the slot's string value is a key in the map, `false` otherwise

#### Examples

```js
on_intent "get_room_info" {
    let room_devices = #{
        "living_room": ["tv", "lights", "thermostat"],
        "bedroom": ["lights", "fan"],
        "kitchen": ["lights", "oven"]
    };
    
    if slots::assert_in_dict(intent, "room", room_devices) {
        let room = slots::get(intent, "room");
        let devices = room_devices[room];
        say(`${room} has ${devices.len()} devices`);
    } else {
        say("I don't recognize that room");
    }
}
```

---

## Best Practices

### 1. Always Check for Existence

```js
// ✅ Good - defensive programming
if slots::exists(intent, "location") {
    let location = slots::get(intent, "location");
    process_location(location);
}

// ❌ Bad - assumes slot exists
let location = slots::get(intent, "location");
// location might be UNIT
```

### 2. Use `require()` for Mandatory Slots

```js
// ✅ Good - fail fast for required data
slots::require(intent, "recipient");
slots::require(intent, "message");

let recipient = slots::get(intent, "recipient");
let message = slots::get(intent, "message");
```

### 3. Validate Slot Values

```js
// ✅ Good - validate before use
let valid_actions = ["play", "pause", "stop", "skip"];
if !slots::assert_in(intent, "action", valid_actions) {
    say("Invalid action. Try play, pause, stop, or skip.");
    return;
}
```

---

## See Also

- [Intent Handling](../language/intent.md) - Working with intents
- [NLU Configuration](../guides/nlu.md) - Configuring slot extraction