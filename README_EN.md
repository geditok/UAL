# UAL v1 — Universal Asset Link

**Status:** Draft
**Version:** v1
**Last updated:** 2026-01-12
**Recommended license (text):** CC BY 4.0 (optional)

## 0. Context and rationale (a GEDITOK proposal)

This document is a **technical proposal** promoted by **GEDITOK** to define an open linking standard that enables **representing and resolving decentralized and immutable content via a URL**.

### 0.1 What GEDITOK.eu is

**GEDITOK.eu** is a **DPPaaS platform (Digital Product Passport as a Service)** designed to help companies worldwide—especially those operating in the **European Union**—implement Digital Product Passports and traceability capabilities across the product life cycle.

### 0.2 ESPR (Ecodesign for Sustainable Products Regulation)

**ESPR** (*Ecodesign for Sustainable Products Regulation*) is the EU regulatory framework aimed at improving the sustainability of products placed on the EU market, introducing information requirements and enabling—through delegated acts—the rollout of the **Digital Product Passport (DPP)** across different product categories.

### 0.3 GS1 Digital Link and why we propose UAL

In the DPP context, one recurring recommendation in the ecosystem is the use of **GS1 Digital Link**, a standard that defines how to **encode GS1 identifiers** (e.g., **GTIN**, GLN, SSCC) into **URLs/URIs** to connect a product’s identity with information accessible on the web.

GEDITOK **supports GS1 Digital Link** when it is necessary or desirable. However, **GS1 Digital Link often relies on GTIN** as a key component of the identifier and, in practice, **not all manufacturers use GTIN across all products**, among other reasons due to **cost** and the operational fit of adopting the scheme for certain segments.

For that reason, GEDITOK proposes **UAL (Universal Asset Link)** as a complementary standard, **agnostic to central organizations**, that enables composing unique URLs by leveraging **native cryptographic identifiers from blockchains** (in particular those used by GEDITOK), avoiding dependency on standards whose adoption may introduce cost or constraints.

UAL makes it possible to identify, **uniquely and unambiguously**:

* the **blockchain/network** where the data is anchored,
* the **asset** (e.g., contract + tokenId on EVM, or equivalent references in other ecosystems), and
* the **resolver entity** (resolver) that returns human (HTML) and machine-readable (JSON/JSON-LD) representations.

---

## 1. Abstract

**Universal Asset Link (UAL)** defines a **resolvable URL** syntax to identify and dereference tokenized assets—especially **physical products / digital twins** (DPP)—in a **multi-chain** way that is **agnostic to the resolver provider**, with the goal of **referencing decentralized and immutable content** through a standard URL.

UAL separates:

* A **canonical identifier** (extractable from the URL) composed of:

  * a network identifier (CAIP-2)
  * an asset reference (Asset Reference)
* A **resolver domain** (resolver-domain) that serves the human and/or machine-readable representation of the asset.

UAL is designed for:

* links embedded in QR/DataMatrix/labels;
* interoperability across resolvers;
* HTML and JSON/JSON-LD responses via content negotiation.

## 2. Terminology and normative language

In this specification, the terms **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, and **MAY** indicate implementation requirements and recommendations.

Terms used:

* **Resolver**: an **HTTPS** service that receives a UAL URL and returns representations of the asset (HTML, JSON, JSON-LD, etc.).
* **UAL URL**: a URL that complies with this standard.
* **CAIP-2**: a network identifier in `namespace:reference` format (e.g., `eip155:11155111`).
* **AssetRef**: an asset reference within UAL (defined in this specification).
* **UAL Canonical ID**: the conceptual concatenation of `{caip2}` + `{assetRef}` (independent of the resolver domain).

## 3. URL syntax

### 3.1 Base structure (normative)

A UAL URL **MUST** have the following form:

```
https://{resolver-domain}/ual/v1/{caip2}/{assetRef}
```

Where:

* `{resolver-domain}` is a valid DNS domain under HTTPS.
* `ual` and `v1` are literals.
* `{caip2}` is a CAIP-2 network identifier.
* `{assetRef}` is an asset reference defined in §4.

The path **MUST** be case-sensitive for the `ual` and `v1` segments (lowercase is recommended). For `{caip2}` and `{assetRef}` see §5.

### 3.2 Scheme and transport requirements

* UAL URLs **MUST** use `https://`.

### 3.3 ABNF (informative)

> Note: ABNF is approximate and may be tightened in future revisions.

```
ual-url      = "https://" resolver "/" "ual" "/" "v1" "/" caip2 "/" assetRef
resolver     = 1*( unreserved / pct-encoded / sub-delims / "." )
caip2        = namespace ":" reference
namespace    = 1*( ALPHA / DIGIT / "-" )
reference    = 1*( ALPHA / DIGIT / "-" / "." / "_" )
assetRef     = nftRef / caip19Ref

nftRef       = "nft" "/" nftStd "/" evmAddress "/" tokenId
nftStd       = "erc721" / "erc1155"
evmAddress   = "0x" 40HEXDIG
tokenId      = decTokenId / hexTokenId

decTokenId   = 1*DIGIT
hexTokenId   = "0x" 1*HEXDIG

caip19Ref    = "asset" "/" 1*( unreserved / pct-encoded / sub-delims / ":" )
```

## 4. AssetRef (asset reference)

UAL v1 defines two branches:

### 4.1 Pragmatic EVM NFT branch (normative)

For EVM NFTs, `{assetRef}` **MUST** use:

* ERC-721:

  ```
  nft/erc721/{contractAddress}/{tokenId}
  ```
* ERC-1155:

  ```
  nft/erc1155/{contractAddress}/{tokenId}
  ```

Where:

* `{contractAddress}` **MUST** be a 20-byte EVM hex address (`0x` + 40 hex).
* `{tokenId}` **MUST** be an unsigned integer. In the URL it **SHOULD** be represented in **decimal (base 10)**; the resolver **MAY** accept **hexadecimal** if prefixed with `0x`, but it **SHOULD** normalize/canonicalize to decimal.

### 4.2 CAIP-19 escape hatch (normative)

For any chain or asset that does not fit 4.1, `{assetRef}` **MUST** be expressible as:

```
asset/{caip19}
```

Where `{caip19}` is a CAIP-19 (or compatible) URL-safe identifier.

> Rationale: this branch ensures extensibility and multi-chain compatibility without forcing bespoke schemes for each ecosystem.

## 5. Canonicalization

To avoid semantic duplicates, UAL v1 defines canonical form rules.

### 5.1 CAIP-2

* `{caip2}` **SHOULD** be lowercase (`eip155:11155111`).
* The resolver **MUST** accept mixed case in input, but **SHOULD** respond (see §6) using the canonical form.

### 5.2 EVM addresses

* `{contractAddress}` **MUST** be accepted in input both in lowercase and checksum form.
* Recommended canonical form: **EIP-55 checksum** (if the resolver can compute it).
* Alternatively, for maximum simplicity, you may set canonical form to **lowercase**.

### 5.3 TokenId

* `{tokenId}` **MUST** be an unsigned integer. In canonical form it **MUST** be represented in **decimal** without leading zeros (except `0`). If the resolver accepts `0x` hexadecimal on input, it **SHOULD** convert it to the canonical decimal form.

### 5.4 General normalization

* The resolver **MUST** reject (400) UAL URLs with non-URL-safe characters or unexpected empty segments.
* The resolver **SHOULD** redirect (308) to the canonical URL when receiving non-canonical variants.

## 6. Resolver behavior

### 6.1 Content negotiation (normative)

Given a valid UAL URL, the resolver:

* **MUST** support `text/html` (human view), or otherwise **MUST** return at least `application/json`.
* **SHOULD** support `application/ld+json` (JSON-LD) for semantic interoperability (DPP/traceability).
* **MAY** support `application/json` as a simple representation.

The resolver **MUST** respect the HTTP `Accept` header (content negotiation). If multiple types are acceptable, it **SHOULD** prioritize:

1. `application/ld+json`
2. `application/json`
3. `text/html`

### 6.2 Minimum response (normative)

Any machine-readable response (`application/json` or `application/ld+json`) **MUST** include at minimum:

* `ual`: the full received URL or its canonical form
* `version`: `"v1"`
* `caip2`: the canonical network identifier
* `assetRef`: the canonical asset reference
* `idCanonical`: a canonical identifier **independent of the domain**, recommended as:

  ```
  ual:v1:{caip2}/{assetRef}
  ```
* `resolver`: object with resolver information, at least:

  * `domain`

Recommended fields for traceability/DPP (SHOULD):

* `granularity`: `"model" | "batch" | "item"`
* `integrity`:

  * `cid` (IPFS/Arweave or similar) and/or `sha256`
* `links`:

  * `dpp` (URL to the DPP record)
  * `events` (event timeline)

### 6.3 Status codes (normative)

* If the URL is syntactically invalid: **400 Bad Request**
* If the asset does not exist (or the resolver cannot resolve it): **404 Not Found**
* If the asset exists but access is restricted: **403 Forbidden** (or 401 if authentication applies)
* If redirecting to canonical form: **308 Permanent Redirect** (recommended)

## 7. Query parameters (recommended)

UAL v1 allows query parameters without affecting `idCanonical`. These parameters **MUST NOT** change the meaning of the identifier—only its **representation**.

Recommended:

* `view=consumer|auditor|regulator`
* `lang=en|es|...`
* `format=html|json|jsonld` (if you do not want to depend solely on `Accept`)
* `scope=model|batch|item` (if the same asset can navigate across granularity levels)

## 8. Examples (multi-chain)

> Note: on non-EVM chains, the examples use `asset/{caip19}` as an extensible form; UAL v1 does not impose a “tokenId” model outside EVM.

### 8.1 EVM Sepolia (testnet), ERC-721, tokenId=1234

```
https://resolver.geditok.eu/ual/v1/eip155:11155111/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234
```

### 8.2 EVM Ethereum mainnet, ERC-721, tokenId=9876

```
https://resolver.geditok.eu/ual/v1/eip155:1/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/9876
```

### 8.3 Bitcoin mainnet (Bitcoin family via BIP122), template

CAIP-2 defines examples with namespace `bip122` (Bitcoin mainnet included). The Bitcoin mainnet reference (genesis hash prefix) is expressed in 32 hex.

```
https://resolver.geditok.eu/ual/v1/bip122:000000000019d6689c085ae165831e93/asset/{caip19}
```

Where `{caip19}` is an **asset identifier** (CAIP-19 or compatible). In UAL v1 it is used here as a **generic placeholder** for non-EVM assets (when `contract/tokenId` does not apply).

### 8.4 Cosmos Hub (Cosmos ecosystem), template

CAIP-2 defines examples with namespace `cosmos` and references like `cosmoshub-*`.

```
https://resolver.geditok.eu/ual/v1/cosmos:cosmoshub-3/asset/{caip19}
```

### 8.5 Solana mainnet, template

In CAIP-2, Solana is identified as `solana:{reference}`. In practice the reference is a well-known identifier of the environment/network.

```
https://resolver.geditok.eu/ual/v1/solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp/asset/{caip19}
```

### 8.6 Polkadot (namespace polkadot), template

CAIP-13 defines that the `polkadot` namespace uses as `reference` the **32-character prefix** of the `genesis-hash` in lowercase hex.

```
https://resolver.geditok.eu/ual/v1/polkadot:{genesisHashPrefix32}/asset/{caip19}
```

### 8.7 Cardano (interoperability note)

UAL v1 **supports any** `{caip2}` defined by CAIP-2 and its namespace profiles. If the Cardano namespace were not yet formalized in the namespaces registry, it is recommended to use `asset/{caip19}` under the `{caip2}` that is officially established when the corresponding profile exists.

Template:

```
https://resolver.geditok.eu/ual/v1/{caip2-cardano}/asset/{caip19}
```

## 9. Example JSON response (minimum)

```json
{
  "ual": "https://resolver.geditok.eu/ual/v1/eip155:11155111/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234",
  "version": "v1",
  "caip2": "eip155:11155111",
  "assetRef": "nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234",
  "idCanonical": "ual:v1:eip155:11155111/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234",
  "resolver": {
    "domain": "resolver.geditok.eu"
  },
  "granularity": "item",
  "integrity": {
    "cid": "ipfs://bafy...example",
    "sha256": "3f0c...example"
  },
  "links": {
    "dpp": "https://resolver.geditok.eu/dpp/item/...",
    "events": "https://resolver.geditok.eu/events/..."
  },
  "schemaOrg": {
    "@context": "https://schema.org",
    "@type": "IndividualProduct",
    "name": "Example Product Name",
    "brand": {
      "@type": "Brand",
      "name": "Example Brand"
    },
    "manufacturer": {
      "@type": "Organization",
      "name": "Example Manufacturer"
    },
    "model": "MODEL-REF-001",
    "serialNumber": "SN-0000001234",
    "productionDate": "2026-01-01",
    "gtin": "00000000000000",
    "identifier": [
      {
        "@type": "PropertyValue",
        "propertyID": "ual",
        "value": "ual:v1:eip155:11155111/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234"
      },
      {
        "@type": "PropertyValue",
        "propertyID": "contract",
        "value": "0xAbCDEF0123456789aBCdef0123456789ABCDef01"
      },
      {
        "@type": "PropertyValue",
        "propertyID": "tokenId",
        "value": "1234"
      }
    ]
  }
}
```

## 10. Security and trust considerations

Implementers **SHOULD** consider:

* **Phishing / malicious resolvers**: the resolver domain may lie about the asset.
  Mitigation: include `integrity` (CID/hash) and, where possible, verifiable proofs (signatures, on-chain anchors).
* **TLS required**: production UAL **MUST** be `https`.
* **Caching**: set appropriate `Cache-Control` (especially for regulated data or revocations).
* **Privacy**: for DPP, separate public data from restricted data; avoid exposing PII in public JSON-LD.

## 11. Versioning and compatibility

* The version is fixed in the path: `/ual/v1/`.
* Incompatible changes **MUST** increment the major version (`v2`, etc.).
* Compatible changes **SHOULD** be delivered via:

  * new fields in JSON/JSON-LD responses
  * new non-semantic query params
  * new `assetRef` subpaths only if they do not break parsing (prefer reserving and documenting)

## 12. Extension registry (optional)

It is recommended to maintain a `REGISTRY.md` file in the repository with:

* additional `assetRef` forms (e.g., `ft/...`, `bundle/...`, `dpp/...`)
* `view` enumerations
* sector-specific extensions for `granularity`

---

## 13. Compliance checklist (practical)

A resolver compliant with UAL v1:

* [ ] Accepts URLs `https://{domain}/ual/v1/{caip2}/{assetRef}`
* [ ] Parses `{caip2}` and `{assetRef}` according to §4
* [ ] Returns **HTML** or **JSON** at minimum
* [ ] Returns a machine-readable representation including `idCanonical`
* [ ] Redirects (308) to canonical form when applicable
* [ ] Implements 400/404/403 status codes consistently

---

## 14. Technical references (non-exhaustive)

* CAIP-2 (Blockchain ID)
* CAIP-4 (BIP122)
* CAIP-5 (Cosmos)
* CAIP-13 (Polkadot)
* CAIP-30 (Solana)
* CAIP-19 (Asset Type & ID)
* Chain Agnostic Namespaces (namespace registry)
