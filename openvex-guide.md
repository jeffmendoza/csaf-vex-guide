# OpenVEX Overview

OpenVEX is a lightweight format for publishing Vulnerability Exploitability eXchange (VEX) information. Like CSAF VEX, it states whether a product is affected by a vulnerability and provides the information required to interpret that status.

OpenVEX organizes this information as a list of statements. Each statement connects:

```text
product + vulnerability + status
```

The document and each statement also have a time context. This allows later statements or document versions to update earlier assessments as more information becomes available.

## Document structure

An OpenVEX document contains document metadata and one or more statements:

```json
{
  "@context": "https://openvex.dev/ns/v0.2.0",
  "@id": "https://example.com/security/openvex/component-data-service",
  "author": "Example Corporation",
  "timestamp": "2026-09-29T16:00:00Z",
  "version": 1,
  "statements": []
}
```

The main document fields are:

| Field | Purpose |
|---|---|
| `@context` | Identifies the OpenVEX JSON-LD context and version. |
| `@id` | A globally addressable IRI identifying the document. |
| `author` | The individual or organization making the statements. |
| `timestamp` | The time the document was issued. Statements inherit this timestamp unless they provide their own. |
| `version` | The document version. Increment it whenever the document content changes. |
| `statements` | The VEX assertions contained in the document. |

## Statements

Each statement identifies a vulnerability, one or more products, and their status:

```json
{
  "vulnerability": {
    "name": "CVE-2026-12345"
  },
  "products": [
    {
      "@id": "pkg:nuget/Example.ComponentDataService@1.0.900-beta"
    }
  ],
  "status": "affected",
  "action_statement": "Upgrade when a fixed version becomes available."
}
```

A complete statement needs:

- A product.
- A vulnerability.
- A status.
- A timestamp, either on the statement or inherited from the document.

A statement may apply to several products when the same vulnerability assessment applies to all of them.

## Products

Products are the subjects of OpenVEX statements. Identify the exact product and version being assessed so a consumer can correlate the statement with an SBOM or asset record.

Package URLs are valid OpenVEX identifiers and are recommended:

```json
"products": [
  {
    "@id": "pkg:nuget/Example.ComponentDataService@1.0.900-beta",
    "identifiers": {
      "purl": "pkg:nuget/Example.ComponentDataService@1.0.900-beta"
    }
  }
]
```

The product `@id` is an IRI that identifies and makes the product referenceable. A Package URL is a convenient choice because it can serve as both the `@id` and the `purl` identifier.

OpenVEX can also identify products with:

- CPE 2.2 or CPE 2.3 identifiers.
- Cryptographic hashes.
- Subcomponents representing dependencies or other software contained in the product.

Use identifiers that describe the product as precisely as possible. The product must be identifiable even though each individual identification field is optional in the data structure.

## Vulnerabilities

The `vulnerability` object identifies the vulnerability addressed by the statement. Only `name` is required:

```json
"vulnerability": {
  "name": "CVE-2026-12345"
}
```

Use a public, globally recognized identifier such as a CVE when one exists. Other public or private identifiers may be used when they are understood by all consumers of the document.

OpenVEX can also include:

- `@id`: an IRI linking to the vulnerability.
- `description`: a human-readable description.
- `aliases`: other identifiers for the same vulnerability, such as a GHSA.

These fields are optional. The main identifier belongs in `name`.

## Status

OpenVEX defines four statuses:

| Status | Meaning |
|---|---|
| `affected` | The product is affected and action is recommended. |
| `not_affected` | The product is known not to be exploitable. |
| `fixed` | The identified product version contains a fix. |
| `under_investigation` | The assessment is still in progress and should be updated later. |

For resolved VEX statements, use `affected` or `not_affected`.

## Affected

Use `affected` when the product is vulnerable:

```json
{
  "vulnerability": {
    "name": "CVE-2026-12345"
  },
  "products": [
    {
      "@id": "pkg:nuget/Example.ComponentDataService@1.0.900-beta"
    }
  ],
  "status": "affected",
  "action_statement": "No fix, workaround, or mitigation is currently available."
}
```

An affected statement requires an `action_statement`. It should describe what the product user should do to remediate or mitigate the vulnerability. If no remediation is available, state that directly rather than omitting the field.

The optional `action_statement_timestamp` records when the action statement was issued.

## Not affected

Use `not_affected` when the vulnerability cannot be exploited in the identified product. Include a machine-readable `justification`:

```json
{
  "vulnerability": {
    "name": "CVE-2026-12345"
  },
  "products": [
    {
      "@id": "pkg:nuget/Example.ComponentDataService@1.0.900-beta"
    }
  ],
  "status": "not_affected",
  "justification": "vulnerable_code_not_in_execute_path"
}
```

Use one of the defined justification labels:

| Justification | Meaning |
|---|---|
| `component_not_present` | The vulnerable component is not included in the product. |
| `vulnerable_code_not_present` | The component is included, but the vulnerable code is not present. |
| `vulnerable_code_not_in_execute_path` | The vulnerable code is present but cannot be reached during execution. |
| `vulnerable_code_cannot_be_controlled_by_adversary` | An attacker cannot control the vulnerable code in a way that exploits it. |
| `inline_mitigations_already_exist` | Built-in protections prevent exploitation. |

The OpenVEX specification permits a free-form `impact_statement` instead of a justification, but strongly discourages relying on text because it reduces automation and interoperability. Use a machine-readable `justification`.

## Fixed and under investigation

Use `fixed` when the exact product version identified by the statement contains the fix:

```json
"status": "fixed"
```

Use `under_investigation` when the assessment is still active:

```json
"status": "under_investigation"
```

An `under_investigation` statement is temporary. Publish an updated document when the investigation determines that the product is affected, not affected, or fixed.

## Relationship to CSAF VEX

The same core information appears in both formats:

| Concept | CSAF VEX | OpenVEX |
|---|---|---|
| Document identity | `document.publisher`, `document.tracking`, and `$schema` | `@context`, `@id`, `author`, `timestamp`, and `version` |
| Product | Product ID defined in `product_tree` | Product object, preferably identified by a Package URL |
| Vulnerability | An item in `vulnerabilities`, identified by `cve` or `ids` | `statement.vulnerability.name` |
| Affected | `product_status.known_affected` | `status: "affected"` |
| Not affected | `product_status.known_not_affected` with a VEX flag | `status: "not_affected"` with a justification |
| Required affected guidance | A matching `remediations` item | `action_statement` |
| Not-affected reason | `flags[].label` | `justification` |

CSAF defines a richer advisory document with a product tree and many optional security-advisory fields. OpenVEX is narrower: each statement directly connects products, one vulnerability, and one status.

## Practical rules

- Give every document a stable `@id`.
- Increment `version` whenever any document content changes.
- Use an accurate issue timestamp.
- Identify products precisely; prefer Package URLs when available.
- Use the primary vulnerability identifier in `vulnerability.name`.
- Use `affected` or `not_affected` for completed assessments.
- Always include an `action_statement` for `affected`.
- Always include a machine-readable `justification` for `not_affected`.
- Do not guess a not-affected justification.
- Publish an updated document when a status or action changes.
