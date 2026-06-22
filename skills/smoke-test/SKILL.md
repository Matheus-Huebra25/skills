---
name: smoke-test
description: Starts PasiAPI locally and runs a smoke test against changed or specified endpoints, reporting whether the response shapes and status codes are correct.
license: MIT
disable-model-invocation: false
---

# Skill: smoke-test

Use this skill to manually validate that the running API returns the expected shapes and status codes for recently changed or newly created endpoints.

## When to invoke

- After implementing a new endpoint
- After modifying a service query
- When the user says "testar", "smoke test", or "verificar endpoint"

## Process

### Step 1 — Confirm environment

Check if the app is already running. If not, ask the user to start it:

```powershell
$env:ASPNETCORE_ENVIRONMENT = "Development"
dotnet run --project .\PasiAPI.csproj --launch-profile http
```

Default base URL: `http://localhost:5208`

Remind the user that a valid connection string must be set in the environment or `appsettings.Development.json`.

### Step 2 — Identify endpoints to test

If specific routes were not provided, derive them from:
- The most recently changed controller (from `git diff`)
- The endpoints listed in `PasiAPI.http`

### Step 3 — Run requests

For each endpoint, use the sample from `PasiAPI.http` or construct a minimal request:

- Provide the exact `curl` command or HTTP snippet the user should run.
- If authentication is needed, note that token acquisition via `POST api/v1/token` is required first.

### Step 4 — Validate responses

For each response, check:

| Check | Expected |
|-------|----------|
| HTTP status | 200 OK (or documented error code) |
| Response shape | Matches the endpoint family's pattern (Resultado/Mensagem, Mensagens, etc.) |
| Data presence | Non-empty result when valid input is given |
| Portuguese field names | All DTO properties are Portuguese |
| No stack traces or raw SQL | Response body contains no exception details |

### Step 5 — Report

```
Smoke Test — {endpoint family}

GET api/v1/{route}?{params}
  Status: 200 ✓ / {actual status} ✗
  Shape:  ✓ / ✗ — {description of mismatch if any}
  Data:   presente / vazio (inesperado)

Resultado geral: PASSOU / FALHOU
```

If any check fails, identify the probable cause and propose a fix (or point to the relevant service method).

### Notes

- Do not expose or log the actual database credentials.
- Do not commit any temporary debug changes introduced during smoke testing.
- If `teste-db` endpoints are used for validation, note they are diagnostics only and not stable.
