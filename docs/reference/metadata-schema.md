# Référence — Schéma de métadonnées

Chaque package du registre embarque un fichier `metadata.json` à la racine du module Python. Ce fichier est à la fois **lisible par les machines** (JSON-LD, CPSV-AP) et **exploitable par les outils du registre** (`regalgo-validator`, index SPARQL).

---

## Fichier `metadata.json` — Structure complète

```json
{
  "@context": {
    "cpsv":    "http://purl.org/vocab/cpsv#",
    "cv":      "http://data.europa.eu/m8g/",
    "dct":     "http://purl.org/dc/terms/",
    "owl":     "http://www.w3.org/2002/07/owl#",
    "xsd":     "http://www.w3.org/2001/XMLSchema#",
    "eli":     "http://data.europa.eu/eli/ontology#",
    "regalgo": "https://registre-algo.gouv.fr/ns#",
    "skos":    "http://www.w3.org/2004/02/skos/core#"
  },

  "@type": ["cpsv:PublicService", "regalgo:RegulatoryAlgorithm"],
  "@id": "https://registre-algo.gouv.fr/algo/finance/lcr/v1",

  "dct:identifier":   "finance.lcr.v1",
  "dct:title":        {"@value": "Liquidity Coverage Ratio", "@language": "fr"},
  "dct:description":  {
    "@value": "Calcul du ratio de couverture des besoins de liquidité selon CRR2 Art. 412.",
    "@language": "fr"
  },
  "dct:language": {
    "@id": "http://publications.europa.eu/resource/authority/language/FRA"
  },

  "regalgo:pypiPackage":       "regalgo-finance-lcr",
  "regalgo:registrySchemaVersion": "1.0",

  "owl:versionInfo": "1.0.0",
  "dct:valid": {
    "@type": "xsd:dateTimeInterval",
    "dct:start": {"@value": "2024-01-01", "@type": "xsd:date"}
  },

  "dct:subject": {
    "@id": "https://registre-algo.gouv.fr/thesaurus/domaine/finance"
  },

  "cv:hasCompetentAuthority": {
    "@type": "cv:PublicOrganisation",
    "@id": "https://registre-algo.gouv.fr/org/eba",
    "dct:title": "European Banking Authority",
    "owl:sameAs": {
      "@id": "http://publications.europa.eu/resource/authority/corporate-body/EBA"
    }
  },

  "cv:hasLegalResource": {
    "@type": "cv:LegalResource",
    "@id": "https://registre-algo.gouv.fr/legal/crr2-art412",
    "dct:title": {"@value": "CRR2 — Article 412 — Exigences de liquidité", "@language": "fr"},
    "eli:id_local": "CRR2/Art.412",
    "owl:sameAs": {"@id": "http://data.europa.eu/eli/reg/2013/575/oj"}
  },

  "cpsv:hasInput": [
    {
      "@type": ["cv:Evidence", "regalgo:AlgorithmInput"],
      "dct:identifier": "hqla",
      "dct:type":       {"@value": "float"},
      "cv:value":       {"@value": "EUR"},
      "dct:description": {"@value": "High Quality Liquid Assets", "@language": "fr"}
    },
    {
      "@type": ["cv:Evidence", "regalgo:AlgorithmInput"],
      "dct:identifier": "net_outflows",
      "dct:type":       {"@value": "float"},
      "cv:value":       {"@value": "EUR"},
      "dct:description": {"@value": "Sorties nettes de trésorerie sur 30 jours", "@language": "fr"}
    }
  ],

  "cpsv:produces": {
    "@type": "cpsv:Output",
    "@id":   "https://registre-algo.gouv.fr/output/lcr-ratio",
    "dct:identifier": "lcr_ratio",
    "dct:type":       {"@value": "float"},
    "cv:value":       {"@value": "ratio"},
    "dct:description": {
      "@value": "Ratio LCR : HQLA / sorties nettes. Seuil réglementaire : >= 1.0",
      "@language": "fr"
    }
  },

  "owl:sameAs": [
    {"@id": "https://pypi.org/project/regalgo-finance-lcr/"},
    {"@id": "https://github.com/your-org/regalgo-finance-lcr"}
  ],

  "skos:broader": {
    "@id": "https://registre-algo.gouv.fr/algo/finance/liquidite"
  },

  "dct:replaces": null,

  "regalgo:status": "stable",

  "regalgo:changelog": [
    {
      "owl:versionInfo": "1.0.0",
      "dct:date":        {"@value": "2024-01-15", "@type": "xsd:date"},
      "regalgo:changeType": "MAJOR",
      "dct:description":    {"@value": "Version initiale", "@language": "fr"}
    }
  ]
}
```

---

## Champs obligatoires vs recommandés

| Champ JSON-LD | Obligation | Validation |
|---|---|---|
| `@context` | **Obligatoire** | Doit inclure `cpsv`, `cv`, `dct`, `regalgo` |
| `@type` | **Obligatoire** | Doit contenir `cpsv:PublicService` |
| `dct:identifier` | **Obligatoire** | Format `<domaine>.<nom>.<vMajeure>` |
| `dct:title` | **Obligatoire** | `@language` : `fr` ou `en` |
| `regalgo:pypiPackage` | **Obligatoire** | Doit correspondre au nom PyPI exact |
| `owl:versionInfo` | **Obligatoire** | SemVer `MAJOR.MINOR.PATCH` |
| `cv:hasLegalResource` | **Obligatoire** | Avec `eli:id_local` ou `owl:sameAs` EUR-Lex |
| `cv:hasCompetentAuthority` | **Obligatoire** | |
| `cpsv:hasInput` | **Obligatoire** | Au moins un élément |
| `cpsv:produces` | **Obligatoire** | Un seul output principal |
| `dct:description` | Recommandé | |
| `owl:sameAs` (EUR-Lex) | Recommandé | Améliore l'interopérabilité |
| `dct:valid` | Recommandé | Période de validité réglementaire |
| `regalgo:changelog` | Recommandé | Traçabilité des évolutions |
| `dct:replaces` | Recommandé si applicable | Référence la version précédente |

---

## Charger les métadonnées depuis Python

```python
import json
from pathlib import Path
from importlib.resources import files

# Depuis le package installé
metadata_path = files("regalgo_finance_lcr").joinpath("metadata.json")
metadata = json.loads(metadata_path.read_text())

# Accès typé
algo_id      = metadata["dct:identifier"]       # "finance.lcr.v1"
legal_ref    = metadata["cv:hasLegalResource"]["eli:id_local"]  # "CRR2/Art.412"
pypi_package = metadata["regalgo:pypiPackage"]  # "regalgo-finance-lcr"
```

---

## Valider le fichier metadata.json

```bash
pip install regalgo-validator
regalgo-validator check-metadata path/to/metadata.json
```

```
✅ @context : namespaces requis présents
✅ @type    : cpsv:PublicService ✓
✅ dct:identifier format valide : finance.lcr.v1
✅ cv:hasLegalResource : eli:id_local présent
✅ cpsv:hasInput : 2 entrées déclarées
✅ cpsv:produces : output déclaré
⚠️  owl:sameAs EUR-Lex absent (recommandé)
```

---

## Voir aussi

- [Vocabulaire commun — définitions](vocabulary.md)
- [Contrat d'interface Python](interface-contract.md)
- [Guide : Déclarer les métadonnées réglementaires](../how-to-guides/metadata.md)
- [CPSV-AP — spécification complète](https://semiceu.github.io/CPSV-AP/)
- [ELI — European Legislation Identifier](https://eur-lex.europa.eu/eli-register/about.html)
