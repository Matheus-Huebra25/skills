---
name: novo-servico
description: Scaffolds a new service class for PasiAPI, wires it into DI in Program.cs, and ensures it follows the thin-controller/fat-service pattern.
license: MIT
disable-model-invocation: false
---

# Skill: novo-servico

Use this skill to create a new service from scratch. Services in PasiAPI own all multi-step query logic and business rules, keeping controllers thin.

## When to invoke

- A new domain area needs its own service (e.g., a new product line or integration)
- An existing controller is getting a new endpoint family that doesn't fit an existing service
- The user says "criar serviço" or "novo serviço"

## Process

### Step 1 — Clarify scope

Ask the user:

1. **What is the name of the new service?** (e.g., `CoberturService`, `RelatorioService`)
2. **Which controller will use it?** (existing or new?)
3. **What entities/tables will it query?**
4. **Should it have an interface?** (default: yes — follow `ICotacaoService` / `CotacaoService` pattern)

### Step 2 — Create the interface

Location: `Services/{Domain}/I{ServiceName}.cs`

- Declare only the methods needed right now. Do not add speculative methods.
- Use Portuguese method names only if the method maps directly to a business term; otherwise English is acceptable.
- All methods must be `Task<T>` or `Task`.

### Step 3 — Create the implementation

Location: `Services/{Domain}/{ServiceName}.cs`

Rules:
- Constructor-inject `PasiDbContext` (and any other approved dependencies).
- All EF read operations: `AsNoTracking()`.
- All methods: `async` with `await`.
- No raw SQL with user input concatenation.
- Use `// TODO: confirm ...` for any domain rule that cannot be confirmed from code.

### Step 4 — Register in DI

Location: `Program.cs`

Add:

```csharp
builder.Services.AddScoped<I{ServiceName}, {ServiceName}>();
```

Place it near the existing service registrations for consistency.

### Step 5 — Wire to controller

If a controller needs to be updated:
- Add a constructor parameter for `I{ServiceName}`.
- Do not add logic to the controller body beyond calling the service and returning the result.

### Step 6 — Verify

```powershell
dotnet build .\PasiAPI.csproj
```

## Output format

```
Serviço criado: {ServiceName}
  Interface: Services/{Domain}/I{ServiceName}.cs
  Implementação: Services/{Domain}/{ServiceName}.cs
  DI registrado em: Program.cs
  Controller atualizado: {file or "nenhum"}

Build: ✓ / ✗
TODOs: {list}
```
