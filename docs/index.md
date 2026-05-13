# Registre des Algorithmes Réglementaires

> **Packager, publier et interconnecter des algorithmes réglementaires sur PyPI — en s'appuyant sur les Core Vocabularies de l'Union Européenne.**

---

## Pourquoi ce registre ?

Les algorithmes réglementaires — éligibilité à un droit, calculs prudentiels, scoring de conformité, formules normatives — sont aujourd'hui dispersés dans des silos applicatifs, réimplémentés à l'identique dans chaque organisation, et impossibles à auditer de manière transversale.

Ce standard de packaging s'appuie sur les **Core Vocabularies SEMIC** pour décrire sémantiquement les algorithmes réglementaires comme des `cpsv:PublicService` producteurs de `cpsv:Output`, encadrés par des `cv:LegalResource`, et fournis par des `cv:PublicOrganisation`.

Résultat : des packages Python qui sont à la fois **installables** et **compréhensibles par les machines**.

| Propriété | Ce que ça signifie concrètement |
|---|---|
| **Découvrables** | Indexés par texte réglementaire, domaine, `cv:LegalResource` |
| **Interopérables** | Contrat d'interface commun (`AlgorithmProtocol`) + métadonnées JSON-LD |
| **Auditables** | Chaque résultat embarque sa référence normative (`cv:hasLegalResource`) |
| **Alignés EU** | Namespaces CPSV-AP, `owl:sameAs` vers EUR-Lex et EU Vocabularies |

---

## Ancrage dans les Core Vocabularies

Les métadonnées de chaque algorithme sont exprimées dans le vocabulaire **CPSV-AP** (Core Public Service Vocabulary Application Profile) :

```turtle
@prefix cpsv: <http://purl.org/vocab/cpsv#> .
@prefix cv:   <http://data.europa.eu/m8g/> .
@prefix dct:  <http://purl.org/dc/terms/> .

<https://registre-algo.gouv.fr/algo/civique/droit-vote/v1>
    a cpsv:PublicService ;
    dct:title               "Droit de vote en France — Code électoral Art. L.2"@fr ;
    cv:hasCompetentAuthority <https://registre-algo.gouv.fr/org/mint> ;
    cv:hasLegalResource     <https://www.legifrance.gouv.fr/codes/id/LEGITEXT000006070239/> ;
    cpsv:produces           <https://registre-algo.gouv.fr/output/peut-voter> ;
    owl:sameAs              <https://registre-algo.gouv.fr/pypi/regalgo-civique-droit-vote> .
```

---

## Par où commencer ?

<div class="grid cards" markdown>

- :material-school: **Tutoriels**

    Apprenez en faisant. Du zéro au package publié sur PyPI en moins d'une heure.

    [→ Commencer le tutoriel](tutorials/index.md)

- :material-wrench: **Guides pratiques**

    Des recettes ciblées : CI/CD, versioning réglementaire, interopérabilité sémantique.

    [→ Voir les guides](how-to-guides/index.md)

- :material-book-open-variant: **Référence**

    Spécifications complètes : schéma de métadonnées JSON-LD, contrat d'interface, conventions.

    [→ Consulter la référence](reference/index.md)

- :material-lightbulb: **Concepts**

    Comprendre l'architecture, les choix de conception, la gouvernance du registre.

    [→ Explorer les concepts](explanation/index.md)

</div>

---

## Exemple express

=== "Python"

    ```python
    # pip install regalgo-civique-droit-vote
    from regalgo_civique_droit_vote import DroitVoteAlgorithm, AlgoInput

    algo = DroitVoteAlgorithm()
    result = algo.compute(AlgoInput(data={
        "nationalite_francaise": True,
        "age": 25,
        "capacite_civique": True,
        "inscrit_listes_electorales": True
    }))

    print(result.value)              # True
    print(result.regulation)        # {'text': 'Code électoral', 'article': 'Art. L.2', ...}
    print(result.jsonld_context())  # Contexte JSON-LD CPSV-AP complet
    ```

=== "Catala"

    ```catala
    > Utilisation Code_Electoral_Français

    déclaration champ d'application DroitDeVote:
      entrée nationalite_francaise contenu booléen
      entrée age contenu entier
      entrée capacite_civique contenu booléen
      entrée inscrit_listes_electorales contenu booléen
      résultat peut_voter contenu booléen

    champ d'application DroitDeVote:
      # Art. L.2 — nationalité, Art. L.3 — majorité,
      # Art. L.5-L.6 — capacité civique, Art. L.7 — inscription
      définition peut_voter égal à
        nationalite_francaise et
        age >= 18 et
        capacite_civique et
        inscrit_listes_electorales
    ```

---

## Architecture en un coup d'œil

```mermaid
graph TB
    subgraph "Standard de packaging"
        A[pyproject.toml] --> B[Package PyPI]
        B --> C[metadata.json-ld]
    end
    subgraph "Core Vocabularies UE"
        D[cpsv:PublicService]
        E[cv:LegalResource]
        F[cpsv:Output]
    end
    subgraph "Registre public"
        G[Index SPARQL]
        H[Découverte]
    end
    C -->|owl:sameAs| D
    D -->|cv:hasLegalResource| E
    D -->|cpsv:produces| F
    B -->|notify| G
    G --> H
```

---

!!! info "Convention de nommage"
    Tous les packages du registre respectent le préfixe `regalgo-<domaine>-<nom>`.
    Exemple : `regalgo-civique-droit-vote`, `regalgo-finance-nsfr`.
    Voir les [conventions de nommage](reference/naming-conventions.md).

!!! note "Relation avec le vocabulaire commun DINUM"
    Ce registre s'inscrit dans la démarche de Web sémantique des administrations françaises
    portée par la DINUM. Les Core Vocabularies utilisés ici sont documentés sur
    [vocabulaire-commun](https://qloridant.github.io/vocabulaire-commun/).
