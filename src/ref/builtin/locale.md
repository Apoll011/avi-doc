# `locale` Namespace

**Namespace:** `locale`  
**Import:** `import "locale";`

The `locale` namespace provides internationalization (i18n) support for skills, enabling multi-language responses and localized content. It manages translation strings, formats messages with parameters, and handles language detection and switching.

> **Note:** Translation files are stored in the skill's `locale/` directory with filenames like `en-US.json`, `pt-BR.json`, etc.

---

## Functions

### `locale::current()`

Returns the currently active language code for the skill.

#### Signature

```rust
fn current() -> ImutableString
```

#### Parameters

None

#### Returns

- `ImutableString` - The current language code in IETF BCP 47 format (e.g., `"en-US"`, `"pt-BR"`, `"es-ES"`)

#### Behavior

- Returns the language code set by the user or system
- Language code follows the format: `language-REGION` (e.g., `en-US`, `fr-CA`)
- Falls back to default language if user preference is not set

#### Examples

**Basic Usage**

```js
let lang = locale::current();
log::info(`Current language: ${lang}`);
// Output: "Current language: en-US"
```

**Conditional Localization**

```js
on_intent "greet" {
    let lang = locale::current();
    
    if lang == "pt-BR" {
        say("Olá! Como posso ajudar?");
    } else if lang == "es-ES" {
        say("¡Hola! ¿Cómo puedo ayudar?");
    } else {
        say("Hello! How can I help?");
    }
}
```

**Language-Specific Features**

```js
on_start {
    let lang = locale::current();
    
    if lang.starts_with("en") {
        // Enable English-specific features
        enable_idiom_detection();
    } else if lang.starts_with("zh") {
        // Enable Chinese-specific features
        enable_tone_analysis();
    }
    
    log::info(`Skill initialized for ${lang}`);
}
```

**Regional Customization**

```js
on_intent "get_weather_units" {
    let lang = locale::current();
    
    // Default to metric, except for US
    let units = "celsius";
    if lang == "en-US" {
        units = "fahrenheit";
    }
    
    say(`Temperature will be shown in ${units}.`);
}
```

---

### `locale::get()`

Retrieves a translation string for the given ID in the current locale.

#### Signature

```rust
fn get(id: ImutableString) -> ImutableString
```

#### Parameters

| Parameter | Type     | Description                           |
|-----------|----------|---------------------------------------|
| `id`      | `ImutableString` | The translation key/ID to retrieve    |

#### Returns

- `ImutableString` - The translated text if found
- `UNIT` - If the translation ID does not exist

#### Behavior

- Looks up the translation in the current locale's translation file
- Returns the raw string without parameter substitution
- Falls back to the translation ID itself if not found (in development mode)
- Case-sensitive key matching

#### Examples

**Basic Usage**

```js
let greeting = locale::get("greeting.hello");
say(greeting);
// Speaks: "Hello!" (if en-US) or "Olá!" (if pt-BR)
```

**With Fallback**

```js
on_intent "welcome" {
    let welcome_msg = locale::get("messages.welcome");
    
    if welcome_msg == UNIT {
        log::warn("Missing translation: messages.welcome");
        say("Welcome!"); // Hardcoded fallback
    } else {
        say(welcome_msg);
    }
}
```

**Error Message Translation**

```js
on_intent "connect_device" {
    let result = try_connect_device();
    
    if !result.success {
        let error_msg = locale::get(`errors.${result.error_code}`);
        say(error_msg);
    }
}
```

**Menu Options**

```js
on_intent "show_menu" {
    let options = [
        locale::get("menu.settings"),
        locale::get("menu.help"),
        locale::get("menu.about")
    ];
    
    say(`Available options: ${options.join(", ")}`);
}
```

---

### `locale::get_fmt()`

Retrieves and formats a translation string with parameter interpolation.

#### Signature

```rust
fn get_fmt(id: ImutableString, params: Map) -> ImutableString
```

#### Parameters

| Parameter | Type     | Description                                    |
|-----------|----------|------------------------------------------------|
| `id`      | `ImutableString` | The translation key/ID to retrieve             |
| `params`  | `Map`    | Key-value pairs for parameter substitution     |

#### Returns

- `ImutableString` - The formatted translation with parameters substituted
- `UNIT` - If the translation ID does not exist

#### Behavior

- Supports placeholder syntax: `{param_name}` in translation strings
- Performs parameter substitution with values from the `params` map
- Missing parameters are left as-is or replaced with empty string
- Supports nested parameter references

#### Examples

**Basic Interpolation**

```js
// Translation file (en-US.json):
// "greeting.personalized": "Hello, {name}! Welcome back."

let msg = locale::get_fmt("greeting.personalized", #{ name: "Alice" });
say(msg);
// Speaks: "Hello, Alice! Welcome back."
```

**Multiple Parameters**

```js
// Translation: "weather.report": "It's {temp} degrees and {condition} in {city}."

let weather_msg = locale::get_fmt("weather.report", #{
    temp: 72,
    condition: "sunny",
    city: "San Francisco"
});
say(weather_msg);
// Speaks: "It's 72 degrees and sunny in San Francisco."
```

**Counting and Pluralization**

```js
// Translation: "items.count": "You have {count} item(s) in your cart."

on_intent "check_cart" {
    let item_count = get_cart_items().len();
    let msg = locale::get_fmt("items.count", #{ count: item_count });
    say(msg);
}
```

**Date Formatting**

```js
// Translation: "reminder.scheduled": "Reminder set for {date} at {time}."

on_intent "set_reminder" {
    let reminder_time = params.datetime;
    
    let msg = locale::get_fmt("reminder.scheduled", #{
        date: reminder_time.date,
        time: reminder_time.time
    });
    
    say(msg);
}
```

**Dynamic Error Messages**

```js
// Translation: "errors.device_unreachable": "Unable to reach {device_name}. Please check if it's powered on."

on_intent "control_device" {
    try {
        call(params.device_id, "turn_on");
    } catch(e) {
        let device = get_device(params.device_id);
        let error_msg = locale::get_fmt("errors.device_unreachable", #{
            device_name: device.name
        });
        say(error_msg);
    }
}
```

**Complex Interpolation**

```js
// Translation: "booking.confirmation": 
//   "Your {service} is booked for {date} at {time}. Total: ${price}."

let confirmation = locale::get_fmt("booking.confirmation", #{
    service: "haircut",
    date: "March 15th",
    time: "2:00 PM",
    price: 45.00
});

say(confirmation);
// Speaks: "Your haircut is booked for March 15th at 2:00 PM. Total: $45.00."
```

---

### `locale::has()`

Checks if a translation exists for the given ID in the current locale.

#### Signature

```rust
fn has(id: ImutableString) -> bool
```

#### Parameters

| Parameter | Type     | Description                     |
|-----------|----------|---------------------------------|
| `id`      | `ImutableString` | The translation key/ID to check |

#### Returns

- `true` if the translation exists
- `false` if the translation does not exist

#### Behavior

- Performs case-sensitive key lookup
- Checks only the current locale
- More efficient than retrieving and checking for `UNIT`
- Useful for optional translations or feature detection

#### Examples

**Basic Usage**

```js
if locale::has("feature.premium_greeting") {
    let greeting = locale::get("feature.premium_greeting");
    say(greeting);
} else {
    say("Hello!");
}
```

**Graceful Degradation**

```js
on_intent "get_help" {
    let help_key = `help.${params.topic}`;
    
    if locale::has(help_key) {
        say(locale::get(help_key));
    } else {
        say(locale::get("help.default"));
        log::warn(`Missing help translation: ${help_key}`);
    }
}
```

**Feature Availability Check**

```js
on_start {
    // Check if skill is fully translated
    let required_keys = [
        "greeting.hello",
        "greeting.goodbye",
        "errors.generic",
        "help.main"
    ];
    
    let missing = [];
    foreach (key in required_keys) {
        if !locale::has(key) {
            missing.push(key);
        }
    }
    
    if missing.len() > 0 {
        log::error(`Missing translations: ${missing.join(", ")}`);
        return false;
    }
    
    log::info("All required translations present");
    return true;
}
```

**Locale Completeness**

```js
// Check translation coverage
let total_keys = 0;
let translated_keys = 0;

let required_translations = [
    "menu.home", "menu.settings", "menu.help",
    "error.network", "error.timeout"
];

foreach (key in required_translations) {
    total_keys += 1;
    if locale::has(key) {
        translated_keys += 1;
    }
}

let coverage = (translated_keys / total_keys) * 100;
log::info(`Translation coverage: ${coverage}%`);
```

---

### `locale::list()`

Returns all translation entries for a specific locale code.

#### Signature

```rust
fn list(code: ImutableString) -> Map
```

#### Parameters

| Parameter | Type     | Description                            |
|-----------|----------|----------------------------------------|
| `code`    | `ImutableString` | The locale code (e.g., `"en-US"`)      |

#### Returns

- `Map` - A map containing all translation key-value pairs for the locale
- Empty map if locale not found

#### Behavior

- Returns complete translation dictionary for the specified locale
- Does not perform parameter substitution
- Useful for debugging, export, or custom translation handling
- Keys are strings, values are translation strings

#### Examples

**Basic Usage**

```js
let translations = locale::list("en-US");
log::info(`Loaded ${translations.len()} translations for en-US`);
```

**Translation Debugging**

```js
on_intent "debug_translations" {
    let current_lang = locale::current();
    let translations = locale::list(current_lang);
    
    say(`Current locale has ${translations.len()} translations.`);
    
    // Log all translation keys
    foreach (key, value in translations) {
        log::info(`${key} = ${value}`);
    }
}
```

**Compare Locales**

```js
on_start {
    let en_translations = locale::list("en-US");
    let pt_translations = locale::list("pt-BR");
    
    // Find missing translations in pt-BR
    let missing = [];
    foreach (key, _ in en_translations) {
        if !pt_translations.contains(key) {
            missing.push(key);
        }
    }
    
    if missing.len() > 0 {
        log::warn(`Portuguese missing ${missing.len()} translations`);
        log::warn(`Missing keys: ${missing.join(", ")}`);
    }
}
```

**Export Translations**

```js
on_intent "export_locale" {
    let locale_code = params.locale ?? "en-US";
    let translations = locale::list(locale_code);
    
    // Convert to JSON for export
    let json_data = json::stringify(translations);
    fs::write(`exports/${locale_code}.json`, json_data);
    
    say(`Exported ${translations.len()} translations for ${locale_code}.`);
}
```

**Translation Statistics**

```js
on_intent "translation_stats" {
    let supported_locales = ["en-US", "pt-BR", "es-ES", "fr-FR"];
    let stats = [];
    
    foreach (locale_code in supported_locales) {
        let translations = locale::list(locale_code);
        stats.push(#{
            locale: locale_code,
            count: translations.len()
        });
    }
    
    foreach (stat in stats) {
        log::info(`${stat.locale}: ${stat.count} translations`);
    }
}
```

---

## Complete Example: Multi-Language Weather Skill

```js
// Translation files structure:
// locale/en-US.json: { "weather.greeting": "Hello!", "weather.current": "It's {temp}°F and {condition}." }
// locale/pt-BR.json: { "weather.greeting": "Olá!", "weather.current": "Está {temp}°C e {condition}." }

on_start {
    let current_lang = locale::current();
    log::info(`Weather skill starting in ${current_lang}`);
    
    // Verify required translations exist
    let required = ["weather.greeting", "weather.current", "weather.error"];
    foreach (key in required) {
        if !locale::has(key) {
            log::error(`Missing translation: ${key} in ${current_lang}`);
            return false;
        }
    }
    
    return true;
}

on_intent "get_weather" {
    let location = params.location ?? "current location";
    
    // Greet user in their language
    let greeting = locale::get("weather.greeting");
    say(greeting);
    
    // Fetch weather data
    let weather = fetch_weather(location);
    
    if weather == UNIT {
        let error_msg = locale::get("weather.error");
        say(error_msg);
        return;
    }
    
    // Format temperature based on locale
    let lang = locale::current();
    let temp = weather.temp;
    
    // Convert temperature for US locale
    if lang == "en-US" {
        temp = celsius_to_fahrenheit(temp);
    }
    
    // Translate condition
    let condition_key = `weather.conditions.${weather.condition}`;
    let condition = locale::has(condition_key) 
        ? locale::get(condition_key)
        : weather.condition;
    
    // Format and speak weather report
    let report = locale::get_fmt("weather.current", #{
        temp: temp.round(),
        condition: condition
    });
    
    say(report);
}
```

---

## Translation File Format

Translation files are JSON objects stored in the `locale/` directory:

**`locale/en-US.json`:**
```json
{
  "greeting.hello": "Hello!",
  "greeting.goodbye": "Goodbye!",
  "weather.current": "It's {temp} degrees and {condition}.",
  "errors.generic": "Something went wrong.",
  "menu.settings": "Settings",
  "menu.help": "Help"
}
```

**`locale/pt-BR.json`:**
```json
{
  "greeting.hello": "Olá!",
  "greeting.goodbye": "Tchau!",
  "weather.current": "Está {temp} graus e {condition}.",
  "errors.generic": "Algo deu errado.",
  "menu.settings": "Configurações",
  "menu.help": "Ajuda"
}
```

---

## Best Practices

### 1. Always Check Translation Existence

```js
// ✅ Good - defensive check
if locale::has("message.key") {
    say(locale::get("message.key"));
} else {
    say("Default message");
}

// ❌ Bad - no fallback
say(locale::get("message.key")); // Could be UNIT
```

### 2. Use Namespaced Keys

```js
// ✅ Good - organized with namespaces
locale::get("weather.greeting")
locale::get("weather.errors.api_failed")

// ❌ Bad - flat structure
locale::get("greeting1")
locale::get("error5")
```

### 3. Extract All User-Facing Text

```js
// ✅ Good - all text is translatable
say(locale::get("welcome.message"));
log::info(locale::get("log.user_connected"));

// ❌ Bad - hardcoded text
say("Welcome to the app!");
log::info("User connected");
```

### 4. Use `get_fmt()` for Dynamic Content

```js
// ✅ Good - parameterized translation
locale::get_fmt("user.greeting", #{ name: user_name })

// ❌ Bad - string concatenation
locale::get("greeting") + ", " + user_name + "!"
```

---

## Common Use Cases

- **Multi-language voice responses**
- **Localized error messages**
- **Region-specific date/time formats**
- **Currency and unit conversions**
- **Dynamic content with user data**
- **Menu and UI text translation**

---

## See Also

- [Internationalization Guide](../guides/i18n.md) - Comprehensive i18n strategies
- [`user` namespace](./user.md) - User preferences including language
- [Translation File Format](../reference/translations.md) - Detailed format specs