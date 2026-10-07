# Changelog

## Xcode 27.1 RC (27A9275)

No changes to `tools/list` compared to 27.0 RC (27A266a).

## Xcode 27.0 RC (27A266a)

**Added tools:** `AddEntitlement`, `AddInfoPlist`, `DeviceInteractionEndSession`, `DeviceInteractionInstallAndRun`, `DeviceInteractionStartSession`, `DeviceInteractionStartWorkspaceSession`, `DeviceInteractionSynthesize`, `GetConsoleOutput`, `GetCrashIssueLogs`, `GetFieldPerformanceIssueLogs`, `GetFileCompilerFlags`, `GetTargetBuildSettings`, `GetTopCrashIssues`, `GetTopFieldPerformanceIssues`, `InvokeDebuggerCommand`, `LocalizationPlanner`, `RunProject`, `StopProject`, `StringCatalogContext`, `StringCatalogEdit`, `StringCatalogRead`, `UpdateFileCompilerFlags`, `UpdateTargetBuildSetting`, `XcodeCloseWorkspace`, `XcodeListRunDestinations`, `XcodeListSchemes`, `XcodeListTargets`, `XcodeListTemplates`, `XcodeListTestPlans`, `XcodeListWorkspaces`, `XcodeNewProject`, `XcodeNewTarget`, `XcodeOpenWorkspace`, `XcodeSwitchRunDestination`, `XcodeSwitchScheme`, `XcodeSwitchTestPlan`

**Removed tools:** `XcodeGetCurrentFile`, `XcodeListNavigatorIssues`, `XcodeListWindows`

**Changed tools:** `BuildProject`, `GetBuildLog`, `GetTestList`, `RenderPreview`, `RunAllTests`, `RunCodeSnippet`, `RunSomeTests`, `XcodeGlob`, `XcodeGrep`, `XcodeLS`, `XcodeMV`, `XcodeMakeDir`, `XcodeRM`, `XcodeRead`, `XcodeRefreshCodeIssuesInFile`, `XcodeUpdate`, `XcodeWrite`

The tool count goes from 21 to 54. Compared against 26.6 (17F113) — the 27.0 betas were not tracked. The list is now returned in alphabetical order. `DocumentationSearch` is the only pre-existing tool that is byte-identical.

### `tabIdentifier` → `workspaceIdentifier` (all 17 changed tools)

Every pre-existing tool that took a **required** `tabIdentifier` input now takes an **optional** `workspaceIdentifier` instead — *"Identifies the target workspace directly (used in headless mode): its workspace identifier (e.g. workspace1) or its absolute path."* Omit it to target the current workspace. Every new workspace-bound tool accepts the same field. Identifiers come from `XcodeListWorkspaces` or `XcodeOpenWorkspace`. Breaking for clients that still send `tabIdentifier` if the schema is enforced.

### Removed tools

- `XcodeListWindows` — superseded by `XcodeListWorkspaces` (windows are no longer the addressing unit)
- `XcodeGetCurrentFile` — added in 26.5, gone without a replacement
- `XcodeListNavigatorIssues` — gone; `XcodeRefreshCodeIssuesInFile` remains the only issue tool

### New tools

#### Workspace lifecycle

- `XcodeListWorkspaces` — no input; returns a `message` describing every open workspace with its identifier and path
- `XcodeOpenWorkspace` — required `path`; returns `workspaceIdentifier`, plus optional `workspacePath`, `activeScheme`, `activeRunDestination`
- `XcodeCloseWorkspace` — required `workspaceIdentifier`; only for workspaces opened with `XcodeOpenWorkspace`

#### Schemes, run destinations, test plans

Three list/switch pairs. The list tools identify the active item, cap inline results (100 schemes, 100 test plans, 40 run destinations), and always write the complete list to a grep-friendly file (`fullSchemeListPath`, `fullRunDestinationListPath`, `fullTestPlanListPath`) with a `truncated` flag.

- `XcodeListSchemes` / `XcodeSwitchScheme` — switch by `schemeName` or disambiguated name; the response reports `activeDestinationDisplayTitle` and `activeTestPlanName` because Xcode may adjust both when the previous ones are incompatible
- `XcodeListRunDestinations` / `XcodeSwitchRunDestination` — grouped like the Xcode picker (Devices, Simulators, Build, Incompatible); `includeIncompatible` opts the Incompatible group into the inline list; switch by `displayTitle`
- `XcodeListTestPlans` / `XcodeSwitchTestPlan` — `usesTestPlans` is false with an empty list for schemes not upgraded to test plans; the testing tools operate on the active plan, so switch before running

#### Project structure and build settings

- `XcodeListTargets` — optional `projectPath` and `productTypeFilter`; each entry carries product type and role flags (`isTestTarget`, `supportsHostingTests`, `isAppExtension`, `isAggregate`); Swift package products are not listed
- `XcodeListTemplates` — discover `templateIdentifier` and option keys for the two creation tools; the unfiltered listing is ~200 target templates, so pass `platformFilter`, `categoryFilter`, `nameFilter`, or `kind`
- `XcodeNewProject` — required `templateIdentifier`, `productName`, `destinationPath`; template options go in `options`; returns `projectPath` and `createdTargets`
- `XcodeNewTarget` — required `templateIdentifier`, `productName`; optional `projectPath`, `embedInAppNamed`, `options`; returns the final `targetName` (some templates append a suffix) and `additionalTargetsCreated`
- `GetTargetBuildSettings` / `UpdateTargetBuildSetting` — required `targetName`, optional `projectPath`; update supports `appendValue` and deletion by omitting `buildSettingValue`; the descriptions insist on never reading or editing `project.pbxproj` directly. `UpdateTargetBuildSetting` declares an empty output schema.
- `GetFileCompilerFlags` / `UpdateFileCompilerFlags` — per-file flags from the Compile Sources phase, positioned for incremental `-fbounds-safety` adoption; the update returns `previousFlags` alongside `compilerFlags`
- `AddEntitlement` / `AddInfoPlist` — required `targetName`, key and value type; scalar, array, or dictionary values; both return `result` plus optional `errorDescription`. The descriptions draw the line between the two: privacy usage strings are Info.plist keys, code-signing capabilities are entitlements.

#### Run, debug, console

- `RunProject` — equivalent to Cmd+R; optional `attachDebugger`; returns `runResult`, `buildErrors`, `fullLogPath`, and optional `launchSessionReference` and `processIdentifier`
- `StopProject` — equivalent to Cmd+.; returns `stopResult`
- `InvokeDebuggerCommand` — runs an lldb `command` in the same session as Xcode's debug console; optional `timeout`; returns `output`, `debugSessionActive`, `isWaitingForMore`
- `GetConsoleOutput` — stdout/stderr/OSLog from a launch session, filtered by `outputType` (`stdio` / `oslog` / `all`), regex `pattern`, `oslogSeverity`, `contextLines`, `tailLimit`; returns `units` with `totalCount` and `truncated`

#### Device interaction

- `DeviceInteractionStartSession` — required `deviceIdentifier` and `sessionIdentifier`; boots the device without a workspace, so it cannot build or install
- `DeviceInteractionStartWorkspaceSession` — same, bound to the workspace: only offers devices the active scheme can run on and enables install-and-run
- `DeviceInteractionInstallAndRun` — builds, installs, and starts the app for the session; optional `commandLineArguments`, `environmentVariables`
- `DeviceInteractionSynthesize` — one `interactionCommand` (e.g. `t 100 200` for tap) then captures state; returns `screenshotPath`, `thumbnailScreenshotPath`, `hierarchyPath`, `logsPath`, `applicationState`
- `DeviceInteractionEndSession` — close the `interactionSessionKey`; the descriptions stress that an open session is expensive and affects the user-facing UI

Both start tools return a `skillToTrigger` field — *"The name of the skill that will handle the next step(s)"* — so the server can hand the agent off to a skill for the interaction loop.

#### Field diagnostics

- `GetTopCrashIssues` / `GetCrashIssueLogs` — top crash signatures by device count for the last 14 days from Apple's crash reporting service, then per-signature logs with triage guidance
- `GetTopFieldPerformanceIssues` / `GetFieldPerformanceIssueLogs` — same shape for `launches`, `hangs`, `diskwrites`, `energy`

All four resolve `bundle_id` and `platform` from the active scheme and run destination when omitted, accept `app_version` and `is_beta`, and return `data` as a string with `success` and `message`. Note the snake_case input names, unlike every other tool.

#### Localization

- `LocalizationPlanner` — required `targetLocaleIdentifier`; prepares the project for a new language and returns `nextStep`
- `StringCatalogRead` — keys grouped by translation state for a locale, with `newCount`, `translatedCount`, `needsReviewCount`, `machineTranslatedCount`, and `offset` / `keyLimit` paging
- `StringCatalogContext` — source values, comment, usage locations, similar strings, plural cases, and existing translations for one `stringKey`
- `StringCatalogEdit` — inserts a `translation`, `variationTranslation`, `templateTranslation`, or `stringSetTranslation` for one key and locale

All four descriptions require activating the `xcode-integration:translation-coordinator` or `xcode-integration:translation` skill before calling — the skills ship in Xcode's packaged agent plugin, not in the `tools/list` response.

### `RenderPreview`

**Input:**
- New optional field `previewCanvasControlOverrides` — object with `timelineIndex`, `groupItemIndex`, `toggleState`, driven by the `supportedCanvasControlOverrides` from a previous invocation
- New optional field `previewLocalizationOverride` — a locale identifier from a previous `supportedLocalizations`

**Output:**
- New optional fields `supportedCanvasControlOverrides` (`timelineIndexes`, `groupItems`, `toggleStates`) and `supportedLocalizations`
- New optional fields `displayName`, `sourceLineNumber`, and `renderedDestination` (`platformName`, `deviceModelName`, `systemVersion`)
- `previewSnapshotPath` description now notes that framebuffer areas not visible on the destination device's screen are transparent

### `BuildProject`

- New optional input field `buildForTesting` — also build test targets that a regular build would skip

### `RunAllTests` & `RunSomeTests`

Both tools received the same addition:

- New optional output field `xcresultBundlePath` — path to the `.xcresult` bundle, parseable with `xcresulttool`; `xccov` can extract coverage from it when enabled

### `GetBuildLog`

- `line` in `emittedIssues` items is now optional (was required) — a relaxation, not a break

### `XcodeMV`

- `operation` is a plain string enum (`"move"` / `"copy"`) again, reverting the 26.5 object-with-`rawValue` encoding

### `RunCodeSnippet`

- `title` changed from `RunCodeSnippet` to `Run Code Snippet`; schemas unchanged apart from `workspaceIdentifier`

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
