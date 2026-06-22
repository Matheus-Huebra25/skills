---
name: nova-entidade
description: Adds a new EF Core entity (or keyless projection) to PasiAPI, including the DbContext mapping and any DTO needed, following the database-first, no-migration conventions.
license: MIT
disable-model-invocation: false
---

# Skill: nova-entidade

Use this skill when a new database table or view needs to be represented in the codebase. This project is **database-first with no EF migrations** — the schema already exists; we only map it.

## When to invoke

- A service query needs to join a table that has no entity class yet
- A SQL view needs a keyless projection class
- The user says "preciso mapear a tabela X" or "adicionar entidade Y"

## Process

### Step 1 — Gather information

Ask the user for the following if not provided:

1. **Table or view name** (exact name as in SQL Server, including schema if not `dbo`)
2. **Is it a table (has a primary key) or a view / keyless projection?**
3. **Which columns need to be mapped?** (name, SQL type, nullable?)
4. **Will it be used in a navigation property on an existing entity?**

Do not guess column names or types — wait for the user to confirm them.

### Step 2 — Create the entity class

Location: `Domain/Entities/{EntityName}.cs`

Rules:
- Use Portuguese class and property names that match existing conventions in the domain.
- Use C# value types with `?` suffix for nullable columns.
- Use `decimal` (not `float`/`double`) for monetary columns.
- Do not add navigation properties unless the user explicitly requests them.

### Step 3 — Register in PasiDbContext

Location: `Infrastructure/Data/PasiDbContext.cs`

Rules:
- Add a `DbSet<EntityName>` property.
- In `OnModelCreating`, call `modelBuilder.Entity<EntityName>(entity => { ... })`.
- Map the table name explicitly with `.ToTable("NOME_TABELA")`.
- For keyless projections (views), call `.HasNoKey()` and `.ToView("NOME_VIEW")` instead.
- Map decimal financial columns with `.HasPrecision(18, 2)` unless the user specifies otherwise.
- Do not use EF conventions for column naming — this project applies upper snake case mapping; verify the convention is applied or add explicit `.HasColumnName("COL_NAME")` if needed.

### Step 4 — Add a DTO if needed

If the entity will be returned directly in an API response:
- Create a DTO in `Domain/Models/` instead of exposing the entity class.
- Name it `{EntityName}Response.cs` or `{EntityName}Payload.cs` following the existing naming pattern.

### Step 5 — Verify

```powershell
dotnet build .\PasiAPI.csproj
```

Report build result. If any mapping error occurs (e.g., missing column, type mismatch), surface it explicitly.

## Output format

```
Entidade criada: {EntityName}
  Arquivo:  Domain/Entities/{EntityName}.cs
  DbSet:    Infrastructure/Data/PasiDbContext.cs
  Tipo:     Tabela com chave / Projeção sem chave (HasNoKey)
  DTO:      Domain/Models/{...}.cs (se criado)

Build: ✓ / ✗
TODOs: {list any unresolved column types or mapping questions}
```
