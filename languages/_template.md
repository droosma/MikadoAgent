# Language Profile Template: {Language Name}

> Copy this file to `languages/{lang}.md` and fill in all sections.
> The Mikado Agent reads this profile at session start to know how to build, test, and parse errors for your language/framework.

## Build Command

```bash
# Command to build the project. Should fail fast on errors.
# Example: dotnet build --no-restore
# Example: mvn compile
# Example: npm run build
```

## Restore Command

```bash
# Command to restore dependencies. Required for fresh worktrees.
# Example: dotnet restore
# Example: npm install
# Example: mvn dependency:resolve
```

## Test Command

```bash
# Command to run tests. Should return non-zero exit code on failure.
# Example: dotnet test --no-restore --no-build
# Example: mvn test
# Example: npm test
```

## Static Analysis

```bash
# Command to run linters/analyzers. Optional but recommended.
# Example: dotnet build /p:TreatWarningsAsErrors=true
# Example: eslint src/
# Example: mvn checkstyle:check
```

## Error Parsing Patterns

### Build Errors

```
# Regex pattern to extract file, line, column, error code, and message
# Pattern: ^(.+)\((\d+),(\d+)\): error (\w+\d+): (.+)$
# Groups: file, line, col, code, message
```

### Test Failures

```
# Regex pattern to extract failed test names
# Pattern: FAILED\s+(\S+)
# Groups: test name
```

### Analyzer Diagnostics

```
# Regex pattern for linter/analyzer warnings
# List diagnostic ID prefixes used by this language's tools
```

## Solution/Project Discovery

### Primary Project File

```
# How to find the main project/workspace file
# Example: *.sln at repo root
# Example: package.json at repo root
# Example: pom.xml at repo root
```

### Sub-Projects

```
# How to discover sub-projects/modules
# Example: parse .sln for Project entries
# Example: workspaces field in package.json
# Example: <modules> in pom.xml
```

## Relevant File Extensions

```
# List all file extensions relevant to this language
# Example: .cs, .csproj, .sln
# Example: .ts, .tsx, .js, .jsx, .json
# Example: .java, .xml, .gradle
```

## Common Error Codes

| Code | Meaning | Typical Fix |
|------|---------|-------------|
| | | |

## Clustering Heuristics

```
# Language-specific rules for grouping errors by root cause
# Example: All errors referencing the same missing type → ONE break
# Example: All import errors for the same module → ONE break
```

## Framework-Specific Notes

```
# Notes about popular frameworks in this language that affect refactoring
# Example: DI container registration updates
# Example: ORM migration impacts
# Example: Test framework specifics
```
