# CLAUDE.md

This file provides guidance specifically to Claude Code (claude.ai/code) when working with this repository.

For comprehensive project documentation, architecture details, build instructions, and development guidelines, please refer to:

- **AGENTS.md** - Main project documentation for all AI coding assistants
- **Source/Data/AGENTS.md** - Dictionary data-specific documentation for AI coding assistants

These files contain detailed information about:

- Project overview and system requirements
- Building and running the application
- Architecture (Swift, Objective-C++, C++ layers)
- State machine implementation
- Development guidelines and best practices
- Testing procedures
- Dictionary data management

Please consult AGENTS.md for all development tasks.

## Build Notes (Fork-specific)

Command-line build **must** use `-scheme` + `-destination` (not `-target`), otherwise local SPM package dependencies fail:

```bash
xcodebuild -project McBopomofo.xcodeproj \
  -scheme McBopomofoInstaller \
  -configuration Debug \
  -destination 'platform=macOS,arch=arm64' \
  build
```

## State Machine Notes

`InputState` has multiple "empty" states that must all be handled consistently:

- `InputState.Empty` — normal idle state
- `InputState.EmptyIgnoringPreviousState` — after backspace clears last character
- `InputState.Deactivated` — input method deactivated

Any logic that checks "is the input buffer empty?" must account for all three.

## SHIFT Toggle During Active Input

Bare SHIFT press during active input (Inputting, Marking, ChoosingCandidate, etc.) abandons the current Chinese input and switches to alphanumeric mode. Implementation in `InputMethodController.swift` `handle(_:client:)` flagsChanged handler:

1. `keyHandler.clear()` — clears BPMF reading buffer and grid
2. Transition to `EmptyIgnoringPreviousState` — clears composing buffer on screen
3. `isAlphanumericMode = true` — switches to English passthrough

## Input Logging Notes

All committed text is logged via `InputLogger.shared` in `InputMethodController.swift`'s Committing handler.

- Log path: `~/Library/Logs/McBopomofo/input-log-YYYY-MM.tsv`
- Format: `timestamp \t reading \t text` (TSV, UTF-8)
- Reading is extracted from `_latestWalk.nodes` via `_walkReadingString` in `KeyHandler.mm` — must be called **before** `[self clear]` which wipes `_latestWalk`
- Non-Bopomofo commits (alphanumeric, punctuation, space) have empty reading column
- `InputState.Committing` carries both `poppedText` and `reading`; the single-arg `init(poppedText:)` defaults reading to `""`
