# Open Fitness Framework Specification

**Version 0.3 draft** · 30 November 2026 · Thorsten Reitz (weTransform)

## Abstract

The Open Fitness Framework (OFF) defines purpose-specific fitness profiles and signed assessment reports so an agent can determine whether a data asset is fit for a declared use. The core reuses EARL test-result semantics and DQV measurement semantics. Optional extensions add capabilities without increasing the minimum implementation requirements.

The main objects defined in the specification are:

- A fitness profile states what is required.
- A report contains authenticated test assertions or quality measurements about a particular asset snapshot.
- An evaluator applies the profile to admissible reports and explains the resulting decision.

This draft defines a small interoperable core and marks optional extensions at their point of use. [Section 2](#2-conformance-and-terminology) specifies the core implementation path and extension declarations.

The core supports conformity tests and typed quality measurements over one whole asset snapshot with complete coverage. Reports identify the responsible issuer, assessment method, tool versions, and evidence references. A measurement can be reused by profiles with different thresholds. Domain definitions, sampling and richer provenance are separately identified extensions.

Principal building blocks and core choices:

- One immutable whole-snapshot binding per core report; one or more EARL-aligned tests or DQV-aligned measurements.
- Verifiable Credentials 2.0 with ES256 JWS remains the concrete baseline for signed exchange; revocation infrastructure is optional.
- Explicit local trust policy combined with each profile’s evidence constraints.
- PASS, FAIL, UNKNOWN, and CONFLICT at requirement level; FIT, NOT_FIT, and NOT_ASSESSABLE at decision level.
- All core requirements are mandatory and combined by conjunction; alternatives and advisory requirements are optional extensions.

## Contents

- [1 Scope and draft status](#1-scope-and-draft-status)
  - [1.1 Relation to other standards](#11-relation-to-other-standards)
- [2 Conformance and terminology](#2-conformance-and-terminology)
  - [2.1 Core implementation path](#21-core-implementation-path)
  - [2.2 Optional extensions](#22-optional-extensions)
- [3 Architecture and responsibilities](#3-architecture-and-responsibilities)
- [4 Shared data types and common requirements](#4-shared-data-types-and-common-requirements)
  - [4.1 JSON representation](#41-json-representation)
  - [4.2 Values and timestamps](#42-values-and-timestamps)
  - [4.3 References and digests](#43-references-and-digests)
  - [4.4 Extension E1: Scope and coverage](#44-extension-e1-scope-and-coverage)
    - [Scope](#scope)
    - [Coverage](#coverage)
  - [4.5 Data Asset binding](#45-data-asset-binding)
- [5 Fitness profiles](#5-fitness-profiles)
  - [5.1 Profile object](#51-profile-object)
  - [5.2 Requirements](#52-requirements)
  - [5.3 Evidence constraints](#53-evidence-constraints)
    - [5.3.1 Extension E2: retrievable evidence](#531-extension-e2-retrievable-evidence)
  - [5.4 Core conjunction rule](#54-core-conjunction-rule)
    - [5.4.1 Extension E3: Composition](#541-extension-e3-composition)
- [6 Fitness reports](#6-fitness-reports)
  - [6.1 Credential structure](#61-credential-structure)
  - [6.2 Report body and assessment](#62-report-body-and-assessment)
    - [6.2.1 Extension E4: Additional Lifespan Information and Assessment Configuration](#621-extension-e4-additional-lifespan-information-and-assessment-configuration)
  - [6.3 EARL assertions and DQV measurements](#63-earl-assertions-and-dqv-measurements)
    - [6.3.1 Normative semantic mapping](#631-normative-semantic-mapping)
  - [6.4 Supporting assessment evidence](#64-supporting-assessment-evidence)
- [7 Assertion definitions and domain semantics](#7-assertion-definitions-and-domain-semantics)
  - [7.1 Definition contract](#71-definition-contract)
  - [7.2 Schema conformity pattern](#72-schema-conformity-pattern)
  - [7.3 Completeness measurement pattern](#73-completeness-measurement-pattern)
  - [7.4 Extension E5: Accuracy and precision](#74-extension-e5-accuracy-and-precision)
  - [7.5 Extension E6: Successful use and attestations](#75-extension-e6-successful-use-and-attestations)
- [8 Signatures and trust](#8-signatures-and-trust)
  - [8.1 Baseline signature binding](#81-baseline-signature-binding)
  - [8.2 Local trust policy](#82-local-trust-policy)
  - [8.3 Extension E7: Credential status](#83-extension-e7-credential-status)
- [9 Evaluation requests](#9-evaluation-requests)
  - [9.1 Extension E8: Context](#91-extension-e8-context)
  - [9.2 Extension E9: Portable replay with verification resources](#92-extension-e9-portable-replay-with-verification-resources)
- [10 Deterministic evaluation procedure](#10-deterministic-evaluation-procedure)
  - [10.1 Validate and resolve inputs](#101-validate-and-resolve-inputs)
  - [10.2 Authenticate reports and check identity](#102-authenticate-reports-and-check-identity)
  - [10.3 Select and admit records](#103-select-and-admit-records)
  - [10.4 Determine requirement outcomes](#104-determine-requirement-outcomes)
  - [10.5 Aggregate the decision](#105-aggregate-the-decision)
- [11 Fitness decisions](#11-fitness-decisions)
- [12 Discovery and lifecycle](#12-discovery-and-lifecycle)
- [13 Privacy and implementation controls](#13-privacy-and-implementation-controls)
- [14 Alignment with existing specifications](#14-alignment-with-existing-specifications)
- [15 Conformance cases and illustrative example](#15-conformance-cases-and-illustrative-example)
  - [15.1 Required evaluator behavior](#151-required-evaluator-behavior)
  - [15.2 Illustrative example and companion status](#152-illustrative-example-and-companion-status)
- [TODOs](#todos)
- [References](#references)

## 1 Scope and draft status

The Open Fitness Framework (OFF) specifies how to describe purpose-specific requirements for data assets, issue authenticated reports about those assets, and evaluate the reports to reach an explainable fitness decision. Its primary consumer is an agent that needs to decide whether a particular data asset can support a declared purpose. Human assessors, catalogs, applications, and governance services can use the same objects. A fitness decision is limited to the declared requirements, and does not grant access or establish legal compliance.

OFF supports expressing and evaluating requirements across structured datasets, raster data, and location data of any type. A report can express many different types of assessment of a data asset, such as schema conformity, completeness, accuracy, precision, a successful use, an expert attestation, or another explicitly defined property. The common contract concerns the meaning, identity, scope, provenance, and acceptance of the assertion. Domain-specific measurement and validation procedures remain separately defined.

The core covers the information model, compact JSON representation, baseline signatures, local issuer authorization, deterministic evaluation and minimum decision traces. Optional extensions are listed in [Section 2](#2-conformance-and-terminology) and marked where their requirements occur.

This document proposes version 0.3. Its normative requirements are draft requirements, not an adopted community standard.

The provisional OFF namespace is <https://www.open-fitness-framework.org/spec/0.3/>.

> [!NOTE]
> A governed namespace, document license, change process and publication process for new versions of the specification must be agreed before public release of 1.0.

### 1.1 Relation to other standards

Data fitness is contextual: a data asset suitable for one purpose may be unsuitable for another. Schema conformity, quality measurements, certification and documented successful use can all provide relevant evidence, but none independently establishes fitness for every application. Automated consumers need a consistent way to identify applicable requirements, interpret available evidence and determine whether that evidence is sufficient.

The Open Fitness Framework (OFF) specifies how purpose-specific fitness profiles are evaluated against verifiable reports about data assets. It connects requirements, assessment assertions and supporting evidence through explicit rules for applicability, trust and decision-making. Its outcomes distinguish demonstrated fulfilment, demonstrated non-fulfilment and insufficient or inconclusive evidence.

OFF builds on elements of the W3C Evaluation and Report Language (EARL). EARL provides a reusable model connecting an assertor, a test subject, a test criterion and a test result. Its distinctions between passed, failed, undetermined, inapplicable and untested outcomes preserve information essential to reliable assessment. OFF adopts this assertion model as its semantic foundation and adds fitness-profile composition, evidence-acceptance rules and reproducible decisions across multiple assertions.

> [!NOTE]
> EARL is a W3C Working Group Note; its reuse does not imply W3C endorsement of OFF. \[[15](#ref-15)\]

Complementary specifications address other parts of this process:

- The W3C Data Quality Vocabulary supplies concepts for quality metrics and measurements;
- PROV-O describes assessment provenance;
- and Verifiable Credentials provide a model for cryptographically verifiable claims;
- SHACL, JSON Schema and XML Schema support structural validation and executable constraints;
- Domain standards such as ISO 19157 supply established quality semantics;
- OSTrails’ FAIR Testing Resource Vocabulary provides related assessment and benchmark concepts.

OFF defines integration points for these specifications without requiring every implementation to adopt every supporting technology. \[[2](#ref-2), [6](#ref-6), [12](#ref-12), [14](#ref-14), [16](#ref-16), [17](#ref-17)\]

OFF’s contribution is the interoperable interpretation of evidence for a stated purpose. It specifies how evidence is bound to an asset and its scope, how acceptable issuers and assessment methods are identified, and how missing, outdated or conflicting evidence affects a decision. A valid signature on the evidence establishes authenticity and integrity, not the truth or sufficiency of a claim. By making these distinctions explicit, OFF enables agents and people to reach transparent, repeatable and appropriately qualified fitness decisions.

## 2 Conformance and terminology

The uppercase terms **MUST**, **MUST NOT**, **REQUIRED**, **SHOULD**, **SHOULD NOT**, and **MAY** have the meanings defined by IETF BCP 14 \[[1](#ref-1)\] when used as normative requirements. Lowercase uses are ordinary prose.

Sections [2](#2-conformance-and-terminology) through [13](#13-privacy-and-implementation-controls) and the cases labelled Core in [Section 15.1](#151-required-evaluator-behavior) are normative for the core, except for passages explicitly marked **Optional extension** or **Informative**. An Optional extension is not required for core conformance; its MUST and SHOULD statements apply only when that extension is declared and supported. Sections [1](#1-scope-and-draft-status), [14](#14-alignment-with-existing-specifications), [15.2](#152-illustrative-example-and-companion-status), TODOs and References are informative.

The field tables and semantic rules in this document govern conformance. JSON Schema and SHACL artifacts may later encode structural constraints, but structural validation alone never establishes semantic or cryptographic conformance.

| Term                 | Meaning                                                                                                    |
| -------------------- | ---------------------------------------------------------------------------------------------------------- |
| Data asset           | A logically identified collection or source of data.                                                       |
| Representation       | A particular distribution, encoding, table, layer or band collection.                                      |
| Snapshot             | An immutable state bound to a content or manifest digest.                                                  |
| Scope                | The part of the data explicitly asserted; whole snapshot in the core; a defined subset under extension E1. |
| Assertion definition | A pinned criterion or metric definition, including parameters, types, units and interpretation.            |
| Test assertion       | An EARL-aligned statement connecting assertor, subject, criterion and result.                              |
| Quality measurement  | A DQV-aligned observation of a defined metric on a bound subject.                                          |
| Fitness report       | A signed credential containing one or more test assertions or measurement records.                         |
| Fitness profile      | A versioned purpose statement and a set of requirements.                                                   |
| Requirement          | A definition and parameter selector, condition, and optional evidence restrictions.                        |
| Trust policy         | Locally authorized issuer/key and definition acceptance rules.                                             |
| Fitness decision     | A purpose-specific conclusion over a bounded evidence set and explicit inputs.                             |

An implementation MUST state its roles and supported extensions. Profile publishers produce valid profiles; report issuers produce valid signed reports; evaluators authenticate, interpret and evaluate reports. A core evaluator MUST implement both test and measurement record forms, all core comparisons and the conservative conflict rule. Repositories and discovery services are optional deployment roles. No RDF store, reasoner, live registry or network discovery service is required.

An agent MAY discover assets, choose a profile, request additional assessments, and act on a decision. If it claims to implement the evaluator role, its decision MUST follow the deterministic rules here. Natural-language inference MUST NOT substitute for a missing assertion, alter a threshold, or make an unsupported semantic equivalence appear established.

### 2.1 Core implementation path

A core implementation uses one immutable whole snapshot of a data asset, complete coverage, a supplied finite report set, a locally authorized pinned-key policy, and all requirements as mandatory. It checks signed reports, matches definitions and parameters exactly, evaluates conditions, and returns requirement outcomes plus FIT, NOT_FIT or NOT_ASSESSABLE. The minimum trace records the supporting or excluded reports and their reasons. No extension may be enabled implicitly.

### 2.2 Optional extensions

The extension identifiers below are provisional and resolve as the OFF namespace followed by extensions/E1 through extensions/E7. A profile, report body or trust policy using extension-dependent semantics MUST list the applicable URI in a `requiredExtensions` array. The request MUST declare the union needed by its profile and policy. Missing declarations are invalid. Unsupported extensions in a profile, request or policy produce an evaluation error with UNSUPPORTED_EXTENSION and result null. An unsupported report extension excludes that report. Informational fields that cannot affect acceptance do not require an extension declaration.

| ID  | Optional capability                              | Sections                                                                                |
| --- | ------------------------------------------------ | --------------------------------------------------------------------------------------- |
| E1  | Selection scopes and sampled coverage            | [4.4](#44-extension-e1-scope-and-coverage)                                              |
| E2  | Required retrieval of supporting evidence        | [5.3.1](#531-extension-e2-retrievable-evidence)                                         |
| E3  | Alternative acceptance and advisory requirements | [5.4.1](#541-extension-e3-composition)                                                  |
| E4  | Additional Lifespan Information                  | [6.2.1](#621-extension-e4-additional-lifespan-information-and-assessment-configuration) |
| E5  | Domain extensions, Accuracy and Precision        | [7.4](#74-extension-e5-accuracy-and-precision)                                          |
| E6  | Successful use and attestations                  | [7.5](#75-extension-e6-successful-use-and-attestations)                                 |
| E7  | Credential Status                                | [8.3](#83-extension-e7-credential-status)                                               |
| E8  | Context                                          | [9.1](#91-extension-e8-context)                                                         |
| E9  | Portable Replay                                  | [9.2](#92-extension-e9-portable-replay-with-verification-resources)                     |

## 3 Architecture and responsibilities

OFF separates reusable *evidence* from the *purpose* against which it is evaluated. A report that records a completeness value of 0.987 can be evaluated against a profile requiring at least 0.98 and another requiring at least 0.995. The measurement can thus lead to different decisions without being regenerated merely because the threshold changed.

The information flow is as follows:

1. A **profile publisher** defines requirements.
2. An **assessor** examines or uses a bound asset snapshot.
3. An **issuer** takes responsibility for the resulting assertions and signs a report.
4. An **evaluator** receives a profile, asset binding, supplied report set, evaluation time and authorized trust policy. It verifies and filters evidence, evaluates conditions and records a decision. Rich operating context and discovery are optional extensions.

The asset publisher, assessor, report issuer, profile publisher, and evaluator are separate roles even when one organization performs several of them. The report issuer is the entity that signs and accepts responsibility for the assertions. A tool name or version is provenance; it is not automatically an issuer identity or an independent assurance claim.

An OFF report authenticates the issuer’s responsibility for its contained assertions and measurements. Cryptographic verification establishes protected content and origin under the configured identity binding; it does not establish truth. The EARL assertor may differ from the credential issuer. The evaluator MUST separately determine whether the responsible issuer and method are acceptable. \[[2](#ref-2), [15](#ref-15)\]

Every decision is relative to the snapshot, scope, profile, purpose, report set, evaluation time and trust policy. Extension inputs are included when used. Two evaluators using different inputs can legitimately differ. A portable OFF decision MUST retain these bindings and MUST NOT consist only of a FIT label.

## 4 Shared data types and common requirements

This chapter describes requirements and objects that are shared across Fitness profiles and Fitness reports.

### 4.1 JSON representation

Core objects use UTF-8 JSON and the case-sensitive field names and `enum` values in this document. `specVersion` is 0.3-draft throughout. Duplicate member names, non-finite numbers, invalid Unicode and malformed values MUST be rejected. `requiredExtensions` is optional and defaults to an empty array. Undeclared semantics MUST NOT affect evaluation. Definitions may require details fields, but these cannot introduce new core operators.

The compact JSON fields have the normative EARL and DQV interpretations in [Section 6.3](#63-earl-assertions-and-dqv-measurements). Core processing does not require JSON-LD expansion. The JSON representation uses the following prefixes:

- `earl` denotes [`http://www.w3.org/ns/earl#`](http://www.w3.org/ns/earl#)
- `dqv` denotes [`http://www.w3.org/ns/dqv#`](http://www.w3.org/ns/dqv#)
- `dct` denotes [`http://purl.org/dc/terms/`](http://purl.org/dc/terms/)
- `xsd` denotes [`http://www.w3.org/2001/XMLSchema#`](http://www.w3.org/2001/XMLSchema#)
- `sdmx-attribute` denotes [`http://purl.org/linked-data/sdmx/2009/attribute#`](http://purl.org/linked-data/sdmx/2009/attribute#)

All identifier fields described as URIs MUST contain absolute URIs. HTTPS is REQUIRED for report identifiers, issuer identifiers, key identifiers, evidence retrieval links, and status-list credential links in the baseline binding. IDs are compared as exact strings. Core processing MUST NOT apply case folding, percent-decoding equivalence, trailing-slash normalization, or inferred URL redirects to establish identity.

Parameter and assertion-details objects MUST contain only JSON values; any JSON numbers within them MUST be integers in the inclusive range from -9007199254740991 to 9007199254740991. Fractional or larger quantities MUST use definition-specified string representations. Parameter equality is equality of their RFC 8785 JSON Canonicalization Scheme representations. Object member order is insignificant; array order, strings, and explicit values remain significant. An omitted parameter and an explicitly supplied parameter are different unless the pinned definition specifies a normalization that both sides perform before publication. \[[3](#ref-3)\]

### 4.2 Values and timestamps

| Value type | Representation and permitted comparisons                        |
| ---------- | --------------------------------------------------------------- |
| boolean    | JSON boolean; eq and ne.                                        |
| decimal    | Decimal string and absolute unit URI; eq, ne, lt, lte, gt, gte. |
| integer    | Integer string and absolute unit URI; eq, ne, lt, lte, gt, gte. |
| string     | JSON string; eq and ne using exact string equality.             |
| iri        | Absolute URI string; eq and ne using exact string equality.     |

Decimal strings use `-?(0|[1-9][0-9]*)(\.[0-9]+)?`; integer strings use `0|-?[1-9][0-9]*`. Negative zero, leading plus signs, exponent notation, whitespace, NaN, and infinity are invalid. Trailing fractional zeroes are allowed and have no numerical effect. Comparisons MUST use exact decimal or integer arithmetic, with no binary floating-point tolerance and no display rounding. Types and units MUST match; the core does not perform unit conversion or integer-to-decimal coercion.

All timestamps MUST use valid UTC calendar timestamps in the form `YYYY-MM-DDTHH:MM:SSZ`, with seconds 00 through 59 and no fractional seconds. Durations are nonnegative integral seconds. The evaluation instant is an explicit input. There is no implicit clock-skew allowance. Missing, invalid, future, or inconsistent required times MUST be handled as specified in [Section 10](#10-deterministic-evaluation-procedure).

### 4.3 References and digests

An `ArtifactRef` consists of `id` and `digest`. The digest consists of `algorithm`, fixed to `sha-256` in this draft, and `value`, a 64-character lowercase hexadecimal SHA-256 digest. The digest MUST cover the complete referenced content bytes after transport content decoding and before text decoding, newline changes, or JSON reformatting. JSON artifact digests are byte digests, not JCS digests. A pretty-printed copy therefore has a different digest even if it represents the same JSON object.

A consumer MUST verify the digest before relying on retrieved artifact content. An identifier and digest together pin a definition, schema, profile, method, configuration, or other input. A URI that still resolves but now returns different bytes does not satisfy the reference. A resolver MAY retrieve matching content from a local cache or authenticated package registry instead of the public URI. A registry mapping identifies bytes; it does not establish trust in their meaning.

### 4.4 Extension E1: Scope and coverage

Scope defines whether the whole or a part of an asset has been assessed, while coverage defines which part of the data within the scope has been evaluated.

The implied defaults for the core of OFF are `{"scope":"whole"}` and `{"coverage":"complete"}`.

As part of this extension, more specific scopes and coverages can be defined:

#### Scope

Scope may have kind selection and a pinned selector `ArtifactRef`. The selector identifies columns, feature classes, geographic extents, time intervals, raster bands or other subsets. Its format must be understood. Matching remains exact: no containment, union, partition aggregation or subset transfer is inferred.

#### Coverage

Coverage may be set to `sampled` and contain a `samplingPlan`.

### 4.5 Data Asset binding

| Field              | Presence | Content                                                                                                          |
| ------------------ | -------- | ---------------------------------------------------------------------------------------------------------------- |
| `assetId`          | Required | Identity of the logical asset; may be a resolvable catalogue URL.                                                |
| `representationId` | Required | Identity of the representation assessed.                                                                         |
| `snapshotId`       | Required | Identity of the immutable state; may include a version or timestamp, provided the digest also binds its content. |
| `digestKind`       | Required | content for one object; manifest for a collection or snapshot.                                                   |
| `digest`           | Required | SHA-256 digest of the content object or manifest.                                                                |

A report applies only when its asset binding equals all attributes of the evaluation request binding. A label such as latest, a mutable download URL, or an unbound version string is insufficient. Conformity of one distribution MUST NOT be generalized to another representation solely because both share an `assetId`.

The manifest MUST unambiguously enumerate or otherwise bind the constituent content and state, using immutable references and integrity information. The manifest format and its interpretation are defined by the assessment method. A manifest containing only mutable links does not establish an immutable snapshot. Database extracts, large raster collections, tiled datasets, and streaming windows can use this mechanism; a continuously changing service must identify a fixed assessed snapshot or interval.

An evaluator MAY operate without downloading the underlying data when the trust policy permits reliance on reports. The consuming application MUST ensure that the data it subsequently uses match the evaluated binding, either by checking content or manifest integrity or through an accepted immutable delivery mechanism. Matching two metadata strings alone does not establish the identity of subsequently downloaded bytes.

## 5 Fitness profiles

### 5.1 Profile object

| Field                       | Meaning                                                                             |
| --------------------------- | ----------------------------------------------------------------------------------- |
| `specVersion`               | Has the value `0.3-draft`.                                                          |
| `type`                      | Has the value `FitnessProfile`.                                                     |
| `id`                        | Profile identity                                                                    |
| `version`                   | Immutable version of this profile                                                   |
| `name`                      | Readable name                                                                       |
| `purpose`                   | Purpose has an absolute id and description.                                         |
| `requirements`              | Nonempty array of uniquely identified requirements; all mandatory in the core.      |
| `requiredExtensions`        | Optional nonempty array when extensions are needed; otherwise omit.                 |
| `acceptance` and `advisory` | E2 only. Omit both in a core profile; see [Section 5.4](#54-core-conjunction-rule). |
| `classification`            | Optional informational metadata; may identify DQV dimensions/categories.            |

The profile MUST identify its purpose explicitly. The request purpose MUST equal the profile purpose id. A purpose description helps a person select a profile, but has no executable semantics. The profile publisher is responsible for choosing requirements sufficient for its intended purpose. OFF evaluation establishes satisfaction of those requirements, rather than proving that an incomplete profile covers every possible operational risk.

Profile versions MUST be immutable once published. Changes to a threshold, definition, evidence policy, purpose, or acceptance expression require a new version and digest. This draft does not define inheritance or runtime imports. Authors MAY compose profiles with external authoring tools, but the evaluated profile MUST be a self-contained, flattened object with explicit requirements.

### 5.2 Requirements

A requirement contains id, description, statement and condition. statement contains a pinned definition and parameters; condition contains operator and a typed expected value. An optional evidence object narrows the local trust policy. For kind test records, passed and failed supply virtual boolean values true and false for comparison. Measurement records supply their measured value. A requirement’s criterion includes its threshold; it is distinct from the reported measurement’s metric.

For example, the condition below requires the defined completeness statistic to be at least 0.98. It is an excerpt from a requirement; the complete profile additionally pins the definition, parameters, and evidence policy.

```json
"condition": {
  "operator": "gte",
  "expected": {
    "type": "decimal",
    "value": "0.98",
    "unit": "https://www.wetransform.to/off/0.3/units/one"
  }
}
```

The generic evaluator MUST support `eq`, `ne`, `lt`, `lte`, `gt`, and `gte` with the type restrictions in [Section 4](#4-shared-data-types-and-common-requirements). A condition using an operator that is invalid for its expected type makes the profile invalid. When a definition is supported, an expected value whose type or unit is incompatible with that definition is also an invalid profile and MUST produce an evaluation error. A recognized, well-formed definition that the evaluator does not implement produces UNKNOWN for that requirement.

### 5.3 Evidence constraints

| Field                     | Rule                                                                                            |
| ------------------------- | ----------------------------------------------------------------------------------------------- |
| `allowedIssuers`          | Optional nonempty allowlist; local issuer authorization always applies.                         |
| `allowedMethods`          | Optional nonempty list of pinned accepted methods.                                              |
| `maxAssessmentAgeSeconds` | Optional upper bound measured from `assessment.endedAt`.                                        |
| `allowedCoverage`         | Core defaults to complete; selecting sampled requires E1.                                       |
| `evidenceAccess`          | Core defaults to `summary`; `retrievable` requires E3. Replaces the ambiguous field name proof. |

Profile and local trust constraints apply together: issuer and coverage sets intersect; the smaller maximum age applies. Missing profile constraints add no restriction to local policy. Core policy defaults are complete coverage and summary evidence access. E3 retrievable is stricter than summary. An empty intersection yields no admissible evidence, never weakened acceptance.

The same admission criteria MUST apply to favorable and unfavorable values. A profile or implementation MUST NOT discard a negative report simply because a positive report has already been found.

#### 5.3.1 Extension E2: retrievable evidence

If either policy requires `evidenceAccess` retrievable, each candidate assertion must reference evidence and every referenced item must be retrieved and digest-verified. This establishes accessibility and integrity, not substantive correctness. Under core summary acceptance, supplementary evidence can be private or inaccessible without automatically excluding an otherwise admissible record.

### 5.4 Core conjunction rule

<!-- Review comment (Word draft, on 5.4 through 5.4.1): "This part feels underspecified" -->

Every requirement is mandatory: FIT requires all to PASS. Any FAIL makes the profile fail; otherwise UNKNOWN or CONFLICT leaves it unresolved. Requirement IDs are unique local identifiers. A missing condition or empty requirements array makes the profile invalid.

#### 5.4.1 Extension E3: Composition

In addition to `conjunction`, this extension defines two additional modes. . Every requirement occurs in `acceptance` or `advisory`, never both:

1. `acceptance` is a requirement reference, nonempty `allOf` array or nonempty `anyOf` array. `allOf` passes if all children pass and fails if any fails; `anyOf` passes if any passes and fails if all fail. Other combinations are unresolved. UNKNOWN and CONFLICT remain distinct in traces. No weighting, negation, quorum or arbitrary executable expression is defined. At JSON level a reference is `{"requirement":"local-id"}`; composition is `{"allOf":[...]}` or `{"anyOf":[...]}`.
2. `advisory` is an array of requirement IDs. References MUST resolve and arrays MUST be nonempty.

## 6 Fitness reports

### 6.1 Credential structure

A core OFF report is a VC 2.0 credential secured under [Section 8](#8-signatures-and-trust). The decoded JSON payload below is insufficient without its verifiable signature. The issuer authenticates all contained records and evidence references. \[[2](#ref-2), [4](#ref-4)\]

| Field               | Rule                                                                |
| ------------------- | ------------------------------------------------------------------- |
| `@context`          | VC v2 context followed by the fixed inline context specified below. |
| `id`                | Unique HTTPS report identifier; MUST NOT contain a fragment.        |
| `type`              | Array `["VerifiableCredential", "OffFitnessReportCredential"]`.     |
| `issuer`            | HTTPS identity of the responsible issuer.                           |
| `validFrom`         | Inclusive start of validity.                                        |
| `validUntil`        | Optional exclusive end of validity.                                 |
| `credentialSubject` | One OffAssetSnapshot; id equals report body subject.snapshotId.     |
| `credentialStatus`  | Absent in the core; E4 only.                                        |

The report body is `credentialSubject.offReport`. The second `@context` entry is exactly the inline object below; the first is `https://www.w3.org/ns/credentials/v2`. The body is carried as @json so ordinary JSON implementations can process the core. This packaging does not itself generate an EARL or DQV RDF graph. [Section 6.3](#63-earl-assertions-and-dqv-measurements) defines the semantic projection, independently of future RDF serialization tooling. \[[5](#ref-5)\]

```json
{
  "OffFitnessReportCredential":
    "https://www.open-fitness-framework.org/spec/0.3/FitnessReportCredential",
  "OffAssetSnapshot":
    "https://www.open-fitness-framework.org/spec/0.3/AssetSnapshot",
  "offReport": {
    "@id": "https://www.open-fitness-framework.org/spec/0.3/report",
    "@type": "@json"
  }
}
```

### 6.2 Report body and assessment

The report body contains `specVersion`, `subject`, `scope`, `issuedAt`, `assessment`, `coverage`, a nonempty `assertions` array and an `evidence` array (possibly empty). `requiredExtensions` is present when needed. The assertions array holds test or measurement records as distinguished by kind. All share the report’s subject, scope, coverage and assessment; materially different assessments require separate reports.

| Assessment field | Meaning                                                                                                               |
| ---------------- | --------------------------------------------------------------------------------------------------------------------- |
| `id`             | Required execution identity.                                                                                          |
| `operator`       | Required responsible person, organization or service                                                                  |
| `mode`           | Required EARL mode token: `earl:automatic`, `earl:manual`, `earl:semiAuto`, `earl:undisclosed` or `earl:unknownMode`. |
| `method`         | Required pinned procedure, distinct from the criterion or metric definition.                                          |
| `endedAt`        | Required completion or attempt time, including failed attempts.                                                       |
| `issuedAt`       | Required time of the issuance of the credential.                                                                      |
| `tools`          | Required array of tool identities and versions; may be empty for manual assessments.                                  |
| `startedAt`      | Optional E6 provenance: start time.                                                                                   |
| `configuration`  | Optional E6 provenance: pinned material configuration. No empty configuration artifact is required by the core.       |

Automatic and semi-automatic assessments MUST name at least one tool and version. A successful-use claim is defined by its criterion, not a special mode value. The method and definition parameters must capture settings essential to interpretation.

`issuedAt` is report issuance, distinct from assessment time and credential validity. `endedAt` must not exceed `issuedAt`, which must not exceed `evaluatedAt` for admission.

#### 6.2.1 Extension E4: Additional Lifespan Information and Assessment Configuration

E4 adds a separately pinned configuration and richer execution history. Under E6, configuration and provenance may align with PROV-O activities, entities and agents. \[[14](#ref-14), [15](#ref-15)\]

If E4 `startedAt` is present it must not exceed `endedAt`. `validUntil`, when present, must be later than `validFrom`. Issuer timestamps do not imply a trusted timestamp service.

### 6.3 EARL assertions and DQV measurements

Each record has id, kind, definition, parameters, evidence (local evidence IDs) and limitations; details is optional. IDs for records and evidence are unique local tokens matching `[A-Za-z][A-Za-z0-9_-]*`. `definition` is a pinned criterion for kind test or metric for kind measurement. `assertedBy` is an optional absolute URI and points to the credential issuer; it identifies the party responsible for the assertion, not automatically the executing tool. The signed issuer remains accountable for attributing that claim.

| Record form                        | Required representation and interpretation                                                                                                                                                                                               |
| ---------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `kind test`                        | result contains outcome. Use exactly earl:passed, earl:failed, earl:cantTell, earl:inapplicable or earl:untested. No executionStatus or value field. Optional result.reason distinguishes EXECUTION_ERROR and INCONCLUSIVE for cantTell. |
| `kind measurement completed`       | executionStatus is completed and value is one typed value from [Section 4.2](#42-values-and-timestamps). No EARL outcome is assigned merely because measurement succeeded.                                                               |
| `kind measurement without a value` | executionStatus is error, inconclusive, notExecuted or inapplicable. value is absent. This is an assessment attempt, not a completed DQV QualityMeasurement.                                                                             |

For a test, `earl:passed` and `earl:failed` state whether the named criterion is satisfied and supply virtual `true` and `false` values for OFF comparisons. The other EARL outcomes supply no decisive value. A crash is `earl:cantTell` with reason `EXECUTION_ERROR`, never `earl:failed`. `earl:inapplicable` does not waive a profile requirement; `earl:untested` does not mean failure. Numerical quality measurements have no intrinsic pass/fail threshold. \[[15](#ref-15)\]

Every evidence reference must resolve within its report. Completed measurements must satisfy the metric’s type, unit, range, parameters and required details. Test outcomes must satisfy their criterion’s semantics. Inconsistent fields exclude the affected record; structural or signature invalidity excludes the report. Unknown definition semantics never authorize an inferred result.

Free-text limitations are disclosed for review, but the generic evaluator does not interpret them. Any limitation that changes whether the assertion applies MUST also be represented by the binding, scope, coverage, parameters, status, or a defined machine-readable field. A producer MUST NOT hide a material qualification only in prose while presenting a stronger machine assertion.

#### 6.3.1 Normative semantic mapping

The following mappings define the meaning of the compact JSON fields. They do not require RDF processing. In a semantic export, record identity is report id plus \#assertion- plus local id; result identity appends -result. A qualified criterion or metric uses report id plus \#criterion- or \#metric- plus local id, and retains the pinned definition and parameters through OFF properties. Distinct parameterizations MUST NOT be collapsed into one unqualified criterion or metric.

| OFF concept                   | Established semantics                                                                                                                                                                            |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Test record                   | earl:Assertion; assertor and subject have types earl:Assertor and earl:TestSubject; assertedBy → earl:assertedBy; subject → earl:subject; qualified criterion → earl:test; result → earl:result. |
| Test result                   | earl:TestResult; result.outcome → earl:outcome; assessment.endedAt → dct:date; assessment.mode → earl:mode on the assertion.                                                                     |
| Completed measurement         | dqv:QualityMeasurement; qualified metric → dqv:isMeasurementOf; subject → dqv:computedOn; typed value → dqv:value.                                                                               |
| Definition and classification | Criterion → earl:TestCriterion; quality metric → dqv:Metric; metric dimension → dqv:inDimension and dqv:Dimension; dimension category → dqv:inCategory and dqv:Category.                         |
| Decision requirement result   | An evaluator-generated earl:Assertion may describe the requirement check; the evaluator is assertor and the complete requirement is its criterion. Original evidence is retained separately.     |

For `whole` scope, the semantic subject is the snapshot URI. Extension E1 selections require a distinct scoped-subject URI bound to snapshot and selector digest; subset results MUST NOT be attributed to the whole snapshot.

JSON boolean, decimal, integer, string and iri values map respectively to xsd:boolean, xsd:decimal, xsd:integer, xsd:string and xsd:anyURI literals. Units map to `sdmx-attribute:unitMeasure`, as illustrated by DQV; comparisons still require exact unit identity. Failed attempts MUST NOT become zero-valued or failed DQV measurements. \[[12](#ref-12)\]

### 6.4 Supporting assessment evidence

Each evidence item contains a local id, a pinned `resource`, a retrieval `href`, a `mediaType`, and a role of `log`, `result`, `reference`, `method`, or `other`. It MAY include a description. The signed report protects these references and their digests. The link itself is not a guarantee of availability or content integrity.

The evidence resource digest MUST identify the intended artifact bytes. The href may be a retrieval location different from the resource id. An authenticated mirror or local package may supply identical bytes. URL credentials and expiring bearer tokens SHOULD NOT be embedded in reports; an authorized retrieval layer can provide access separately.

Evidence may include validator output, sampling plans, calibration records, reference comparisons or workflow results. Evidence is optional under core summary acceptance. Required retrieval belongs to Extension E3. A report remains interpretable without parsing an untyped log. Cryptographic proof authenticates a report; supporting assessment evidence substantiates or explains its claims. Neither term implies the other.

## 7 Assertion definitions and domain semantics

### 7.1 Definition contract

Each pinned definition declares kind test or measurement, immutable identity/version, parameter structure, property or event assessed, interpretation and indeterminate cases. A test definition specifies criterion satisfaction and failure. A measurement definition specifies value type, unit, range, required details and coverage interpretation, and aligns to `dqv:Metric`. Semantically relevant dependencies must be immutable. Dimension/category annotations may reuse DQV but do not change matching or thresholds. Numeric metric types may be declared with `dqv:expectedDataType`. A definition’s semantic type is fixed: a criterion cannot silently become a measurement or vice versa.

The generic evaluator uses exact definition and parameter matching. It MUST NOT infer that two definitions, schemas, purposes, methods, or units are equivalent from similar names. Implementations MUST declare which definitions they understand. Domain plug-ins can implement further definitions, but downloading a definition MUST NOT execute code supplied by that resource.

Sections [7.2](#72-schema-conformity-pattern) and [7.3](#73-completeness-measurement-pattern) give definition patterns (informative examples; the adapter safeguards are normative); their concrete schemas and missing-value conventions are chosen by domain publishers. Core conformance requires the record model and declared definition support, not a universal catalogue of metrics. Sections [7.4](#74-extension-e5-accuracy-and-precision) and [7.5](#75-extension-e6-successful-use-and-attestations) are optional domain extension E7. None of these patterns constitutes a released companion definition.

### 7.2 Schema conformity pattern

A schema-conformity test identifies a pinned schema and validation dialect through its criterion parameters and method. Its EARL outcome states whether the snapshot conforms under that procedure. earl:passed is meaningful only with that criterion; a successful validator execution alone is insufficient.

The procedure MUST define how warnings, unsupported features, references, and validator failures affect the assertion. An adapter for SHACL, JSON Schema, or another validation system MUST preserve that system's declared result semantics. For example, the SHACL specification distinguishes validation failure from ordinary validation results; an execution failure must not be rewritten as demonstrated data nonconformity. \[[6](#ref-6)\]

The VC `credentialSchema` concept concerns the credential's structure. A data asset's target schema belongs in OFF assertion parameters. Confusing these two checks could produce a valid report document that says nothing about whether the data conform.

### 7.3 Completeness measurement pattern

Completeness requires an explicitly identified population and missing-value rule. Attribute completeness, record coverage against a reference inventory, spatial coverage, temporal continuity, and raster no-data coverage are different quantities. A definition MUST identify the denominator, numerator, duplicate handling, excluded records, and behavior when the denominator is zero.

In the illustrative height-completeness definition, the denominator is all eligible records and the numerator is records with present, nonnull height. The record is kind measurement with a decimal ratio and a dimensionless unit. Counts in details are integer strings. For this example only, the ratio is truncated downward to 12 decimal places; a zero denominator is inconclusive and produces no measurement value. An implementation declaring support for this definition must check consistency of value and counts.

This explicit rounding rule makes the example interoperable and conservative at a minimum threshold. It is a proposal for this example definition, not a universal completeness rule. A definition for inventory completeness would additionally need a pinned inventory and matching procedure. A profile MUST NOT silently substitute one of these definitions for another.

### 7.4 Extension E5: Accuracy and precision

Accuracy, precision, resolution, and uncertainty MUST be distinguished by their definitions. A one-meter grid spacing is not a measurement of one-meter positional accuracy. An accuracy definition should identify the quantity, reference system, reference data, statistic, assessment population, sampling method, and uncertainty as applicable.

The generic comparator can compare RMSE, percentile errors or confidence bounds once an implemented domain definition specifies the quantity precisely. They are distinct metrics; a point estimate cannot replace a requested confidence bound. ISO 19157 may supply geospatial measure semantics. No universal uncertainty propagation, accuracy metric or statistical extrapolation is required by the core. \[[16](#ref-16)\]

### 7.5 Extension E6: Successful use and attestations

A `successful-use` test identifies workflow/version, relevant context, asset role and success criterion through pinned parameters or method. `earl:passed` means that criterion was met in that execution; normal application termination is not automatically domain success. The EARL assertion model is unchanged.

Successful use may satisfy a requirement that explicitly asks for that use. It MUST NOT automatically satisfy a schema, completeness, or accuracy requirement. Transfer to another workflow or operating context requires a separate supported definition that describes the compatibility claim. Textual similarity of purposes is insufficient.

Extension E6 also covers domain-specific expert opinions and certificates. Their definitions must be machine-interpretable, with manual provenance when appropriate. Acceptability of self-attestation or independent review is determined by the profile and local policy. Domain conformance claims must name the implemented definitions, not merely “E6”.

## 8 Signatures and trust

### 8.1 Baseline signature binding

Report issuers and evaluators MUST support VC Data Model 2.0 with direct JWS Compact Serialization under Securing Verifiable Credentials using JOSE and COSE. This draft pins the 15 May 2025 VC Recommendation and the 13 October 2026 JOSE and COSE Recommendation. Other securing mechanisms require a separately defined binding and do not establish baseline conformance here. \[[2](#ref-2), [4](#ref-4)\]

The complete unsecured report credential is the JWS payload. It MUST NOT be placed inside a legacy `vc` wrapper claim. The baseline uses the normal base64url-encoded payload and has no detached or unencoded-payload mode. The media type is application/vc+jwt. The protected header has exactly these members:

```json
{
  "alg": "ES256",
  "typ": "vc+jwt",
  "cty": "vc",
  "kid": "https://validator.example/issuer#key-1"
}
```

ES256 uses the P-256 curve and SHA-256. Its JWS signature encoding is the 64-byte concatenation of the 32-byte R and S values. Issuers and verifiers MUST use the JOSE encoding, not an ASN.1 DER signature. These restrictions narrow the underlying standards to one interoperable OFF baseline. \[[7](#ref-7), [8](#ref-8)\]

The verifier MUST verify the original protected-header and payload bytes. It MUST NOT reformat or canonicalize the JSON before signature verification. It MUST reject `alg none`, HMAC algorithms, other unnegotiated algorithms, unknown header members, malformed encodings, invalid key types, duplicate JSON members, and payloads outside the fixed OFF credential shape. The baseline has no embedded proof member and omits registered JWT claims such as iss, iat, exp, and nbf; OFF and VC time fields are evaluated explicitly.

An implementation MUST use an application-selected algorithm `allowlist` and a key authenticated for the issuer. A key delivered by the report or an arbitrary token URL MUST NOT become trusted merely because it verifies the signature. These requirements are consistent with JWT security guidance on algorithm and issuer validation. \[[9](#ref-9)\]

### 8.2 Local trust policy

A core TrustPolicy has `specVersion`, type `TrustPolicy`, `id`, `version` and `trustedIssuers`. Optional `maxAssessmentAgeSeconds` and `allowedMethods` narrow admission; issuer acceptance is defined by `trustedIssuers`. Core coverage and evidence access default to complete and summary. `allowNoCredentialStatus` MUST be true in a core policy, explicitly acknowledging no revocation check. Extension settings require `requiredExtensions` declarations. The application must authorize the pinned policy independently of report content.

Each trusted issuer entry names an HTTPS issuer identity, one or more accepted key identifiers with ES256 P-256 public JWKs, and an explicit set of allowed assertion-definition references. Empty or missing authorization does not mean universal trust. The evaluator MUST match `issuer`, key `id`, `algorithm`, `key`, and `assertion` definition. This baseline uses pinned keys; live identity-resolution and key-rotation services can prepare a new authorized policy snapshot.

Key authorization and claim acceptance are separate. An organization may control a key while lacking accepted authority for a particular kind of measurement. A profile's issuer restrictions further narrow the locally accepted issuers. Identifiers representing organizations under common control MUST NOT be described as independent merely because their strings differ; independence assessment is outside the baseline's one-observation admission model.

Profiles and policies SHOULD be distributed through authenticated channels or signed publication mechanisms. A signed report cannot protect an evaluator from an attacker replacing its profile with weaker requirements. Policy updates and key removal require a new policy snapshot and reevaluation for decisions intended for current use.

### 8.3 Extension E7: Credential status

The supported status mechanism is Bitstring Status List 1.0. A status-bearing report declares E7 and contains one `BitstringStatusListEntry` for revocation or suspension. An evaluator without E7 excludes such a report with UNSUPPORTED_EXTENSION rather than ignoring status. Under E7 the status-list issuer is restricted to the report issuer. \[[10](#ref-10)\]

E7 evaluators obtain and authenticate the status-list credential, check its authorized issuer, validity, purpose and index, and evaluate the bit. Status lists use the [Section 8.1](#81-baseline-signature-binding) direct ES256 JWS format but the `BitstringStatusListCredential` payload model. Revoked or suspended reports are inadmissible. Unavailable, stale, invalid or unsupported status is unresolved and never treated as clear.

Under E7, the list contains `validFrom` as publication time and a later `validUntil`, and must be valid at `evaluatedAt`. Both signed publication age and trusted observation age must satisfy the policy’s required `maxStatusAgeSeconds`; observation time cannot be in the future. Fetching an old list today cannot make it fresh. Retain the list and acquisition record; E5 adds portable replay packaging.

E7 policies declare `allowNoCredentialStatus`; when false, reports without a status entry are excluded. Core policies permit absent status and decisions record that no revocation channel was checked. Expiry and assessment freshness still apply independently. Credential-status support is not a prerequisite for core conformance.

## 9 Evaluation requests

A core `EvaluationRequest` contains `specVersion`, type `EvaluationRequest`, `id`, `subject`, `scope`, `purpose`, `profile`, `trustPolicy`, `evaluatedAt` and `reportResources`. Profile, policy and secured JWS report bytes are pinned by ArtifactRef; `reportResources` may be empty. Optional `requiredExtensions` declares extension needs. `context` and `verificationResources` are absent unless the applicable extensions are used. A core evaluator may operate entirely on supplied local artifacts.

### 9.1 Extension E8: Context

An added `context` document describes the operating situation for audit. Any context affecting admission must also appear in exact parameters of a supported definition; free-text similarity does not establish applicability.

### 9.2 Extension E9: Portable replay with verification resources

The goal of this extension is to enable a later evaluator to replay the evaluation, e.g. for scientific purposes. The extension adds a schema for the resources needed to replay an evaluation.

`verificationResources` pins external status evidence, retrieval observations and any other externally acquired material used. The evaluator records actual inputs and can assemble the frozen request after acquisition. The core retains used inputs and admission reasons locally, but does not require this portable acquisition-record format.

## 10 Deterministic evaluation procedure

### 10.1 Validate and resolve inputs

The evaluator validates the request, declared extensions, pinned profile and authorized policy, including digests and semantic constraints. Invalid required input, unsupported request/profile/policy extensions, unresolved profile/policy or an unestablished subject produces `evaluationStatus` error and result null. These are evaluation errors, not asset failures.

The evaluator retains the supplied report inventory. Order must not affect the result; identical reports do not count as additional evidence. Resource-limit termination produces an evaluation error, never FIT from partial processing.

### 10.2 Authenticate reports and check identity

For each candidate, verify artifact digest, JWS form, signature, credential structure and issuer-key binding. Check declared extension support, subject, representation, snapshot/digest, scope, timestamps and credential validity. Any failed check excludes the report with a reason.

Apply status checks only under E4; a status-bearing report without supported E4 is excluded.

The credential validity interval is half-open: `validFrom` is inclusive and `validUntil`, when present, is exclusive. Assessment freshness is calculated as `evaluatedAt` minus `assessment.endedAt` and is accepted when that nonnegative age is less than or equal to the effective maximum. Reissuing yesterday's assessment today does not make the assessment newer. Data acquisition age is a separate property and requires a separate assertion if the purpose depends on it.

A source may provide several signatures over identical logical report content. After successful authentication, equivalent payloads with the same issuer and report id MAY be deduplicated using JCS equality. If the same issuer and report id identify different authenticated payloads, the evaluator MUST quarantine both variants with REPORT_ID_EQUIVOCATION. A forged or invalidly signed variant MUST NOT poison an otherwise valid report. A new correction MUST use a new report id.

### 10.3 Select and admit records

For each requirement, collect all records with exactly matching definition references and parameters. Check implemented semantics, issuer authorization, method, complete coverage (or E1), assessment age, record validity and `evidenceAccess`. For tests, compare virtual true for earl:passed or false for earl:failed to the boolean condition. For completed measurements, compare the typed value. All other outcomes or execution statuses are nondecisive.

Mismatching definition, method, scope, schema version, parameters or snapshot supplies no evidence for that requirement. No implicit compatibility is inferred. earl:cantTell, earl:inapplicable and earl:untested produce distinct trace reasons but no decisive value; they cannot turn an unmet mandatory requirement into a pass. Admission must consider all relevant reports, including negative ones.

Invalid signatures, untrusted issuers, stale evidence, failed executions and unavailable required evidence are evidence problems, not demonstrated asset defects. An authentic, applicable, admitted measurement below a minimum threshold establishes a negative requirement observation. Successful measurement execution alone never establishes threshold satisfaction.

### 10.4 Determine requirement outcomes

| Admitted decisive observations                            | Requirement outcome |
| --------------------------------------------------------- | ------------------- |
| At least one satisfies the condition and none violates it | PASS                |
| At least one violates the condition and none satisfies it | FAIL                |
| No decisive observation                                   | UNKNOWN             |
| At least one satisfies and at least one violates          | CONFLICT            |

PASS, FAIL, UNKNOWN and CONFLICT are OFF requirement aggregation states, not replacement EARL assertion outcomes. For semantic export, PASS maps to earl:passed, FAIL to earl:failed, and UNKNOWN or CONFLICT to earl:cantTell while retaining the OFF state and reasons. Conflicting tests or measurements remain individually intact. Measurements that differ but both satisfy the predicate do not create a requirement conflict.

Newer evidence does not automatically displace older contrary evidence. The core defines no newest-wins, issuer ranking, majority vote or implicit supersession. A consistency requirement needs its own defined metric or criterion.

### 10.5 Aggregate the decision

In the core, all requirements are mandatory: all PASS yields FIT; any FAIL yields NOT_FIT; otherwise the result is NOT_ASSESSABLE. Under E5, an otherwise passing root becomes NOT_ASSESSABLE if a required discovery source is incomplete; a failing root remains NOT_FIT. A core submitted-only evaluation has no discovery prerequisites.

A mandatory failure can establish NOT_FIT even if another condition is unknown. All evaluated leaf outcomes, including conflicts, remain visible; an extension must not hide evidence that did not determine the root result.

FIT means that the declared requirements are supported within the specified admissible evidence set and policy. It does not claim that no contradictory evidence exists elsewhere, that the profile is complete for every real-world hazard, or that future data revisions will remain fit.

## 11 Fitness decisions

A `FitnessDecision` contains `specVersion`, type `FitnessDecision`, id, a pinned request reference, evaluator identity/version, `decidedAt`, `evaluationStatus`, `result`, `requirementResults`, `errors` and `limitations`. Each requirement result identifies its requirement `id`, `outcome`, `acceptedTrace` and `excludedTrace`. The request binds core inputs and any declared extensions. `decidedAt` must not precede `evaluatedAt`. No separate portable acquisition bundle is required by the core.

| Field or state             | Required behavior                                                                                                                                           |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| evaluationStatus completed | result is FIT, NOT_FIT, or NOT_ASSESSABLE; errors is empty.                                                                                                 |
| evaluationStatus error     | result is null; errors contains at least one machine code and message.                                                                                      |
| requirementResults         | For completed evaluations, one result for every defined requirement, including advisory ones. Error responses may retain partial results or an empty array. |
| acceptedTrace              | Each decisive report reference, assertion id, and its PASS or FAIL predicate result.                                                                        |
| excludedTrace              | Each relevant excluded report or assertion, with machine-readable reasons.                                                                                  |
| limitations                | Material bounds of the decision, such as a submitted-only evidence set or absent revocation channels.                                                       |

Every input report MUST be accounted for as accepted, excluded, duplicate, or irrelevant in the retained evaluation record. Reports unrelated to a particular requirement need not be repeated in every requirement trace; a report rejected before matching can be recorded against affected requirements. The retained request inventory prevents omitted candidates from being mistaken for an exhaustive search.

Reason codes carry machine-readable categories; explanations may add detail. Core implementations support applicable core codes below. E-prefixed rows apply only to the named extensions. Extensions may refine but cannot contradict core meanings.

| Code family         | Examples                                                                                                            |
| ------------------- | ------------------------------------------------------------------------------------------------------------------- |
| Core input          | INVALID_REQUEST, INVALID_PROFILE, POLICY_UNAVAILABLE, UNSUPPORTED_DEFINITION, UNSUPPORTED_EXTENSION, RESOURCE_LIMIT |
| Core authentication | INVALID_SIGNATURE, UNTRUSTED_ISSUER, UNAUTHORIZED_DEFINITION, REPORT_ID_EQUIVOCATION                                |
| Core applicability  | SUBJECT_MISMATCH, SCOPE_MISMATCH, DEFINITION_MISMATCH, PARAMETERS_MISMATCH, METHOD_NOT_ALLOWED                      |
| Core time           | INVALID_TIME, NOT_YET_VALID, EXPIRED, STALE_ASSESSMENT                                                              |
| Core observation    | EXECUTION_ERROR, INCONCLUSIVE, NOT_EXECUTED, INAPPLICABLE, INVALID_VALUE, UNIT_MISMATCH, DIGEST_MISMATCH            |
| Core outcome        | NO_ADMISSIBLE_EVIDENCE, PREDICATE_SATISFIED, PREDICATE_VIOLATED, CONFLICTING_EVIDENCE                               |
| E1 and E2           | COVERAGE_NOT_ALLOWED; MISSING_EVIDENCE, EVIDENCE_UNAVAILABLE                                                        |
| E4 and E7           | REVOKED, SUSPENDED, STATUS_UNAVAILABLE, STALE_STATUS, STATUS_REQUIRED; REQUIRED_SOURCE_INCOMPLETE                   |

A decision MAY be signed for transport or audit. This draft does not define recursive use of a FitnessDecision as an atomic fitness report. An evaluator MUST NOT accept a previous FIT label as a substitute for the underlying requirements unless a future supported assertion definition explicitly specifies that reuse, dependencies, expiration, and cycle handling.

## 12 Discovery and lifecycle

Reports may be published beside assets, linked from catalogs or returned by APIs. No central registry is required. Discovery yields candidates; the signed binding determines applicability. DCAT can describe discovery links without altering OFF admission rules. \[[13](#ref-13)\]

Repositories preserve secured bytes and must not rewrite payloads under a signature. They should expose asset, snapshot, definition and issuer metadata. Unsigned catalogue claims of validity or fitness are not OFF reports. Core evaluation only requires directly supplied reports.

Changed assets need a new snapshot binding; changed profiles or policies need new pinned inputs and reevaluation. New evidence can change a decision without rewriting historical decisions. Under E4, status changes may also trigger reevaluation. Automated monitoring and scheduling are outside core conformance.

Optional provenance and domain extensions may record lineage. Lineage alone must not transfer conformity, completeness, accuracy or fitness to a transformed asset. Property-preservation claims require separately defined, supported assertions about the output binding; no such inference is included in the core.

Existing validators, report issuance, evidence collection, evaluation, and reassessment workflows can be wrapped as OFF report producers. Report producers remain free to use other implementations and to run assessments close to protected data, exposing only the authorized signed summaries and evidence references.

## 13 Privacy and implementation controls

Reports, profiles, context documents, and evidence links may reveal sensitive attributes, geographic coverage, operational activity, or data defects. Issuers SHOULD minimize disclosed details and use access-controlled evidence stores when appropriate. Signature verification does not grant permission to retrieve the asset or its supporting evidence. Existing access and usage policies continue to apply.

Evaluators MUST treat retrieved reports, descriptions, logs, and linked documents as untrusted input. Text inside evidence MUST NOT change system instructions, authorize tool execution, select a weaker trust policy, or trigger arbitrary commands. OFF's declarative comparison language does not require running report-supplied code.

Network retrieval MUST use configured scheme, destination, redirect, size, and timeout controls. A resolver MUST NOT bypass access controls to obtain supporting evidence. File and network parsers MUST enforce resource limits and validate expected formats. Failure to fetch a required resource is diagnosed according to its role; it MUST NOT be converted to acceptance.

Implementations MUST keep provenance and security outcomes distinct in explanations. A valid signature is not independent corroboration, a tool version is not evidence of tool correctness, a digest is not proof of truth, and accessible evidence is not necessarily substantively verified evidence. These distinctions determine which assertion the application is entitled to rely upon.

## 14 Alignment with existing specifications

EARL and DQV semantics are normative as profiled in [Section 6.3](#63-earl-assertions-and-dqv-measurements); their full vocabularies are not mandatory implementation dependencies. The following table identifies core dependencies and optional alignments. OFF adds the contract for admissible evidence and purpose-specific decisions.

| Specification                                     | OFF relationship and limit                                                                                                                         |
| ------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| EARL 1.0 Schema \[[15](#ref-15)\]                 | Core test assertion semantics, five outcomes and modes. Working Group Note, not a Recommendation. Compact JSON does not itself create RDF triples. |
| DQV \[[12](#ref-12)\]                             | Core metric/measurement distinction and semantic mapping. Working Group Note. A measurement is not a threshold decision.                           |
| VC 2.0; VC JOSE/COSE \[[2](#ref-2), [4](#ref-4)\] | Core signed envelope and ES256 binding. Credential status is E4.                                                                                   |
| JSON Schema; SHACL \[[11](#ref-11), [6](#ref-6)\] | Optional structural validation tooling and assessment adapters. No engine is mandated; future artifacts must match the prose.                      |
| DCAT 3 \[[13](#ref-13)\]                          | Optional E5 discovery/catalogue mapping. Catalog labels do not replace immutable binding.                                                          |
| PROV-O \[[14](#ref-14)\]                          | Optional E6 execution and derivation graphs. Core provenance fields remain sufficient without an RDF graph.                                        |
| ISO 19157-1 \[[16](#ref-16)\]                     | Optional E7 geospatial measure definitions. No complete ISO crosswalk or mandatory catalogue is claimed.                                           |
| OSTrails FTR \[[17](#ref-17)\]                    | Informative mapping target for metrics, tests, executions and benchmarks; not a core dependency.                                                   |

Domain publishers should reuse established metric identifiers and meanings where they match, including ISO 19157 definitions. Parameterized OFF metrics retain their precise measurement basis and scope; similar names do not establish equivalence. FTR integration should map metrics, tests and executions explicitly rather than treating a benchmark score as an OFF fitness decision.

A complete RDF/JSON-LD serialization, richer PROV graphs, ODCS requirement import and Gaia-X trust-registry integration are future optional bindings, not core prerequisites. EARL/DQV semantic alignment does not claim a tested lossless conversion. If future exports are implemented, they must preserve the [Section 6.3](#63-earl-assertions-and-dqv-measurements) mappings, typed values, scope, attribution and OFF-specific admission/decision metadata.

## 15 Conformance cases and illustrative example

### 15.1 Required evaluator behavior

Core cases are mandatory for core evaluators. Unless stated otherwise, the case has one mandatory requirement, valid profile/policy, whole scope, complete coverage and trusted signed evidence. Lack of external discovery is not a core error.

| Case and scope                                                            | Expected outcome                                                  |
| ------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Core: No matching report                                                  | Requirement UNKNOWN; NOT_ASSESSABLE.                              |
| Core: Admitted value 0.987 with minimum 0.98                              | PASS; FIT.                                                        |
| Core: Admitted value 0.987 with minimum 0.995                             | FAIL; NOT_FIT.                                                    |
| Core: Value 0.979999 displayed as 98 percent                              | FAIL against gte 0.98.                                            |
| Core: Validator crash                                                     | UNKNOWN with EXECUTION_ERROR                                      |
| Core: Old snapshot or different representation                            | Exclude with SUBJECT_MISMATCH.                                    |
| Core: Schema version differs                                              | No implicit schema compatibility; exclude.                        |
| Core: Valid trusted pass and valid trusted fail                           | CONFLICT; NOT_ASSESSABLE.                                         |
| Core: Valid trusted pass and invalidly signed fail                        | PASS; record rejection of the invalid report.                     |
| Core: Only an untrusted issuer reports pass                               | UNKNOWN.                                                          |
| Core: Old assessment reissued today                                       | Apply age to assessment end; exclude if stale.                    |
| Core: Supplementary supporting evidence inaccessible under summary policy | No automatic rejection solely for inaccessibility.                |
| Core: evaluatedAt equals validUntil                                       | Exclude as EXPIRED.                                               |
| Core: Age equals the allowed maximum                                      | Freshness condition passes.                                       |
| Core: Mandatory set contains FAIL and UNKNOWN                             | NOT_FIT.                                                          |
| Core: Reordered or identical duplicated reports                           | Same semantic outcome.                                            |
| Core: Same issuer and id with different authenticated payloads            | Quarantine variants; REPORT_ID_EQUIVOCATION.                      |
| Core: Unsupported assertion definition                                    | UNKNOWN; UNSUPPORTED_DEFINITION.                                  |
| Core: Invalid acceptance reference or condition type                      | Evaluation error; result null.                                    |
| Core: earl:inapplicable or earl:untested                                  | UNKNOWN with distinct reason; no automatic waiver or failure.     |
| Core: completed measurement without EARL outcome                          | Compare value to threshold; success of measurement is not a pass. |
| Core: unsupported extension in profile or policy                          | Evaluation error, result null; UNSUPPORTED_EXTENSION.             |
| Core: measurement execution failure                                       | UNKNOWN; do not emit a zero-valued DQV measurement.               |

### 15.2 Illustrative example and companion status

Consider a synthetic snapshot of 1000 building records: height is present and `nonnull` for 987 and null for 13. A schema permitting null height may still be satisfied. Schema conformity is therefore a kind test record with earl:passed, while height completeness is a separate kind measurement record with decimal value 0.987.

A core profile requiring conformity eq true and completeness gte 0.98 yields FIT if both records are admitted and no contradictory admitted evidence exists. Changing the threshold to 0.995 yields NOT_FIT without changing the measurement. A successful-use record requires its own E7 definition; making that requirement advisory additionally requires E2.

This example illustrates the semantics and is not a signed test vector or certification claim.

## TODOs

**Domain extensions:**

- Prioritize definitions for the data uses to be supported first, including raster coverage, spatial accuracy, semantic conformity, and successful use.
- Each extension must define its scope, measurement basis, units, uncertainty, and evidence conditions. Scope transfer and derived evidence need separate rules before they can safely broaden reuse.

**Conformance testing:**

- Build at least two independent producers and evaluators against the same positive and adversarial cases. Validate stable decisions under reordered reports, conflicting observations, missing evidence, time boundaries, and policy changes.

**Publication and implementation:**

- Agree the document license independently of OFE’s EUPL implementation license. Stabilize the core field model, EARL/DQV mapping and extension declarations before producing new companion schemas, contexts, definitions and test vectors.
- A proof of concept should first demonstrate the full asset-to-report-to-decision cycle for one structured-data and one geospatial or raster use case.

## References

<a id="ref-1"></a>\[1\] IETF. RFC 2119 and RFC 8174, BCP 14, requirement language. <https://www.rfc-editor.org/rfc/rfc2119> and <https://www.rfc-editor.org/rfc/rfc8174>

<a id="ref-2"></a>\[2\] W3C. Verifiable Credentials Data Model v2.0. Recommendation, 15 May 2025. <https://www.w3.org/TR/2025/REC-vc-data-model-2.0-20250515/>

<a id="ref-3"></a>\[3\] IETF Independent Submission. RFC 8785, JSON Canonicalization Scheme. Informational, June 2020. <https://www.rfc-editor.org/rfc/rfc8785.html>

<a id="ref-4"></a>\[4\] W3C. Securing Verifiable Credentials using JOSE and COSE. Recommendation, 13 October 2026. <https://www.w3.org/TR/2026/REC-vc-jose-cose-20261013/>

<a id="ref-5"></a>\[5\] W3C. JSON-LD 1.1. Recommendation, 16 July 2020, JSON literals. <https://www.w3.org/TR/json-ld11/#json-literals>

<a id="ref-6"></a>\[6\] W3C. Shapes Constraint Language. Recommendation, 20 July 2017. <https://www.w3.org/TR/2017/REC-shacl-20170720/>

<a id="ref-7"></a>\[7\] IETF. RFC 7515, JSON Web Signature. Standards Track, May 2015. <https://www.rfc-editor.org/rfc/rfc7515>

<a id="ref-8"></a>\[8\] IETF. RFC 7518, JSON Web Algorithms, Section 3.4. Standards Track, May 2015. <https://www.rfc-editor.org/rfc/rfc7518.html#section-3.4>

<a id="ref-9"></a>\[9\] IETF. RFC 8725, JSON Web Token Best Current Practices. BCP 225, February 2020. <https://www.rfc-editor.org/rfc/rfc8725.html>

<a id="ref-10"></a>\[10\] W3C. Bitstring Status List v1.0. Recommendation, 15 May 2025. <https://www.w3.org/TR/2025/REC-vc-bitstring-status-list-20250515/>

<a id="ref-11"></a>\[11\] JSON Schema. Draft 2020-12 core specification and validation vocabularies. <https://json-schema.org/draft/2020-12>

<a id="ref-12"></a>\[12\] W3C. Data on the Web Best Practices Data Quality Vocabulary. Working Group Note, 15 December 2016. <https://www.w3.org/TR/2016/NOTE-vocab-dqv-20161215/>

<a id="ref-13"></a>\[13\] W3C. Data Catalog Vocabulary Version 3. Recommendation, 22 August 2024. <https://www.w3.org/TR/2024/REC-vocab-dcat-3-20240822/>

<a id="ref-14"></a>\[14\] W3C. PROV O The PROV Ontology. Recommendation, 30 April 2013. <https://www.w3.org/TR/2013/REC-prov-o-20130430/>

<a id="ref-15"></a>\[15\] W3C. Evaluation and Report Language (EARL) 1.0 Schema. Working Group Note, 2 February 2017. <https://www.w3.org/TR/2017/NOTE-EARL10-Schema-20170202/>

<a id="ref-16"></a>\[16\] ISO. ISO 19157-1:2023 Geographic information — Data quality — Part 1 General requirements. <https://www.iso.org/standard/78900.html>

<a id="ref-17"></a>\[17\] OSTrails. FAIR Testing Resource Vocabulary (FTR). Evolving community specification. <https://github.com/OSTrails/FAIR_testing_resource_vocabulary>
