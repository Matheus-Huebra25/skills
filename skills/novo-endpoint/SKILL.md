---
name: novo-endpoint
description: Guides the developer through creating a new API endpoint in PasiAPI, covering controller, service, DTOs, EF mapping, and HTTP file update.
license: MIT
disable-model-invocation: false
---

# Skill: novo-endpoint

Use this skill whenever a new API endpoint needs to be added to PasiAPI. It walks through every required step — from route design through smoke testing — and enforces the project's architectural rules automatically.

## When to invoke

- Adding a brand-new route to an existing controller
- Creating a new controller family
- Implementing a feature that requires a new GET, POST, PATCH, or DELETE endpoint

## Process

### Step 1 — Clarify before touching code

Ask the user for the following if not already provided:

1. **What domain does this endpoint serve?** (cotacao, localidade, convencao-coletiva, proposta, or a new family)
2. **What is the Portuguese route name?** (kebab-case, e.g. `listar-coberturas`)
3. **What does it return?** (a list, a single object, a status message, a generated document?)
4. **What inputs does it accept?** (query params, route params, or request body?)
5. **Does it require authentication?** (for future-proofing; note that auth is not currently enforced)

Do not proceed until points 1–4 are answered.

### Step 2 — Identify affected files

Before writing any code, locate and read:

- The controller for the target family under `Controllers/`
- The service for the target family under `Services/`
- Relevant entities in `Domain/Entities/`
- Relevant response models in `Domain/Models/`
- `Infrastructure/Data/PasiDbContext.cs` — check if new entities need mapping

### Step 3 — Design the contract

Decide:

- Full route: `api/v1/{family}/{route-name}`
- HTTP verb and parameters
- Response shape — **must match the endpoint family's existing pattern**:
  - `cotacao` family → `Resultado` (bool) + `Mensagem` (string) + data payload
  - `convencao-coletiva` family → data collections + `Mensagens` array (each with `Descricao` and `Erro`)
  - `teste-db` family → plain HTTP status codes + anonymous objects
  - New families → propose a shape and confirm with the user before writing

### Step 4 — Implement

Order of edits:

1. **DTO/Model** — add or reuse a file in `Domain/Models/`
2. **Service method** — add logic in the relevant service, using `async`, `AsNoTracking()` for reads
3. **Controller action** — thin method that calls the service and returns `Ok(result)` or the appropriate status
4. **DbContext** — only if a new entity or projection is required; add `HasNoKey()` for read-only projections
5. **`PasiAPI.http`** — append a sample request for the new route

### Step 5 — Constraints to enforce

- Keep controllers thin. If the method body is more than ~5 lines, move logic to the service.
- Use Portuguese names for all route segments and DTO properties.
- Use `AsNoTracking()` on all read-only EF queries.
- Do not add NuGet packages without explicit user approval.
- Do not invent domain rules — use `// TODO: confirm ...` for any unknown business logic.
- Do not change the response shape of other endpoints in the same controller.

### Step 6 — Verify

Run:

```powershell
dotnet build .\PasiAPI.csproj
```

If the build succeeds, prompt the user to smoke-test the new route using Swagger or `PasiAPI.http`.

## Output format

After completing implementation, report:

```
Endpoint adicionado:
  Rota:    {full route}
  Verbo:   {GET/POST/...}
  Arquivo: {controller file:line}
  Serviço: {service file:line}
  DTO:     {model file}

Build: ✓ / ✗
Smoke test: pendente
```

Highlight any TODOs or unresolved domain questions at the end.
