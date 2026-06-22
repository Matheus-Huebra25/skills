# Skills

Repository containing skills for the PasiAPI project. Each skill lives under `skills/<name>/SKILL.md` and is picked up automatically by Claude Code.

---

## How skills work

- **Automatic skills** activate when Claude recognizes the task matches the skill's description. You don't need to type anything special.
- **Manual skills** (`disable-model-invocation: true`) only run when you explicitly type `/skill-name` in your prompt.

---

## Skill reference

### `/novo-endpoint` — Criar um novo endpoint
**Ativação:** automática (ou `/novo-endpoint`)

Use when adding a new route to an existing controller or creating a new controller family. The skill asks clarifying questions (route name, domain family, inputs, response shape, auth), then guides the implementation through DTO → service method → controller action → DbContext (if needed) → `PasiAPI.http` update, and finishes with a build check.

**Quando usar:**
- "Quero adicionar um endpoint para listar coberturas"
- "Criar rota POST para gerar proposta"
- "Novo GET em cotacao para buscar produto por código"

---

### `/novo-servico` — Criar um novo serviço
**Ativação:** automática (ou `/novo-servico`)

Scaffolds a new service class with its interface, registers it in `Program.cs`, and wires it to the target controller. Enforces the thin-controller / fat-service pattern and the project's async + `AsNoTracking()` conventions.

**Quando usar:**
- "Preciso de um serviço para relatórios"
- "Criar serviço para integração com um sistema externo"
- "Nova área de domínio que ainda não tem serviço"

---

### `/nova-entidade` — Mapear uma nova entidade EF Core
**Ativação:** automática (ou `/nova-entidade`)

Adds an entity class in `Domain/Entities/` and its `PasiDbContext` mapping. Handles both regular tables (with primary key) and keyless projections for views. Reminds you that this project is database-first — no EF migrations are used.

**Quando usar:**
- "A tabela X ainda não tem entidade"
- "Preciso mapear uma view do banco"
- "Adicionar entidade para junção nova"

---

### `/build-e-verifica` — Compilar e verificar
**Ativação:** automática (ou `/build-e-verifica`)

Runs `dotnet build .\PasiAPI.csproj`, interprets the output, and decides whether a smoke test is also needed based on what changed. If the build fails, it identifies which recent edit caused the error and proposes a fix, retrying up to 3 times before asking for guidance.

**Quando usar:**
- "Build"
- "Compilar"
- Após qualquer implementação antes de commitar

---

### `/debug-query` — Depurar uma query EF Core
**Ativação:** automática (ou `/debug-query`)

Diagnoses EF Core queries that return wrong results, throw translation exceptions, or are suspected to be slow. Checks for missing `AsNoTracking()`, incorrect column name mappings, accent-insensitive filter issues (`FN_REMOVE_CARACTERES_ESPECIAIS`), nullable mismatches, and N+1 patterns.

**Quando usar:**
- "A query está retornando vazio mas tem dado no banco"
- "EF Core está jogando erro de tradução"
- "Resultado da busca está errado"

---

### `/smoke-test` — Testar endpoints em execução
**Ativação:** automática (ou `/smoke-test`)

Walks through manually validating changed or new endpoints against a running instance of the API. Produces the exact `curl` / HTTP request to run and checks that the status code, response shape, and data presence are correct.

**Quando usar:**
- "Quero testar o endpoint que acabei de criar"
- "Verificar se o retorno está correto"
- Após qualquer mudança de contrato de API

---

### `/revisar-pr` — Revisar um PR antes de abrir
**Ativação:** manual — você precisa digitar `/revisar-pr`

Reviews the full `git diff main...HEAD` against PasiAPI's hard rules and PR checklist. Produces a structured report with **Blockers**, **Avisos**, and **Sugestões**, and issues a final APROVADO / REQUER AJUSTES verdict. Will not approve if any blocker is found.

**Quando usar:**
- Antes de abrir um pull request
- Para uma segunda opinião antes de pedir review humano
- `/revisar-pr`

---

### `/mapear-contrato` — Documentar a superfície da API
**Ativação:** manual — você precisa digitar `/mapear-contrato [família]`

Reads the controllers and DTOs directly and produces a precise Markdown contract table (routes, verbs, parameters, response shapes) for one or all endpoint families. Useful before integration work or when writing external documentation.

**Quando usar:**
- Antes de escrever um cliente que consome a API
- Para gerar documentação para outro time
- `/mapear-contrato cotacao`
- `/mapear-contrato` (todas as famílias)
