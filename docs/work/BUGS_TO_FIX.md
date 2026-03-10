# Bugs Found During Coverage Work

Discovered while writing unit tests (commit db33956). Tests were adjusted
to match actual behavior -- source code was not modified.

## 1. decoratorFunctionName does not trim whitespace before @

**File:** `internal/pipeline/decorates.go`
**Symptom:** `decoratorFunctionName("  @spaced  ")` returns `"@spaced"` instead of `"spaced"`.
The function strips the `@` prefix but does not trim leading/trailing whitespace first.
**Severity:** Low -- decorators with surrounding whitespace are unlikely in parsed AST output.

## 2. ProjectNameFromPath returns "." for empty string

**File:** `internal/pipeline/pipeline.go`
**Symptom:** `ProjectNameFromPath("")` returns `"."` (from `filepath.Base("")`).
No guard for empty input.
**Severity:** Low -- empty path is an edge case that should not occur in normal usage.

## 3. resolveFileConfiguresCBM deduplicates by funcQN+targetModule, not by env key

**File:** `internal/pipeline/pipeline_cbm.go`
**Symptom:** Two different env vars (`DB_URL`, `API_KEY`) accessed from the same function
targeting the same config module produce only 1 CONFIGURES edge instead of 2.
Dedup key is `funcQN + targetModuleQN`, so all env vars from the same function
to the same module collapse into one edge.
**Severity:** Medium -- loses granularity in the graph. A function reading 5 env vars
from the same `.env` file gets one edge instead of 5.

## 4. isDependencyChild never matches camelCase npm sections

**File:** `internal/pipeline/configlink_strategies.go`
**Symptom:** `isDependencyChild` lowercases QN parts before looking up in `depSectionNames`,
but the map contains camelCase keys (`"devDependencies"`, `"peerDependencies"`).
`strings.ToLower("devDependencies")` = `"devdependencies"` which never matches
the key `"devDependencies"`.
**Effect:** npm `devDependencies`, `peerDependencies`, `optionalDependencies` etc.
are never recognized as dependency sections. Only lowercase keys like
`"dependencies"` (which survives lowercasing) work.
**Severity:** High -- breaks dependency linking for all camelCase package.json sections
across JavaScript/TypeScript projects.
