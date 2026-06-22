---
name: mapear-contrato
description: Produces a precise API contract summary (routes, verbs, params, response shapes) for one or all PasiAPI endpoint families, useful before integration or documentation work.
license: MIT
disable-model-invocation: true
---

# Skill: mapear-contrato

Run this skill to get a precise, up-to-date summary of PasiAPI's API surface. This is useful before integration work, writing client code, or producing external documentation.

## When to invoke

Invoke manually with `/mapear-contrato`. Optionally pass a family name:
- `/mapear-contrato cotacao`
- `/mapear-contrato convencao-coletiva`
- `/mapear-contrato proposta`
- `/mapear-contrato` (no argument = all families)

## Process

### Step 1 — Read controllers

For each requested family, read the corresponding controller file(s) under `Controllers/`.

### Step 2 — Read DTOs

For each action, read the request DTO (query params, route params, body) and response DTO from `Domain/Models/`.

### Step 3 — Note response shape pattern

Identify which pattern the family uses:
- **cotacao**: `{ Resultado: bool, Mensagem: string, ...data }`
- **convencao-coletiva**: `{ ...data, Mensagens: [{ Descricao: string, Erro: bool }] }`
- **teste-db**: plain HTTP status + anonymous object
- **proposta**: document the actual shape from the DTO

### Step 4 — Produce the contract document

Output a Markdown table per family:

```
## {Family} — api/v1/{family}

| Rota | Verbo | Parâmetros | Resposta | Auth requerida |
|------|-------|------------|----------|----------------|
| /rota | GET | param1 (query, string), param2 (query, bool?) | { ... } | Não (ainda) |
```

After the table, add:
- **Shape de resposta** — full typed shape for each distinct response type
- **Notas** — any known quirks, nullable fields, or TODO items in the current code

### Step 5 — Flag drift

If `PasiAPI.http` is out of date (missing routes or wrong parameters), note which entries need updating.

## Output format

Produce clean Markdown suitable for pasting into a Confluence page or PR description.

End the output with:

```
Gerado em: {today's date}
Fonte: leitura direta dos controllers e DTOs em {commit hash or "working tree"}
```
