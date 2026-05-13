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
  "@id": "https://registre-algo.gouv.fr/algo/civique/droit-vote/v1",

  "dct:identifier":   "civique.droit-vote.v1",
  "dct:title":        {"@value": "Droit de vote en France", "@language": "fr"},
  "dct:description":  {
    "@value": "Détermine si une personne a le droit de voter en France selon le Code électoral Art. L.2 à L.7.",
    "@language": "fr"
  },
  "dct:language": {
    "@id": "http://publications.europa.eu/resource/authority/language/FRA"
  },

  "regalgo:pypiPackage":           "regalgo-civique-droit-vote",
  "regalgo:registrySchemaVersion": "1.0",

  "owl:versionInfo": "1.0.0",
  "dct:valid": {
    "@type": "xsd:dateTimeInterval",
    "dct:start": {"@value": "2024-01-01", "@type": "xsd:date"}
  },

  "dct:subject": {
    "@id": "https://registre-algo.gouv.fr/thesaurus/domaine/civique"
  },

  "cv:hasCompetentAuthority": {
    "@type": "cv:PublicOrganisation",
    "@id": "https://registre-algo.gouv.fr/org/mint",
    "dct:title": "Ministère de l'Intérieur",
    "owl:sameAs": {
      "@id": "http://publications.europa.eu/resource/authority/corporate-body/MINT"
    }
  },

  "cv:hasLegalResource": {
    "@type": "cv:LegalResource",
    "@id": "https://registre-algo.gouv.fr/legal/code-electoral-l2",
    "dct:title": {"@value": "Code électoral — Articles L.2, L.5, L.6, L.7 — Conditions du droit de vote", "@language": "fr"},
    "eli:id_local": "CodeElectoral/Art.L2",
    "owl:sameAs": {"@id": "https://www.legifrance.gouv.fr/codes/id/LEGITEXT000006070239/"}
  },

  "cpsv:hasInput": [
    {
      "@type": ["cv:Evidence", "regalgo:AlgorithmInput"],
      "dct:identifier": "nationalite_francaise",
      "dct:type":       {"@value": "bool"},
      "dct:description": {"@value": "Possède la nationalité française (Art. L.2)", "@language": "fr"}
    },
    {
      "@type": ["cv:Evidence", "regalgo:AlgorithmInput"],
      "dct:identifier": "age",
      "dct:type":       {"@value": "int"},
      "cv:value":       {"@value": "années"},
      "dct:description": {"@value": "Âge de la personne en années", "@language": "fr"}
    },
    {
      "@type": ["cv:Evidence", "regalgo:AlgorithmInput"],
      "dct:identifier": "capacite_civique",
      "dct:type":       {"@value": "bool"},
      "dct:description": {"@value": "Non privé de ses droits civiques (Art. L.5, L.6)", "@language": "fr"}
    },
    {
      "@type": ["cv:Evidence", "regalgo:AlgorithmInput"],
      "dct:identifier": "inscrit_listes_electorales",
      "dct:type":       {"@value": "bool"},
      "dct:description": {"@value": "Inscrit sur les listes électorales (Art. L.7)", "@language": "fr"}
    }
  ],

  "cpsv:produces": {
    "@type": "cpsv:Output",
    "@id":   "https://registre-algo.gouv.fr/output/peut-voter",
    "dct:identifier": "peut_voter",
    "dct:type":       {"@value": "bool"},
    "dct:description": {
      "@value": "Vrai si la personne remplit les quatre conditions cumulatives du droit de vote",
      "@language": "fr"
    }
  },

  "owl:sameAs": [
    {"@id": "https://pypi.org/project/regalgo-civique-droit-vote/"},
    {"@id": "https://github.com/your-org/regalgo-civique-droit-vote"}
  ],

  "skos:broader": {
    "@id": "https://registre-algo.gouv.fr/algo/civique/eligibilite"
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
| `cv:hasLegalResource` | **Obligatoire** | Avec `eli:id_local` ou `owl:sameAs` Légifrance/EUR-Lex |
| `cv:hasCompetentAuthority` | **Obligatoire** | |
| `cpsv:hasInput` | **Obligatoire** | Au moins un élément |
| `cpsv:produces` | **Obligatoire** | Un seul output principal |
| `dct:description` | Recommandé | |
| `owl:sameAs` (Légifrance/EUR-Lex) | Recommandé | Améliore l'interopérabilité |
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
metadata_path = files("regalgo_civique_droit_vote").joinpath("metadata.json")
metadata = json.loads(metadata_path.read_text())

# Accès typé
algo_id      = metadata["dct:identifier"]       # "civique.droit-vote.v1"
legal_ref    = metadata["cv:hasLegalResource"]["eli:id_local"]  # "CodeElectoral/Art.L2"
pypi_package = metadata["regalgo:pypiPackage"]  # "regalgo-civique-droit-vote"
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
✅ dct:identifier format valide : civique.droit-vote.v1
✅ cv:hasLegalResource : eli:id_local présent
✅ cpsv:hasInput : 4 entrées déclarées
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
