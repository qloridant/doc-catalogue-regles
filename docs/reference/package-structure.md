# Référence — Structure de package standard

---

## Arborescence complète

```
regalgo-finance-lcr/
│
├── pyproject.toml                  # Configuration du build et métadonnées PyPI
├── README.md                       # Description affichée sur PyPI
├── LICENSE                         # Licence open source
│
├── src/
│   └── regalgo_finance_lcr/
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
__algo_id__  = "finance.lcr.v1"
```

---

## `pyproject.toml` — sections minimales

```toml
[build-system]
# Hatchling, setuptools ou poetry-core — au choix

[project]
name            = "regalgo-finance-lcr"
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
    packages = ["src/regalgo_finance_lcr"]
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
    packages = [{include = "regalgo_finance_lcr", from = "src"}]
    include  = ["src/regalgo_finance_lcr/metadata.json"]
    ```

---

## Tests de conformité requis

Le fichier `tests/test_compliance.py` doit au minimum vérifier :

```python
from regalgo_core import AlgorithmProtocol
from regalgo_finance_lcr import LCRAlgorithm, AlgoInput
import json
from importlib.resources import files

def test_implements_protocol():
    assert isinstance(LCRAlgorithm(), AlgorithmProtocol)

def test_metadata_json_present():
    path = files("regalgo_finance_lcr").joinpath("metadata.json")
    assert path.is_file()

def test_metadata_json_valid():
    path = files("regalgo_finance_lcr").joinpath("metadata.json")
    meta = json.loads(path.read_text())
    assert "dct:identifier" in meta
    assert "cv:hasLegalResource" in meta
    assert "cpsv:produces" in meta
    assert meta["regalgo:pypiPackage"].startswith("regalgo-")

def test_result_has_regulation():
    algo = LCRAlgorithm()
    result = algo.compute(AlgoInput(data={"hqla": 100.0, "net_outflows": 80.0}))
    assert result.regulation.get("text")
    assert result.regulation.get("article")
    assert result.inputs_snapshot is not None
```
