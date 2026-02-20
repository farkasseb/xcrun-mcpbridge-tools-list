# Changelog

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
