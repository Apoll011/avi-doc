# `rand` Namespace

**Namespace:** `rand`  
**Import:** Built-in (globally available)

The `rand` namespace provides functions for generating random numbers and boolean values. Use these functions to add variability, make random selections, or implement probabilistic behavior in your skills.

---

## Functions

### `rand::rand()`

Generates a random integer number.

#### Signatures

```rhai
fn rand() -> int
fn rand(range: Range) -> int
fn rand(start: int, end: int) -> int
```

#### Parameters

**Overload 1: No parameters**
- Returns a random integer across the full integer range

**Overload 2: Range**
| Parameter | Type    | Description                                    |
|-----------|---------|------------------------------------------------|
| `range`   | `Range` | A range object (e.g., `0..10` or `0..=10`)    |

**Overload 3: Start and End**
| Parameter | Type  | Description                  |
|-----------|-------|------------------------------|
| `start`   | `int` | The start of the range       |
| `end`     | `int` | The end of the range         |

#### Returns

- `int` - A random integer

#### Examples

```js
// Generate any random integer
let number = rand::rand();
log::info(`Random number: ${number}`);

// Generate number in range (exclusive end)
let dice = rand::rand(1, 7); // 1-6
dialogue::say(`You rolled a ${dice}`);

// Generate number in range using range syntax
let percent = rand::rand(0..101); // 0-100
log::info(`Random percentage: ${percent}%`);

// Inclusive range
let inclusive = rand::rand(1..=10); // 1-10 inclusive
```

---

### `rand::rand_bool()`

Generates a random boolean value.

#### Signatures

```rhai
fn rand_bool() -> bool
fn rand_bool(probability: float) -> bool
```

#### Parameters

**Overload 1: No parameters**
- 50/50 chance of true or false

**Overload 2: With probability**
| Parameter     | Type    | Description                                |
|--------------|---------|---------------------------------------------|
| `probability`| `float` | Probability of returning true (0.0 to 1.0) |

#### Returns

- `bool` - A random boolean value

#### Examples

```js
// 50/50 coin flip
let coin = rand::rand_bool();
if coin {
    dialogue::say("Heads!");
} else {
    dialogue::say("Tails!");
}

// 75% chance of true
let likely = rand::rand_bool(0.75);
if likely {
    dialogue::say("This happens 75% of the time");
}

// 10% chance (rare event)
if rand::rand_bool(0.1) {
    dialogue::say("You hit the jackpot!");
}
```

---

### `rand::rand_float()`

Generates a random floating-point number between 0.0 and 1.0 (exclusive).

#### Signatures

```rhai
fn rand_float() -> float
fn rand_float(start: float, end: float) -> float
```

#### Parameters

**Overload 1: No parameters**
- Returns a float between 0.0 (inclusive) and 1.0 (exclusive)

**Overload 2: Custom range**
| Parameter | Type    | Description              |
|-----------|---------|--------------------------|
| `start`   | `float` | The start of the range   |
| `end`     | `float` | The end of the range     |

#### Returns

- `float` - A random floating-point number

#### Behavior

> **Note:** 1.0 is excluded from the possibilities in the no-parameter version.

#### Examples

```js
// Generate float between 0.0 and 1.0
let factor = rand::rand_float();
log::info(`Random factor: ${factor}`);

// Generate float in custom range
let temperature = rand::rand_float(20.0, 30.0);
dialogue::say(`Random temperature: ${temperature} degrees`);

// Use for weighted calculations
let multiplier = rand::rand_float(0.5, 1.5);
let adjusted_value = base_value * multiplier;
```

---

## Common Use Cases

### Random Selection from Array

```js
let greetings = ["Hello", "Hi", "Hey", "Greetings"];
let index = rand::rand(0, greetings.len());
let greeting = greetings[index];
dialogue::say(greeting);
```

### Weighted Probability

```js
let chance = rand::rand_float();
if chance < 0.6 {
    // 60% chance
    dialogue::say("Common event");
} else if chance < 0.9 {
    // 30% chance
    dialogue::say("Uncommon event");
} else {
    // 10% chance
    dialogue::say("Rare event!");
}
```

### Random Delays

```js
// Random delay between 1 and 5 seconds
let delay_ms = rand::rand(1000, 5001);
timer::sleep(delay_ms);
```

### Shuffle-like Behavior

```js
let items = ["A", "B", "C", "D"];
let shuffled = [];

while items.len() > 0 {
    let index = rand::rand(0, items.len());
    shuffled.push(items[index]);
    items.remove(index);
}
```

---

## Best Practices

### 1. Use Appropriate Ranges

```js
// ✅ Good - clear, inclusive dice roll
let dice = rand::rand(1, 7); // 1-6

// ❌ Bad - confusing range
let dice = rand::rand(0, 6) + 1; // Works but less clear
```

### 2. Use Probability for Binary Outcomes

```js
// ✅ Good - use rand_bool for binary choices
if rand::rand_bool(0.3) {
    trigger_rare_event();
}

// ❌ Bad - unnecessary complexity
if rand::rand_float() < 0.3 {
    trigger_rare_event();
}
```

### 3. Cache Random Values

```js
// ✅ Good - generate once, use multiple times
let random_factor = rand::rand_float();
let adjusted_a = value_a * random_factor;
let adjusted_b = value_b * random_factor;

// ❌ Bad - different random values
let adjusted_a = value_a * rand::rand_float();
let adjusted_b = value_b * rand::rand_float();
```

---

## Complete Example: Interactive Game

```js
on_intent "play_number_game" {
    dialogue::say("I'm thinking of a number between 1 and 10. Can you guess it?");
    
    let secret_number = rand::rand(1, 11);
    context::set("secret_number", secret_number);
    context::set("attempts", 0);
    
    dialogue::on_reply("handle_guess", dialogue::any_validator());
}

fn handle_guess(guess_str) {
    let secret = context::get("secret_number");
    let attempts = context::get("attempts") + 1;
    context::set("attempts", attempts);
    
    let guess = parse_int(guess_str);
    
    if guess == UNIT {
        dialogue::say("Please say a number between 1 and 10");
        dialogue::on_reply("handle_guess", dialogue::any_validator());
        return;
    }
    
    if guess == secret {
        dialogue::say(`Correct! You got it in ${attempts} attempts!`);
        
        // Vary encouragement based on attempts
        if attempts == 1 {
            dialogue::say("Amazing! First try!");
        } else if attempts <= 3 {
            dialogue::say("Great job!");
        } else {
            dialogue::say("You did it!");
        }
    } else if guess < secret {
        dialogue::say("Higher! Try again.");
        dialogue::on_reply("handle_guess", dialogue::any_validator());
    } else {
        dialogue::say("Lower! Try again.");
        dialogue::on_reply("handle_guess", dialogue::any_validator());
    }
}
```

---

## See Also

- [Control Flow](../language/control-flow.md) - Using randomness in conditionals
- [Arrays](../language/arrays.md) - Random selection from collections