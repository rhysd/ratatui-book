# Migrate from tui-rs

[Ratatui](https://github.com/tui-rs-revival/ratatui) is a fork of
[tui-rs](https://github.com/fdehau/tui-rs/), created to continue maintenance of the project.

Several options are available to migrate apps and libs:

- Use `ratatui` as a drop in replacement aliased as tui
- Replace `ratatui` fully
- Support both `tui` and `ratatui` (useful for libraries)

## Drop in replacement

The simplest approach to migrating to `ratatui` is to use it as drop in replacement for tui and
updating the terminal libraries used (`crossterm` / `termion`). E.g.:

```toml
tui = { package = "ratatui", version = "0.21.0", features = ["crossterm"] }
crossterm = { version = "0.26.1" }
```

Or:

```toml
tui = { package = "ratatui", version = "0.21.0", default-features = false, features = ["termion"] }
termion = { version = "2.0" }
```

## Fully replace Tui with Ratatui

Most new code would instead use the following. To take this approach to migration requires find and
replace `tui::`->`ratatui::` on the entire codebase.

```toml
ratatui = { version = "0.21.0" }
crossterm = { version = "0.26.1" }
```

## Support both tui and ratatui

For more complex scenarios where a library (or in some cases an app) needs to support both ratatui
and maintain existing support for tui, it may be feasible to use feature flags to select which
library to use. See [tui-logger](https://github.com/gin66/tui-logger) for an example of this
approach.

## Backwards compatibility and breaking changes

[You can see the list of breaking changes here](https://github.com/ratatui-org/ratatui/pulls?q=is%3Apr+label%3A%22breaking+change%22+is%3Aclosed)
