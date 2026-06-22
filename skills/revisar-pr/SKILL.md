---
name: revisar-pr
description: Reviews staged or committed changes against PasiAPI's architecture rules, conventions, and PR checklist before a pull request is opened.
license: MIT
disable-model-invocation: true
---

# Skill: revisar-pr

Run this skill before opening a pull request on PasiAPI. It checks the diff against the project's hard rules and PR checklist, catches common mistakes, and produces a concise review report.

## When to invoke

Invoke manually with `/revisar-pr`. Do not invoke automatically.

## Process

### Step 1 — Collect the diff

Run:

```powershell
git diff main...HEAD
git status
```

Read the full diff of all changed files.

### Step 2 — Check against hard rules

For each changed file, verify:

**Controllers**
- [ ] No business logic in the controller body (max ~5 lines per action)
- [ ] Route segments are Portuguese and kebab-case
- [ ] HTTP verb and status codes are appropriate
- [ ] Response shape matches the endpoint family's existing pattern
- [ ] `[Authorize]` is not accidentally uncommented without implementing `UseAuthentication()`

**Services**
- [ ] All EF read queries use `AsNoTracking()`
- [ ] No raw SQL strings with user-supplied input (SQL injection risk)
- [ ] Async/await is used throughout; no `.Result` or `.Wait()` calls
- [ ] No domain rules were invented; TODOs were used for unknowns

**Entities / DTOs**
- [ ] Portuguese naming preserved
- [ ] Decimal financial columns are mapped with correct precision
- [ ] No entity is returned directly from an endpoint (DTO is used)
- [ ] Keyless entities use `HasNoKey()`

**DbContext**
- [ ] No EF migrations added
- [ ] New table mappings have explicit `.ToTable()` calls
- [ ] No changes to existing mappings unless the task explicitly requires it

**General**
- [ ] No new NuGet packages added (or user approved them)
- [ ] `PasiAPI.http` updated if routes changed
- [ ] No credentials, connection strings, or secrets committed
- [ ] Build passes (`dotnet build .\PasiAPI.csproj`)

### Step 3 — Summarize findings

Group findings by severity:

- **Blocker** — must fix before merge (logic bug, security issue, broken contract, failing build)
- **Aviso** — should fix but not a blocker (missing `AsNoTracking`, thin-controller violation)
- **Sugestão** — optional improvement

### Step 4 — Report

Output a structured report:

```
## Revisão de PR — PasiAPI

### Arquivos alterados
- {list of files}

### Blockers ({n})
- {file:line} — {description}

### Avisos ({n})
- {file:line} — {description}

### Sugestões ({n})
- {description}

### Build
✓ / ✗

### Checklist de PR
- [x] Rota documentada em PasiAPI.http
- [x] Shape de resposta preservada
- [x] Sem credenciais commitadas
- [ ] ... (items not met)

### Veredicto
APROVADO / REQUER AJUSTES
```

Do not approve if there are any Blockers.
