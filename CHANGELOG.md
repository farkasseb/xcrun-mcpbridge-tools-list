# Changelog

## Xcode 26.4 RC (17E192)

**Changed tools:** `ExecuteSnippet`, `RunAllTests`, `RunSomeTests`, `XcodeGrep`, `XcodeRead`, `XcodeUpdate`, `XcodeWrite`

No tools were added or removed — the tool count remains at 20.

### Description-only changes: `XcodeGrep`, `XcodeRead`, `XcodeUpdate`, `XcodeWrite`

All four tools received **JSON encoding / escaping guidance** appended to their descriptions. These clarify how backslashes, quotes, and newlines behave in input and output:

- **`XcodeGrep`** — new text: *"Input pattern uses standard regex syntax, not JSON escaping. To find `\d` in source code, use pattern `\\d`. Output results are JSON-encoded e.g. backslashes, quotes, and newlines appear escaped (`\\`, `\"`, `\n`)."*
- **`XcodeRead`** — new text: *"Output is JSON-encoded e.g. backslashes, quotes, and newlines appear escaped (`\\`, `\"`, `\n`). Account for this when interpreting file content."*
- **`XcodeUpdate`** — new text: *"Input oldString and newString use literal characters e.g. if XcodeRead shows `\\d`, use `\d` in parameters to match it."*
- **`XcodeWrite`** — new text: *"Input content uses literal characters e.g. if XcodeRead shows `\\d`, use `\d` in the content parameter to write it."*

### `ExecuteSnippet`

**Input:**
- New required input field `purpose` — a short human-readable description of the purpose of running the code snippet (must not use the word "test")

**Output (`error` object):**
- `message` description changed from *"The error message with potential underlying errors included"* to *"A very short message describing the error"*
- New required output field `summary` — a longer summary of the error and its cause
- New optional output fields: `detailsPath` (path to a file with full error details), `recoveryAdvice` (advice on how to resolve the error)

### `RunAllTests` & `RunSomeTests`

Both tools received the same addition:

- New optional output field `fullConsoleLogsPath` — path to a text file containing all logs that would be printed in the console from `print`, `NSLog`, etc. during test build and execution

## Xcode 26.3 (17C529)

No changes to `tools/list` compared to RC 2 (17C528).

## Xcode 26.3 RC 2 (17C528)

**Changed tools:** `GetTestList`, `RunAllTests`, `RunSomeTests`

All changes are **output-only** — no input schemas were modified. All new fields were added to both `properties` and `required`, making this a **breaking change** for strict decoders written against RC.

### `GetTestList`

- Description expanded to document the 100-test truncation limit, `fullTestListPath`, and grep-friendly format with keys `TEST_TARGET`, `TEST_IDENTIFIER`, `TEST_FILE_PATH`
- New required output fields:
  - `fullTestListPath` — path to a file containing the complete test list
  - `summary` — human-readable summary string
  - `counts` — object with `total`, `enabled`, `disabled` test counts
  - `totalTests` — total count before truncation
  - `truncated` — boolean flag

### `RunAllTests` & `RunSomeTests`

Both tools received the same additions:

- New required output field `fullSummaryPath` — path to a text file with all test results and complete issue details
- Each result item now includes a required `errors` field — array of error message strings
