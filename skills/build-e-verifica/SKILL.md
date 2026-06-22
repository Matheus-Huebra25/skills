---
name: build-e-verifica
description: Runs the PasiAPI build and reports the result clearly, then guides smoke-testing changed endpoints.
license: MIT
disable-model-invocation: false
---

# Skill: build-e-verifica

Use this skill to compile the project and validate that recent changes do not break the build. It also prompts for smoke testing when API contracts may have changed.

## When to invoke

- Before committing any change
- After finishing an implementation task
- When the user says "build", "compilar", "verificar", or "checar se funciona"
- Automatically at the end of any skill that modifies C# source files

## Process

### Step 1 — Run the build

```powershell
dotnet build .\PasiAPI.csproj
```

### Step 2 — Interpret results

**If the build succeeds (exit code 0):**
- Report "Build: ✓" with the number of warnings (if any).
- List warnings only if there are more than 0; summarize them in one line each.

**If the build fails:**
- Extract each compiler error: file path, line number, error code, message.
- For each error, identify which recent change caused it and propose the fix.
- Apply fixes and re-run the build. Repeat up to 3 times before asking the user for guidance.

### Step 3 — Decide if smoke testing is needed

Prompt the user to smoke-test manually when any of the following is true:

- A controller action was added, removed, or its parameters changed
- A service query was modified (different filter, different join, different projection)
- A DTO property was added, removed, or renamed
- `PasiDbContext` mappings were modified

If smoke testing is recommended, output the exact `PasiAPI.http` request(s) to run and the expected response shape.

### Step 4 — Update PasiAPI.http if needed

If a new endpoint was added and `PasiAPI.http` was not yet updated, add a sample request entry now.

## Output format

```
Build: ✓ 0 erros, N avisos
  [aviso] {file:line} — {mensagem}

Smoke test recomendado: sim / não
  Rota: {full route}
  Verbo: {GET/POST/...}
  Shape esperada: {brief description}
```
