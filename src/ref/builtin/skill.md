# `skill` Namespace

**Namespace:** `skill`  
**Import:** Built-in (globally available)

The `skill` namespace provides access to metadata and configuration information about the current skill. Use these functions to query skill properties, check permissions, and access runtime information.

---

## Functions

### `skill::dir()`

Gets the root directory of the current skill.

#### Signature

```rhai
fn dir() -> String
```

#### Parameters

None

#### Returns

- `String` - The absolute path to the skill's root directory

#### Examples

```js
// Access skill directory
let skill_path = skill::dir();
log::info(`Skill located at: ${skill_path}`);

// Load a resource file from skill directory
let config_path = `${skill::dir()}/config.json`;
let config = file::read(config_path);
```

---

### `skill::get_permissions()`

Gets the permissions required by the current skill.

#### Signature

```rhai
fn get_permissions() -> Array
```

#### Parameters

None

#### Returns

- `Array` - A list of permission strings required by the skill

#### Examples

```js
// Check what permissions are enabled
let perms = skill::get_permissions();
log::info(`This skill requires ${perms.len()} permissions`);

foreach (perm in perms) {
    log::info(`  - ${perm}`);
}
```

---

### `skill::is_disabled()`

Checks if the current skill is disabled.

#### Signature

```rhai
fn is_disabled() -> bool
```

#### Parameters

None

#### Returns

- `bool` - `true` if the skill is disabled, `false` otherwise

#### Examples

```js
// Conditionally execute based on skill state
if skill::is_disabled() {
    log::warn("Skill is disabled, skipping execution");
    return;
}

// Continue normal execution
execute_skill_logic();
```

---

### `skill::manifest()`

Gets the manifest information of the current skill.

#### Signature

```rhai
fn manifest() -> Manifest
```

#### Parameters

None

#### Returns

- `Manifest` - An object containing the skill's manifest data (name, version, author, description, etc.)

#### Examples

```js
// Access manifest data
let manifest = skill::manifest();
log::info(`Running ${manifest.name} v${manifest.version}`);
log::info(`Author: ${manifest.author}`);
```

---

### `skill::version()`

Gets the version of the current skill.

#### Signature

```rhai
fn version() -> String
```

#### Parameters

None

#### Returns

- `String` - The version string of the skill (e.g., "1.2.3")

#### Examples

```js
// Display version information
let version = skill::version();
say(`I'm running version ${version}`);

// Version-based feature gating
if version >= "2.0.0" {
    enable_new_features();
}
```

---

## See Also

- [`constant` namespace](./constant.md) - Access skill-level constants
- [Skill Development Guide](../guides/skills.md) - Building skills for Avi