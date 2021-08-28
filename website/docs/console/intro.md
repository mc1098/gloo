---
sidebar_position: 1
title: Introduction
slug: /console
---

The JavaScript's `console` object provides access to the browser's console.
Using the `console` object in Rust/WASM directly is cumbersome as it requires JavaScript glue code.
This crate exists to solve this problem by providing a set of ergonomic Rust APIs to deal
with the browser console.

## Counter

A `Counter` instance can be created with a label (`str`) and can be used to increment the count
for that label while it remains in scope. Once the `Counter` is dropped it will reset the count.
As the count increments the label and current count is logged to the console.

```rust
use gloo_console::Counter;
// Output to console:
// foo: 1
let counter = Counter::new("foo");
// Output to console:
// foo: 2
counter.count();
// Output to console:
// foo: 0
drop(counter);
// Output to console:
// foo: 1
let counter = Counter::new("foo");
```

See the following links for more information on the console methods that `Counter` abstracts over:
- [`console.count`](https://developer.mozilla.org/en-US/docs/Web/API/console/count)
- [`console.countReset`](https://developer.mozilla.org/en-US/docs/Web/API/console/countReset)

## Macros

Gloo provides macros for console functions, most of which require variadic parameters.

:::note
In macros that support variadic parameters; string substitution works and the formatting
must match what would be expected in JavaScript.
:::

### assert!

The `assert` macro writes an error message to the console if the assertion is false and 
does nothing otherwise. The program will **not** panic if the assertion fails.

:::tip
Use `gloo_console` prefix to avoid conflicts with Rust's `assert` macro.
:::

A passing assertion which does **not** write an error message to the console:
```rust
// No output to console
gloo_console::assert!(true, "This string won't be displayed!");
```

A failing assertion which writes an error message to the console:
```rust
// Output to console:
// `This assertion has failed!`
gloo_console::assert!(false, "This assertion has %s!", "failed");
```

See [`console.assert`](https://developer.mozilla.org/en-US/docs/Web/API/console/assert) for more information on the console method.

### clear! 

The `clear` macro clears the console, if the environment allows it.

```rust
use gloo_console::clear;

// clear the console!
clear!();
```

See [`console.clear`](https://developer.mozilla.org/en-US/docs/Web/API/console/clear) for more information on the console method.

### dir!

The `dir` macro displays an interactive list of the properties of the specified JavaScript object.

```rust
use web_sys::Window;
use gloo_console::dir;

let window = web_sys::window().expect("No global window");
// Output to console:
// > Window
//  [List of Window properties]
dir!(window);
```

See [`console.dir`](https://developer.mozilla.org/en-US/docs/Web/API/console/dir) for more information on the console method.

### dirxml!

The `dirxml` macro displays an interactive tree of elements of the specified XML or HTML element.
When the passed in item cannot be displayed a XML or HTML tree then the JavaScript Object view is 
shown instead.

```html title="index.html"
<!-- .. -->
<body>
    <div id="mydiv">
        <span>Hello, World!</span>
    </div>
</body>
```

Can use `web_sys` window to get the Document and then the body:
```rust
use web_sys::{Document, Window};
use gloo_console::dirxml;

let body = web_sys::window()
    .expect("No global window")
    .document()
    .expect("No document")
    .body()
    .expect("No body");

// Output to console will display interactive tree of html snippet above.
dirxml!(body);
```

See [`console.dirxml`](https://developer.mozilla.org/en-US/docs/Web/API/console/dirxml) for more information on the console method.

### Grouping

The `group` macro creates a new inline group in the console log, this indents following console
messages until the `group_end` macro is called.

```rust
use gloo_console::{group, group_end, log};

// Output to console:
// console
// v `Group 1`
// |____`Welcome to group 1`
// |____v `Group 1.1`
//      |____`Group inside another group`
// > `Group 2`
log!("console");
group!("Group 1");
log!("Welcome to group 1");
group!("Group 1.1");
log!("Group inside another group");
group_end!();
group_end!();
group!(collapsed "Group 2");
log!("This is hidden - unless the user expands group 2");
group_end!();
```

See the following links for more information on the console methods:
- [`console.group`](https://developer.mozilla.org/en-US/docs/Web/API/console/group)
- [`console.groupCollapsed`](https://developer.mozilla.org/en-US/docs/Web/API/console/groupCollapsed)
- [`console.groupEnd`](https://developer.mozilla.org/en-US/docs/Web/API/console/groupEnd)

### Logging 

Gloo provides the following macros for logging to the console:
- `debug!`
- `error!`
- `info!`
- `log!`
- `warn!`

They all log text to the console but do so at the log level of their respective names:

```rust
use gloo_console::{debug, error, info, log};

// Output to console (if configured to display "debug" log level):
// `Did this bit of code run?`
debug!("Did this bit of code run?");

// Output to console (if configured to display "error" log level):
// `oops...`
error!("oops...");

// Output to console (if configured to display "info" log level):
// `Connection established!`
info!("Connection established!");

// Output to console (if configured to display "log" log level):
// `This message came from gloo.`
let name = "gloo";
log!("This message came from %s.", name);

// Output to console (if configured to display "warn" log level):
// `[WS CLOSED] retrying in 2 seconds.`
let seconds = 2u32;
let warn_msg = format!("[WS CLOSED] retrying in {} seconds.", seconds);
warn!(warn_msg);
```

See the following links for more information on the console methods:
- [`console.debug`](https://developer.mozilla.org/en-US/docs/Web/API/console/debug)
- [`console.error`](https://developer.mozilla.org/en-US/docs/Web/API/console/error)
- [`console.info`](https://developer.mozilla.org/en-US/docs/Web/API/console/info)
- [`console.log`](https://developer.mozilla.org/en-US/docs/Web/API/console/log)
- [`console.warn`](https://developer.mozilla.org/en-US/docs/Web/API/console/warn)

### table!

The `table` macro displays tabular data as a table.

This can be useful for displaying imported JavaScript objects:
```js title="JavaScript"
class User {
    constructor(username, email) {
        this.username = username;
        this.email = email;
    }
}
```

```rust
use wasm_bindgen::prelude::*;
use gloo_console::table;

// import the `User` object
#[wasm_bindgen]
extern "C" {
    type User;

    #[wasm_bindgen(constructor)]
    fn new(username: String, email: String) -> User;
}

let user = User::new("gloo".to_owned(), "gloo@email.com".to_owned());

// Output to console:
// | (index)  | Value            |
// | username | `gloo`           |
// | email    | `gloo@email.com` |
// > User
table!(user);
```

See [`console.table`](https://developer.mozilla.org/en-US/docs/Web/API/console/table) for more information on the console method.

### trace!

The `trace` macro outputs a stack trace to the console.

The stack trace can be quite verbose and will include stack traces for wasm_bindgen too.

```rust
use gloo_console::trace;

// Output to console:
// imports.wbg.__wbg_trace_a195a001805aea84
// $gloo_console::externs::trace::h2fb98723c1b2128c
// .. etc
trace!("Stack trace:");
```

See [`console.trace`](https://developer.mozilla.org/en-US/docs/Web/API/console/trace) for more information on the console method.

## Timer

The `Timer` wraps both the `time` and `timeEnd` calls into a single type, ensuring both are called.

### Scoped measurement

Wrap code to be measured in a closure with [`Timer::scope`].

```rust
use gloo_console::Timer;

let value = Timer::scope("foo", || {
    // Place code to be measured here
    // Optionally return a value.
});
// Output to console:
// answer time: [time taken]ms
```

### RAII-style measurement

For scenarios where [`Timer::scope`] can't be used, like with
asynchronous operations, you can use `Timer::new` to create a timer.
The measurement ends when the timer object goes out of scope.

```rust
use gloo_console::Timer;
use gloo_timers::callback::Timeout;

// Start timing a new operation.
let timer = Timer::new("foo");

// And then asynchronously finish timing.
let timeout = Timeout::new(1_000, move || {
    drop(timer);
});
```

See the following links for more information on the console methods that `Timer` abstracts over:
- [`console.time`](https://developer.mozilla.org/en-US/docs/Web/API/console/time)
- [`console.timeEnd`](https://developer.mozilla.org/en-US/docs/Web/API/console/timeEnd)
