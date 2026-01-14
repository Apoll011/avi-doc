# `json` Namespace

**Namespace:** `json`  
**Import:** Built-in (globally available)

The `json` namespace provides functions for parsing and serializing JSON data. Use these functions to work with JSON APIs, configuration files, and data interchange.

---

## Functions

### `json::parse()`

Parses a JSON string into a Rhai object.

#### Signature

```rhai
fn parse(json_str: ImutableString) -> ?
```

#### Parameters

| Parameter  | Type     | Description              |
|------------|----------|--------------------------|
| `json_str` | `ImutableString` | The JSON string to parse |

#### Returns

- A Rhai object (Map, Array, ImutableString, Number, Bool) representing the JSON data
- Throws an error if the JSON is invalid

#### Examples

```js
// Parse JSON string
let json_str = '{"name": "Alice", "age": 30}';
let data = json::parse(json_str);
log::info(`Name: ${data.name}, Age: ${data.age}`);

// Parse API response
let response = http::get("https://api.example.com/user");
let user = json::parse(response.body);
dialogue::say(`Hello, ${user.name}!`);

// Parse array
let json_array = '[1, 2, 3, 4, 5]';
let numbers = json::parse(json_array);
log::info(`First number: ${numbers[0]}`);
```

---

### `json::to_json()`

Converts a Rhai object into a pretty-printed JSON string.

#### Signature

```rhai
fn to_json(value: ?) -> ImutableString
```

#### Parameters

| Parameter | Type | Description                    |
|-----------|------|--------------------------------|
| `value`   | `?`  | The Rhai object to convert     |

#### Returns

- `ImutableString` - A pretty-printed JSON string

#### Examples

```js
// Convert map to JSON
let user = #{
    name: "Bob",
    age: 25,
    active: true
};
let json_str = json::to_json(user);
log::info(json_str);

// Convert array to JSON
let items = ["apple", "banana", "cherry"];
let json_array = json::to_json(items);
log::info(json_array);

// Send JSON in API request
let payload = #{
    action: "create",
    data: #{
        title: "New Item",
        description: "Description here"
    }
};
http::post("https://api.example.com/items", json::to_json(payload));
```

---

## Best Practices

### 1. Handle Parse Errors

```js
// ✅ Good - error handling
try {
    let data = json::parse(json_str);
    process_data(data);
} catch (e) {
    log::error(`Failed to parse JSON: ${e}`);
    dialogue::say("Invalid data format");
}

// ❌ Bad - no error handling
let data = json::parse(json_str); // May throw
```

### 2. Validate Parsed Data

```js
// ✅ Good - validation
let data = json::parse(response);
if data.status == UNIT || data.result == UNIT {
    log::error("Missing required fields");
    return;
}

// ❌ Bad - assumes structure
let result = json::parse(response).result.items[0];
```

### 3. Use for API Integration

```js
// ✅ Good - structured API request/response
let request = #{
    method: "search",
    query: "weather",
    limit: 10
};

let response = http::post(
    "https://api.example.com/v1/search",
    json::to_json(request)
);

let result = json::parse(response.body);
```

---

## Complete Example: API Integration

```js
on_intent "fetch_weather" {
    let city = slots::get(intent, "city");
    
    // Build API request
    let request_body = #{
        city: city,
        units: "metric",
        lang: user::language()
    };
    
    log::debug(`Request: ${json::to_json(request_body)}`);
    
    try {
        // Make API call
        let response = http::post(
            "https://weather-api.example.com/current",
            json::to_json(request_body)
        );
        
        // Parse response
        let weather = json::parse(response.body);
        
        // Validate response structure
        if weather.status == UNIT {
            log::error("Invalid API response structure");
            dialogue::say("Sorry, I couldn't get the weather");
            return;
        }
        
        // Check for API errors
        if weather.status != "success" {
            log::warn(`API error: ${weather.message}`);
            dialogue::say(`Weather service error: ${weather.message}`);
            return;
        }
        
        // Extract weather data
        let temp = weather.data.temperature;
        let conditions = weather.data.conditions;
        
        dialogue::say(`It's ${temp} degrees and ${conditions} in ${city}`);
        
    } catch (e) {
        log::error(`Weather fetch failed: ${e}`);
        dialogue::say("I couldn't fetch the weather right now");
    }
}
```

---

## Common Patterns

### Working with Nested Data

```js
let json_str = '{
    "user": {
        "profile": {
            "name": "Alice",
            "settings": {
                "theme": "dark"
            }
        }
    }
}';

let data = json::parse(json_str);
let theme = data.user.profile.settings.theme;
log::info(`Theme: ${theme}`);
```

### Array Operations

```js
let json_array = '[
    {"id": 1, "name": "Item 1"},
    {"id": 2, "name": "Item 2"},
    {"id": 3, "name": "Item 3"}
]';

let items = json::parse(json_array);
foreach (item in items) {
    log::info(`${item.id}: ${item.name}`);
}
```

### Configuration Files

```js
// Load JSON config file
let config_path = `${skill::dir()}/config.json`;
let config_str = file::read(config_path);
let config = json::parse(config_str);

// Access configuration values
let api_url = config.api.url;
let timeout = config.api.timeout;
```

---

## See Also

- [`http` namespace](./http.md) - Making API requests
- [`file` namespace](./file.md) - Reading/writing JSON files
- [Data Structures](../language/data-structures.md) - Working with Maps and Arrays