---
name: debug-query
description: Helps diagnose and fix EF Core query problems in PasiAPI — wrong results, N+1 issues, missing joins, or type mismatches against the SQL Server schema.
license: MIT
disable-model-invocation: false
---

# Skill: debug-query

Use this skill when a service query returns wrong results, throws an exception, or is suspected to be slow due to N+1 or missing index hints.

## When to invoke

- A service method returns empty results when data is expected
- EF Core throws a translation exception ("could not be translated")
- A query is slower than expected
- The user says "a query não está funcionando" or "resultado errado"

## Process

### Step 1 — Identify the query

Ask the user to point to the service file and method, or locate it from context. Read the full method.

### Step 2 — Check common PasiAPI query mistakes

**Missing AsNoTracking**
- All read paths must use `.AsNoTracking()`. If absent, add it.

**Incorrect column name mapping**
- This project uses upper snake case. Check that the entity property maps to the correct column name in `PasiDbContext`. If the column name diverges, the filter or join will silently fail.

**EF-untranslatable calls**
- C# string methods like `.ToLower()` inside a LINQ `.Where()` may not translate. Use the mapped SQL function `FN_REMOVE_CARACTERES_ESPECIAIS` for accent-insensitive lookups (already registered in DbContext).

**Nullable mismatches**
- If a filter compares a `string` to a nullable column (or vice versa), EF may produce `IS NULL` instead of an equality check.

**Missing `.Include()` or projection mismatch**
- If a navigation property is expected but not loaded, the result will be null. Check if `.Include()` is needed, or if the query should be rewritten as a projection to avoid loading full graphs.

**`HasNoKey()` entities**
- These cannot be used in `.Where()` on the server side after `.FromSqlRaw()`. Verify the query structure.

### Step 3 — Log the generated SQL

If the query behavior is unclear, add a temporary log to capture the SQL:

```csharp
// TODO: remove after debugging
var sql = query.ToQueryString();
```

Ask the user to run the app and share the generated SQL.

### Step 4 — Propose and apply fix

Explain what was wrong, apply the smallest fix, then re-run:

```powershell
dotnet build .\PasiAPI.csproj
```

### Step 5 — Clean up

Remove any debugging `.ToQueryString()` calls before the fix is committed.

## Output format

```
Problema identificado: {one-line description}
  Arquivo: {service file:line}
  Causa:   {root cause}
  Fix:     {description of change made}

Build: ✓ / ✗
SQL gerado (se fornecido): {paste or N/A}
```
