# `ml` Namespace

**Namespace:** `ml`  
**Import:** Built-in (globally available)

The `ml` namespace provides machine learning capabilities powered by [smartcore](https://smartcorelib.org/). Use these functions to train models on data and make predictions, enabling your skills to learn patterns and make intelligent decisions.

---

## Functions

### `ml::train()`

Trains a machine learning model on provided data.

#### Signature

```rhai
fn train(x: Array, y: Array, algorithm: String) -> Model
```

#### Parameters

| Parameter   | Type     | Description                                    |
|-------------|----------|------------------------------------------------|
| `x`         | `Array`  | 2D array of input features (training data)    |
| `y`         | `Array`  | 1D array of target values (labels)            |
| `algorithm` | `String` | The algorithm to use for training              |

#### Returns

- `Model` - A trained model that can be used for predictions

#### Available Algorithms

1. **`"linear"`** - Ordinary least squares linear regression
2. **`"logistic"`** - Logistic regression for binary classification
3. **`"lasso"`** - Lasso regression with L1 regularization

#### Examples

```js
// Train a linear regression model
let x_data = [
    [1.0, 2.0],
    [2.0, 3.0],
    [3.0, 4.0],
    [4.0, 5.0]
];
let y_data = [3.0, 5.0, 7.0, 9.0];

let model = ml::train(x_data, y_data, "linear");
log::info("Model trained successfully");

// Train a logistic regression model
let x_binary = [
    [1.0, 2.0],
    [2.0, 1.0],
    [3.0, 4.0],
    [4.0, 3.0]
];
let y_binary = [0.0, 0.0, 1.0, 1.0];

let classifier = ml::train(x_binary, y_binary, "logistic");
```

---

### `ml::predict()`

Uses a trained model to predict dependent variables for new input data.

#### Signature

```rhai
fn predict(x: Array, model: Model) -> Array
```

#### Parameters

| Parameter | Type    | Description                              |
|-----------|---------|------------------------------------------|
| `x`       | `Array` | 2D array of input features to predict on |
| `model`   | `Model` | A trained model from `ml::train()`       |

#### Returns

- `Array` - Predictions for each input sample

#### Examples

```js
// Make predictions with trained model
let x_test = [
    [5.0, 6.0],
    [6.0, 7.0]
];

let predictions = ml::predict(x_test, model);
log::info(`Predictions: ${predictions}`);

// Predict single value
let single_input = [[3.5, 4.5]];
let result = ml::predict(single_input, model);
dialogue::say(`The predicted value is ${result[0]}`);
```

---

## Complete Example: Temperature Prediction

```js
on_intent "train_temperature_model" {
    dialogue::say("Training temperature prediction model...");
    
    // Historical data: [hour, humidity] -> temperature
    let x_data = [
        [6.0, 65.0],   // 6 AM, 65% humidity
        [9.0, 60.0],   // 9 AM, 60% humidity
        [12.0, 55.0],  // 12 PM, 55% humidity
        [15.0, 50.0],  // 3 PM, 50% humidity
        [18.0, 60.0],  // 6 PM, 60% humidity
        [21.0, 70.0]   // 9 PM, 70% humidity
    ];
    
    let y_data = [15.0, 20.0, 25.0, 28.0, 23.0, 18.0]; // Temperatures
    
    try {
        let model = ml::train(x_data, y_data, "linear");
        
        // Store model for later use
        context::set("temperature_model", model);
        
        dialogue::say("Model trained successfully!");
        log::info("Temperature model ready");
        
    } catch (e) {
        log::error(`Training failed: ${e}`);
        dialogue::say("Failed to train the model");
    }
}

on_intent "predict_temperature" {
    let hour = slots::get(intent, "hour");
    let humidity = slots::get(intent, "humidity");
    
    if hour == UNIT || humidity == UNIT {
        dialogue::say("Please specify both the hour and humidity");
        return;
    }
    
    let model = context::get("temperature_model");
    if model == UNIT {
        dialogue::say("Please train the model first");
        return;
    }
    
    // Make prediction
    let input = [[hour, humidity]];
    let prediction = ml::predict(input, model);
    
    let temp = prediction[0];
    dialogue::say(`Predicted temperature at ${hour}:00 with ${humidity}% humidity is ${temp} degrees`);
}
```

---

## Complete Example: User Preference Classification

```js
on_intent "train_preference_model" {
    dialogue::say("Learning your music preferences...");
    
    // Features: [tempo (BPM), energy (0-100), danceability (0-100)]
    let x_songs = [
        [120.0, 60.0, 70.0],  // Mid-tempo, moderate energy
        [140.0, 80.0, 85.0],  // Fast, high energy
        [90.0, 40.0, 50.0],   // Slow, low energy
        [128.0, 75.0, 80.0],  // Upbeat
        [100.0, 50.0, 60.0],  // Mellow
        [135.0, 85.0, 90.0]   // Energetic
    ];
    
    // Labels: 1.0 = liked, 0.0 = disliked
    let y_preference = [0.0, 1.0, 0.0, 1.0, 0.0, 1.0];
    
    let model = ml::train(x_songs, y_preference, "logistic");
    context::set("preference_model", model);
    
    dialogue::say("I've learned your preferences!");
}

on_intent "recommend_song" {
    let model = context::get("preference_model");
    if model == UNIT {
        dialogue::say("Train the model first by saying 'learn my preferences'");
        return;
    }
    
    // New song characteristics
    let new_song = [[125.0, 70.0, 75.0]];
    let prediction = ml::predict(new_song, model);
    
    if prediction[0] > 0.5 {
        dialogue::say("I think you'll like this song!");
    } else {
        dialogue::say("This song might not be your style");
    }
}
```

---

## Best Practices

### 1. Validate Input Data

```js
// ✅ Good - validate data structure
fn validate_training_data(x, y) {
    if x.len() != y.len() {
        log::error("X and Y must have same length");
        return false;
    }
    
    if x.len() == 0 {
        log::error("Cannot train on empty dataset");
        return false;
    }
    
    return true;
}

if validate_training_data(x_data, y_data) {
    let model = ml::train(x_data, y_data, "linear");
}
```

### 2. Handle Training Errors

```js
// ✅ Good - error handling
try {
    let model = ml::train(x_data, y_data, "linear");
    context::set("model", model);
} catch (e) {
    log::error(`Training failed: ${e}`);
    dialogue::say("Unable to train the model with this data");
}
```

### 3. Normalize Features

```js
// ✅ Good - normalize data for better results
fn normalize(value, min, max) {
    return (value - min) / (max - min);
}

let x_normalized = [
    [normalize(100, 0, 200), normalize(60, 0, 100)],
    [normalize(150, 0, 200), normalize(80, 0, 100)]
];
```

### 4. Validate Predictions

```js
// ✅ Good - check prediction results
let predictions = ml::predict(x_test, model);
foreach (pred in predictions) {
    if pred < 0 || pred > 100 {
        log::warn(`Unusual prediction: ${pred}`);
    }
}
```

---

## Algorithm Selection Guide

| Algorithm    | Use Case                          | Output Type         |
|--------------|-----------------------------------|---------------------|
| `linear`     | Continuous value prediction       | Real numbers        |
| `logistic`   | Binary classification (yes/no)    | 0.0 to 1.0 (probability) |
| `lasso`      | Feature selection, sparse models  | Real numbers        |

---

## See Also

- [Arrays](../language/arrays.md) - Working with training data
- [Context Management](./context.md) - Storing trained models
- [smartcore Documentation](https://smartcorelib.org/) - Underlying ML library