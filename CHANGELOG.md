# Changelog

## Xcode 26.6 (17F113)

**Renamed tools:** `ExecuteSnippet` → `RunCodeSnippet`

**Changed tools:** `DocumentationSearch`

The tool count remains at 21.

### `ExecuteSnippet` → `RunCodeSnippet`

Only `name` and `title` changed — the input and output schemas are byte-identical. Breaking for clients calling the tool by its old name.

### `DocumentationSearch`

- New **required** field `kind` (*"The kind of document"*) in the result `documents` items — breaking change for strict decoders

## Xcode 26.5 (17F42)

**Added tools:** `XcodeGetCurrentFile`

**Changed tools:** `BuildProject`, `ExecuteSnippet`, `RenderPreview`, `RunAllTests`, `RunSomeTests`, `XcodeGlob`, `XcodeGrep`, `XcodeLS`, `XcodeMV`, `XcodeRead`

The tool count goes from 20 to 21.

### New tool: `XcodeGetCurrentFile`

Gets information about the currently active file in the Xcode editor — file path, content, and selection. Content is returned in `cat -n` format with line numbers, up to 600 lines by default with optional `offset`/`limit` for large files.

- **Input:** required `tabIdentifier`; optional `includeContent`, `includeSelection`, `offset`, `limit`
- **Output:** required `isEditable`; optional `filePath`, `content`, `totalLines`, `linesRead`, `startLine`, and a `selection` object with `text`, `lineRange`, `characterRange`

### `RenderPreview`

**Input:**
- New optional field `previewVariantOverrides` — a dictionary mapping variant group names to variant names, using keys/values returned in `supportedPreviewVariantOverrides` by a previous invocation with the same active scheme and run destination

**Output:**
- New optional field `supportedPreviewVariantOverrides` — the supported preview variant overrides, usable as `previewVariantOverrides` in subsequent invocations
- `error` (single object) replaced by `errors` — an array of the same `{message}` objects; the description now also mentions input validation failures

### `BuildProject`

- New **required** output field `fullLogPath` — path of the full log in textual format, containing the complete command lines and any output from the build tasks. Breaking change for strict decoders.

### `RunAllTests` & `RunSomeTests`

Both tools received the same change:

- Required `errors` field in result items renamed to `errorMessages` — breaking change for strict decoders

### `XcodeGrep`, `XcodeGlob` & `XcodeLS`

All three received a new optional output field:

- `packageDependencies` — names of package dependencies whose files are included in results (for `XcodeLS`: items that are package dependencies rather than regular project directories)

`XcodeLS` also gained a new optional output field `message` — *"Optional message about the operation"*.

### `XcodeRead`

- New optional output field `message` — *"Optional message about the operation"*

### `XcodeMV`

- Input field `operation` changed from a string enum (`"move"` / `"copy"`) to an object with a required `rawValue` string property; the `enum` values remain in the schema. Looks like a Swift `RawRepresentable` encoding change — clients sending plain strings may break if the schema is enforced.

### `ExecuteSnippet`

Description-only schema changes, but one documents a behavior change:

- `timeout` default is now documented as **600 seconds** (previously 120)
- `codeSnippet` and `error` wording changed from "executed" to "run"; the `error` description now notes it can be a compile-time or runtime error

## Xcode 26.4.1 (17E202)

No changes to `tools/list` compared to 26.4 RC (17E192).

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
