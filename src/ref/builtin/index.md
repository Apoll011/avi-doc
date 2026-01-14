# Built-in Modules

This section documents the **built-in modules of AviScript**.  
Built-ins are native modules provided by the Avi runtime and are always available to scripts without explicit installation.

They expose core functionality such as data manipulation, I/O, dialogue handling, configuration access, randomness, and runtime context.  
All built-in APIs are implemented in the host environment and executed with full engine privileges, subject to sandbox rules.

Built-in modules are stable, versioned with the Avi runtime, and form the foundation upon which skills and external modules operate.

---

## Module Namespace

Each built-in module is exposed as a **namespace**.  
Functions, constants, and types are accessed using the `namespace::item` syntax.

Example:

```js
import "log" as l;
import "rand";

l::info("Skill started");
let value = rand::int(0, 10);
```