# UAL v1 — Universal Asset Link

**Estado:** Borrador (Draft)
**Versión:** v1
**Última actualización:** 2026-01-12
**Licencia recomendada del texto:** CC BY 4.0 (opcional)

## 0. Contexto y motivación (propuesta de GEDITOK)

Este documento es una **propuesta técnica** impulsada por **GEDITOK** para definir un estándar abierto de enlaces que permitan **representar y resolver contenidos descentralizados e inmutables mediante una URL**.

### 0.1 Qué es GEDITOK.eu

**GEDITOK.eu** es una plataforma **DPPaaS (Digital Product Passport as a Service / Pasaporte Digital de Producto como Servicio)** orientada a ayudar a empresas de todo el mundo —y de forma especial a empresas que operan en la **Unión Europea**— a implementar Pasaportes Digitales de Producto y capacidades de trazabilidad a lo largo del ciclo de vida.

### 0.2 ESPR (Ecodesign for Sustainable Products Regulation)

El **ESPR** (*Ecodesign for Sustainable Products Regulation*, Reglamento de Ecodiseño para Productos Sostenibles) es el marco regulatorio europeo orientado a mejorar la sostenibilidad de los productos comercializados en el mercado de la UE, introduciendo requisitos de información y habilitando, mediante actos delegados, la implantación del **Pasaporte Digital de Producto (DPP)** en diferentes categorías de producto.

### 0.3 GS1 Digital Link y por qué proponemos UAL

En el contexto del DPP, una de las recomendaciones recurrentes del ecosistema es el uso de **GS1 Digital Link**, un estándar que define cómo **codificar identificadores GS1** (por ejemplo, **GTIN**, GLN, SSCC) en **URLs/URIs** para conectar la identidad del producto con información accesible en la web.

GEDITOK **soporta GS1 Digital Link** cuando es necesario o deseable. Sin embargo, **GS1 Digital Link suele apoyarse en GTIN** como componente clave del identificador y, en la práctica, **no todos los fabricantes lo utilizan en todos sus productos**, entre otros motivos por el **coste y el encaje operativo** de adoptar dicho esquema para ciertos segmentos.

Por ello, GEDITOK propone **UAL (Universal Asset Link)** como un estándar complementario, **agnóstico a organismos centrales**, que permite componer URLs únicas apoyándose en **identificadores criptográficos nativos de las blockchains** (y en particular las que utiliza GEDITOK), evitando depender de estándares cuya adopción suponga un corsé o un coste adicional.

UAL permite identificar de forma **única e inequívoca**:

* la **blockchain/red** donde se alojan los datos,
* el **activo** (p. ej., contrato + tokenId en EVM, o referencias equivalentes en otros ecosistemas),
* y la **entidad resolutora (resolver)** que devuelve representaciones humanas (HTML) y machine-readable (JSON/JSON-LD).

---

## 1. Resumen (Abstract)

**Universal Asset Link (UAL)** define una sintaxis de **URL resoluble** para identificar y dereferenciar activos tokenizados, especialmente **productos físicos / gemelos digitales** (DPP), de forma **multi-chain** y **agnóstica al proveedor del resolver**, con el objetivo de **referenciar contenidos descentralizados e inmutables** desde una URL estándar.

UAL separa:

* Un **identificador canónico** (extraíble de la URL) compuesto por:

  * identificador de red (CAIP-2)
  * referencia de activo (Asset Reference)
* Un **dominio resolutor** (resolver-domain) que sirve la representación humana y/o machine-readable del activo.

UAL se diseña para:

* enlaces en QR/DataMatrix/etiquetas;
* interoperabilidad entre resolvers;
* respuestas HTML y JSON/JSON-LD mediante negociación de contenido.

## 2. Terminología y lenguaje normativo

En esta especificación, los términos **DEBE**, **NO DEBE**, **DEBERÍA**, **NO DEBERÍA** y **PUEDE** indican requisitos y recomendaciones de implementación.

Términos usados:

* **Resolver**: servicio **HTTPS** que recibe una URL UAL y devuelve representaciones del activo (HTML, JSON, JSON-LD, etc.).
* **UAL URL**: URL que cumple la sintaxis de este estándar.
* **CAIP-2**: identificador de red en formato `namespace:reference` (por ejemplo `eip155:11155111`).
* **AssetRef**: referencia de activo dentro de UAL (definida en esta especificación).
* **IdCanónicoUAL**: concatenación conceptual de `{caip2}` + `{assetRef}` (independiente del dominio del resolver).

## 3. Sintaxis de URL

### 3.1 Estructura base (normativa)

Una UAL URL **DEBE** tener esta forma:

```
https://{resolver-domain}/ual/v1/{caip2}/{assetRef}
```

Donde:

* `{resolver-domain}` es un dominio DNS válido bajo HTTPS.
* `ual` y `v1` son literales.
* `{caip2}` es un identificador de red CAIP-2.
* `{assetRef}` es una referencia de activo definida en §4.

La ruta **DEBE** ser sensible a mayúsculas/minúsculas para los segmentos `ual` y `v1` (se recomienda en minúsculas). Para `{caip2}` y `{assetRef}` ver §5.

### 3.2 Requisitos de esquema y transporte

* Las UAL URL **DEBEN** usar `https://`.

### 3.3 ABNF (orientativa)

> Nota: ABNF es aproximada y puede endurecerse en futuras revisiones.

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

## 4. AssetRef (referencia de activo)

UAL v1 define dos ramas:

### 4.1 NFT EVM pragmático (normativo)

Para EVM NFTs, `{assetRef}` **DEBE** usar:

* ERC-721:

  ```
  nft/erc721/{contractAddress}/{tokenId}
  ```
* ERC-1155:

  ```
  nft/erc1155/{contractAddress}/{tokenId}
  ```

Donde:

* `{contractAddress}` **DEBE** ser una dirección hex EVM de 20 bytes (`0x` + 40 hex).
* `{tokenId}` **DEBE** ser un entero sin signo. En la URL **DEBERÍA** representarse en **decimal (base 10)**; el resolver **PUEDE** aceptar representación **hexadecimal** si va prefijada por `0x`, pero **DEBERÍA** normalizar/canonicalizar a decimal.

### 4.2 Escape hatch CAIP-19 (normativo)

Para cualquier cadena o activo que no encaje en 4.1, `{assetRef}` **DEBE** poder expresarse como:

```
asset/{caip19}
```

Donde `{caip19}` es un identificador CAIP-19 (o compatible) URL-safe.

> Motivación: esta rama garantiza extensibilidad y compatibilidad multi-chain sin forzar “inventos” para cada ecosistema.

## 5. Canonicalización

Para evitar duplicados semánticos, UAL v1 define reglas de forma canónica.

### 5.1 CAIP-2

* `{caip2}` **DEBERÍA** estar en minúsculas (`eip155:11155111`).
* El resolver **DEBE** aceptar mayúsculas/minúsculas en entrada, pero **DEBERÍA** responder (ver §6) usando la forma canónica.

### 5.2 Direcciones EVM

* `{contractAddress}` **DEBE** aceptarse en entrada tanto en minúsculas como en checksum.
* Forma canónica recomendada: **checksum EIP-55** (si el resolver la puede calcular).
* Alternativamente, si quieres máxima simplicidad, puedes fijar como canónica **minúsculas**.

### 5.3 TokenId

* `{tokenId}` **DEBE** ser un entero sin signo. En forma canónica **DEBE** representarse en **decimal** sin ceros a la izquierda (excepto `0`). Si el resolver acepta `0x` hexadecimal en entrada, **DEBERÍA** convertirlo a la forma canónica decimal.

### 5.4 Normalización general

* El resolver **DEBE** rechazar (400) UAL con caracteres no URL-safe o con segmentos vacíos inesperados.
* El resolver **DEBERÍA** redirigir (308) a la URL canónica cuando reciba variantes no canónicas.

## 6. Comportamiento del resolver

### 6.1 Negociación de contenido (normativa)

Dada una UAL URL válida, el resolver:

* **DEBE** soportar `text/html` (vista humana), o en su defecto **DEBE** devolver al menos `application/json`.
* **DEBERÍA** soportar `application/ld+json` (JSON-LD) para interoperabilidad semántica (DPP/traceability).
* **PUEDE** soportar `application/json` como representación simple.

El resolver **DEBE** respetar la cabecera `Accept` de HTTP (content negotiation). Si hay múltiples tipos aceptables, **DEBERÍA** priorizar en este orden:

1. `application/ld+json`
2. `application/json`
3. `text/html`

### 6.2 Respuesta mínima (normativa)

Cualquier respuesta machine-readable (`application/json` o `application/ld+json`) **DEBE** incluir como mínimo:

* `ual`: la URL completa recibida o su forma canónica
* `version`: `"v1"`
* `caip2`: el identificador de red canónico
* `assetRef`: la referencia de activo canónica
* `idCanonico`: un identificador canónico **independiente del dominio**, recomendado como:

  ```
  ual:v1:{caip2}/{assetRef}
  ```
* `resolver`: objeto con información del resolver, al menos:

  * `domain`

Campos recomendados para trazabilidad/DPP (DEBERÍA):

* `granularity`: `"model" | "batch" | "item"`
* `integrity`:

  * `cid` (IPFS/Arweave o similar) y/o `sha256`
* `links`:

  * `dpp` (URL a la ficha DPP)
  * `events` (timeline de eventos)

### 6.3 Códigos de estado (normativos)

* Si la URL es sintácticamente inválida: **400 Bad Request**
* Si el asset no existe (o el resolver no lo reconoce): **404 Not Found**
* Si existe pero está restringido: **403 Forbidden** (o 401 si aplica autenticación)
* Si hay redirección a forma canónica: **308 Permanent Redirect** (recomendado)

## 7. Parámetros de query (recomendados)

UAL v1 permite query params sin afectar al `idCanonico`. Estos parámetros **NO DEBEN** cambiar el significado del identificador, solo la **representación**.

Recomendados:

* `view=consumer|auditor|regulator`
* `lang=es|en|...`
* `format=html|json|jsonld` (si no quieres depender solo de `Accept`)
* `scope=model|batch|item` (si un mismo asset puede navegar a niveles de granularidad)

## 8. Ejemplos (multi-chain)

> Nota: en cadenas no-EVM, el ejemplo usa `asset/{caip19}` como forma extensible; UAL v1 no impone un modelo concreto de “tokenId” fuera de EVM.

### 8.1 EVM Sepolia (testnet), ERC-721, tokenId=1234

```
https://resolver.geditok.eu/ual/v1/eip155:11155111/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234
```

### 8.2 EVM Ethereum mainnet, ERC-721, tokenId=9876

```
https://resolver.geditok.eu/ual/v1/eip155:1/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/9876
```

### 8.3 Bitcoin mainnet (familia Bitcoin vía BIP122), plantilla

CAIP-2 define ejemplos con namespace `bip122` (Bitcoin mainnet incluido). La referencia de Bitcoin mainnet (prefijo del hash génesis) se expresa en 32 hex.

```
https://resolver.geditok.eu/ual/v1/bip122:000000000019d6689c085ae165831e93/asset/{caip19}
```

Donde `{caip19}` es un **identificador de activo** (CAIP-19 o compatible). En UAL v1 se usa aquí como **marcador genérico** para activos no-EVM (cuando no aplica `contrato/tokenId`).

### 8.4 Cosmos Hub (ecosistema Cosmos), plantilla

CAIP-2 define ejemplos con namespace `cosmos` y referencias tipo `cosmoshub-*`.

```
https://resolver.geditok.eu/ual/v1/cosmos:cosmoshub-3/asset/{caip19}
```

### 8.5 Solana mainnet, plantilla

En CAIP-2, Solana se identifica como `solana:{reference}`. En la práctica se usa como referencia un identificador de la red (p. ej. un identificador conocido del entorno).

```
https://resolver.geditok.eu/ual/v1/solana:5eykt4UsFv8P8NJdTREpY1vzqKqZKvdp/asset/{caip19}
```

### 8.6 Polkadot (namespace polkadot), plantilla

CAIP-13 define que el namespace `polkadot` usa como `reference` el **prefijo de 32 caracteres** del `genesis-hash` en hex minúscula.

```
https://resolver.geditok.eu/ual/v1/polkadot:{genesisHashPrefix32}/asset/{caip19}
```

### 8.7 Cardano (nota de interoperabilidad)

UAL v1 **admite cualquier** `{caip2}` definido por CAIP-2 y sus perfiles de namespace. Si el namespace Cardano no estuviera aún formalizado en el catálogo de namespaces, se recomienda usar `asset/{caip19}` bajo el `{caip2}` que se establezca oficialmente cuando exista el perfil correspondiente.

Plantilla:

```
https://resolver.geditok.eu/ual/v1/{caip2-cardano}/asset/{caip19}
```

## 9. Ejemplo de respuesta JSON (mínima)

```json
{
  "ual": "https://resolver.geditok.eu/ual/v1/eip155:11155111/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234",
  "version": "v1",
  "caip2": "eip155:11155111",
  "assetRef": "nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234",
  "idCanonico": "ual:v1:eip155:11155111/nft/erc721/0xAbCDEF0123456789aBCdef0123456789ABCDef01/1234",
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

## 10. Seguridad y consideraciones de confianza

Los implementadores **DEBERÍAN** considerar:

* **Phishing / resolvers maliciosos**: el dominio del resolver puede mentir sobre el asset.
  Mitigación: incluir `integrity` (CID/hash) y, cuando sea posible, pruebas verificables (firmas, anclajes on-chain).
* **TLS obligatorio**: UAL producción **DEBE** ser `https`.
* **Cache**: representar correctamente `Cache-Control` (especialmente si hay datos regulados o revocaciones).
* **Privacidad**: en DPP, separar datos públicos de datos restringidos; evitar exponer PII en JSON-LD público.

## 11. Versionado y compatibilidad

* La versión está fijada en el path: `/ual/v1/`.
* Cambios incompatibles **DEBEN** incrementar versión mayor (`v2`, etc.).
* Cambios compatibles **DEBERÍAN** hacerse mediante:

  * nuevos campos en la respuesta JSON/JSON-LD
  * nuevos query params no semánticos
  * nuevas subrutas dentro de `assetRef` solo si no rompen parseo (preferible reservar y documentar)

## 12. Registro de extensiones (opcional)

Se recomienda mantener en el repositorio un archivo `REGISTRY.md` con:

* `assetRef` adicionales (p. ej., `ft/...`, `bundle/...`, `dpp/...`)
* enumeraciones de `view`
* extensiones sectoriales para `granularity`

---

## 13. Checklist de conformidad (práctico)

Un resolver conforme con UAL v1:

* [ ] Acepta URLs `https://{domain}/ual/v1/{caip2}/{assetRef}`
* [ ] Parsea `{caip2}` y `{assetRef}` según §4
* [ ] Devuelve **HTML** o **JSON** como mínimo
* [ ] Devuelve representación machine-readable con `idCanonico`
* [ ] Redirige (308) a forma canónica cuando procede
* [ ] Implementa códigos 400/404/403 de forma coherente

---

## 14. Referencias técnicas (no exhaustivas)

* CAIP-2 (Blockchain ID)
* CAIP-4 (BIP122)
* CAIP-5 (Cosmos)
* CAIP-13 (Polkadot)
* CAIP-30 (Solana)
* CAIP-19 (Asset Type & ID)
* Chain Agnostic Namespaces (catálogo de namespaces)
