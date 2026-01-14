# `array` Namespace

**Namespace:** `rand::array`  
**Import:** `import "rand::array";`

The `array` namespace provides utility functions for working with arrays, including random sampling and shuffling operations. These functions are particularly useful for randomization, game logic, and creating varied user experiences.

> **Note:** Array manipulation functions require the `array` feature to be enabled in the runtime.

---

## Functions

### `array::sample()`

Randomly selects and returns one or more elements from an array.

#### Signatures

```rust
fn sample(array: Array) -> ?
fn sample(array: Array, amount: int) -> Array
```

#### Parameters

| Parameter | Type    | Description                                    |
|-----------|---------|------------------------------------------------|
| `array`   | `Array` | The source array to sample from                |
| `amount`  | `int`   | (Optional) Number of random elements to return |

#### Returns

- **Single element mode**: Returns a random element of any type (`?`)
- **Multiple elements mode**: Returns an `Array` containing `amount` random elements

#### Behavior

- When called with one argument, returns a single random element
- When called with two arguments, returns an array of random samples
- The original array is **not modified**
- Samples are selected with equal probability (uniform distribution)

#### Examples

**Basic Usage - Single Random Element**

```js
let numbers = [1, 2, 3, 4, 5];
let random_num = numbers.sample();

print(`I'll give you a random number between 1 and 5: ${random_num}`);
// Output: "I'll give you a random number between 1 and 5: 3" (varies)
```

**Advanced Usage - Multiple Random Elements**

```js
let colors = ["red", "blue", "green", "yellow", "purple"];
let selected = colors.sample(3);

print(`Your random colors are: ${selected}`);
// Output: "Your random colors are: ["blue", "yellow", "red"]" (varies)
```

**Practical Example - Random Greeting**

```js
let greetings = [
    "Hello there!",
    "Good to see you!",
    "Welcome back!",
    "Hi! How can I help?"
];

say(greetings.sample());
```

> **Tip:** Use `sample()` to add variety to your assistant's responses and make interactions feel more natural.

---

### `array::shuffle()`

Randomly reorders all elements in an array in place.

#### Signature

```rust
fn shuffle(array: Array)
```

#### Parameters

| Parameter | Type    | Description                |
|-----------|---------|----------------------------|
| `array`   | `Array` | The array to shuffle       |

#### Returns

- `void` (modifies the array in place)

#### Behavior

- Shuffles elements using a uniform random distribution
- The original array is **modified directly**
- All elements remain in the array (no elements added or removed)
- Uses Fisher-Yates algorithm for fair randomization

#### Examples

**Basic Usage**

```js
let deck = [1, 2, 3, 4, 5];
deck.shuffle();

print(deck);
// Output: [3, 1, 5, 2, 4] (order varies each time)
```

**Practical Example - Playlist Shuffle**

```js
let playlist = [
    "song_01.mp3",
    "song_02.mp3",
    "song_03.mp3",
    "song_04.mp3"
];

playlist.shuffle();
log::info(`Shuffled playlist: ${playlist}`);

// Play songs in random order
foreach (song in playlist) {
    call("speaker-main", "play_audio", { url: song });
}
```

**Advanced Example - Quiz Question Randomization**

```js
let questions = [
    { q: "What is 2+2?", a: "4" },
    { q: "Capital of France?", a: "Paris" },
    { q: "Largest planet?", a: "Jupiter" }
];

questions.shuffle();

foreach (item in questions) {
    say(item.q);
    // Wait for user answer...
}
```

> **Warning:** `shuffle()` modifies the original array. If you need to preserve the original order, create a copy first using `array.clone()`.

---

## Complete Example: Random Quiz Game

This example combines both `sample()` and `shuffle()` to create a dynamic quiz experience:

```js
module QuizGame {
    intent "start_quiz" {
        on_enter {
            let categories = ["history", "science", "geography", "sports"];
            
            // Let user choose or pick random category
            let category = categories.sample();
            say(`Let's play a ${category} quiz!`);
            
            // Get questions for category
            let questions = get_questions(category);
            
            // Shuffle questions for variety
            questions.shuffle();
            
            // Ask first 5 questions
            let quiz_set = questions.sample(5);
            
            foreach (q in quiz_set) {
                say(q.question);
                // Handle user response...
            }
        }
    }
}
```

---

## Requirements

- **Feature Flag:** `array` must be enabled in runtime configuration
- **Import:** Functions available globally when imported via `rand` module
- **Runtime:** Requires Avi runtime v1.0 or higher

---

## See Also

- [`rand` namespace](./rand.md) - Random number generation
- [`util` namespace](./util.md) - General utility functions
- [Arrays in AviScript](../language/arrays.md) - Array syntax and operations