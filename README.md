# UHKM Specification — Version 1 (Draft)

> Status: **Draft** (specification is subject to change)

## Table of Contents

- [UHKM Specification — Version 1 (Draft)](#uhkm-specification--version-1-draft)
  - [Table of Contents](#table-of-contents)
  - [What is UHKM?](#what-is-uhkm)
    - [Motivation](#motivation)
  - [Design Philosophy](#design-philosophy)
  - [File Format](#file-format)
    - [Naming and Encoding](#naming-and-encoding)
    - [Formatting](#formatting)
  - [Preamble Pragmas](#preamble-pragmas)
    - [Pragma Reference](#pragma-reference)
  - [Sample Macros](#sample-macros)
    - [LED Text Flash](#led-text-flash)
    - [Repeat Word](#repeat-word)
  - [References](#references)
  - [License](#license)
  - [Change Log](#change-log)

## What is UHKM?

UHKM (Ultimate Hacking Keyboard Macro) is an unofficial, standalone, text-based file format for representing a single UHK Smart Macro independently of [Ultimate Hacking Keyboard](https://ultimatehackingkeyboard.com/) device configuration.

A `.uhkm` file is a plain-text, one-macro-per-file artifact that pairs UHK Smart Macro commands with a metadata preamble. It is portable, versionable, and human-readable.

UHKM does not replace UHK firmware, UHK Agent, or the Smart Macro language itself. It defines macro logic only — trigger bindings and device configuration are external.

### Motivation

At time of writing, UHK macros remain tightly coupled to the UHK device configuration. As macros are shared more widely, a standalone format offers clear benefits: standardized distribution, independent organization, offline validation decoupled from the agent, and broader ecosystem tooling, all while preserving the underlying Smart Macro syntax.

## Design Philosophy

This specification is intentionally lightweight. It defines just enough structure to enable portable, shareable macro files without imposing unnecessary constraints. Where the UHK Smart Macro syntax is permissive, the UHKM spec defers to it.

Each published spec version is tagged in Git (e.g., `v1`, `v2`). Approved specifications are published as GitHub releases. Previous versions remain available as tags.

## File Format

### Naming and Encoding

* Files must use the `.uhkm` extension and UTF-8 encoding (no BOM).
* Kebab-case, lowercase filenames are recommended: `on-init.uhkm`, `doubletap-caps-word.uhkm`.
* Event macros should use the event name without the `$` prefix: `$onInit` → `on-init.uhkm`.
* The filename does not determine macro identity — the `@uhkm-name` pragma is authoritative.
* The first line of the file must be a comment containing the filename (e.g., `// on-init.uhkm`). This allows the filename to be inferred when the macro is viewed outside its original file context.

### Formatting

* **Comments**: `//` to end-of-line, matching UHK Smart Macro syntax.
* **Indentation**: Not prescribed. The UHK parser is indentation-agnostic.
* **Trailing whitespace**: Should be stripped.
* **Block braces**: Use Stroustrup-style braces — opening `{` at the end of a line, closing `}` on its own line, and each subsequent block statement (`else`, `else if`, etc.) on a new line. `} else {` on a single line is not valid. Short-hand conditionals without braces (e.g., `ifShift suppressMods write 4`) are exempt. See [Control Flow](https://github.com/UltimateHackingKeyboard/firmware/blob/master/doc-dev/user-guide.md#control-flow) in the official docs for more on this.

## Preamble Pragmas

The preamble is a block of `// @`-prefixed pragma lines at the top of the file, before any macro content. It carries machine-readable metadata (name, version, constraints) without altering the macro body.

**Syntax:** `// @<key>: <value>` — a single space after `//`, no space between `@`, key, and `:`, a single space before the value.

**Rules:**
* The preamble ends at the first line that is not a `// @` pragma, blank line, or `//` comment.
* Pragma ordering is not significant.
* Duplicate pragma keys are an error.
* Unrecognized pragma keys should produce a warning (for forward compatibility).

`@uhkm-name` and `@uhkm-version` are **required**. All other pragmas are optional. Published `.uhkm` files must include the required pragmas.

### Pragma Reference

| Pragma | Required | Description |
|--------|----------|-------------|
| `@uhkm-name` | Yes | Macro's canonical name as used in UHK Agent (e.g., `$onInit`, `My Macro`). Must be unique within a macro set. |
| `@uhkm-version` | Yes | Author's semver version (`MAJOR.MINOR.PATCH`). Informational only. |
| `@uhkm-description` | No | Single-line summary. UHKM-only metadata — UHK Agent has no description field. |
| `@uhkm-firmware` | No | Firmware version constraint (e.g., `>=16.0.0`, `>=16.0.0 <17.0.0`). Operators: `>=`, `<=`, `>`, `<`, `=`. |
| `@uhkm-author` | No | Author identity in `Name <email>` format where email is optional. |
| `@uhkm-license` | No | SPDX license identifier (e.g., `MIT`, `Apache-2.0`, `Unlicense`). UHKM-only metadata. |
| `@uhkm-os` | No | Target operating system (e.g., `linux`, `macos`, `windows`). Indicates OS-specific behavior such as shortcuts or key mappings. |
| `@uhkm-spec` | No | UHKM spec version (defaults to `1` if omitted). |

**Macro naming details:**

* User-defined names are free-form strings as allowed by UHK Agent.
* Event macro names use the `$` prefix: `$onInit`, `$onKeymapChange`, `$onLayerChange`, `$onKeymapLayerChange`, `$onCapsLockStateChange`, `$onNumLockStateChange`, `$onScrollLockStateChange`, `$onError`, `$onJoin`, `$onSplit`.
* Some event macros support parameterized suffixes (e.g., `$onKeymapChange DOE`). Tooling must accept these forms.

## Sample Macros

### LED Text Flash

A macro that briefly displays "UHK" on the LED display:

```
// led-flash.uhkm
// @uhkm-name: LED Text Flash UHK
// @uhkm-description: Briefly display UHK on the LED display
// @uhkm-license: MIT
// @uhkm-version: 1.0.0

setLedTxt 1000 "UHK"
delayUntil 1000
```

### Repeat Word

A macro that types "uhkm" five times using a loop:

```
// repeat.uhkm
// @uhkm-name: Repeat UHKM
// @uhkm-description: Type the word uhkm five times
// @uhkm-license: MIT
// @uhkm-version: 1.0.0

setVar count 5
write "uhkm "
repeatFor count ($currentAddress - 1)
```

## References

* [UHK Smart Macro Reference — User Guide](https://github.com/UltimateHackingKeyboard/firmware/blob/master/doc-dev/user-guide.md)
* [UHK Smart Macro Reference — Reference Manual](https://github.com/UltimateHackingKeyboard/firmware/blob/master/doc-dev/reference-manual.md)
* [UHK Firmware v16.1.1](https://github.com/UltimateHackingKeyboard/firmware/releases/tag/v16.1.1) — Minimum firmware version for UHKM spec 1.
* [UHK Firmware Releases](https://github.com/UltimateHackingKeyboard/firmware/releases) — Full release history and Smart Macros engine changelog.
* [Ultimate Hacking Keyboard](https://ultimatehackingkeyboard.com/) — Official UHK site.
* [UHK Community Forums](https://forum.ultimatehackingkeyboard.com/) — Community discussion and support.

## License

This specification is released under the MIT License.

## Change Log

* **2026-02-20** — Added `@uhkm-os` pragma.
* **2026-02-18** — Added `@uhkm-license` pragma, block brace formatting rules, and filename requirement.
* **2026-02-17** — Simplified spec structure.
* **2026-02-16** — Initial draft.
