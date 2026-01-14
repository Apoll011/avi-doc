# `dialogue` Namespace

**Namespace:** `dialogue`  
**Import:** Built-in (globally available)

The `dialogue` namespace provides functions for managing conversational interactions with users. This includes text-to-speech output, listening for input, validating responses, and creating multi-turn dialogues.

---

## Output Functions

### `dialogue::say()`

Speaks the given text to the user.

#### Signature

```rust
fn say(text: ImutableString)
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `text`    | `ImutableString` | The text to speak     |

#### Returns

Nothing

#### Examples

```rust
on_intent "greeting" {
    dialogue::say("Hello! How can I help you today?");
}
```

---

### `dialogue::say_once()`

Speaks the given text only once per session. Subsequent calls with the same text are ignored.

#### Signature

```rust
fn say_once(text: ImutableString)
```

#### Parameters

| Parameter | Type     | Description           |
|-----------|----------|-----------------------|
| `text`    | `ImutableString` | The text to speak     |

#### Returns

Nothing

#### Examples

```rust
on_intent "tip_of_day" {
    dialogue::say_once("Did you know you can control your lights by voice?");
}
```

---

## Input Functions

### `dialogue::listen()`

Makes the device start listening for voice input.

#### Signature

```rust
fn listen()
```

#### Parameters

None

#### Returns

Nothing

#### Examples

```rust
on_intent "start_dictation" {
    dialogue::say("I'm listening. Please speak now.");
    dialogue::listen();
}
```

---

### `dialogue::request_attention()`

Requests the user's attention with an audio cue and starts listening.

#### Signature

```rust
fn request_attention()
```

#### Parameters

None

#### Returns

Nothing

#### Examples

```rust
on_timer "reminder" {
    dialogue::request_attention();
    dialogue::say("Your meeting starts in 5 minutes");
}
```

---

### `dialogue::repeat()`

Repeats the last thing spoken or heard.

#### Signature

```rust
fn repeat()
```

#### Parameters

None

#### Returns

Nothing

#### Examples

```rust
on_intent "repeat_request" {
    dialogue::repeat();
}
```

---

## Conversation Management

### `dialogue::on_reply()`

Registers a handler for the next user response with validation.

#### Signature

```rust
fn on_reply(handler: FnPtr, validator: Validator)
```

#### Parameters

| Parameter   | Type     | Description                                    |
|-------------|----------|------------------------------------------------|
| `handler`   | `FnPtr`  | The function to call with response             |
| `validator` | `Validator`| The validator to use for the response          |

#### Returns

Nothing

#### Examples

```rust
on_intent "order_pizza" {
    dialogue::say("What size would you like?");
    dialogue::on_reply(handle_size, dialogue::mapped_validator(#{
        "small": "small",
        "medium": "medium",
        "large": "large"
    }));
}

fn handle_size(size) {
    dialogue::say(`Great! Ordering a ${size} pizza.`);
}
```

---

### `dialogue::confirm()`

Asks the user a yes/no question and handles the response.

#### Signature

```rust
fn confirm(question_locale_id: ImutableString, handler: ImutableString)
```

#### Parameters

| Parameter            | Type     | Description                           |
|---------------------|----------|---------------------------------------|
| `question_locale_id`| `ImutableString` | The locale ID of the question to ask  |
| `handler`           | `FnPtr`  | The function to call      |

#### Returns

Nothing

#### Examples

```rust
on_intent "delete_item" {
    dialogue::confirm("confirm_delete", handle_delete_confirm);
}

fn handle_delete_confirm(confirmed) {
    if confirmed {
        delete_item();
        dialogue::say("Item deleted");
    } else {
        dialogue::say("Cancelled");
    }
}
```

---

## Validators

### `dialogue::any_validator()`

Creates a validator that accepts any input.

#### Signature

```rust
fn any_validator() -> AnyValidator
```

#### Returns

- `AnyValidator` - A validator that accepts all input

#### Examples

```rust
dialogue::say("Tell me anything");
dialogue::on_reply("handle_response", dialogue::any_validator());
```

---

### `dialogue::bool_validator()`

Creates a validator that accepts boolean input (yes/no).

#### Signature

```rust
fn bool_validator(fuzzy: bool) -> BoolValidator
```

#### Parameters

| Parameter | Type   | Description                  |
|-----------|--------|------------------------------|
| `fuzzy`   | `bool` | Whether to use fuzzy matching|

#### Returns

- `BoolValidator` - A validator for yes/no responses

#### Examples

```rust
dialogue::say("Do you want to continue?");
dialogue::on_reply("handle_continue", dialogue::bool_validator(true));

fn handle_continue(response) {
    if response {
        continue_process();
    }
}
```

---

### `dialogue::mapped_validator()`

Creates a validator that maps string input to values.

#### Signature

```rust
fn mapped_validator(map: Map) -> MappedValidator
```

#### Parameters

| Parameter | Type  | Description                               |
|-----------|-------|-------------------------------------------|
| `map`     | `Map` | A map of possible inputs to their values  |

#### Returns

- `MappedValidator` - A validator that maps recognized inputs

#### Examples

```rust
let color_map = #{
    "red": "ff0000",
    "blue": "0000ff",
    "green": "00ff00"
};

dialogue::say("What color would you like?");
dialogue::on_reply("handle_color", dialogue::mapped_validator(color_map));
```

---

### `dialogue::list_or_none_validator()`

Creates a validator that accepts values from a list or nothing.

#### Signature

```rust
fn list_or_none_validator(allowed_values: Array) -> ListOrNoneValidator
```

#### Parameters

| Parameter        | Type    | Description                      |
|-----------------|---------|----------------------------------|
| `allowed_values`| `Array` | A list of accepted string values |

#### Returns

- `ListOrNoneValidator` - A validator for list values or none

#### Examples

```rust
let options = ["option1", "option2", "option3"];
dialogue::say("Choose an option or say none");
dialogue::on_reply("handle_choice", dialogue::list_or_none_validator(options));
```

---

### `dialogue::optional_validator()`

Creates a validator wrapper that makes another validator optional.

#### Signature

```rust
fn optional_validator() -> OptionalValidator
```

#### Returns

- `OptionalValidator` - A validator that allows empty/no input

#### Examples

```rust
dialogue::say("Enter a note, or say skip");
dialogue::on_reply("handle_note", dialogue::optional_validator());
```

---

## Complete Example: Multi-Turn Dialogue

```rust
on_intent "book_appointment" {
    dialogue::say("What day would you like?");
    dialogue::on_reply(handle_day, dialogue::any_validator());
}

fn handle_day(day) {
    // Store the day
    context::set("appointment_day", day);
    
    dialogue::say("What time works for you?");
    dialogue::on_reply(handle_time, dialogue::any_validator());
}

fn handle_time(time) {
    let day = context::get("appointment_day");
    
    dialogue::say(`Booking for ${day} at ${time}. Is this correct?`);
    dialogue::on_reply(handle_confirm, dialogue::bool_validator(true));
}

fn handle_confirm(confirmed) {
    if confirmed {
        let day = context::get("appointment_day");
        // Book the appointment
        dialogue::say("Your appointment is booked!");
    } else {
        dialogue::say("Let's start over");
        // Restart the flow
    }
}
```

---

## Best Practices

### 1. Use Appropriate Validators

```rust
// ✅ Good - constrained input for clear choices
dialogue::on_reply(handler, dialogue::bool_validator(true));

// ❌ Bad - any_validator for yes/no question
dialogue::on_reply(handler, dialogue::any_validator());
```

### 2. Provide Clear Prompts

```rust
// ✅ Good - clear options
dialogue::say("Say small, medium, or large");

// ❌ Bad - ambiguous
dialogue::say("What do you want?");
```

### 3. Handle Edge Cases

```rust
fn handle_response(value) {
    if value == () {
        dialogue::say("I didn't catch that. Please try again.");
        return;
    }
    // Process value...
}
```

---

## See Also

- [Context Management](./context.md) - Storing conversation state
- [Localization](./locale.md) - Multi-language support