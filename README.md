[![MSeeP.ai Security Assessment Badge](https://mseep.net/pr/cicatriiz-cdisc-library-mcp-server-badge.png)](https://mseep.ai/app/cicatriiz-cdisc-library-mcp-server)

## CDISC Library MCP Server (DXT)

This MCP server provides a compact, task‑oriented wrapper around the CDISC Library. It covers discovery (suggest + search), Biomedical Concepts (latest + versioned packages), and SDTM Dataset Specializations (latest + packages). Use it to search, fetch a concept or specialization with one call, then follow `_links` for deeper context. For reproducible results tied to an effective date, use package routes; for exploration, use latest routes.

### About CDISC and the CDISC Library
- **CDISC**: A global, non‑profit that develops data standards to streamline clinical research and enable interoperability. See the [CDISC website](https://www.cdisc.org).
- **CDISC Library**: Programmatic access to CDISC standards and artifacts (e.g., SDTM, CDASH, ADaM, Controlled Terminology), including Biomedical Concepts and SDTM Dataset Specializations. Browse via the [Library Browser](https://library.cdisc.org/browser) and obtain API access from the [Developer Portal](https://api.developer.library.cdisc.org).

### Requirements
- **Node.js**: >= 18
- **CDISC Library API key**: subscription key from the Developer Portal

### Get an API key
1. Create/sign in to a CDISC Library account.
   - Browser login: [library.cdisc.org](https://library.cdisc.org/browser)
2. Open the API Management Developer Portal and sign in with the same credentials.
   - Developer Portal: [api.developer.library.cdisc.org](https://api.developer.library.cdisc.org)
3. Retrieve your subscription key (API key) on the portal.
4. Use the key as `api-key` in a header (recommended) or query string.

### Run locally
```bash
export CDISC_API_KEY=your_key_here
npm start
```

#### Shell examples
- **bash/zsh**:
```bash
export CDISC_API_KEY=your_key
export CDISC_AUTH_LOCATION=header   # or: query
npm start
```
- **fish**:
```fish
set -x CDISC_API_KEY your_key
set -x CDISC_AUTH_LOCATION header   # or: query
npm start
```
- **PowerShell**:
```powershell
$env:CDISC_API_KEY = "your_key"
$env:CDISC_AUTH_LOCATION = "header"   # or: "query"
npm start
```

### Configuration (environment variables)
- **CDISC_API_KEY** (required): your subscription key
- **CDISC_AUTH_LOCATION** (optional): `header` (default) or `query`
- **CDISC_AUTH_HEADER** (optional): header name when using header auth, default `api-key`
- **CDISC_AUTH_QUERY** (optional): query param name when using query auth, default `api-key`
- **CDISC_API_BASE_URL** (optional): override server base URL, default `https://api.library.cdisc.org`
- **CDISC_OPENAPI_DIR** (optional): directory for OpenAPI YAML; defaults to `server/openapi`. The `manifest.json` sets this to `${DXT_DIR}/server/openapi` when packaged as DXT so the extension is self‑contained.

#### Caching and resilience
- **CDISC_CACHE_ENABLED** (optional): set to `0` to disable caching. Default: enabled.
- **CDISC_CACHE_TTL_MS** (optional): TTL for GET responses in milliseconds. Default: `60000`.
- **CDISC_CACHE_MAX_ENTRIES** (optional): max cache entries (LRU). Default: `500`.
- **CDISC_RETRY_COUNT** (optional): number of retries for transient 5xx/network errors. Default: `2`.
- **CDISC_RETRY_BACKOFF_MS** (optional): base backoff per attempt in ms. Default: `300`.
- **CDISC_CACHE_DEBUG** (optional): set to `1` to log cache/retry activity to stderr.
 - **CDISC_CACHE_PERSIST_PATH** (optional): file path to persist cache to disk (JSON). Default: disabled.
 - **CDISC_CACHE_EXCLUDE_REGEX** (optional): regex to exclude cache matches; defaults also exclude search/suggest.
 - **CDISC_CACHE_INCLUDE_REGEX** (optional): regex to force-include cache matches.

### Package as a DXT extension
- Spec: [DXT MANIFEST.md](https://github.com/anthropics/dxt/blob/main/MANIFEST.md)
- CLI: [DXT CLI.md](https://github.com/anthropics/dxt/blob/main/CLI.md)

```bash
# Option A: one‑off
npm install -g @anthropic-ai/dxt
dxt pack        # creates cdisc-library-mcp.dxt in the project directory

# Option B: via npm script
npm run pack:dxt
```

### Install in clients
- **Claude Desktop**: open the generated `.dxt` file to install the local MCP server.
- Any DXT‑compatible app can load this extension per its instructions. See the DXT repo: `https://github.com/anthropics/dxt`.

### Tools exposed by the server
- **Generic**:
  - `list_operations(filter?)`
  - `call_operation(operationId, pathParams?, query?, body?, headers?, timeoutMs?)`
- **Cache management**:
  - `cache.clear()` — clear all in-memory cache entries
  - `cache.invalidate(contains?, regex?)` — invalidate entries by substring or regex match
- **Discovery**:
  - `search.suggest(q, top?, select?)`
  - `search.query(q, start?, pageSize?, facets?, filters?)`
- **Biomedical Concepts (latest)**:
  - `bc.list(category?)`, `bc.get(conceptId)`, `bc.categories()`
- **Biomedical Concepts (packages)**:
  - `bc.packages.list()`, `bc.packages.listConcepts(package)`, `bc.packages.getConcept(package, conceptId)`
- **SDTM Specializations (latest)**:
  - `sdtm.list(domain?)`, `sdtm.get(datasetSpecializationId)`, `sdtm.domains()`, `sdtm.byBiomedicalConcept(conceptId)`
- **SDTM Specializations (packages)**:
  - `sdtm.packages.list()`, `sdtm.packages.listSpecializations(package)`, `sdtm.packages.getSpecialization(package, datasetSpecialization)`

### Quick API reference
- **Auth & bases**:
  - API key (header or query): `api-key: <YOUR_KEY>`
  - Library base: `https://api.library.cdisc.org/api`
  - Cosmos BC base: `https://api.library.cdisc.org/api/cosmos/v2/mdr/bc`
  - Cosmos SDTM Specializations base: `https://api.library.cdisc.org/api/cosmos/v2/mdr/specializations`

- **Discovery**:
  - GET `/mdr/suggest` — params: `q`, `top`, `select`
  - GET `/mdr/search` — params: `q` (req), `start`, `pageSize`, `facets`, filters `domain|codelist|conceptId|product|version|dataset|variable|standard`

- **Biomedical Concepts**:
  - Latest: `/biomedicalconcepts`, `/biomedicalconcepts/{biomedicalconcept}`, `/categories`
  - Version‑locked: `/packages`, `/packages/{package}/biomedicalconcepts`, `/packages/{package}/biomedicalconcepts/{biomedicalconcept}`
  - Key fields: `_links`, `conceptId`, `href`, `categories`, `shortName`, `synonyms`, `resultScales`, `definition`, `coding`, `dataElementConcepts`, `ncitCode`

- **SDTM Dataset Specializations**:
  - Latest: `/sdtm/datasetspecializations?domain=VS|LB|…`, `/sdtm/datasetspecializations/{dataset_specialization_id}`, `/sdtm/domains`, by BC: `/datasetspecializations?biomedicalconcept={conceptId}`
  - Version‑locked: `/sdtm/packages`, `/sdtm/packages/{package}/datasetspecializations`, `/sdtm/packages/{package}/datasetspecializations/{datasetspecialization}`
  - Key fields: `_links`, `datasetSpecializationId`, `domain`, `shortName`, `source`, `sdtmigStartVersion`, `sdtmigEndVersion`, `variables[]`
  - `variables[]` keys: `name`, `dataElementConceptId`, `isNonStandard`, `codelist`, `subsetCodelist`, `valueList`, `assignedTerm`, `role`, `dataType`, `length`, `format`, `significantDigits`, `relationship{subject,linkingPhrase,predicateTerm,object}`, `mandatoryVariable`, `mandatoryValue`, `originType`, `originSource`, `comparator`, `vlmTarget`

- **Behavior**:
  - Pagination on `/mdr/search`: `start`, `pageSize`
  - Errors surfaced as‑is: 401, 422, 404/406/5xx
  - Links: follow `_links.self`, `_links.parentPackage`, `_links.parentBiomedicalConcept`

### Test the server
Scripts in `package.json`:
- `npm run test:mcp` — smoke tests via a minimal MCP stdio client (`scripts/test_mcp.js`). Requires `CDISC_API_KEY`.
- `npm run test:plan` — broader happy‑path + error scenarios (`scripts/test_plan.js`). Requires `CDISC_API_KEY`.

### Troubleshooting
- **401 Unauthorized**: ensure `CDISC_API_KEY` is set; switch `CDISC_AUTH_LOCATION` between `header` and `query` if needed.
- **Timeouts/network errors**: check connectivity to `https://api.library.cdisc.org`; set `timeoutMs` when using `call_operation`.
- **API changes**: update the YAML specs under `server/openapi`, then restart. You can override the base via `CDISC_API_BASE_URL`.
- **Corporate proxies**: configure your environment (e.g., `HTTPS_PROXY`, `HTTP_PROXY`).

### Governance
- **License**: MIT (see `LICENSE.md`).
- **Code of Conduct**: see `CODE_OF_CONDUCT.md`.


