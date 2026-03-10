# Coding Standards & Best Practices

**Audience:** Junior Developers
**Version:** 1.0 — March 2026

---

## Table of Contents

1. [File & Folder Organization](#1-file--folder-organization)
2. [Naming Conventions](#2-naming-conventions)
3. [Function Design](#3-function-design)
4. [Batch Processing — Map/Reduce](#4-batch-processing--mapreducer-over-scheduled-scripts)
5. [Platform Patterns](#5-platform-patterns) (Suitelets, Secrets, REST Web Services)
6. [Error Handling](#6-error-handling)
7. [Logging](#7-logging)
8. [Security](#8-security)
9. [Comments & Documentation](#9-comments--documentation)
10. [Constants & Configuration](#10-constants--configuration)
11. [Boolean & Data Handling](#11-boolean--data-handling)
12. [Testing](#12-testing)
13. [Code Smells to Avoid](#13-code-smells-to-avoid)
14. [Git, Repos & Documentation](#14-git-repos--documentation)
15. [Deployment Pipeline](#15-deployment-pipeline)
16. [Performance](#16-performance)
17. [Quick Reference Checklist](#17-quick-reference-checklist)

---

## 1. File & Folder Organization

### One file = one responsibility

Each file should represent a single service, feature, or concern. Don't put unrelated logic in the same file.

### Project structure

```
Project-Root/
├── src/
│   ├── FileCabinet/SuiteScripts/AppName/
│   │   ├── lib/                  # Shared library modules (reusable services)
│   │   │   ├── fp_lib_settings.js
│   │   │   ├── fp_lib_api_connect.js
│   │   │   ├── fp_lib_tax_calc.js
│   │   │   ├── fp_lib_address_utils.js
│   │   │   ├── fp_lib_transaction_workspace.js
│   │   │   └── ...               # One file per service
│   │   ├── user-event/           # Transaction-driven entry points (beforeSubmit/afterSubmit)
│   │   │   ├── fp_ue_tax_calc.js
│   │   │   ├── fp_ue_settings.js
│   │   │   └── ...
│   │   ├── client/               # Client-side UI scripts (pageInit, saveRecord, fieldChanged)
│   │   │   ├── fp_cs_validation_user_alert.js
│   │   │   └── fp_cs_address_snapshot.js
│   │   ├── map-reduce/           # Batch/background processing jobs
│   │   │   ├── fp_mr_batch_processor.js
│   │   │   ├── fp_mr_update_addresses.js
│   │   │   └── fp_mr_upsert_item_tax_codes.js
│   │   ├── suitelet/             # Web UI endpoints (forms, viewers, tools)
│   │   │   ├── fp_sl_transaction_viewer.js
│   │   │   ├── fp_sl_batch_processor.js
│   │   │   └── ...
│   │   ├── portlet/              # Dashboard widgets
│   │   │   └── fp_portlet_tax_dashboard.js
│   │   └── bundle/               # Bundle install/upgrade scripts
│   │       └── fp_bundle_install.js
│   │
│   └── Objects/                  # Platform configuration (XML definitions)
│       ├── custom_records/       # Custom record types
│       │   ├── customrecord_fp_settings.xml
│       │   ├── customrecord_fp_sub_api_link.xml
│       │   └── customrecord_fp_item_tax_codes.xml
│       ├── custom_fields/        # Custom fields on transactions, entities, items
│       │   ├── custbody_fp_tax_history_steps.xml
│       │   ├── custentity_fp_cust_exemp_code.xml
│       │   ├── custitem_fp_item_tax_code.xml
│       │   └── ...
│       ├── custom_lists/         # Dropdown/selection lists
│       │   ├── customlist_fp_cust_ex_codes.xml
│       │   └── customlist_fp_trans_purpose_codes.xml
│       ├── custom_scripts/       # Script deployment records (mirrors script folders)
│       │   ├── user-event/
│       │   ├── client/
│       │   ├── map-reduce/
│       │   ├── suitelet/
│       │   ├── portlet/
│       │   └── bundle/
│       ├── custom_forms/         # Custom entry/transaction forms
│       ├── custom_saved_searches/ # Saved searches
│       ├── custom_centers/       # Navigation center customizations
│       ├── custom_center_categories/
│       ├── custom_center_links/
│       └── custom_tabs/          # Custom subtabs on records
│
├── __tests__/                    # Unit tests (mirrors src structure)
│   ├── ut_fp_lib_settings.js
│   ├── ut_fp_lib_api_connect.js
│   ├── ut_fp_ue_tax_calc.js
│   ├── ut_fp_mr_update_addresses.js
│   ├── ...                       # One test file per source file
│   ├── utils/                    # Test helpers and mock utilities
│   └── test-records/             # JSON fixtures (sample transactions, records)
│
├── playwright/                   # End-to-end tests
│   ├── scripts/                  # Test spec files
│   └── helpers/                  # Login, API, navigation helpers
│
└── scripts/                      # Build, deploy, and CI utilities
```

**Key principles:**
- **Scripts** go under `src/FileCabinet/` — organized by script type (user-event, client, map-reduce, etc.)
- **Objects** go under `src/Objects/` — XML definitions for custom records, fields, lists, script deployments
- **Script deployments** in `Objects/custom_scripts/` mirror the folder names under `FileCabinet/`
- **Tests** live in `__tests__/` with naming that mirrors the source file (`ut_` prefix + source name)
- **One lib file per service** — don't dump unrelated helpers into one utils file

### File naming

Use a consistent prefix or pattern so anyone can identify a file's purpose at a glance.

```
Scripts:     [prefix]_[type]_[name].js
Tests:       ut_[prefix]_[type]_[name].js
Records:     customrecord_[prefix]_[name].xml
Fields:      cust[location]_[prefix]_[name].xml   (body, entity, item, record)
Lists:       customlist_[prefix]_[name].xml
Deployments: customscript_[prefix]_[type]_[name].xml

Examples (using prefix 'fp'):
  fp_lib_settings.js              # Library: settings service
  fp_ue_tax_calc.js               # User Event: tax calculation entry point
  fp_mr_batch_processor.js        # Map/Reduce: batch processing job
  fp_cs_validation_user_alert.js  # Client Script: save-time validation
  fp_sl_transaction_viewer.js     # Suitelet: transaction viewer UI
  ut_fp_lib_settings.js           # Unit test for settings service
  customrecord_fp_settings.xml    # Custom record: app settings
  custbody_fp_tax_history_steps.xml   # Body field: tax audit trail
```

---

## 2. Naming Conventions

### Use `let` and `const` — never `var`

Always use `const` for values that don't change, `let` for values that do. Never use `var` — it has function-scoping bugs that cause hard-to-find issues.

```javascript
// GOOD
const MAX_RETRIES = 3;               // Won't change — use const
const settings = getSettings();       // Object reference won't change — use const
let retryCount = 0;                   // Will be incremented — use let

for (let i = 0; i < lineCount; i++) {  // Loop variable — use let
    const lineAmount = getAmount(i);   // New value each iteration, but const within scope
}
```

```javascript
// BAD: var leaks out of blocks and gets hoisted — causes subtle bugs
for (var i = 0; i < 5; i++) { }
console.log(i);  // 5 — still accessible! With let, this would throw ReferenceError
```

### Write plain JavaScript — no TypeScript unless approved

All SuiteScript code is written in **plain JavaScript (ES6/ES2015+)**. Do not use TypeScript. The transpilation step from TS → JS introduces compatibility risks with the SuiteScript runtime, and most dev teams are not experienced with TypeScript. Only use TypeScript if explicitly approved by the dev manager (directive may come from the client or implementation partner).

```
GOOD:  fp_lib_settings.js      ← Plain JavaScript, runs directly on the platform
BAD:   fp_lib_settings.ts      ← Requires transpilation, risk of runtime issues
```

### Use platform enums — never hardcode string/number literals

When the platform provides enums or constants, always use them instead of raw strings or numbers. This prevents typos, enables IDE autocomplete, and survives platform changes.

```javascript
// GOOD: Platform enums
const ctx = runtime.executionContext;
if (ctx === runtime.ContextType.CSV_IMPORT) { }
if (ctx === runtime.ContextType.WEB_SERVICES) { }
if (ctx === runtime.ContextType.USER_INTERFACE) { }

const rec = record.create({ type: record.Type.SALES_ORDER });
rec.setValue({ fieldId: 'orderstatus', value: record.Status.PENDING_FULFILLMENT });
```

```javascript
// BAD: Magic strings — typos won't be caught, breaks if platform renames
if (ctx === 'csvimport') { }
if (ctx === 'webservices') { }

const rec = record.create({ type: 'salesorder' });
```

### Variables and functions — `camelCase`

```javascript
const customerName = 'Acme Corp';
function calculateTax(amount, rate) { }
```

### Constants — `UPPER_SNAKE_CASE`

```javascript
const MAX_RETRIES = 3;
const CACHE_TTL = 60 * 60;  // 1 hour in seconds
const API_BASE_URL = 'https://api.example.com';
```

### Booleans — prefix with `is`, `has`, `should`, `can`

```javascript
const isActive = true;
const hasPermission = user.role === 'admin';
const shouldRetry = attempts < MAX_RETRIES;
```

### Private members — prefix with `_`

```javascript
class UserService {
    constructor(db) {
        this._db = db;          // Private: internal dependency
        this._cache = {};       // Private: internal state
    }
    getUser(id) { }            // Public: no underscore
}
```

### Arrays — use plural nouns

```javascript
const users = [];              // Not: userList, userArr
const orderIds = [101, 102];   // Not: orderIdArray
```

---

## 3. Function Design

### Public/exported functions go first

The functions returned by your module (entry points like `beforeSubmit`, `afterSubmit`, `saveRecord`, `pageInit`) should be defined **at the top** of the script. Private helper functions go below them. This way anyone opening the file immediately sees what the script does and what it exports.

```javascript
define(['N/record', 'N/log'], function(record, log) {

    // ── PUBLIC (exported) entry points — first ──────────────

    function beforeSubmit(context) {
        validateTransaction(context.newRecord);
        calculateTotals(context.newRecord);
    }

    function afterSubmit(context) {
        postToExternalApi(context.newRecord);
    }

    // ── PRIVATE helpers — below ─────────────────────────────

    function validateTransaction(rec) { /* ... */ }

    function calculateTotals(rec) { /* ... */ }

    function postToExternalApi(rec) { /* ... */ }

    // ── Module exports — last ───────────────────────────────

    return { beforeSubmit, afterSubmit };
});
```

```javascript
// BAD: Entry points buried at the bottom, hard to find what the script does
define(['N/record'], function(record) {
    function helperA() { /* ... */ }
    function helperB() { /* ... */ }
    function helperC() { /* ... */ }
    // ... 200 lines later ...
    function beforeSubmit(context) { /* ... */ }  // Buried!
    return { beforeSubmit };
});
```

### Declare the return value at the top of the function

Establish what you're going to return at the **beginning** of the function. This makes the function's intent clear from line one — anyone reading it immediately knows what's being built. Build it up through the function, then return it at the end.

```javascript
// GOOD: Return value declared upfront — intent is clear
function buildTaxPayload(record) {
    const payload = {
        transactionId: record.id,
        lines: [],
        totalAmount: 0
    };

    const lineCount = record.getLineCount({ sublistId: 'item' });
    for (let i = 0; i < lineCount; i++) {
        const lineData = getLineData(record, i);
        payload.lines.push(lineData);
        payload.totalAmount += lineData.amount;
    }

    return payload;
}
```

```javascript
// GOOD: Even simple functions — declare what you return
function getTransactionIds(filters) {
    const ids = [];

    const results = search.create({ /* ... */ }).run();
    results.each(function(result) {
        ids.push(result.getValue('internalid'));
        return true;
    });

    return ids;
}
```

```javascript
// BAD: Return value appears out of nowhere at the end
function buildTaxPayload(record) {
    const lineCount = record.getLineCount({ sublistId: 'item' });
    // ... 40 lines of processing ...
    // ... more processing ...
    return { transactionId: record.id, lines: processedLines, totalAmount: sum };  // Surprise!
}
```

### Keep functions small — aim for under 30 lines

If a function is getting long, break it into helpers. Each function should do **one thing**.

```javascript
// GOOD: Small, focused functions
function validateEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
}

function validatePassword(password) {
    return password.length >= 8;
}

function validateUser(user) {
    return validateEmail(user.email) && validatePassword(user.password);
}
```

```javascript
// BAD: One giant function doing everything
function processUser(user) {
    // validate email... (15 lines)
    // validate password... (10 lines)
    // check duplicates... (20 lines)
    // save to database... (15 lines)
    // send welcome email... (10 lines)
}
```

### Use early returns to reduce nesting

```javascript
// GOOD: Flat, easy to follow
function getDiscount(customer) {
    if (!customer) return 0;
    if (!customer.isActive) return 0;
    if (customer.isPremium) return 0.20;
    return 0.05;
}
```

```javascript
// BAD: Deeply nested, hard to follow
function getDiscount(customer) {
    if (customer) {
        if (customer.isActive) {
            if (customer.isPremium) {
                return 0.20;
            } else {
                return 0.05;
            }
        }
    }
    return 0;
}
```

### Use options objects for 3+ parameters

```javascript
// GOOD: Clear, extendable
function createOrder({ customerId, items, shippingMethod = 'standard', notes = '' }) {
    // ...
}

createOrder({ customerId: 42, items: cart, shippingMethod: 'express' });
```

```javascript
// BAD: Positional args are confusing
function createOrder(customerId, items, shippingMethod, notes, priority, coupon) { }

createOrder(42, cart, 'express', '', 'high', null);  // What are these values?
```

### Return `this` for fluent chaining (when it makes sense)

```javascript
class QueryBuilder {
    where(field, value) {
        this._filters.push({ field, value });
        return this;  // Enables chaining
    }
    limit(n) {
        this._limit = n;
        return this;
    }
}

// Clean, readable usage
const results = query.where('status', 'active').where('role', 'admin').limit(10).run();
```

---

## 4. Batch Processing — Map/Reduce Over Scheduled Scripts

Always prefer **Map/Reduce** over Scheduled Scripts for batch work. Map/Reduce gives you automatic parallelism, built-in retry on errors, governance tracking, and a clear pipeline that reduces data at each stage.

### Always implement all four stages

Every Map/Reduce script must define `getInputData`, `map`, `reduce`, and `summarize`. No skipping stages.

```javascript
return { getInputData, map, reduce, summarize };
```

### The pipeline: each stage reduces the data set

Think of it as a funnel — data gets smaller and more processed at each stage:

| Stage | Purpose | Input | Output |
|-------|---------|-------|--------|
| `getInputData` | Set up the search or query | Parameters/config | Search object, SuiteQL query, or API call |
| `map` | Break down, validate, group | One item from input | `context.write({ key, value })` grouped by key |
| `reduce` | Create/update/delete records | Grouped values per key | `context.write({ key, value })` result per key |
| `summarize` | Report totals, log errors | All output + error iterators | Final log entry |

### `getInputData` — return a search or query, don't return data

You don't need to build an array and return it. Just return a **saved search**, **`search.create()` object**, or **SuiteQL query** — the platform handles pagination automatically. Keep it minimal and delegate query construction to a library function.

All search/query logic lives in a **separate lib file** (e.g., `fp_lib_query.js`). The Map/Reduce script just calls the lib.

**Saved Search — platform paginates for you:**

```javascript
// ── fp_mr_batch_processor.js ──────────────────────────────
// getInputData just calls the lib — no search logic here
function getInputData() {
    return fp_lib_query.createTransactionSearch();
}
```

```javascript
// ── fp_lib_query.js ───────────────────────────────────────
// Search construction lives here — testable, reusable
function createTransactionSearch() {
    return search.create({
        type: search.Type.TRANSACTION,
        filters: [
            ['type', 'anyof', 'SalesOrd', 'CustInvc'],
            'AND', ['mainline', 'is', 'T'],
            'AND', ['custbody_fp_needs_processing', 'is', 'T']
        ],
        columns: ['internalid', 'type', 'tranid', 'entity']
    });
    // Return the search object — DON'T .run().each() it
    // NetSuite's Map/Reduce engine iterates the search automatically
}
```

**SuiteQL with pagination (`OFFSET`/`FETCH`):**

When using SuiteQL, the platform does **not** auto-paginate. The lib handles paging with `OFFSET` and `FETCH FIRST N ROWS ONLY`:

```javascript
// ── fp_mr_sync_records.js ─────────────────────────────────
function getInputData() {
    return fp_lib_query.getAllTransactionIds();
}
```

```javascript
// ── fp_lib_query.js ───────────────────────────────────────
function getAllTransactionIds() {
    const PAGE_SIZE = 1000;
    let offset = 0;
    const allResults = [];
    let hasMore = true;

    while (hasMore) {
        const results = query.runSuiteQL({
            query: `
                SELECT id, type, tranid
                FROM transaction
                WHERE custbody_fp_needs_processing = 'T'
                  AND type IN ('SalesOrd', 'CustInvc')
                ORDER BY id
                OFFSET ${offset} ROWS
                FETCH FIRST ${PAGE_SIZE} ROWS ONLY
            `
        }).asMappedResults();

        allResults.push(...results);
        hasMore = results.length === PAGE_SIZE;
        offset += PAGE_SIZE;
    }

    return allResults;
}
```

**External API call — also in a lib:**

```javascript
// ── fp_mr_upsert_tax_codes.js ─────────────────────────────
function getInputData() {
    return fp_lib_api_connect.getTaxCodes();
}
```

```javascript
// ── fp_lib_api_connect.js ─────────────────────────────────
function getTaxCodes() {
    const response = https.post({ url: endpoint, headers: authHeaders, body: payload });
    return JSON.parse(response.body);
}
```

**What NOT to do:**

```javascript
// BAD: Search logic inline in the Map/Reduce script — move to a lib file
function getInputData() {
    const results = [];
    search.create({
        type: 'transaction',
        filters: [['type', 'anyof', 'SalesOrd']],
        columns: ['internalid', 'tranid']
    }).run().each(function(result) {
        results.push(result);  // Building array manually — unnecessary
        return true;
    });
    return results;  // Just return the search object instead
}
```

```javascript
// BAD: SuiteQL without pagination — misses rows beyond first page
function getInputData() {
    return query.runSuiteQL({
        query: `SELECT id FROM transaction WHERE type = 'SalesOrd'`
    }).asMappedResults();  // Only gets first 5000 rows!
}
```

### `map` — break down, validate, and group by key

Map processes each input item individually. Use it to **look up data, validate, and write grouped keys** for reduce. Don't do creates/updates here — that's reduce's job.

```javascript
function map(context) {
    let transactionId = null;
    try {
        transactionId = context.value;

        // Look up the data (don't return it — write it forward)
        const txnData = txnQuery.getTransactionById(transactionId);
        if (!txnData) {
            context.write({ key: transactionId, value: { success: false, error: 'Not found' } });
            return;
        }

        // Validate and group
        const recordType = TYPE_MAP[txnData.type];
        if (!recordType) {
            context.write({ key: transactionId, value: { success: false, error: 'Unknown type: ' + txnData.type } });
            return;
        }

        // Write forward to reduce — grouped by key
        context.write({
            key: transactionId,
            value: { success: true, recordType, tranid: txnData.tranid }
        });

    } catch (e) {
        logError('map', e);
        context.write({ key: transactionId || 'unknown', value: { success: false, error: e.message } });
    }
}
```

### `reduce` — do the actual creates/updates/deletes

Reduce receives grouped values per key. This is where records get loaded, modified, and saved. Each key's results get written forward to summarize.

```javascript
function reduce(context) {
    try {
        const transactionId = context.key;
        const values = context.values.map(v => {
            try { return JSON.parse(v); } catch (e) { return { success: false }; }
        });

        // Check if map reported failure
        const mapResult = values[0];
        if (!mapResult.success) {
            context.write({ key: 'failed', value: transactionId });
            return;
        }

        // Do the actual record operation
        const txnRecord = record.load({
            type: mapResult.recordType,
            id: transactionId,
            isDynamic: true
        });
        txnRecord.save({ enableSourcing: true, ignoreMandatoryFields: true });

        context.write({ key: 'success', value: transactionId });

    } catch (e) {
        logError('reduce', e);
        context.write({ key: 'failed', value: context.key || 'unknown' });
    }
}
```

### `summarize` — report results and log all errors

Always iterate through all error summaries (input, map, reduce) so failures are visible.

```javascript
function summarize(summary) {
    try {
        let successCount = 0;
        let failedCount = 0;

        summary.output.iterator().each(function(key, value) {
            if (key === 'success') successCount++;
            else if (key === 'failed') failedCount++;
            return true;
        });

        log.audit('Batch Complete', JSON.stringify({
            success: successCount,
            failed: failedCount,
            totalSeconds: summary.seconds,
            usage: summary.usage
        }));

        // Always log errors from each stage
        if (summary.inputSummary && summary.inputSummary.error) {
            log.error('Input Error', summary.inputSummary.error);
        }
        summary.mapSummary.errors.iterator().each(function(key, error) {
            log.error('Map Error', `Key: ${key}, Error: ${error}`);
            return true;
        });
        summary.reduceSummary.errors.iterator().each(function(key, error) {
            log.error('Reduce Error', `Key: ${key}, Error: ${error}`);
            return true;
        });

    } catch (e) {
        logError('summarize', e);
    }
}
```

### Why not Scheduled Scripts?

| | Map/Reduce | Scheduled Script |
|---|---|---|
| Parallelism | Automatic (platform manages workers) | Manual (you manage batches) |
| Error recovery | Platform retries failed keys | You build retry logic |
| Governance | 10,000 units per stage | 10,000 units total |
| Progress tracking | Built-in summary with counts | You track manually |
| Data pipeline | Natural funnel (input → map → reduce → summarize) | One big loop |

---

## 5. Platform Patterns

### Suitelets — raw HTML, not N/ui/serverWidget forms

We write Suitelet UIs with **raw HTML/CSS/JS** instead of the built-in `serverWidget.createForm()` widgets. This gives us full control over layout, styling, and interactivity. Use `serverWidget` only as the outer container — inject your HTML into an `inlinehtml` field.

```javascript
function onRequest(context) {
    try {
        if (context.request.method === 'GET') {
            handleGet(context);
        } else {
            handlePost(context);
        }
    } catch (e) {
        logError('onRequest', e);
        context.response.write('Error: ' + e.message);
    }
}
```

**Suitelet structure:**
- `onRequest` at the top — routes GET vs POST
- `handleGet` / `handlePost` as separate functions — keep request handling manageable
- Build HTML strings with template literals — no string concatenation chains
- Extract reusable HTML builders into lib files when shared across suitelets

```javascript
function handleGet(context) {
    const form = serverWidget.createForm({ title: 'Transaction Viewer' });

    // Build raw HTML — full control over layout
    let html = `
        <style>
            .fp-container { max-width: 1200px; margin: 0 auto; }
            .fp-header { display: flex; justify-content: space-between; }
            .fp-stat-box { padding: 12px; border: 1px solid #ddd; }
        </style>
        <div class="fp-container">
            <div class="fp-header">
                <h2>Results</h2>
                <div class="fp-stats">Showing ${results.length} of ${totalCount}</div>
            </div>
            ${buildResultsTable(results)}
            ${buildPagination(currentPage, totalPages)}
        </div>
        <script>
            // Client-side interactivity — filters, sorting, etc.
            document.getElementById('filterBtn').addEventListener('click', applyFilters);
        </script>
    `;

    const htmlField = form.addField({
        id: 'custpage_html',
        type: serverWidget.FieldType.INLINEHTML,
        label: ' '
    });
    htmlField.defaultValue = html;

    context.response.writePage(form);
}
```

```javascript
// BAD: serverWidget forms — limited styling, rigid layout
function handleGet(context) {
    const form = serverWidget.createForm({ title: 'Viewer' });
    form.addField({ id: 'custpage_name', type: 'text', label: 'Name' });
    form.addSublist({ id: 'custpage_results', type: 'list', label: 'Results' });
    // Can't customize layout, colors, or add interactivity
}
```

### Secrets library — never hardcode tokens or API keys

Store all sensitive credentials (API keys, OAuth tokens, integration secrets) in the **NetSuite Secrets Management** system (`N/https.createSecureString`). Reference secrets by their `custsecret_` ID — never store actual values in code, fields, or logs.

```javascript
// GOOD: Secret resolved at runtime by the platform — never exposed
const AUTH_TOKEN = https.createSecureString({ input: `Bearer {${apiKeyName}}` });
const req = {
    url: endpoint,
    headers: { 'Authorization': AUTH_TOKEN, 'Content-Type': 'application/json' },
    body: JSON.stringify(payload)
};
```

```javascript
// BAD: Hardcoded token — exposed in code, logs, and version control
const AUTH_TOKEN = 'Bearer sk-abc123-my-real-api-key';
```

**Naming convention for secrets:**
```
Production:  custsecret_fp_api_secret
Sandbox:     custsecret_sandbox_fp_api_secret
Subsidiary:  custsecret_fp_sub_[id]_secret    (per-subsidiary keys)
```

Use environment detection to pick the right secret:
```javascript
function getApiKeyName(env, subsidiaryId) {
    const prefix = env === runtime.EnvType.SANDBOX ? 'custsecret_sandbox_' : 'custsecret_';
    if (subsidiaryId && subsidiaryApiKeys[subsidiaryId]) {
        return prefix + subsidiaryApiKeys[subsidiaryId].apiKey;
    }
    return prefix + 'fp_api_secret';
}
```

### REST Web Services — prefer over SOAP/SuiteTalk

When integrating with NetSuite externally or building internal integrations, use **REST Web Services** (RESTlets, SuiteQL via REST) over SOAP/SuiteTalk. REST is simpler, faster, and easier to debug.

```javascript
// RESTlet — clean JSON in, JSON out
function post(context) {
    try {
        const payload = context;  // Already parsed JSON
        const result = processRequest(payload);
        return { success: true, data: result };
    } catch (e) {
        logError('RESTlet.post', e);
        return { success: false, error: e.message };
    }
}

return { post, get: get };
```

**When to use REST Web Services:**
- External system needs to create/read/update NetSuite records
- Running SuiteQL queries from outside NetSuite
- Building API endpoints for third-party integrations
- Automated testing (Playwright E2E tests use REST to create test data)

---

## 6. Error Handling

Our error handling strategy has one goal: **when something fails, you know exactly what failed, where, and on which record/line.** Errors bubble up from the deepest level with full context so the top-level catch can log everything in one place.

### Top-level try...catch in every entry point

Every exported function (`beforeSubmit`, `afterSubmit`, `map`, `reduce`, `pageInit`, etc.) gets **one** top-level try...catch. This is the safety net — it logs the full error and prevents the script from crashing silently.

```javascript
function beforeSubmit(context) {
    try {
        validateTransaction(context.newRecord);
        calculateTax(context.newRecord);
    } catch (e) {
        logError('beforeSubmit', e);
        throw e;  // Re-throw so the platform knows it failed
    }
}

function afterSubmit(context) {
    try {
        postToExternalApi(context.newRecord);
    } catch (e) {
        logError('afterSubmit', e);
        // Don't re-throw in afterSubmit — transaction is already saved
    }
}
```

### Never leave an empty catch block

An empty catch is a **silent failure** — the worst kind of bug. If you catch, you **must** do something: log it, add context, or re-throw.

```javascript
// BAD: Error disappears — debugging becomes impossible
try {
    processLine(record, i);
} catch (e) {
    // Nothing here. No one will ever know this failed.
}

// BAD: Equally useless
try {
    processLine(record, i);
} catch (e) {
    // ignore
}
```

```javascript
// GOOD: Always log or handle
try {
    processLine(record, i);
} catch (e) {
    logError('processLine', e);
    throw e;
}

// GOOD: If you genuinely want to continue, log WHY you're ignoring it
try {
    sendOptionalNotification(record);
} catch (e) {
    log.debug('beforeSubmit', `Notification skipped (non-critical): ${e.message}`);
}
```

### Bubble up granular errors — include line, item, and record context

When processing loops (line items, child records, sublists), **wrap each iteration** and attach context so the top-level catch knows exactly which line/record failed and why.

```javascript
// GOOD: Error tells you exactly which line failed and what was on it
function setTaxByLine(record) {
    const lineCount = record.getLineCount({ sublistId: 'item' });

    for (let i = 0; i < lineCount; i++) {
        const itemName = record.getSublistText({ sublistId: 'item', fieldId: 'item', line: i });
        try {
            const rate = calculateLineRate(record, i);
            record.setSublistValue({ sublistId: 'item', fieldId: 'taxrate1', line: i, value: rate });
        } catch (e) {
            // Attach line context before bubbling up
            throw new Error(
                `Failed to set tax on line ${i} (item: ${itemName}): ${e.message}`
            );
        }
    }
}
```

```javascript
// BAD: All you get is "Cannot read property 'taxcode' of undefined" — which line? which item?
function setTaxByLine(record) {
    const lineCount = record.getLineCount({ sublistId: 'item' });
    for (let i = 0; i < lineCount; i++) {
        const rate = calculateLineRate(record, i);
        record.setSublistValue({ sublistId: 'item', fieldId: 'taxrate1', line: i, value: rate });
    }
}
```

### The full pattern: entry point → helper → line-level

Errors flow **up** with more context at each level:

```javascript
// LEVEL 3 (deepest): Line-level operation — attaches line context
function processLine(record, line) {
    const itemName = record.getSublistText({ sublistId: 'item', fieldId: 'item', line });
    const taxCode = record.getSublistValue({ sublistId: 'item', fieldId: 'taxcode', line });
    try {
        const rate = api.getTaxRate(taxCode);
        record.setSublistValue({ sublistId: 'item', fieldId: 'taxrate1', line, value: rate });
    } catch (e) {
        throw new Error(`Line ${line} "${itemName}" (taxcode: ${taxCode}): ${e.message}`);
    }
}

// LEVEL 2 (middle): Helper — attaches operation context
function calculateTax(record) {
    const lineCount = record.getLineCount({ sublistId: 'item' });
    for (let i = 0; i < lineCount; i++) {
        processLine(record, i);  // Errors bubble up with line context
    }
}

// LEVEL 1 (top): Entry point — catches everything, logs full context
function beforeSubmit(context) {
    try {
        calculateTax(context.newRecord);
    } catch (e) {
        // Error message: "Line 3 "Widget-A" (taxcode: AVATAX): Connection timeout"
        logError('beforeSubmit', e);
        throw e;
    }
}
```

When this fails, the log reads:
```
[beforeSubmit] Line 3 "Widget-A" (taxcode: AVATAX): Connection timeout
```
Not just:
```
[ERROR] Connection timeout
```

### Use a consistent `logError` helper

```javascript
function logError(context, error) {
    log.error(context, [
        `Message: ${error.message || 'Unknown error'}`,
        `Stack: ${error.stack || 'No stack trace'}`
    ].join('\n'));
}
```

### Validate inputs early — fail fast

```javascript
function processPayment(orderId, amount) {
    if (!orderId) throw new Error('orderId is required');
    if (amount <= 0) throw new Error('amount must be positive');

    // Now safe to proceed
    return gateway.charge(orderId, amount);
}
```

### Return safe defaults only for genuinely non-critical operations

```javascript
// Non-critical: return empty result instead of crashing
function getRecentNotifications(userId) {
    try {
        return notificationService.fetch(userId);
    } catch (e) {
        log.debug('getRecentNotifications', `Skipped (non-critical): ${e.message}`);
        return [];  // Page still loads, just without notifications
    }
}
```

### External API calls must have retry with max 3 attempts

Every call to an external API or system **must** have retry logic. Retries must not exceed **3 attempts**, and each failure must log exactly what went wrong — HTTP status, error message, which attempt it was, and what was being called.

```javascript
const MAX_RETRIES = 3;

function callExternalApi(method, url, payload) {
    let lastError = null;

    for (let attempt = 1; attempt <= MAX_RETRIES; attempt++) {
        try {
            const response = https.post({
                url,
                headers: { 'Authorization': authToken, 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            });

            // Handle specific HTTP errors differently
            if (response.code >= 200 && response.code < 300) {
                return JSON.parse(response.body);
            }

            // 4xx = client error — don't retry, it will fail again
            if (response.code >= 400 && response.code < 500) {
                throw new Error(
                    `API returned ${response.code} (client error, not retrying): ${response.body}`
                );
            }

            // 5xx = server error — worth retrying
            lastError = new Error(`API returned ${response.code}: ${response.body}`);
            log.debug('callExternalApi',
                `Attempt ${attempt}/${MAX_RETRIES} failed: HTTP ${response.code} from ${method} ${url}`
            );

        } catch (e) {
            lastError = e;

            // Connection errors (timeout, DNS, network) — worth retrying
            if (e.name === 'SSS_REQUEST_TIME_EXCEEDED' || e.message.includes('timeout')) {
                log.debug('callExternalApi',
                    `Attempt ${attempt}/${MAX_RETRIES} timed out: ${method} ${url}`
                );
            } else if (e.message.includes('client error, not retrying')) {
                // 4xx errors — stop immediately
                throw e;
            } else {
                log.debug('callExternalApi',
                    `Attempt ${attempt}/${MAX_RETRIES} failed: ${e.message} — ${method} ${url}`
                );
            }
        }
    }

    // All retries exhausted
    throw new Error(
        `${method} ${url} failed after ${MAX_RETRIES} attempts. Last error: ${lastError.message}`
    );
}
```

**Key rules for retry:**
- **Max 3 attempts** — never more. If it fails 3 times, it's a real problem, not a transient blip.
- **Don't retry 4xx errors** — 400 Bad Request, 401 Unauthorized, 403 Forbidden, 404 Not Found mean *your request* is wrong. Retrying won't fix it.
- **Do retry 5xx and timeouts** — 500 Internal Server Error, 502/503/504, and connection timeouts are often transient.
- **Log every attempt** — include attempt number, HTTP status, URL, and method so you can trace the failure in logs.
- **Final error includes all context** — the thrown error after all retries must say what was called, how many attempts, and the last error message.

---

## 7. Logging

### Use structured, contextual log messages

```javascript
// GOOD: Clear context + relevant data
log.debug('OrderService.create', `Created order #${orderId} for customer ${customerId}`);
log.audit('PaymentService.charge', `Charged $${amount} to card ending ${last4}`);
log.error('APIClient.post', `HTTP ${response.code} from ${url}`);
```

```javascript
// BAD: Vague, unhelpful
console.log('done');
console.log('error happened');
console.log(data);  // Dumps entire object with no context
```

### Log levels — use them correctly

| Level | When to Use | Example |
|-------|-------------|---------|
| `debug` | Development details, step-by-step flow | "Processing line 3 of 12" |
| `info` / `audit` | Significant business events | "Order #1234 created" |
| `warn` | Recoverable issues, degraded behavior | "Cache miss, falling back to DB" |
| `error` | Failures that need attention | "Payment gateway timeout" |

### Never log sensitive data

```javascript
// GOOD: Redact secrets
log.info('API Request', `POST ${url} (auth: [REDACTED])`);

// BAD: Leaks credentials
log.info('API Request', `POST ${url} with key ${apiKey}`);
```

---

## 8. Security

### Never expose secrets in logs, responses, or source code

```javascript
// GOOD: Reference secrets by name, resolve at runtime
const apiKey = config.getSecret('API_KEY');

// Format logs safely
function formatRequestForLog(request) {
    const safeHeaders = { ...request.headers };
    if (safeHeaders.Authorization) safeHeaders.Authorization = '[REDACTED]';
    if (safeHeaders['X-API-Key']) safeHeaders['X-API-Key'] = '[REDACTED]';
    return { method: request.method, url: request.url, headers: safeHeaders };
}
```

### Validate at system boundaries

Validate data coming from **outside your system** — user input, API responses, file uploads. Trust internal function calls between your own modules.

```javascript
// Boundary: HTTP endpoint — VALIDATE
function handleApiRequest(req) {
    const email = req.body.email;
    if (!email || !validateEmail(email)) {
        return { status: 400, error: 'Invalid email' };
    }
    return userService.create(email);  // Internal call — no need to re-validate
}
```

### Use environment-specific configuration

```javascript
function getApiKeyName(environment) {
    return environment === 'sandbox'
        ? 'secret_sandbox_api_key'
        : 'secret_prod_api_key';
}
```

---

## 9. Comments & Documentation

### Comment **why**, not **what**

```javascript
// GOOD: Explains intent
// NetSuite returns null for unfilled address fields, but the API expects empty strings
const city = address.city ?? '';

// BAD: Restates the code
// Set city to empty string if null
const city = address.city ?? '';
```

### Use JSDoc for public functions

```javascript
/**
 * Calculate tax for a transaction's line items.
 * Skips lines with non-standard tax codes.
 *
 * @param {Object} transaction - The transaction record
 * @param {string} apiKey - Name of the secret (not the actual key)
 * @returns {{ totalTax: number, lineResults: Array }} Calculation results
 */
function calculateTax(transaction, apiKey) { }
```

### When to comment vs. when to rename

If you need a comment to explain **what** code does, consider renaming instead.

```javascript
// Before: Needs a comment
const d = new Date(ts * 1000);  // Convert Unix timestamp to Date

// After: Self-documenting
const dateFromUnixTimestamp = new Date(unixSeconds * 1000);
```

### Don't leave commented-out code

Commented-out code is clutter. If you might need it later, that's what version control is for — delete it.

```javascript
// BAD: Dead code
// function oldCalculation(x) {
//     return x * 0.07;
// }
function newCalculation(x) {
    return x * taxRate;
}

// GOOD: Just the current code. Old version lives in git history.
function calculateTax(amount) {
    return amount * taxRate;
}
```

---

## 10. Constants & Configuration

### No magic numbers or strings

Every literal value with business meaning should be a named constant.

```javascript
// BAD: What do these numbers mean?
if (response.code >= 200 && response.code < 300) { }
if (retries > 3) { }
setTimeout(fn, 3600000);

// GOOD: Self-documenting
const HTTP_SUCCESS_MIN = 200;
const HTTP_SUCCESS_MAX = 299;
const MAX_RETRIES = 3;
const ONE_HOUR_MS = 60 * 60 * 1000;

if (response.code >= HTTP_SUCCESS_MIN && response.code <= HTTP_SUCCESS_MAX) { }
if (retries > MAX_RETRIES) { }
setTimeout(fn, ONE_HOUR_MS);
```

### Use `Object.freeze()` for constant maps

```javascript
const ORDER_STATUS = Object.freeze({
    PENDING: 'pending',
    APPROVED: 'approved',
    SHIPPED: 'shipped',
    CANCELLED: 'cancelled'
});

// Prevents accidental mutation
ORDER_STATUS.PENDING = 'oops';  // Silently fails (or throws in strict mode)
```

### Group related config together

```javascript
const CACHE_CONFIG = {
    name: 'app_settings',
    ttl: 60 * 60,        // 1 hour
    scope: 'protected'
};

const API_CONFIG = {
    baseUrl: 'https://api.example.com',
    timeout: 30000,       // 30 seconds
    maxRetries: 2
};
```

---

## 11. Boolean & Data Handling

### Normalize booleans from external sources

APIs, databases, and config files often return booleans as strings or numbers. Normalize them early.

```javascript
// External systems may return: true, 'true', 'T', 1, '1', 'yes'
function toBool(value) {
    return value === true || value === 'T' || value === 1 || value === '1' || value === 'true';
}

// Use at the boundary
const isActive = toBool(record.getValue('is_active'));
```

### Use `??` (nullish coalescing) for defaults, not `||`

```javascript
// GOOD: Only falls back on null/undefined
const count = input.count ?? 10;       // 0 is a valid count
const name = input.name ?? 'Unknown';  // '' (empty string) is a valid name

// BAD: Treats 0, '', false as missing
const count = input.count || 10;       // If count is 0, you get 10!
const name = input.name || 'Unknown';  // If name is '', you get 'Unknown'!
```

### Normalize data at the boundary, use it clean internally

```javascript
// Normalize ONCE when data enters your system
function normalizeAddress(raw) {
    return {
        line1: (raw.line1 || '').trim(),
        city: (raw.city || '').trim(),
        state: (raw.state || '').toUpperCase().trim(),
        zip: (raw.zip || '').replace(/[\s-]/g, '')   // Strip spaces and hyphens
    };
}

// Internal functions receive clean data — no defensive checks needed
function calculateShipping(address) {
    // address.zip is already normalized — just use it
    return shippingRates[address.state] || defaultRate;
}
```

---

## 12. Testing

### Test file naming mirrors source files

```
Source:  lib/settings.js
Test:    __tests__/ut_lib_settings.js

Source:  controllers/order.js
Test:    __tests__/ut_ctrl_order.js
```

### Structure: Arrange → Act → Assert

```javascript
test('calculateTax returns correct tax for standard rate', () => {
    // Arrange: Set up data and mocks
    const order = { subtotal: 100.00, state: 'CA' };
    taxRateService.getRate.mockReturnValue(0.0725);

    // Act: Call the function under test
    const result = calculateTax(order);

    // Assert: Verify the outcome
    expect(result.tax).toBe(7.25);
    expect(result.total).toBe(107.25);
});
```

### One assertion per concept (not necessarily one per test)

```javascript
// GOOD: Related assertions grouped logically
test('createUser returns a complete user object', () => {
    const user = createUser({ name: 'Alice', email: 'alice@example.com' });

    expect(user.id).toBeDefined();
    expect(user.name).toBe('Alice');
    expect(user.email).toBe('alice@example.com');
    expect(user.createdAt).toBeInstanceOf(Date);
});
```

### Mock external dependencies, not your own code

```javascript
// GOOD: Mock the HTTP client (external boundary)
jest.mock('axios');
axios.get.mockResolvedValue({ data: { id: 1, name: 'Test' } });

// BAD: Mocking your own internal helper defeats the purpose of testing
jest.mock('./helpers/formatName');  // Now you're not testing real behavior
```

### Use `jest.isolateModules()` for environment-specific tests

```javascript
function loadModuleWithConfig(env) {
    let moduleExports;
    jest.isolateModules(() => {
        process.env.NODE_ENV = env;
        moduleExports = require('./myModule');
    });
    return moduleExports;
}

test('uses sandbox URL in development', () => {
    const mod = loadModuleWithConfig('development');
    expect(mod.apiUrl).toBe('https://sandbox.api.example.com');
});

test('uses production URL in production', () => {
    const mod = loadModuleWithConfig('production');
    expect(mod.apiUrl).toBe('https://api.example.com');
});
```

### Use fixtures for complex test data

```javascript
// __tests__/fixtures/sample-order.json
{
    "id": "ORD-001",
    "customerId": 42,
    "items": [
        { "sku": "WIDGET-A", "qty": 2, "price": 19.99 },
        { "sku": "GADGET-B", "qty": 1, "price": 49.99 }
    ]
}

// In test file
const sampleOrder = require('./fixtures/sample-order.json');

test('processOrder handles multi-item orders', () => {
    const result = processOrder(sampleOrder);
    expect(result.lineCount).toBe(2);
});
```

### Clean up between tests

```javascript
beforeEach(() => {
    jest.clearAllMocks();    // Reset call counts and return values
});

afterEach(() => {
    delete process.env.CUSTOM_FLAG;  // Clean up env changes
});
```

---

## 13. Code Smells to Avoid

### Deeply nested conditionals

```javascript
// SMELL
if (user) {
    if (user.isActive) {
        if (user.hasPermission('edit')) {
            doEdit();
        }
    }
}

// FIX: Guard clauses
if (!user) return;
if (!user.isActive) return;
if (!user.hasPermission('edit')) return;
doEdit();
```

### Giant functions (>50 lines)

Break them into named steps:

```javascript
// SMELL: 120-line processOrder function

// FIX: Decompose into steps
function processOrder(order) {
    validateOrder(order);
    const pricing = calculatePricing(order);
    const tax = calculateTax(order, pricing);
    return saveOrder(order, pricing, tax);
}
```

### Copy-paste code (DRY — Don't Repeat Yourself)

If you see the same 5+ lines in multiple places, extract a function.

```javascript
// SMELL: Same formatting logic in 3 files
const name = `${first.trim()} ${last.trim()}`.toUpperCase();

// FIX: Shared utility
function formatFullName(first, last) {
    return `${first.trim()} ${last.trim()}`.toUpperCase();
}
```

### Boolean parameters that change behavior

```javascript
// SMELL: What does `true` mean here?
processOrder(order, true, false);

// FIX: Use an options object
processOrder(order, { expedited: true, sendNotification: false });
```

### Catch-all error handling

```javascript
// SMELL: Catches everything, hides real problems
try {
    doEverything();
} catch (e) {
    console.log('something went wrong');
}

// FIX: Specific handling with context
try {
    const order = await fetchOrder(id);
    const payment = await chargePayment(order);
    await sendConfirmation(order, payment);
} catch (e) {
    logError('processCheckout', e);
    throw e;  // Don't hide the failure
}
```

---

## 14. Git, Repos & Documentation

### Every project lives in a Git repo

No exceptions. Every project — no matter how small — must have:
- A **Git repository** (GitHub) from day one
- A **`CLAUDE.md`** at the project root — project instructions, conventions, and architecture for AI-assisted development
- A **`wiki/`** folder — markdown docs covering architecture, data flows, troubleshooting, and how the system works
- **Inline code comments** where logic is non-obvious (see Section 9)

The goal: any developer (or AI) can pick up the project and understand it without asking you.

### Pull requests are required

**Never push directly to `main`.** All changes go through pull requests:
- Create a feature/fix branch
- Push your branch and open a PR
- PR must describe what changed and why
- Get review before merge
- Squash or merge to `main`

### Claude Code (Max Pro) is a core development tool

We use **Claude Code** (via Claude Max Pro subscription) as a day-to-day development tool for:
- Writing and refactoring code with full codebase context
- Running tests and debugging failures
- Generating and updating wiki documentation (automated via `nspublish.ps1`)
- Code review assistance
- Exploring unfamiliar codebases

Every project should have a `CLAUDE.md` that gives Claude the context it needs — project structure, naming conventions, key patterns, and common commands. This file is loaded automatically when Claude Code opens the project.

### Commit messages — imperative mood, explain the "why"

```
GOOD:
  Add retry logic to payment gateway calls
  Fix tax rounding on multi-line invoices
  Remove deprecated address validation endpoint

BAD:
  Fixed stuff
  Updates
  WIP
  asdfasdf
```

### Commit small and often

Each commit should represent **one logical change**. If you can't describe it in one sentence, it's probably too big.

### Code must build and pass before you push

**Never push broken code to the repo.** Another developer must be able to pull your branch and be immediately working — no build errors, no SDF validation failures, no failing tests.

Before every push, verify:
1. **Unit tests pass** — `npm test`
2. **SDF validates** — `suitecloud project:validate` (no XML errors, no missing dependencies)
3. **No syntax errors** — the code runs without throwing on load

If tests fail or SDF reports errors, **fix them before pushing**. The repo is the source of truth — if it's broken in the repo, it's broken for everyone.

```
# Pre-push checklist
npm test                          # All unit tests green
suitecloud project:validate       # SDF validation clean
git push origin feature/my-branch # Only after both pass
```

### Branch naming

```
feature/add-user-search
fix/tax-calculation-rounding
chore/update-dependencies
```

### Keep the repo clean — no junk files

Only source code, configuration, tests, and documentation belong in the repo. **Never commit:**
- Binaries, compiled output, build artifacts (`.exe`, `.dll`, `.zip`)
- Dependencies (`node_modules/`, `vendor/`)
- IDE/editor files (`.idea/`, `.vscode/settings.json` with personal settings)
- OS files (`.DS_Store`, `Thumbs.db`)
- Logs, temp files, cache directories
- Secrets and credentials (see below)

Set up `.gitignore` from day one:

```
# Dependencies
node_modules/

# Secrets
.env
*.secret
credentials.json

# Build/output
dist/
build/
*.exe
*.dll
*.zip

# IDE/OS
.idea/
.DS_Store
Thumbs.db
*.log
```

### Never commit secrets

If you accidentally commit a secret, **rotate it immediately** — removing it from git history is not enough.

---

## 15. Deployment Pipeline

We use **SDF (SuiteCloud Development Framework)** for deploying to NetSuite. The full pipeline is automated via `nspublish.ps1` — never deploy by manually uploading files.

### The deploy protocol

Every deploy follows this exact sequence:

```
1. Account setup       — (optional) run suitecloud account:setup if switching accounts
2. Version bump        — auto-increment patch version in settings record XML
3. Wiki update         — (optional) AI-generated documentation refresh if code changed
4. Git fetch & compare — ensure local is up to date with remote (abort if behind)
5. Stage & commit      — git add -A + commit with message (default: "Auto-deploy")
6. Push to remote      — push to origin branch (abort on failure)
7. SDF deploy          — suitecloud project:deploy (runs unit tests via pre-deploy hook)
8. E2E tests           — (optional) run Playwright tests after successful deploy
```

### Key rules

- **Never deploy without pushing first** — the repo must always reflect what's in NetSuite
- **Unit tests run automatically** — the `suitecloud.config.js` pre-deploy hook runs `npm test` before every deploy. If tests fail, deploy is blocked.
- **Version is always incremented** — the settings record tracks the deployed version. This is bumped automatically by the publish script.
- **Fetch before push** — always check that remote hasn't diverged. If remote has commits you don't have, pull and resolve first.

### SDF project types

```powershell
# Account Customization Project (ACP) — standard deploy
suitecloud project:deploy

# SuiteApp — deploy with install preferences
suitecloud project:deploy --applyinstallprefs
```

The publish script auto-detects project type from `project.json`.

### Running the pipeline

```powershell
# From the project root
.\nspublish.ps1
```

The script is interactive — it prompts for commit message (28s timeout, defaults to "Auto-deploy") and optional Playwright test run after deploy.

---

## 16. Performance

### Cache expensive operations

```javascript
// GOOD: Compute once, reuse
class ReportGenerator {
    get settings() {
        if (!this._settings) {
            this._settings = loadSettingsFromDb();  // Expensive — only do it once
        }
        return this._settings;
    }
}
```

### Use `Set` for membership checks on large collections

```javascript
// BAD: O(n) lookup on every check
const exemptIds = [101, 202, 303, /* ... hundreds more */];
if (exemptIds.includes(customerId)) { }

// GOOD: O(1) lookup
const exemptIds = new Set([101, 202, 303]);
if (exemptIds.has(customerId)) { }
```

### Don't fetch more data than you need

```javascript
// BAD: Load everything, use one field
const allUsers = await db.query('SELECT * FROM users');
const names = allUsers.map(u => u.name);

// GOOD: Only fetch what you need
const names = await db.query('SELECT name FROM users');
```

### Avoid work inside loops that can be done outside

```javascript
// BAD: Regex compiled on every iteration
for (const item of items) {
    if (/^EXEMPT-/.test(item.code)) { }
}

// GOOD: Compile once
const exemptPattern = /^EXEMPT-/;
for (const item of items) {
    if (exemptPattern.test(item.code)) { }
}
```

---

## 17. Quick Reference Checklist

Use this before submitting code for review:

### Before writing code
- [ ] Do I understand the requirement? (Ask if unclear — don't guess)
- [ ] Is there existing code I can reuse or extend?

### While writing code
- [ ] Functions are under 30 lines
- [ ] No magic numbers or strings — all named constants
- [ ] Variables and functions have descriptive names
- [ ] Booleans prefixed with `is`/`has`/`should`/`can`
- [ ] Error handling at entry points with context in log messages
- [ ] No secrets in code or logs
- [ ] Input validated at system boundaries

### Before committing
- [ ] Tests written for new/changed logic
- [ ] Tests pass (`npm test`)
- [ ] No `console.log` debugging left behind
- [ ] No commented-out code
- [ ] Commit message describes **why**, not just what
- [ ] No secrets or `.env` files staged

### Before requesting review
- [ ] Code is consistent with the existing codebase style
- [ ] I can explain every line I wrote if asked

---

*This is a living document. As the team grows and patterns evolve, update it.*
