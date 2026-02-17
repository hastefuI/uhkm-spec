# UHKM Specification — Version 1 (Draft)

> Status: **Draft** (specification is subject to change)

## Table of Contents

- [UHKM Specification — Version 1 (Draft)](#uhkm-specification--version-1-draft)
  - [Table of Contents](#table-of-contents)
  - [What is UHKM?](#what-is-uhkm)
  - [Motivation](#motivation)
  - [Design Philosophy](#design-philosophy)
    - [Spec Versioning](#spec-versioning)
  - [Goals](#goals)
  - [Proposed Artifact Types](#proposed-artifact-types)
  - [Macro Syntax Architecture](#macro-syntax-architecture)
    - [Scope of Syntax Proposal](#scope-of-syntax-proposal)
    - [Separation of Concerns](#separation-of-concerns)
  - [File Format](#file-format)
    - [File Naming](#file-naming)
    - [Encoding](#encoding)
    - [Comments](#comments)
    - [Deterministic Formatting](#deterministic-formatting)
  - [Preamble Pragmas](#preamble-pragmas)
    - [`@uhkm-author`](#uhkm-author)
    - [`@uhkm-description`](#uhkm-description)
    - [`@uhkm-firmware`](#uhkm-firmware)
    - [`@uhkm-name`](#uhkm-name)
    - [`@uhkm-spec`](#uhkm-spec)
    - [`@uhkm-version`](#uhkm-version)
  - [Sample Macros](#sample-macros)
    - [LED Text Flash](#led-text-flash)
    - [Repeat Word](#repeat-word)
  - [References](#references)
  - [License](#license)
  - [Change Log](#change-log)

## What is UHKM?

**UHKM (Ultimate Hacking Keyboard Macro)** is a proposal for a standalone, text-based artifact format for representing a single UHK Smart Macro outside of the standard UHK configuration.

UHKM is:

* A one-macro-per-file format (`.uhkm`).
* A plain-text representation of UHK's native Smart Macro syntax, the scripting language used by the firmware to define macro behavior.
* Independent from UHK Agent as a required runtime.
* Designed to be portable, versionable, and human-readable.

UHKM is not:

* A replacement for UHK firmware.
* A replacement for UHK Agent.
* A new macro language.
* A configuration bundle containing layers, lighting, or device settings.

## Motivation

UHK Smart Macros are a powerful and flexible feature of the Ultimate Hacking Keyboard. Today, macros are defined and managed within UHK Agent as part of full device configuration exports. This approach works well for configuring a device as a whole, but it also means that macros are primarily treated as components of larger configuration files rather than as standalone artifacts.

As macros are reused and shared more broadly, structural improvements become valuable:
* **Standardized sharing.** Providing a consistent structure for exchanging macros beyond informal code snippets.
* **Standalone organization.** Saving and organizing individual macros independently from full device configurations.
* **Offline inspection and validation.** Enabling parsing, linting, and inspection workflows that do not require importing into UHK Agent.
* **Ecosystem growth.** Establishing a stable file format that community tooling can build around while remaining fully compatible with the existing Smart Macro syntax and firmware.

UHKM aims to complement the existing UHK workflow by defining a minimal, portable macro file format that gives each macro a standalone identity, without changing the underlying Smart Macro syntax or firmware behavior.

## Design Philosophy

This specification is intentionally lightweight. It defines just enough structure to enable portable, shareable macro files without imposing unnecessary constraints on authors or tooling. Where the UHK Smart Macro syntax is permissive, the UHKM spec defers to it.

### Spec Versioning

Each published specification version is tagged in Git (e.g., `v1`, `v2`). Previous versions remain available as tags so that tooling authors can always reference or target a specific spec revision. Approved specifications (i.e., those that have moved past draft status) are published as GitHub releases.

## Goals

* Provide a single-macro-per-file format (e.g., `*.uhkm`) that is version-controllable, diff-friendly, and suitable for easy sharing and publishing of standalone macros.
* Enable deterministic linting and validation of macro syntax.
* Enable safe transformation into/out of UHK configuration exports.
* Ensure the format supports fully offline workflows: `.uhkm` files must be buildable, testable, and validatable entirely offline, on a machine the user controls. There must be no requirement to use UHK Agent in order to parse, lint, inspect, or test a macro.

## Proposed Artifact Types

* **Macro + Metadata**: `*.uhkm` contains UHK Smart Macro commands plus metadata preamble. The preamble consists of `//`-prefixed pragma lines (e.g., `// @uhkm-firmware`, `// @uhkm-version`, `// @uhkm-name`, `// @uhkm-description`, `// @uhkm-author`) that appear before any macro content. The contents of the macro must be valid if pasted directly into the UHK Agent's command action box.

## Macro Syntax Architecture

This proposal focuses exclusively on the macro artifact format and syntax. Tooling (CLI, editors, integrations) is intentionally out of scope.

### Scope of Syntax Proposal

* Define what constitutes a valid `.uhkm` file.
* Define formatting and structural conventions.
* Define comment usage rules.
* Define determinism and normalization expectations.

### Separation of Concerns

* `.uhkm` defines macro logic only.
* Trigger bindings (keymaps, layer positions) are external.
* Device configuration (modules, lighting, etc.) is external.

## File Format

### File Naming

* Files must use the `.uhkm` extension.
* Kebab-case filenames are recommended: `doubletap-caps-word.uhkm`, `on-init.uhkm`.
* Filenames should be lowercase and use only `a-z`, `0-9`, and `-` (hyphens). Avoid spaces, underscores, and special characters for maximum cross-platform portability.
* Event macros should use the event name without the `$` prefix: `$onInit` → `on-init.uhkm`, `$onKeymapChange` → `on-keymap-change.uhkm`.
* The filename does not determine the macro's identity — the `@uhkm-name` pragma is authoritative. Filenames are a convention for human readability and filesystem organization only.
* Including the filename as a comment on the first line (e.g., `// led-flash.uhkm`) is recommended. This allows the filename to be inferred when the macro is viewed outside its original file context, such as in a paste, snippet, or embed.

### Encoding

* `.uhkm` files use UTF-8 encoding.
* Since UHK Smart Macros support arbitrary Unicode input (including emoji and other non-ASCII characters), UTF-8 ensures these characters can be represented and round-tripped without loss.
* Files must not include a Byte Order Mark (BOM).

### Comments

* Line comments start with `//` and continue to end-of-line, matching the UHK Smart Macro native comment syntax.

### Deterministic Formatting

* **Indentation**: Not prescribed. Spaces and tabs are both valid; the UHK macro parser is indentation-agnostic. Authors may use whatever style they prefer.
* **Trailing whitespace**: Should be stripped. Lines must not end with trailing spaces or tabs.
* **Block braces**: `{ }` must be followed by newlines, per UHK Smart Macro rules. A formatter must not reflow brace-delimited blocks onto a single line.

## Preamble Pragmas

The metadata preamble is a block of pragma lines at the top of the file, before any macro content.

Its intent is to carry shareable, machine-readable context (name, version, constraints) without altering the macro body so tools can validate, index, and exchange macros deterministically.

**Pragma syntax:** Each pragma line must match the form:

```
// @<key>: <value>
```

Where `<key>` is a recognized pragma name (e.g., `uhkm-name`), followed by a colon, a single space, and the value.

A single space is required between `//` and `@`, and between `:` and the value. The `@`, key, and `:` must have no intervening whitespace.

**Preamble boundary:** The preamble ends at the first line that is neither a `// @` pragma, a blank line, nor a regular `//` comment. Blank lines and `//` comments are permitted within the preamble for readability and are skipped by tooling. Once any other line is encountered, all subsequent lines are treated as macro body.

**Pragma ordering:** The order of pragma lines is not significant. Tooling must not depend on or enforce a specific ordering.

**Duplicate pragmas:** A file must not contain more than one instance of any pragma key. Tooling must produce an error if a duplicate is encountered.

**Unknown pragmas:** Tooling must ignore unrecognized `// @` pragma keys with a warning. This ensures forward compatibility when new pragmas are introduced in future spec versions.

All pragmas are optional except for `@uhkm-name` and `@uhkm-version`, which are required

Published `.uhkm` files (shared publicly on the internet, submitted to a registry, or distributed as standalone artifacts) must include these pragmas, and tooling that publishes or indexes macros should reject files that are missing them.

### `@uhkm-author`

* Identifies the macro author: `// @uhkm-author: Jane Doe <jane@example.com>`
* Must follow the format `Name <email>` (e.g., `Jane Doe <jane@example.com>`).
* Has no effect on macro behavior or validation.

### `@uhkm-description`

* A free-text, single-line summary of the macro's purpose: `// @uhkm-description: Initialize trackball settings and backlight defaults`
* Intended for display by tooling, registries, or directory listings.
* This pragma is UHKM-only metadata. UHK Agent does not have a description field for macros, so this value is not preserved when importing into or exporting from Agent configurations.
* Has no effect on macro behavior or validation.

### `@uhkm-firmware`

* Declares a UHK firmware version constraint for the macro, using the firmware's semver version number (e.g., `16.1.1`).
* The value is a version constraint consisting of an operator and a version, or two constraints for a range:
  * `// @uhkm-firmware: >=16.0.0` — Minimum version. The macro requires at least this firmware version.
  * `// @uhkm-firmware: =16.1.1` — Pinned version. The macro is designed for exactly this firmware version.
  * `// @uhkm-firmware: >=16.0.0 <17.0.0` — Range. The macro requires a firmware version within this range.
* Supported operators: `>=`, `<=`, `>`, `<`, `=`.
* If omitted, no firmware version constraint is assumed. Tooling should not warn or error on the absence of this pragma.
* Tooling that knows the target firmware version should warn when the constraint is not satisfied.

### `@uhkm-name`

* Files declare the macro's canonical name with: `// @uhkm-name: $onInit`
* The name must match the macro name used in the UHK configuration (e.g., `$onInit`, `My Macro`).
* **Valid names:** User-defined macro names are free-form strings as allowed by UHK Agent (e.g., `My Macro`, `ledFlashUhk`). Event macro names are prefixed with `$` (e.g., `$onInit`, `$onKeymapChange`). Tooling should accept any non-empty name that UHK Agent would accept.
* Event macros (`$onInit`, `$onKeymapChange`, `$onLayerChange`, `$onKeymapLayerChange`, `$onCapsLockStateChange`, `$onNumLockStateChange`, `$onScrollLockStateChange`, `$onError`, `$onJoin`, `$onSplit`) must use the exact event name expected by the firmware.
* **Parameterized event macro names:**
  * Some event macros support an additional selector/suffix after the event identifier, separated by whitespace (e.g., `$onKeymapChange DOE`, `$onKeymapChange any`). UHKM tooling must allow these parameterized forms and must not reject a valid event macro name solely because it includes a suffix parameter.
  * Tooling may validate selector syntax when the parameter rules are known, but should prefer warnings over hard errors to avoid becoming stricter than the firmware.
* Macro names must be unique within a given macro set. The UHK firmware does not allow multiple macros with the same name, and tooling should error if two `.uhkm` files declare the same `@uhkm-name`.
* **This pragma is required.**

### `@uhkm-spec`

* Declares the UHKM specification version that this file conforms to.
* Files declare their spec version with: `// @uhkm-spec: 1`
* Tooling must error if it encounters a file declaring a higher spec version than it supports.
* If omitted, spec version `1` is assumed for backward compatibility in UHKM v1.

### `@uhkm-version`

* The macro author's own version number for the macro, using Semantic Versioning (`MAJOR.MINOR.PATCH`).
* Files declare their macro version with: `// @uhkm-version: 2.1.0`
* This is the version that users, registries, and sharing tools should display and compare when determining whether a macro has been updated.
* Has no effect on parser compatibility or validation. Tooling must not reject a file based on this value.
* **This pragma is required.**

## Sample Macros

Sample macros that follow the UHKM specification.

### LED Text Flash

A macro that briefly displays "UHK" on the LED display:

```
// led-flash.uhkm
// @uhkm-name: LED Text Flash UHK
// @uhkm-description: Briefly display UHK on the LED display
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

---

## Change Log

* **2026-02-16** — Initial draft.
