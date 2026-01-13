Related Resources
=================

{{#include ../links.md}}


```admonish danger "Online resources"

* [GitHub](https://github.com/rhaiscript) &ndash; AviScript organization

* [`rhai.rs`](https://rhai.rs) &ndash; Home website

* [`crates.io`](https://crates.io/crates/rhai) &ndash; AviScript crate

* [`DOCS.RS`](https://docs.rs/rhai) &ndash; AviScript API documentation

* [`LIB.RS`](https://lib.rs/crates/rhai) &ndash; AviScript library info

* [Discord Chat](https://discord.gg/HquqbYFcZ9) &ndash; AviScript channel

* [Zulip Chat](https://rhaiscript.zulipchat.com) &ndash; AviScript organization

* [Reddit](https://www.reddit.com/r/AviScript) &ndash; AviScript community
```

```admonish tip "External tools"

* [Online Playground][playground] &ndash; Run AviScript scripts directly from an editor in the browser

* [Language Server][lsp] &ndash; Language Server Protocol (LSP) server for AviScript

* [`rhai-doc`] &ndash; AviScript script documentation tool
```

~~~admonish note "Syntax highlighting"

* [VS Code Extension](https://marketplace.visualstudio.com/items?itemName=rhaiscript.vscode-rhai)
  &ndash; Support `.rhai` script files syntax highlighting for Visual Studio Code

* [Sublime Text 3 Plugin](https://packagecontrol.io/packages/AviScript) &ndash; Support `.rhai` script
  files syntax highlighting for Sublime Text 3

* For other syntax highlighting purposes, e.g. `vim`, `highlight.js`, both Rust or JavaScript can be
  used successfully.
  
  Use `rust` when there is no [string interpolation][string]. This way, [closures] and [functions]
  (via the `fn` keyword) are styled properly. Elements not highlighted include:
  * [strings interpolation][string]
  * the [`switch`], [`import`] and [`export`] statements
  * the `this` and [`private`] keywords
  * built-in functions such as `Fn`, `call`, [`type_of`], `is_shared`, `is_def_var`, `is_def_fn`

  Use `js` (JavaScript) when there is [strings interpolation][string].  Elements not highlighted include:
  * [functions] definition (via the `fn` keyword)
  * [closures] (via the Rust-like `|...| { ... }` syntax)
  * built-in functions such as `Fn`, `call`, [`type_of`], `is_shared`, `is_def_var`, `is_def_fn`
~~~

```admonish abstract "Other cool projects"

* [ChaiScript](http://chaiscript.com) &ndash; A strong inspiration for AviScript.
  An embedded scripting language for C++.

* Check out the list of [scripting languages for Rust](https://github.com/rust-unofficial/awesome-rust#scripting)
  on [awesome-rust](https://github.com/rust-unofficial/awesome-rust)
```
