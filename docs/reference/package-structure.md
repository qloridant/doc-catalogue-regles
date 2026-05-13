# Référence — Structure de package standard

---

## Arborescence complète

```
regalgo-civique-droit-vote/
│
├── pyproject.toml                  # Configuration du build et métadonnées PyPI
├── README.md                       # Description affichée sur PyPI
├── LICENSE                         # Licence open source
│
├── src/
│   └── regalgo_civique_droit_vote/
│       ├── __init__.py             # Exports publics du package
│       ├── algorithm.py            # Implémentation principale
│       ├── metadata.json           # Métadonnées JSON-LD CPSV-AP (OBLIGATOIRE)
│       └── exceptions.py           # Exceptions spécifiques (optionnel)
│
├── tests/
│   ├── test_algorithm.py           # Tests unitaires
│   ├── test_compliance.py          # Tests de conformité réglementaire
│   └── test_metadata.py            # Tests de validité du metadata.json
│
└── .github/
    └── workflows/
        ├── ci.yml                  # Tests sur PR
        └── release.yml             # Publication automatique
```

---

## Fichiers obligatoires

| Fichier | Rôle | Validation |
|---|---|---|
| `pyproject.toml` | Configuration build + métadonnées PyPI | `twine check` |
| `src/<module>/metadata.json` | Métadonnées JSON-LD CPSV-AP | `regalgo-validator check-metadata` |
| `src/<module>/__init__.py` | Exports publics | `AlgorithmProtocol` doit être implémenté |
| `README.md` | Description PyPI | Rendu Markdown valide |

---

## `__init__.py` — exports attendus

```python
# Exports publics minimaux requis par le registre
from .algorithm import <NomAlgorithme>
from regalgo_core import AlgoInput, AlgoResult

__all__ = ["<NomAlgorithme>", "AlgoInput", "AlgoResult"]

# Métadonnées de version — doit correspondre à metadata.json
__version__ = "1.0.0"
__algo_id__  = "civique.droit-vote.v1"
```

---

## `pyproject.toml` — sections minimales

```toml
[build-system]
# Hatchling, setuptools ou poetry-core — au choix

[project]
name            = "regalgo-civique-droit-vote"
version         = "1.0.0"
requires-python = ">=3.10"
dependencies    = ["regalgo-core>=1.0,<2.0"]

# Classifiers obligatoires pour l'indexation registre
classifiers = [
    "Programming Language :: Python :: 3",
    "Intended Audience :: Developers",
    "Topic :: Office/Business",
    "License :: OSI Approved :: MIT License",
]

# Keyword obligatoire pour la découverte
keywords = ["regalgo", "<domaine>", "<nom-court>"]

# Lien vers le registre public (obligatoire)
[project.urls]
"Registry" = "https://registre-algo.gouv.fr/algo/<domaine>/<nom>/<vMajeure>"
```

---

## Inclusion de `metadata.json` dans le wheel

Le fichier `metadata.json` **doit être embarqué** dans le wheel Python. Selon le build backend :

=== "Hatchling"
    ```toml
    [tool.hatch.build.targets.wheel]
    packages = ["src/regalgo_civique_droit_vote"]
    # Les fichiers non-Python sont inclus automatiquement
    ```

=== "Setuptools"
    ```toml
    [tool.setuptools.package-data]
    "*" = ["metadata.json"]
    ```

=== "Poetry"
    ```toml
    [tool.poetry]
    packages = [{include = "regalgo_civique_droit_vote", from = "src"}]
    include  = ["src/regalgo_civique_droit_vote/metadata.json"]
    ```

---

## Tests de conformité requis

Le fichier `tests/test_compliance.py` doit au minimum vérifier :

```python
from regalgo_core import AlgorithmProtocol
from regalgo_civique_droit_vote import DroitVoteAlgorithm, AlgoInput
import json
from importlib.resources import files

def test_implements_protocol():
    assert isinstance(DroitVoteAlgorithm(), AlgorithmProtocol)

def test_metadata_json_present():
    path = files("regalgo_civique_droit_vote").joinpath("metadata.json")
    assert path.is_file()

def test_metadata_json_valid():
    path = files("regalgo_civique_droit_vote").joinpath("metadata.json")
    meta = json.loads(path.read_text())
    assert "dct:identifier" in meta
    assert "cv:hasLegalResource" in meta
    assert "cpsv:produces" in meta
    assert meta["regalgo:pypiPackage"].startswith("regalgo-")

def test_result_has_regulation():
    algo = DroitVoteAlgorithm()
    result = algo.compute(AlgoInput(data={
        "nationalite_francaise": True,
        "age": 25,
        "capacite_civique": True,
        "inscrit_listes_electorales": True,
    }))
    assert result.regulation.get("text")
    assert result.regulation.get("article")
    assert result.inputs_snapshot is not None
```
