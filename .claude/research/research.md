# Research — Platypus

Background research collected during development. Internal for now, with the option to promote to an official project file (e.g. a repo wiki) when a permanent home is decided.

---

## Service Repository File Roles

Each file in a fully developed service repository can be characterised along multiple axes.

**Axis 1: Provenance** — where does the file come from?

| Framework-provided                 | Developer-provided                     |
|:-----------------------------------|:---------------------------------------|
| `AGENTS.md`, Workflow file, README | Config, Config Schema, Terraform files |

> Note: framework-provided files are installed by the framework in a service repository. Developer-provided files are created by the service developer during the development of the service.

**Axis 2: Lifecycle** — when is the file used?

| Dev-time    | Runtime                                                       |
|:------------|:--------------------------------------------------------------|
| `AGENTS.md` | Config, Config Schema, Terraform files, Workflow file, README |

**Axis 3: Audience** — who is the file for?

| Service developer                                          | Service user   |
|:-----------------------------------------------------------|:---------------|
| `AGENTS.md`, Config Schema, Terraform files, Workflow file | Config, README |

---

## JSON Schema to Markdown Conversion Tool Evaluation

Background research for the service docs generation (service README) from the service's JSON schema. The goal is to get an overview of how existing tools convert JSON Schema to Markdown documentation. To do so, a set of JSON Schema to Markdown conversion tools has been identified and thoroughly analysed. Each tool has been run on an example schema (full outputs in [`jsonschema2markdown/`](jsonschema2markdown)). The [Tool Evaluation Results](#tool-evaluation-results) section contains the results of this tool evaluation.

### Output Categories

During the analysis, we identified the following two output categories which we will use in the discussion of the individual tools:

**Hierarchical:** the schema's hierarchical structure is preserved — each field, object, and array is represented as an independent entity (a section, subsection, or linked file) that stands in a structural relationship with other entities, expressed, for example, through nesting or hyperlinks.

**Flat:** the schema hierarchy is flattened — all fields at all nesting levels are presented as uniform, standalone entries with no structural relationship to each other; each entry carries its full ancestry path (e.g. `items[].mode.strategy`) to preserve context.

### Tool Evaluation Results

> Note: all GitHub data (stars, number of commits, etc.) has been retrieved on 2026-04-19.

#### 1. [`prmd`](https://github.com/interagent/prmd) (Ruby)

- **GitHub:** 2,093 stars | 636 commits | last: 2025-02-06 (14 months ago)
- **Category:** Flat
- **Formatting:** single table with all fields; nesting expressed via underscore-separated names (e.g. `mode_settings_timeout`)
- **Metadata:** *Name*, *Type*, *Description*, *Example* as table columns
- **Verbosity:** low for schema content; padded by irrelevant REST API scaffolding
- **Constraint:** not applicable to arbitrary JSON schemas, expects schema to be in a certain format (HTTP REST API documentation)

#### 2. [`json-schema-for-humans`](https://github.com/coveooss/json-schema-for-humans) (Python)

- **GitHub:** 732 stars | 448 commits | last: 2026-04-14 (less than 1 week ago)
- **Category:** Hierarchical
- **Formatting:** nested sections mirroring the schema hierarchy; each object field includes a child-property summary table followed by individual subsections per child
- **Metadata:** *Type*, *Required*, *Additional properties* in a per-field vertical table; child-property summary tables include *Property*, *Pattern*, *Type*, *Deprecated*, *Definition*, *Title/Description*; array fields include an additional table with *Min items*, *Max items*, *Items unicity*, *Additional items*, *Tuple validation*; description and examples as separate paragraphs
- **Verbosity:** very high — every field including simple scalars gets a full section; array item sections get an auto-generated "items items" heading when no `title` is set on the array item schema

#### 3. [`@adobe/jsonschema2md`](https://github.com/adobe/jsonschema2md) (Node.js)

- **GitHub:** 715 stars | 1,411 commits | last: 2026-04-14 (less than 1 week ago)
- **Category:** Hierarchical
- **Formatting:** multi-file output — 12 files total: an index, a root overview, and one file per field/object/array (10 files for our schema); nested content lives in sibling files linked by relative paths; designed for static site generators
- **Metadata:** every file includes a header table with *Abstract*, *Extensible*, *Status*, *Identifiable*, *Custom Properties*, *Additional Properties*, *Access Restrictions*, *Defined In*; root properties summary table adds *Property*, *Type*, *Required*, *Nullable*, *Defined by*; examples as separate code blocks
- **Verbosity:** very high — every field gets its own file; header table is noise-heavy with mostly irrelevant metadata

#### 4. [`wetzel`](https://github.com/CesiumGS/wetzel) (Node.js)

- **GitHub:** 137 stars | 174 commits | last: 2022-07-18 (3 years 9 months ago)
- **Category:** Hierarchical
- **Formatting:** single file; nested sections to represent hierarchy; each object gets a properties table with each field as a row
- **Metadata:** *Type*, *Description*, *Required* as table columns
- **Verbosity:** medium
- **Constraint:** only resolves `$ref` references — does not recurse into inline nested objects; for our schema only the root object and the `items` array are documented

#### 5. [`jsonschema-markdown`](https://github.com/elisiariocouto/jsonschema-markdown) (Python)

- **GitHub:** 35 stars | 150 commits | last: 2026-03-25 (less than 1 month ago)
- **Category:** Flat
- **Formatting:** single table with all fields; nesting expressed via dot-notation with array notation (e.g. `items[].mode.strategy`)
- **Metadata:** *Property*, *Type*, *Required*, *Possible values*, *Deprecated*, *Default*, *Description*, *Examples* as table columns
- **Verbosity:** low — all fields in a single compact table, no per-field sections or prose

#### 6. [`json-schema-static-docs`](https://github.com/tomcollins/json-schema-static-docs) (Node.js)

- **GitHub:** 31 stars | 298 commits | last: 2025-01-06 (15 months ago)
- **Category:** Hierarchical
- **Formatting:** single file; each field gets a section at the same heading level; nesting expressed via dot-notation in section headings (e.g. `items.mode.strategy`)
- **Metadata:** per-field vertical table with only present attributes as rows — *Description*, *Type*, *Required*, *Enum*, *Examples* (and others when present in the schema)
- **Verbosity:** low — sparse per-field tables with only present attributes

#### 7. [`json-schema-to-markdown`](https://github.com/saibotsivad/json-schema-to-markdown) (Node.js)

- **GitHub:** 30 stars | 30 commits | last: 2019-06-26 (6 years 10 months ago)
- **Category:** Hierarchical
- **Formatting:** single file; heading level deepens with each nesting level; type and required status appended in parentheses to each heading (e.g. `` `strategy` (enum, required) ``); no tables
- **Metadata:** *Type* and *Required* in the section heading; enum values as a bullet list; description as prose
- **Verbosity:** low — section body contains only description and enum values
- **Note:** [`jsonschema-2-markdown`](https://github.com/hugorper/jsonschema-2-markdown) (1 star, last commit 2018) is a direct fork with identical output; only additions are cosmetic refactoring and a document template wrapper

#### 8. [`jsonschema2mk`](https://github.com/simonwalz/jsonschema2mk) (Node.js)

- **GitHub:** 10 stars | 131 commits | last: 2026-03-29 (less than 1 month ago)
- **Category:** Hierarchical
- **Formatting:** single file; nested sections for objects and arrays mirroring the schema hierarchy; scalar properties remain as rows in the parent section's properties table
- **Metadata:** *Name*, *Type*, *Description*, *Required* as table columns; enum values and constraints inlined into the *Description* cell
- **Verbosity:** low — compact section-per-object structure with no per-scalar sections

#### 9. [`json-schema-to-markdown-table`](https://www.jsdelivr.com/package/npm/json-schema-to-markdown-table) (Node.js)

- **Source:** published on jsDelivr only — no GitHub repository
- **Category:** Flat
- **Formatting:** single table with all fields; nesting expressed via dot-notation with array notation
- **Metadata:** *Name*, *Type*, *Description* as table columns; optional fields marked with `*Optional*` in the *Description* cell; no enum values, defaults, or constraints
- **Verbosity:** very low — minimal three-column table, least metadata of all evaluated tools

### Conclusions

- JSON Schema to Markdown conversion tools use both output categories that we identified: 6 are Hierarchical, 3 are Flat
- The two output categories seem to be suited for different purposes
- **Hierarchical:** suited for a navigable reference
  - Each field or object is an addressable entity that can be linked to individually (e.g. from an index or search result)
  - The schema's structural relationships map to navigation operations (drill-down via nested sections or hyperlinks)
  - Metadata can be rich without overwhelming the reader since each entity has its own dedicated space
  - Scales well with schema complexity
- **Flat:** suited for an at-a-glance overview of an entire schema
  - All fields are represented as uniform standalone entities (each one including full ancestral path) which is easy to parse for small schemas
  - Structural relationship mapping is given up for the potential of compact representation that allows showing the entire schema at a glance
  - Close visual resemblance to the YAML or JSON that the schema defines
  - Suitable to represent schemas that can be comprehended at a glance — becomes unwieldy for large schemas that require segmentation and navigation to comprehend
- Conclusion for the Platypus docs generation:
  - The Flat output category seems to be the better fit as schema complexity is expected to be low, navigation or search is not intended, and ideally the config structure should become apparent to the user at a single glance (while still allowing for the casual lookup of the necessary details for writing a valid config).
  - None of the tested tools is suitable to be used directly for the docs generation. The way to go is to implement the tailored custom conversion logic from scratch.

---

## Programmatic Output Consumption Options

Options considered for machine-readable access to provisioning outputs, all rejected in favour of manual human-readable output via the GitHub Actions job summary.

- **Terraform remote state:** requires the consumer to also use Terraform — a dealbreaker for clients that should need no IaC tooling
- **SSM Parameter Store:** clients need AWS credentials just to read a value, introducing an AWS dependency for unrelated consumers
- **Shared state repo / output file committed to a repo:** pollutes git history with machine-generated commits; concurrency risk; conflates human intent (config) with machine state (outputs)
- **HCP Terraform API:** workable but against the grain — HCP Terraform is not designed for cross-system output distribution
- **Purpose-built outputs store:** cleanest interface, but requires building and hosting a new service

---

## JSON Schema 2020-12 keyword reference

Source documents: `draft-bhutton-json-schema-01` (core spec), `draft-bhutton-json-schema-validation-01` (validation companion spec).

### Annotation keywords

Annotation keywords are added directly in any schema object (at root level or on any field), alongside validation keywords. They have no effect on validation — they are purely informational, used by tooling to produce documentation.

| Keyword | Type |
|---|---|
| `title` | string |
| `description` | string |
| `default` | any |
| `deprecated` | boolean |
| `readOnly` | boolean |
| `writeOnly` | boolean |
| `examples` | array |

> Source: `draft-bhutton-json-schema-validation-01`, section 9 (Meta-Data vocabulary)

### Validation keywords

Validation keywords are added directly in any schema object alongside annotation keywords. They assert constraints on the instance value — validation fails if a constraint is not satisfied.

Type assertion keywords — every schema object must have one:

| Keyword | Valid values |
|---|---|
| `type` | `"array"`, `"boolean"`, `"integer"`, `"null"`, `"number"`, `"object"`, `"string"` (or an array of these) |
| `enum` | An array of allowed values of any type |
| `const` | Any single value |

Constraint keywords — each category only applies to fields of the corresponding type:

| Category | Keywords |
|---|---|
| Number constraints | `multipleOf`, `maximum`, `exclusiveMaximum`, `minimum`, `exclusiveMinimum` |
| String constraints | `maxLength`, `minLength`, `pattern` |
| Array constraints | `maxItems`, `minItems`, `uniqueItems`, `maxContains`, `minContains` |
| Object constraints | `maxProperties`, `minProperties`, `required`, `dependentRequired` |

> Source: `draft-bhutton-json-schema-validation-01`, section 6

### Format keywords

The `format` keyword is added to a `string` field with one of the following spec-defined values (e.g. `"format": "uri"`). By default it is an annotation only (no effect on validation); it can be configured as an assertion by declaring the Format-Assertion vocabulary.

| Category | Values |
|---|---|
| Dates & times | `date-time`, `date`, `time`, `duration` |
| Email | `email`, `idn-email` |
| Hostnames | `hostname`, `idn-hostname` |
| IP addresses | `ipv4`, `ipv6` |
| Resource identifiers | `uri`, `uri-reference`, `iri`, `iri-reference`, `uuid` |
| URI template | `uri-template` |
| JSON pointers | `json-pointer`, `relative-json-pointer` |
| Regular expressions | `regex` |

> Source: `draft-bhutton-json-schema-validation-01`, section 7.3

