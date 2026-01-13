# Skill Lifecycle Hooks

AviScript provides lifecycle hooks that allow your skills to respond to specific events during their execution. These hooks enable you to run code at critical moments in a skill's lifecycle, such as when it starts, ends, or receives external events.

---

## Lifecycle Hooks

### `on_start`

Executes when the skill is first loaded or initialized.

#### Syntax

```js
on_start {
    // Initialization code
}
```

#### Use Cases

- Initialize settings and configuration
- Load persistent data
- Validate required constants
- Set up connections to external services
- Register background tasks

#### Examples

```js
on_start {
    log::info("Weather skill starting up");
    
    // Validate required configuration
    if !constant::has("API_KEY") {
        log::error("API_KEY not configured");
        return false;
    }
    
    // Initialize cache directory
    let cache_dir = `${skill::dir()}/cache`;
    if !fs::exists(cache_dir) {
        fs::mkdir(cache_dir);
    }
    
    log::info("Weather skill initialized successfully");
    return true;
}
```

---

### `on_end`

Executes when the skill is being shut down or unloaded.

#### Syntax

```js
on_end {
    // Cleanup code
}
```

#### Use Cases

- Save state before shutdown
- Close connections to external services
- Clean up temporary files
- Log shutdown information
- Perform final data synchronization

#### Examples

```js
on_end {
    log::info("Weather skill shutting down");
    
    // Save current cache
    let cache_data = context::get("weather_cache");
    if cache_data != UNIT {
        let cache_path = `${skill::dir()}/cache/weather.json`;
        fs::write(cache_path, json::to_json(cache_data));
    }
    
    // Clean up temporary files
    let temp_dir = `${skill::dir()}/temp`;
    if fs::exists(temp_dir) {
        let files = fs::list_files(temp_dir);
        foreach (file in files) {
            fs::delete(`${temp_dir}/${file}`);
        }
    }
    
    log::info("Weather skill shutdown complete");
}
```

---

## Event Subscriptions

### `subscribe`

Subscribes to external events or message topics, executing code when messages are received.

#### Syntax

```js
subscribe topic "topic_name" as <identifier> {
    // Event handling code
    // Use 'identifier' to access message data
    // Use 'from' to access sender information
}

subscribe event "event_name" as <identifier> {
    // Event handling code
}
```

#### Parameters

| Parameter | Description                                      |
|-----------|--------------------------------------------------|
| `topic`   | Subscribe to a message topic (pub/sub pattern)   |
| `event`   | Subscribe to a named event                       |
| `identifier` | Variable name for the received data           |

#### Special Variables

- `from` - Contains information about the event sender
- `<identifier>` - Contains the event/message data

#### Examples

**Topic Subscription**

```js
// Subscribe to intent execution events
subscribe topic "intent/execute/text" as <input> {
    let text = util::to_string(input);
    log::info(`Received intent: ${text}`);
    
    // Process the intent text
    if text.contains("emergency") {
        trigger_alert(text);
    }
}

// Subscribe to system events
subscribe topic "system/battery/low" as <level> {
    let battery_level = util::to_string(level);
    log::warn(`Low battery: ${battery_level}%`);
    dialogue::say("Battery is running low");
}
```

**Event Subscription**

```js
// Subscribe to custom skill events
subscribe event "timer_expired" as <data> {
    log::info(`Timer expired from: ${from}`);
    
    let timer_id = data.timer_id;
    let duration = data.duration;
    
    dialogue::say(`Your ${duration} minute timer has expired`);
    play_alarm_sound();
}

// Subscribe to device state changes
subscribe event "device_connected" as <device> {
    log::info(`Device connected: ${device.name}`);
    dialogue::say(`${device.name} is now connected`);
}
```

---

## Complete Example: Smart Home Skill

```js
// Initialize skill on startup
on_start {
    log::info("Smart home skill starting");
    
    // Validate configuration
    if !constant::has("HOME_NETWORK") {
        log::error("HOME_NETWORK not configured");
        return false;
    }
    
    // Initialize device registry
    let devices_file = `${skill::dir()}/devices.json`;
    if fs::exists(devices_file) {
        let devices_json = fs::read(devices_file);
        let devices = json::parse(devices_json);
        context::set("devices", devices);
        log::info(`Loaded ${devices.len()} devices`);
    } else {
        context::set("devices", []);
    }
    
    return true;
}

// Listen for device state changes
subscribe topic "home/devices/state" as <state_data> {
    let state = util::to_string(state_data);
    let state_obj = json::parse(state);
    
    log::info(`Device ${state_obj.device_id} changed to ${state_obj.state}`);
    
    // Update device registry
    let devices = context::get("devices");
    foreach (device in devices) {
        if device.id == state_obj.device_id {
            device.state = state_obj.state;
            break;
        }
    }
    context::set("devices", devices);
}

// Listen for automation triggers
subscribe event "automation_trigger" as <trigger> {
    log::info(`Automation triggered: ${trigger.name}`);
    
    let action = trigger.action;
    let target = trigger.target;
    
    if action == "turn_on" {
        turn_on_device(target);
        dialogue::say(`Turning on ${target}`);
    } else if action == "turn_off" {
        turn_off_device(target);
        dialogue::say(`Turning off ${target}`);
    }
}

// Clean up on shutdown
on_end {
    log::info("Smart home skill shutting down");
    
    // Save device states
    let devices = context::get("devices");
    if devices != UNIT {
        let devices_file = `${skill::dir()}/devices.json`;
        fs::write(devices_file, json::to_json(devices));
        log::info("Device states saved");
    }
    
    log::info("Smart home skill shutdown complete");
}
```

---

## Best Practices

### 1. Always Return from on_start

```js
// ✅ Good - indicates success/failure
on_start {
    if !initialize() {
        return false;
    }
    return true;
}

// ❌ Bad - no return value
on_start {
    initialize();
}
```

### 2. Clean Up Resources in on_end

```js
// ✅ Good - proper cleanup
on_end {
    close_connections();
    save_state();
    cleanup_temp_files();
}
```

### 3. Handle Subscription Data Safely

```js
// ✅ Good - error handling
subscribe topic "data/stream" as <data> {
    try {
        let text = util::to_string(data);
        let obj = json::parse(text);
        process(obj);
    } catch (e) {
        log::error(`Failed to process data: ${e}`);
    }
}
```
---

## See Also

- [`context` namespace](../namespaces/context.md) - Managing skill state
- [Event System](./events.md) - Understanding Avi's event system
- [Message Topics](./topics.md) - Available message topics