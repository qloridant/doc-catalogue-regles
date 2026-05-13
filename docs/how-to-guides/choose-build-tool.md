# Guide — Choisir son outil de build

Ce guide vous aide à choisir entre Hatchling, Setuptools et Poetry pour packager votre algorithme réglementaire.

---

## Tableau comparatif

| Critère | Hatchling | Setuptools | Poetry |
|---|---|---|---|
| Standard `pyproject.toml` pur | ✅ | ✅ | ✅ |
| Recommandé par PyPA (2024) | ✅ Oui | ⚠️ Héritage | — |
| Gestion des dépendances intégrée | ❌ | ❌ | ✅ |
| Lock file | ❌ (séparé) | ❌ | ✅ `poetry.lock` |
| Configuration `pyproject.toml` | Simple | Verbeuse | Propre |
| Monorepo / src layout | ✅ Natif | ✅ | ✅ |
| Inclusion de fichiers non-Python | ✅ Auto | ⚠️ Manuel | ⚠️ Manuel |

---

## Recommandation

**Pour un nouveau package :** → Hatchling

- Standard actuel recommandé par la Python Packaging Authority
- Inclut automatiquement `metadata.json` dans le wheel (pas de configuration supplémentaire)
- Syntaxe `pyproject.toml` la plus concise

**Si votre organisation utilise déjà Poetry :** → Gardez Poetry

**Si vous migrez un package existant avec Setuptools :** → Conservez Setuptools pour éviter une migration risquée

---

## Configuration minimale par outil

=== "Hatchling"

    ```toml
    [build-system]
    requires      = ["hatchling"]
    build-backend = "hatchling.build"

    [project]
    name    = "regalgo-civique-droit-vote"
    version = "1.0.0"
    requires-python = ">=3.10"
    dependencies = ["regalgo-core>=1.0,<2.0"]

    [tool.hatch.build.targets.wheel]
    packages = ["src/regalgo_civique_droit_vote"]
    ```

=== "Setuptools"

    ```toml
    [build-system]
    requires      = ["setuptools>=68", "wheel"]
    build-backend = "setuptools.backends.legacy:build"

    [project]
    name    = "regalgo-civique-droit-vote"
    version = "1.0.0"

    [tool.setuptools.packages.find]
    where = ["src"]

    [tool.setuptools.package-data]
    "*" = ["metadata.json"]
    ```

=== "Poetry"

    ```toml
    [tool.poetry]
    name     = "regalgo-civique-droit-vote"
    version  = "1.0.0"
    packages = [{include = "regalgo_civique_droit_vote", from = "src"}]
    include  = ["src/regalgo_civique_droit_vote/metadata.json"]

    [tool.poetry.dependencies]
    python      = "^3.10"
    regalgo-core = ">=1.0,<2.0"

    [build-system]
    requires      = ["poetry-core"]
    build-backend = "poetry.core.masonry.api"
    ```

---

## Voir aussi

- [Structure de package standard](../reference/package-structure.md)
- [Tutoriel 01 — Mon premier algorithme packagé](../tutorials/01-first-package.md)
