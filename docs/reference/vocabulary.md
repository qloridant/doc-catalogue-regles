# Référence — Vocabulaire commun

Cette page définit les termes et namespaces utilisés dans le registre des algorithmes réglementaires. Elle s'appuie sur les **Core Vocabularies de l'Union Européenne** maintenus par [SEMIC](https://joinup.ec.europa.eu/collection/semantic-interoperability-community-semic).

---

## Namespaces utilisés

| Préfixe | Namespace | Vocabulaire |
|---|---|---|
| `cpsv:` | `http://purl.org/vocab/cpsv#` | Core Public Service Vocabulary |
| `cv:` | `http://data.europa.eu/m8g/` | Core Vocabulary (CCCEV, CPSV-AP extensions) |
| `dct:` | `http://purl.org/dc/terms/` | Dublin Core Terms |
| `owl:` | `http://www.w3.org/2002/07/owl#` | Web Ontology Language |
| `skos:` | `http://www.w3.org/2004/02/skos/core#` | SKOS |
| `xsd:` | `http://www.w3.org/2001/XMLSchema#` | XML Schema |
| `eli:` | `http://data.europa.eu/eli/ontology#` | European Legislation Identifier |
| `regalgo:` | `https://registre-algo.gouv.fr/ns#` | Namespace propre au registre |

---

## Mapping : concept du registre → Core Vocabulary

| Concept du registre | Classe CPSV-AP | Description |
|---|---|---|
| **Algorithme réglementaire** | `cpsv:PublicService` | Un algorithme est modélisé comme un service public numérique |
| **Résultat de calcul** | `cpsv:Output` | La valeur produite par l'algorithme |
| **Texte réglementaire** | `cv:LegalResource` | Le texte de loi ou règlement encadrant l'algorithme |
| **Autorité émettrice** | `cv:PublicOrganisation` | L'organisme responsable (EBA, AMF, HAS…) |
| **Entrée de l'algorithme** | `cv:Evidence` | Les données nécessaires à l'exécution |
| **Domaine réglementaire** | `dct:subject` + `skos:Concept` | Finance, santé, environnement… |
| **Version normative** | `owl:versionInfo` + `dct:valid` | Plage de validité temporelle |

---

## Termes du registre

### `regalgo:RegulatoryAlgorithm`

Sous-classe de `cpsv:PublicService`. Tout algorithme enregistré dans le registre est une instance de cette classe.

**Propriétés obligatoires :**

| Propriété | Type | Description |
|---|---|---|
| `dct:title` | `rdfs:Literal` | Nom complet de l'algorithme |
| `dct:identifier` | `xsd:string` | Identifiant unique : `<domaine>.<nom>.<version_majeure>` |
| `cv:hasLegalResource` | `cv:LegalResource` | Texte réglementaire de référence |
| `cv:hasCompetentAuthority` | `cv:PublicOrganisation` | Autorité réglementaire |
| `cpsv:produces` | `cpsv:Output` | Résultat produit |
| `regalgo:pypiPackage` | `xsd:string` | Nom du package PyPI |

**Propriétés recommandées :**

| Propriété | Type | Description |
|---|---|---|
| `dct:description` | `rdfs:Literal` | Description en langage naturel |
| `owl:sameAs` | `owl:Thing` | Alignement EUR-Lex, Wikidata, etc. |
| `dct:valid` | `xsd:dateTimeInterval` | Période de validité réglementaire |
| `dct:replaces` | `regalgo:RegulatoryAlgorithm` | Version précédente remplacée |
| `dct:language` | `dct:LinguisticSystem` | Langue(s) de référence de la norme |

---

### `cv:LegalResource` dans le contexte du registre

Chaque algorithme doit référencer son texte réglementaire via le namespace **ELI** (European Legislation Identifier) :

```turtle
<https://registre-algo.gouv.fr/legal/crr2-art412>
    a cv:LegalResource ;
    dct:title    "CRR2 — Article 412 — Exigences de liquidité"@fr ;
    eli:id_local "CRR2/Art.412" ;
    owl:sameAs   <http://data.europa.eu/eli/reg/2013/575/oj> .
```

---

### `cv:PublicOrganisation` — Autorités réglementaires alignées

| Autorité | URI de référence |
|---|---|
| EBA (finance) | `https://www.eba.europa.eu` |
| AMF (marchés) | `https://www.amf-france.org` |
| HAS (santé) | `https://www.has-sante.fr` |
| ADEME (environnement) | `https://www.ademe.fr` |
| ACPR | `https://acpr.banque-france.fr` |

Utiliser `owl:sameAs` pour aligner avec le référentiel EU :
```turtle
<https://registre-algo.gouv.fr/org/eba>
    a cv:PublicOrganisation ;
    owl:sameAs <http://publications.europa.eu/resource/authority/corporate-body/EBA> .
```

---

### `regalgo:AlgorithmInput` (sous-classe de `cv:Evidence`)

Représente une entrée de l'algorithme.

| Propriété | Type | Description |
|---|---|---|
| `dct:identifier` | `xsd:string` | Nom du paramètre (ex : `hqla`) |
| `dct:type` | `xsd:string` | Type Python (`float`, `int`, `str`…) |
| `cv:value` | `rdfs:Literal` | Unité (EUR, ratio, jours…) |
| `dct:description` | `rdfs:Literal` | Description en langage naturel |

---

## Domaines réglementaires (`dct:subject`)

Les domaines sont des `skos:Concept` issus du thésaurus réglementaire du registre :

| Concept | URI |
|---|---|
| Finance / prudentiel | `https://registre-algo.gouv.fr/thesaurus/domaine/finance` |
| Santé publique | `https://registre-algo.gouv.fr/thesaurus/domaine/sante` |
| Environnement / ESG | `https://registre-algo.gouv.fr/thesaurus/domaine/environnement` |
| Fiscalité | `https://registre-algo.gouv.fr/thesaurus/domaine/fiscalite` |
| Urbanisme | `https://registre-algo.gouv.fr/thesaurus/domaine/urbanisme` |

---

## Voir aussi

- [Schéma de métadonnées complet](metadata-schema.md)
- [Contrat d'interface Python](interface-contract.md)
- [Vocabulaire commun DINUM](https://qloridant.github.io/vocabulaire-commun/)
- [CPSV-AP sur Joinup](https://joinup.ec.europa.eu/collection/semantic-interoperability-community-semic/solution/core-public-service-vocabulary-application-profile)
- [EU Vocabularies — Publications Office](https://op.europa.eu/en/web/eu-vocabularies)
