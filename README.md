# Practical CSAF 2.1 VEX Guide

This guide describes one simple, consistent way to create a compliant CSAF 2.1 Vulnerability Exploitability eXchange (VEX) document. CSAF supports many kinds of security documents and therefore leaves much of its schema optional. The VEX profile and the conformance tests add the rules that matter here.

This guide is intentionally opinionated. It does not describe every valid CSAF representation. Follow these conventions unless the information you need to publish cannot be represented by them. The goal is for different authors to produce VEX documents with the same structure and semantics.

This is independent guidance, not an official OASIS specification. The specification baseline for this guide is [CSAF 2.1 Committee Specification Draft 04, dated 21 October 2026](https://docs.oasis-open.org/csaf/csaf/v2.1/csd04/csaf-v2.1-csd04.html). CSAF 2.1 is still a draft; later revisions may change its requirements.

The rules below are based on the [CSAF Base profile](https://docs.oasis-open.org/csaf/csaf/v2.1/csd04/csaf-v2.1-csd04.html#profile-1-csaf-base), the [VEX profile](https://docs.oasis-open.org/csaf/csaf/v2.1/csd04/csaf-v2.1-csd04.html#profile-5-vex), and the [mandatory tests](https://docs.oasis-open.org/csaf/csaf/v2.1/csd04/csaf-v2.1-csd04.html#mandatory-tests). Chapter 6 also defines recommended and informative tests. Those tests may suggest optional content, such as metrics or language metadata, but a recommendation is not an additional VEX conformance requirement. This guide prioritizes mandatory compliance and consistent output.

For the other format covered in this repository, see the [OpenVEX overview](openvex-guide.md).

## The basic model

A VEX document answers this question:

> What is the status of each product with respect to each vulnerability, and what evidence or action accompanies that status?

Build the document in three parts:

1. `document` identifies the VEX document and its publisher.
2. `product_tree` defines every product that will be referenced.
3. `vulnerabilities` identifies each vulnerability, assigns a status to each relevant product, and supplies the information required by that status.

Use one vulnerability object per vulnerability. A document may contain multiple products and multiple vulnerabilities.

## Start with this document skeleton

Replace every value in angle brackets. Use RFC 3339 timestamps, including a timezone.

This skeleton is incomplete and is not ready to publish. Populate `vulnerabilities` with at least one assessed vulnerability and reference every defined product before publishing. See the [complete example](#complete-example) for a filled-in document.

```json
{
  "$schema": "https://docs.oasis-open.org/csaf/csaf/v2.1/schema/csaf.json",
  "document": {
    "category": "csaf_vex",
    "csaf_version": "2.1",
    "distribution": {
      "tlp": {
        "label": "CLEAR"
      }
    },
    "publisher": {
      "category": "vendor",
      "name": "<publisher name>",
      "namespace": "https://<publisher-controlled namespace>/"
    },
    "title": "<canonical document title>",
    "tracking": {
      "current_release_date": "<YYYY-MM-DDThh:mm:ssZ>",
      "id": "<publisher-unique document ID>",
      "initial_release_date": "<YYYY-MM-DDThh:mm:ssZ>",
      "revision_history": [
        {
          "date": "<YYYY-MM-DDThh:mm:ssZ>",
          "number": "1.0.0",
          "summary": "Initial release."
        }
      ],
      "status": "final",
      "version": "1.0.0"
    }
  },
  "product_tree": {
    "branches": [
      {
        "category": "vendor",
        "name": "<vendor name>",
        "branches": [
          {
            "category": "product_name",
            "name": "<product name>",
            "branches": [
              {
                "category": "product_version",
                "name": "<product version>",
                "product": {
                  "name": "<complete product name and version>",
                  "product_id": "CSAFPID-0001"
                }
              }
            ]
          }
        ]
      }
    ]
  },
  "vulnerabilities": []
}
```

Use these conventions:

- Publish public VEX documents as `TLP:CLEAR`.
- This guide assumes the publisher is the authoritative vendor or maintainer of the product, so `publisher.category` is `vendor`.
- Use a stable URI controlled by the publisher for `publisher.namespace`. It identifies the publisher; it does not have to resolve to a web page.
- Use a non-empty canonical `title` that is sufficiently unique to distinguish the document from similar documents.
- Make `tracking.id` unique within the publisher namespace and never reuse it for another document.
- Choose semantic versioning or integer versioning. CSAF prefers semantic versioning. Use only the selected system throughout the document.
- For a first release, use `"1.0.0"` with semantic versioning or `"1"` with integer versioning. Set both release dates and the revision date to the release timestamp.
- For an update, keep `initial_release_date` unchanged, update `current_release_date`, update `version` according to the selected versioning system, and append a matching revision-history entry.

## Define products

Use `product_tree.branches` with the hierarchy recommended by the specification:

```text
vendor → product_name → product_version
```

Define the assessed product with one exact `product_version` branch:

```json
"product_tree": {
  "branches": [
    {
      "category": "vendor",
      "name": "Example Corporation",
      "branches": [
        {
          "category": "product_name",
          "name": "Example Server",
          "branches": [
            {
              "category": "product_version",
              "name": "4.2.1",
              "product": {
                "name": "Example Server 4.2.1",
                "product_id": "CSAFPID-0001"
              }
            }
          ]
        }
      ]
    }
  ]
}
```

The leaf `product` is a full product name object. Its `name` must contain the complete canonical product name and version even though the version also appears in the surrounding `product_version` branch. Consumers must not have to reconstruct the full name from the branch hierarchy.

The `product_id` is an internal identifier used throughout this document. It is not a package identifier. It must be unique in the document, must be defined exactly once, and must always refer to the same product.

Use `product_version` only for one exact version. Do not put a version range such as `before 4.2` in its `name`. The defined product should be referenced by at least one vulnerability.

## Fill out every vulnerability

Every vulnerability object must contain:

- A vulnerability identifier: `cve` or `ids`.
- At least one `notes` item.
- A `product_status` containing `known_affected` or `known_not_affected`.
- Any evidence or action required by the selected status.

### Identify the vulnerability

If a CVE exists, use `cve` and do not repeat it in `ids`:

```json
"cve": "CVE-2026-12345"
```

If there is no CVE, use a public identifier from the source system:

```json
"ids": [
  {
    "system_name": "GHSA",
    "text": "GHSA-xxxx-yyyy-zzzz"
  }
]
```

Do not bind the identifier to products with `product_ids` or `group_ids`; an unbound identifier applies to the vulnerability as a whole.

### Add the required note

Use one short summary note for every vulnerability:

```json
"notes": [
  {
    "category": "summary",
    "text": "<plain-language description of the vulnerability>"
  }
]
```

The note's `title` property is optional for the VEX profile; do not include it. If copying the official CVE description unchanged, use `category: "description"` instead. Do not present edited or summarized text as the official CVE description.

## Choose the product status

Use only the following two status fields. This produces a resolved, binary VEX statement: the product is either exploitable or it is not. A product must not appear in both fields for the same vulnerability.

| Status | Use when | Additional content |
|---|---|---|
| `known_affected` | The product is vulnerable. | A matching `remediations` action statement is required. |
| `known_not_affected` | The product is known not to be exploitable. | A matching machine-readable `flags` justification is required. |

Treat a fixed version as `known_not_affected` and justify it with `vulnerable_code_not_present`. Do not publish a status for a product whose assessment is incomplete; finish the investigation and then classify it.

CSAF also defines `fixed` and `under_investigation`, and the VEX profile permits them. This guide deliberately does not use them. Do not use `unknown`, `first_affected`, `last_affected`, `first_fixed`, or `recommended`; they add states and version relationships that are not needed for this binary convention.

### Known affected

List the affected product IDs in `known_affected` and provide an action statement in `remediations`. Every affected product must be covered by a remediation's `product_ids`.

```json
{
  "cve": "CVE-2026-12345",
  "notes": [
    {
      "category": "summary",
      "text": "A crafted request can cause an out-of-bounds read."
    }
  ],
  "product_status": {
    "known_affected": [
      "CSAFPID-0001"
    ]
  },
  "remediations": [
    {
      "category": "vendor_fix",
      "details": "Upgrade to Example Server 5.0.0.",
      "product_ids": [
        "CSAFPID-0001"
      ]
    }
  ]
}
```

Choose the remediation category that states the action:

| Category | Use when |
|---|---|
| `vendor_fix` | An official fix is available. |
| `workaround` | A configuration or deployment change avoids exposure. |
| `mitigation` | An action reduces risk but does not remove the vulnerability. |
| `fix_planned` | A fix is planned but is not yet available. |
| `none_available` | No remediation is currently available. |
| `no_fix_planned` | No fix will be provided. |

Do not use `optional_patch` for an affected product. It is intended for a product that is not affected but may still be updated, for example to satisfy a scanner.

If several actions apply, add several remediation objects. Each object must contain `category`, `details`, and the `product_ids` it covers.

### Known not affected

List the product IDs in `known_not_affected` and provide a machine-readable `flags` justification. Use exactly one VEX justification flag per product.

```json
{
  "cve": "CVE-2026-12345",
  "notes": [
    {
      "category": "summary",
      "text": "A crafted request can cause an out-of-bounds read."
    }
  ],
  "product_status": {
    "known_not_affected": [
      "CSAFPID-0002"
    ]
  },
  "flags": [
    {
      "label": "vulnerable_code_not_in_execute_path",
      "product_ids": [
        "CSAFPID-0002"
      ]
    }
  ]
}
```

Select the flag from facts established by the assessment; do not guess:

| Flag | Meaning |
|---|---|
| `component_not_present` | The vulnerable component is not in the product. |
| `vulnerable_code_not_present` | The component is present, but the vulnerable code is not. |
| `vulnerable_code_cannot_be_controlled_by_adversary` | The code is present, but an attacker cannot control it in a way that triggers the vulnerability. |
| `vulnerable_code_not_in_execute_path` | The vulnerable code cannot be reached during execution. |
| `inline_mitigations_already_exist` | Built-in controls prevent exploitation. |

If none of these flags accurately explains the assessment, do not classify the product as `known_not_affected` under this convention. Revisit the assessment until it can be supported by one of the defined VEX justification flags.

Do not add a remediation for `known_not_affected`. Remediations such as `vendor_fix`, `workaround`, and `mitigation` contradict that status.

## Vulnerability field map

The following table covers every top-level field available in a vulnerability object. Omit fields marked **Do not include**. They are not required by the CSAF schema, the VEX profile, or a mandatory VEX profile test.

| Field | What it contains | Convention in this guide |
|---|---|---|
| `acknowledgments` | Credits for vulnerability discovery or handling. | **Do not include.** |
| `cve` | The CVE identifier. | Include when a CVE exists; otherwise use `ids`. |
| `cwes` | CWE classifications. | **Do not include.** |
| `disclosure_date` | When the vulnerability was publicly disclosed. | **Do not include.** |
| `discovery_date` | When the vulnerability was discovered. | **Do not include.** |
| `first_known_exploitation_dates` | Product-specific dates of known exploitation. | **Do not include.** |
| `flags` | Machine-readable VEX not-affected justification. | Always include exactly one applicable flag for each `known_not_affected` product. |
| `ids` | Non-CVE vulnerability identifiers. | Include only when there is no CVE. |
| `metrics` | Product-specific CVSS, EPSS, or SSVC metrics. | **Do not include.** |
| `notes` | Human-readable vulnerability information. | Always include one summary or official CVE description. |
| `product_status` | Product IDs grouped by VEX status. | Always include `known_affected` or `known_not_affected`. |
| `references` | Links related to the vulnerability. | **Do not include.** |
| `remediations` | Product-specific action statements. | Required for `known_affected`; otherwise omit. |
| `threats` | Time-sensitive exploit, impact, or target information. | **Do not include.** |
| `title` | A publisher-assigned vulnerability title. | **Do not include.** |
| `x_extensions` | CSAF extensions. | **Do not include.** |

## Other optional fields

The skeleton deliberately omits the following optional fields. Leave them out unless another publishing policy explicitly requires them.

| Location | Optional fields to omit | Purpose |
|---|---|---|
| Root | `x_extensions` | Extends CSAF with another schema. |
| `document` | `acknowledgments` | Credits applying to the whole document. |
| `document` | `aggregate_severity` | Publisher-defined severity for the document as a whole. |
| `document` | `involvement` | Coordination actions and participating entities. |
| `document` | `lang` | Document language. |
| `document` | `license_expression` | SPDX license expression for the document. |
| `document` | `notes` | Notes applying to the whole document. |
| `document` | `references` | References applying to the whole document. |
| `document` | `source_lang` | Source language of a translation. |
| `document` | `x_extensions` | Document-level extensions. |
| `document.distribution` | `sharing_group`, `text` | Additional sharing restrictions. |
| `document.publisher` | `contact`, `issuing_authority` | Publisher contact and authority details. |
| `document.tracking` | `aliases`, `generator` | Alternate document IDs and generator metadata. |
| `product_tree` | `full_product_names`, `product_groups`, `product_paths` | Alternative flat, grouping, and relationship-based product representations. |

Within objects that this guide does use, also omit optional dates, URLs, group references, audience fields, note titles, entitlements, restart instructions, and extensions. Use direct `product_ids` instead of product groups.

## Complete example

This fictional document assesses Example Server 4.2.1 as affected and recommends upgrading to 5.0.0. Replace the illustrative product, vulnerability identifier, publisher, and assessment with your own verified information.

```json
{
  "$schema": "https://docs.oasis-open.org/csaf/csaf/v2.1/schema/csaf.json",
  "document": {
    "category": "csaf_vex",
    "csaf_version": "2.1",
    "distribution": {
      "tlp": {
        "label": "CLEAR"
      }
    },
    "publisher": {
      "category": "vendor",
      "name": "Example Corporation",
      "namespace": "https://example.com/"
    },
    "title": "Example Server 4.2.1 VEX assessment for CVE-2026-12345",
    "tracking": {
      "current_release_date": "2026-10-22T16:00:00Z",
      "id": "EXAMPLE-VEX-2026-001",
      "initial_release_date": "2026-10-22T16:00:00Z",
      "revision_history": [
        {
          "date": "2026-10-22T16:00:00Z",
          "number": "1.0.0",
          "summary": "Initial release."
        }
      ],
      "status": "final",
      "version": "1.0.0"
    }
  },
  "product_tree": {
    "branches": [
      {
        "category": "vendor",
        "name": "Example Corporation",
        "branches": [
          {
            "category": "product_name",
            "name": "Example Server",
            "branches": [
              {
                "category": "product_version",
                "name": "4.2.1",
                "product": {
                  "name": "Example Server 4.2.1",
                  "product_id": "CSAFPID-0001"
                }
              }
            ]
          }
        ]
      }
    ]
  },
  "vulnerabilities": [
    {
      "cve": "CVE-2026-12345",
      "notes": [
        {
          "category": "summary",
          "text": "A crafted request can cause an out-of-bounds read."
        }
      ],
      "product_status": {
        "known_affected": [
          "CSAFPID-0001"
        ]
      },
      "remediations": [
        {
          "category": "vendor_fix",
          "details": "Upgrade to Example Server 5.0.0.",
          "product_ids": [
            "CSAFPID-0001"
          ]
        }
      ]
    }
  ]
}
```

## Final checks

Before publishing, verify all of the following:

- `$schema` is the CSAF 2.1 schema URL.
- `document.category` is exactly `csaf_vex`.
- The document contains both `product_tree` and `vulnerabilities`.
- Every product ID is defined exactly once and every defined product is used.
- Every vulnerability has `cve` or `ids`.
- Every vulnerability has at least one note.
- Every vulnerability uses `known_affected` or `known_not_affected`.
- No product has contradictory statuses for the same vulnerability.
- Every `known_affected` product is covered by a remediation.
- Every `known_not_affected` product is covered by exactly one justification flag.
- Every flag and remediation has matching `product_ids`.
- Dates use RFC 3339 date-time syntax, for example `2026-09-29T15:59:34Z`.
- The tracking version matches the newest revision-history number, and all version values consistently use either semantic or integer versioning.

## License

Except for the JSON examples, this guide is licensed under the
[Creative Commons Attribution 4.0 International License (CC BY 4.0)](LICENSE).
The JSON examples, including complete documents, templates, and fragments, are
dedicated to the public domain under [CC0 1.0 Universal (CC0-1.0)](LICENSE-CC0)
and may be reused without attribution. Third-party material retains its existing
license.