# Language Profile: C #

## Build Command

```bash
dotnet build --no-restore
```

## Restore Command

```bash
dotnet restore
```

## Test Command

```bash
dotnet test --no-restore --no-build
```

For scoped testing (when affected projects are known):

```bash
dotnet test --no-restore --no-build --filter "FullyQualifiedName~{namespace}"
```

## Static Analysis

```bash
dotnet build --no-restore /p:TreatWarningsAsErrors=true /p:EnforceCodeStyleInBuild=true
```

## Error Parsing Patterns

### Build Errors

Pattern: `^(.+)\((\d+),(\d+)\): error (CS\d{4}): (.+)$`

- Group 1: File path
- Group 2: Line number
- Group 3: Column number
- Group 4: Error code (e.g., CS0246, CS0103)
- Group 5: Error message

### Build Warnings (when treated as errors)

Pattern: `^(.+)\((\d+),(\d+)\): warning (CS\d{4}|CA\d{4}|IDE\d{4}): (.+)$`

### Test Failures

Pattern: `Failed\s+(\S+)\s+\[`

- Group 1: Fully qualified test name

Detailed failure pattern (from `--logger trx` or console output):

```
Failed {TestName} [{Duration}]
  Error Message:
   {message}
  Stack Trace:
   at {location}
```

### Analyzer Diagnostics

Roslyn diagnostic IDs:

- `CS####` — Compiler errors/warnings
- `CA####` — Code analysis (FxCop/NetAnalyzers)
- `IDE####` — IDE code style rules
- `SA####` — StyleCop rules (if enabled)

## Solution/Project Discovery

### Primary Solution

1. Look for `*.sln` files at repo root: `ls *.sln`
2. If multiple found, prefer the one matching the repo name
3. If none at root, search recursively: `find . -name "*.sln" -not -path "./.worktrees/*" -maxdepth 3`

### Project Files

1. Parse `.sln` file for `Project("...")` entries to find all `.csproj` files
2. Fallback: `find . -name "*.csproj" -not -path "./.worktrees/*"`

### Project References

Parse `.csproj` for `<ProjectReference>` elements to understand dependency graph between projects.

## Relevant File Extensions

`.cs`, `.csproj`, `.sln`, `.props`, `.targets`, `.editorconfig`, `.globalconfig`

## Common Error Codes

| Code | Meaning | Typical Fix |
|------|---------|-------------|
| CS0246 | Type or namespace not found | Add using statement, add project reference, or update type name |
| CS0103 | Name does not exist in current context | Add using, qualify name, or inject dependency |
| CS0234 | Namespace does not contain type | Wrong namespace, missing reference |
| CS0535 | Class does not implement interface member | Add missing method implementation |
| CS0012 | Type defined in assembly not referenced | Add NuGet/project reference |
| CS1061 | Type does not contain member | Update method call, check API change |
| CS0029 | Cannot implicitly convert type | Update type, add cast, or change interface |
| CS7036 | No argument for required parameter | Update constructor/method call |
| CS0115 | No suitable method to override | Base class signature changed |

## Clustering Heuristics

When grouping errors into logical breaks:

1. **Same missing type**: All CS0246 errors referencing the same type name → ONE break
2. **Same missing member**: All CS1061 errors for the same member on the same type → ONE break
3. **Same missing reference**: All CS0012 errors for the same assembly → ONE break (add the reference)
4. **Related interface errors**: CS0535 errors for the same interface → ONE break (implement the interface)
5. **Constructor changes**: CS7036 errors from the same constructor → ONE break (update all callers)

## Framework-Specific Notes

### ASP.NET Core

- Check `Program.cs` / `Startup.cs` for DI registrations after extracting interfaces
- Controller dependencies are resolved via DI — extracting a service requires updating DI registrations
- Middleware pipeline may need updates for structural changes

### Entity Framework Core

- DbContext changes can cascade to migrations
- Changing entity types may require migration updates
- Repository pattern extraction should preserve query semantics

### xUnit / NUnit / MSTest

- Test projects need references to the projects they test
- Mock frameworks (Moq, NSubstitute) may need updated interfaces
- Integration tests may use `WebApplicationFactory` which needs working DI
