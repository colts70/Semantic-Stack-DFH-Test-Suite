# Semantic-Stack-DFH-Test-Suite
A repo that contains:  JSON-LD examples  Common mistakes  Valid/invalid stacks  CI validation tests  This makes adoption easier and prevents implementation divergence.
DFH-Semantic-Stack-Examples/
├─ README.md
├─ schema/
│  └─ stack.schema.json
├─ examples/
│  ├─ valid/
│  │  ├─ minimal-stack.json
│  │  └─ full-stack.json
│  └─ invalid/
│     ├─ missing-anchors.json
│     ├─ wrong-context.json
│     └─ bad-types.json
├─ docs/
│  └─ common-mistakes.md
├─ tools/
│  └─ validate-stacks.js
├─ package.json
└─ .github/
   └─ workflows/
      └─ validate-stacks.yml
Below are the actual file contents you can copy straight into GitHub.

2. README.md
md
Copy code
# DFH Semantic Stack — Examples & Validator

This repository provides **canonical JSON-LD examples** for DFH / Semantic Stack,
along with **valid/invalid stacks**, **common mistakes**, and a **CI validator**
to keep implementations aligned with the spec.

The goal:  
> Make adoption trivial and prevent silent divergence across stacks.

---

## Contents

- `schema/stack.schema.json`  
  Minimal JSON Schema describing a DFH /.well-known/stack file.

- `examples/valid/`  
  Canonical valid stacks (minimal + full).

- `examples/invalid/`  
  Common failure cases: missing anchors, wrong context, bad types, etc.

- `docs/common-mistakes.md`  
  Human-readable list of traps and examples.

- `tools/validate-stacks.js`  
  Simple Node.js validator used by CI.

- `.github/workflows/validate-stacks.yml`  
  GitHub Actions CI that runs validation on every push/PR.

---

## 1. What is being validated?

This repo focuses on **structural correctness** of a DFH stack file, not
semantic truth. In other words:

- Are the **core anchors** present?
  - `type`
  - `entity`
  - `url`
  - `map`
  - `index` (or whatever the current 5th anchor is in the spec)

- Is the JSON syntax valid?
- Is the JSON-LD structure sane enough to be consumed by typical tooling?

The **spec itself** still lives in the main Semantic Stack / DFH spec repos.
This repo just keeps **implementations from drifting**.

---

## 2. Quickstart

### Option A: Run validation locally

```bash
git clone https://github.com/YOUR-USER/DFH-Semantic-Stack-Examples.git
cd DFH-Semantic-Stack-Examples

npm install
npm test
This will:

Load schema/stack.schema.json

Validate every .json file under examples/valid and examples/invalid

Fail the build if:

any valid example does not conform, or

any invalid example accidentally passes

Option B: Use this repo as a reference
Copy a file from examples/valid/.

Adapt the values (entity, url, map, etc.) for your topic.

Run your own validation (or plug the schema into your stack).

3. JSON-LD Example (Minimal Stack)
See examples/valid/minimal-stack.json:

json
Copy code
{
  "@context": "https://schema.org",
  "type": "SemanticStackTopic",
  "entity": {
    "@id": "https://example.com/#entity",
    "@type": "Thing",
    "name": "Example Topic"
  },
  "url": "https://example.com/",
  "map": [
    "https://example.com/sitemap.xml"
  ],
  "index": "https://example.com/.well-known/stack"
}
4. Valid vs Invalid Examples
Valid examples show:

Minimal required anchors

A richer, “full” stack with extra metadata

Invalid examples deliberately break:

Missing anchors (e.g. no map)

Wrong types (map as an object instead of array/string)

Wrong or missing @context

These are meant to be used in tests so people don’t accidentally ship a broken
stack and discover it only when AI/search tooling fails.

5. CI Integration
GitHub Actions workflow:
./github/workflows/validate-stacks.yml

Runs on every push and pull_request

Uses Node.js

Executes npm test

Fails fast if stacks or examples drift from the schema

You can either:

Fork this repo and customize

Or copy the workflow + tools/ + schema/ into your own project

6. Contributing
Add new valid examples when new patterns become common.

Add new invalid examples when you see real-world mistakes.

Keep schema/stack.schema.json minimal but accurate to the DFH spec.

PRs that tighten the schema or add high-signal examples are welcome.

7. License
MIT — keep it open, simple, and forkable.

pgsql
Copy code

---

## 3. `schema/stack.schema.json`

Minimal JSON Schema for the stack file. This is intentionally **loose** so it
doesn’t over-constrain the spec.

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/schema/stack.schema.json",
  "title": "DFH Semantic Stack",
  "description": "Minimal JSON Schema for a DFH /.well-known/stack file.",
  "type": "object",
  "required": ["type", "entity", "url", "map", "index"],
  "properties": {
    "@context": {
      "type": ["string", "array"],
      "description": "JSON-LD context. Often https://schema.org or a compatible context."
    },
    "type": {
      "type": "string",
      "minLength": 1,
      "description": "The semantic type of this stack anchor object."
    },
    "entity": {
      "type": "object",
      "description": "Canonical entity for this topic/domain.",
      "required": ["@id"],
      "properties": {
        "@id": {
          "type": "string",
          "format": "uri"
        },
        "@type": {
          "type": ["string", "array"]
        },
        "name": {
          "type": "string"
        }
      },
      "additionalProperties": true
    },
    "url": {
      "type": ["string", "object"],
      "description": "Canonical URL for this topic/domain.",
      "oneOf": [
        {
          "type": "string",
          "format": "uri"
        },
        {
          "type": "object",
          "required": ["@id"],
          "properties": {
            "@id": {
              "type": "string",
              "format": "uri"
            }
          },
          "additionalProperties": true
        }
      ]
    },
    "map": {
      "description": "One or more sitemap-like resources that map the topic.",
      "oneOf": [
        {
          "type": "string",
          "format": "uri"
        },
        {
          "type": "array",
          "items": {
            "type": "string",
            "format": "uri"
          }
        }
      ]
    },
    "index": {
      "description": "Index resource for this stack (often the /.well-known/stack URL itself or a hub).",
      "oneOf": [
        {
          "type": "string",
          "format": "uri"
        },
        {
          "type": "array",
          "items": {
            "type": "string",
            "format": "uri"
          }
        }
      ]
    }
  },
  "additionalProperties": true
}
4. Valid examples
examples/valid/minimal-stack.json
json
Copy code
{
  "@context": "https://schema.org",
  "type": "SemanticStackTopic",
  "entity": {
    "@id": "https://example.com/#entity",
    "@type": "Thing",
    "name": "Example Topic"
  },
  "url": "https://example.com/",
  "map": [
    "https://example.com/sitemap.xml"
  ],
  "index": "https://example.com/.well-known/stack"
}
examples/valid/full-stack.json
json
Copy code
{
  "@context": [
    "https://schema.org",
    "https://example.com/context/stack.jsonld"
  ],
  "type": "SemanticStackTopic",
  "entity": {
    "@id": "https://example.com/#entity",
    "@type": [
      "Thing",
      "WebSite"
    ],
    "name": "Example Topic",
    "alternateName": "Example Topic (DFH Stack)",
    "sameAs": [
      "https://en.wikipedia.org/wiki/Example",
      "https://knowledge-graph.example.org/entity/example-topic"
    ]
  },
  "url": {
    "@id": "https://example.com/"
  },
  "map": [
    "https://example.com/sitemap.xml",
    "https://example.com/sitemaps/topic.xml"
  ],
  "index": [
    "https://example.com/.well-known/stack",
    "https://example.com/stack/index.json"
  ],
  "meta": {
    "version": "1.0.0",
    "lastUpdated": "2025-12-03",
    "spec": "https://your-main-spec-domain/.well-known/stack"
  }
}
5. Invalid examples
examples/invalid/missing-anchors.json
json
Copy code
{
  "@context": "https://schema.org",
  "type": "SemanticStackTopic",
  "entity": {
    "@id": "https://broken.example.com/#entity",
    "@type": "Thing",
    "name": "Broken Stack"
  },
  "url": "https://broken.example.com/"
  /* ERROR: map + index are missing */
}
(When you save this for real, remove the comment, but the idea is:
missing map + index.)

Use this actual JSON in the file (no comment):

json
Copy code
{
  "@context": "https://schema.org",
  "type": "SemanticStackTopic",
  "entity": {
    "@id": "https://broken.example.com/#entity",
    "@type": "Thing",
    "name": "Broken Stack"
  },
  "url": "https://broken.example.com/"
}
examples/invalid/wrong-context.json
json
Copy code
{
  "@context": 123,
  "type": "SemanticStackTopic",
  "entity": {
    "@id": "https://wrong-context.example.com/#entity",
    "@type": "Thing",
    "name": "Wrong Context"
  },
  "url": "https://wrong-context.example.com/",
  "map": "https://wrong-context.example.com/sitemap.xml",
  "index": "https://wrong-context.example.com/.well-known/stack"
}
(Here @context is the wrong type.)

examples/invalid/bad-types.json
json
Copy code
{
  "@context": "https://schema.org",
  "type": 42,
  "entity": "not-an-object",
  "url": ["https://bad-types.example.com/"],
  "map": {
    "primary": "https://bad-types.example.com/sitemap.xml"
  },
  "index": 123
}
Breaks multiple type expectations.

6. docs/common-mistakes.md
md
Copy code
# Common DFH / Semantic Stack Implementation Mistakes

This document lists **realistic ways implementations drift** and shows how to
fix them.

---

## 1. Missing anchors

**Problem**

People publish:

```json
{
  "@context": "https://schema.org",
  "type": "SemanticStackTopic",
  "entity": {
    "@id": "https://example.com/#entity"
  },
  "url": "https://example.com/"
}
Missing:

map

index

Impact

AI/search clients don’t know where the sitemap / index lives.

The stack becomes an orphaned blob of JSON-LD with no mapping surface.

Fix

Always include all required anchors:

type

entity

url

map

index

2. Wrong JSON types
Common mistakes:

type as a number

entity as a string

map as an object instead of a string/array of URLs

index as a number

Fix

Use these patterns:

json
Copy code
"type": "SemanticStackTopic",
"entity": {
  "@id": "https://example.com/#entity",
  "@type": "Thing",
  "name": "Example Topic"
},
"url": "https://example.com/",
"map": [
  "https://example.com/sitemap.xml"
],
"index": "https://example.com/.well-known/stack"
3. Forgetting JSON-LD context
Some implementations omit @context or set it to non-JSON-LD values.

Impact

JSON-LD tooling can’t properly expand or interpret your graph.

Interop with other systems becomes fragile.

Fix

Provide a stable context (or contexts):

json
Copy code
"@context": [
  "https://schema.org",
  "https://your-domain/.well-known/context/stack.jsonld"
]
4. Using non-canonical URLs
If your url and map don’t match what your public site actually uses
(canonical domain, HTTPS, etc.), you create split-brain semantics.

Fix

Use your canonical scheme + host (https://example.com, not http:// or www.)

Match your actual sitemap locations.

5. Silent drift between environments
People manually edit JSON in prod and forget to update examples or schema.

Fix

Treat this repo (or your fork) as the source of truth.

Add a CI step that validates your deployed / .well-known/stack against
this schema before release.

pgsql
Copy code

---

## 7. Node validator script — `tools/validate-stacks.js`

This is what CI runs.

```js
#!/usr/bin/env node

const fs = require("fs");
const path = require("path");
const Ajv = require("ajv");
const addFormats = require("ajv-formats");

const ROOT = path.join(__dirname, "..");
const SCHEMA_PATH = path.join(ROOT, "schema", "stack.schema.json");
const EXAMPLES_DIR = path.join(ROOT, "examples");

function loadJson(filePath) {
  const raw = fs.readFileSync(filePath, "utf8");
  return JSON.parse(raw);
}

function getJsonFiles(dir) {
  return fs
    .readdirSync(dir, { withFileTypes: true })
    .flatMap((entry) => {
      const full = path.join(dir, entry.name);
      if (entry.isDirectory()) return getJsonFiles(full);
      if (entry.isFile() && entry.name.endsWith(".json")) return [full];
      return [];
    });
}

function main() {
  const schema = loadJson(SCHEMA_PATH);

  const ajv = new Ajv({ allErrors: true, strict: false });
  addFormats(ajv);

  const validate = ajv.compile(schema);

  const validDir = path.join(EXAMPLES_DIR, "valid");
  const invalidDir = path.join(EXAMPLES_DIR, "invalid");

  const validFiles = getJsonFiles(validDir);
  const invalidFiles = getJsonFiles(invalidDir);

  let hasError = false;

  console.log("Validating VALID examples...");
  for (const file of validFiles) {
    const data = loadJson(file);
    const ok = validate(data);
    if (!ok) {
      hasError = true;
      console.error(`❌ VALID example failed schema: ${file}`);
      console.error(validate.errors);
    } else {
      console.log(`✅ ${file}`);
    }
  }

  console.log("\nValidating INVALID examples (should fail)...");
  for (const file of invalidFiles) {
    const data = loadJson(file);
    const ok = validate(data);
    if (ok) {
      hasError = true;
      console.error(`❌ INVALID example unexpectedly passed schema: ${file}`);
    } else {
      console.log(`✅ ${file} (correctly rejected)`);
    }
  }

  if (hasError) {
    console.error("\nOne or more validation checks failed.");
    process.exit(1);
  } else {
    console.log("\nAll validation checks passed.");
  }
}

main();
8. package.json
json
Copy code
{
  "name": "dfh-semantic-stack-examples",
  "version": "1.0.0",
  "description": "DFH / Semantic Stack JSON-LD examples, common mistakes, and CI validation tests.",
  "license": "MIT",
  "type": "module",
  "scripts": {
    "test": "node tools/validate-stacks.js"
  },
  "dependencies": {
    "ajv": "^8.17.0",
    "ajv-formats": "^3.0.0"
  }
}
(If you’d rather avoid ESM, drop "type": "module"; the script as written works fine in CJS.)

9. GitHub Actions CI — .github/workflows/validate-stacks.yml
yaml
Copy code
name: Validate DFH Semantic Stacks

on:
  push:
    branches: [ main, master ]
  pull_request:

jobs:
  validate:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"

      - name: Install dependencies
        run: npm install

      - name: Run validation tests
        run: npm test
