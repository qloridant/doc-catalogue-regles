# Tutoriel 01 — Mon premier algorithme packagé

**Durée estimée :** ~20 minutes  
**Objectif :** Transformer un algorithme réglementaire Python brut en package installable, conforme au standard du registre.

À la fin de ce tutoriel, vous aurez un package qui :

- s'installe avec `pip install`
- expose le contrat d'interface standard (`AlgorithmProtocol`)
- embarque les métadonnées réglementaires minimales requises

---

## Prérequis

- Python 3.10 ou supérieur
- `pip` à jour : `pip install --upgrade pip`
- Connaissance basique de la structure d'un module Python

---

## 1. Créer la structure du projet

Créez un dossier pour votre algorithme. Nous utilisons ici un exemple fictif : un calcul de **ratio de liquidité** (domaine finance).

```bash
mkdir regalgo-finance-lcr
cd regalgo-finance-lcr
```

Créez la structure suivante :

```
regalgo-finance-lcr/
├── pyproject.toml
├── README.md
├── src/
│   └── regalgo_finance_lcr/
│       ├── __init__.py
│       ├── algorithm.py
│       └── metadata.json
└── tests/
    └── test_algorithm.py
```

```bash
mkdir -p src/regalgo_finance_lcr tests
touch pyproject.toml README.md
touch src/regalgo_finance_lcr/__init__.py
touch src/regalgo_finance_lcr/algorithm.py
touch src/regalgo_finance_lcr/metadata.json
touch tests/test_algorithm.py
```

---

## 2. Déclarer les métadonnées réglementaires

Ouvrez `src/regalgo_finance_lcr/metadata.json` et renseignez :

```json
{
  "registry_schema_version": "1.0",
  "algo_id": "finance.lcr.v1",
  "name": "Liquidity Coverage Ratio",
  "short_name": "LCR",
  "domain": "finance",
  "regulation": {
    "text": "CRR2",
    "article": "Art. 412",
    "authority": "EBA"
  },
  "version": "1.0.0",
  "effective_date": "2024-01-01",
  "inputs": [
    {"name": "hqla", "type": "float", "unit": "EUR", "description": "High Quality Liquid Assets"},
    {"name": "net_outflows", "type": "float", "unit": "EUR", "description": "Net cash outflows over 30 days"}
  ],
  "output": {
    "name": "lcr_ratio", "type": "float", "unit": "ratio"
  },
  "tags": ["liquidite", "prudentiel", "bale3"]
}
```

!!! note "Schéma complet"
    Toutes les clés possibles sont documentées dans la [référence du schéma de métadonnées](../reference/metadata-schema.md).

---

## 3. Implémenter l'algorithme

Ouvrez `src/regalgo_finance_lcr/algorithm.py` :

```python
from __future__ import annotations

import json
from pathlib import Path
from dataclasses import dataclass, field
from typing import Any


# --- Structures de données standard ---

@dataclass
class AlgoInput:
    """Entrée normalisée d'un algorithme réglementaire."""
    data: dict[str, Any]
    context: dict[str, Any] = field(default_factory=dict)


@dataclass
class AlgoResult:
    """Sortie normalisée d'un algorithme réglementaire."""
    value: Any
    algo_id: str
    regulation: dict[str, str]
    inputs_snapshot: dict[str, Any]
    metadata: dict[str, Any] = field(default_factory=dict)


# --- Implémentation de l'algorithme ---

class LCRAlgorithm:
    """
    Calcul du Liquidity Coverage Ratio (LCR) selon CRR2 Art. 412.

    Le LCR mesure la capacité d'un établissement à couvrir ses sorties
    nettes de trésorerie sur 30 jours avec des actifs liquides de haute qualité.

    Formula:
        LCR = HQLA / Net Cash Outflows >= 100%
    """

    def __init__(self) -> None:
        _meta_path = Path(__file__).parent / "metadata.json"
        self._metadata = json.loads(_meta_path.read_text())

    @property
    def algo_id(self) -> str:
        return self._metadata["algo_id"]

    @property
    def regulation(self) -> dict[str, str]:
        return self._metadata["regulation"]

    def compute(self, algo_input: AlgoInput) -> AlgoResult:
        """
        Calcule le LCR à partir des entrées fournies.

        Args:
            algo_input: Entrée contenant `hqla` et `net_outflows` (en EUR).

        Returns:
            AlgoResult avec le ratio LCR et la traçabilité complète.

        Raises:
            ValueError: Si net_outflows est nul ou négatif.
        """
        hqla = float(algo_input.data["hqla"])
        net_outflows = float(algo_input.data["net_outflows"])

        if net_outflows <= 0:
            raise ValueError(
                f"net_outflows doit être strictement positif, reçu : {net_outflows}"
            )

        lcr = hqla / net_outflows

        return AlgoResult(
            value=round(lcr, 6),
            algo_id=self.algo_id,
            regulation=self.regulation,
            inputs_snapshot=algo_input.data,
            metadata={
                "compliant": lcr >= 1.0,
                "threshold": 1.0,
                "unit": "ratio",
            },
        )
```

---

## 4. Exposer le package

Ouvrez `src/regalgo_finance_lcr/__init__.py` :

```python
from .algorithm import LCRAlgorithm, AlgoInput, AlgoResult

__all__ = ["LCRAlgorithm", "AlgoInput", "AlgoResult"]
```

---

## 5. Configurer `pyproject.toml`

=== "Hatchling (recommandé)"

    ```toml
    [build-system]
    requires = ["hatchling"]
    build-backend = "hatchling.build"

    [project]
    name = "regalgo-finance-lcr"
    version = "1.0.0"
    description = "Liquidity Coverage Ratio — CRR2 Art. 412"
    readme = "README.md"
    requires-python = ">=3.10"
    license = { text = "MIT" }
    keywords = ["reglementation", "finance", "lcr", "liquidite", "regalgo"]
    classifiers = [
        "Programming Language :: Python :: 3",
        "Topic :: Office/Business :: Financial",
        "Intended Audience :: Financial and Insurance Industry",
    ]

    [project.urls]
    "Registry" = "https://registre-algo.example.com/finance/lcr"
    "Source" = "https://github.com/your-org/regalgo-finance-lcr"

    [tool.hatch.build.targets.wheel]
    packages = ["src/regalgo_finance_lcr"]

    # Inclusion des métadonnées JSON dans le wheel
    [tool.hatch.build.targets.wheel.force-include]
    "src/regalgo_finance_lcr/metadata.json" = "regalgo_finance_lcr/metadata.json"
    ```

=== "Setuptools"

    ```toml
    [build-system]
    requires = ["setuptools>=68", "wheel"]
    build-backend = "setuptools.backends.legacy:build"

    [project]
    name = "regalgo-finance-lcr"
    version = "1.0.0"
    description = "Liquidity Coverage Ratio — CRR2 Art. 412"
    readme = "README.md"
    requires-python = ">=3.10"
    license = { text = "MIT" }
    keywords = ["reglementation", "finance", "lcr", "regalgo"]

    [tool.setuptools.packages.find]
    where = ["src"]

    [tool.setuptools.package-data]
    "*" = ["metadata.json"]
    ```

=== "Poetry"

    ```toml
    [tool.poetry]
    name = "regalgo-finance-lcr"
    version = "1.0.0"
    description = "Liquidity Coverage Ratio — CRR2 Art. 412"
    packages = [{include = "regalgo_finance_lcr", from = "src"}]

    [tool.poetry.dependencies]
    python = "^3.10"

    [build-system]
    requires = ["poetry-core"]
    build-backend = "poetry.core.masonry.api"
    ```

---

## 6. Écrire un test minimal

Ouvrez `tests/test_algorithm.py` :

```python
import pytest
from regalgo_finance_lcr import LCRAlgorithm, AlgoInput


def test_lcr_above_threshold():
    algo = LCRAlgorithm()
    result = algo.compute(AlgoInput(data={"hqla": 150.0, "net_outflows": 100.0}))
    assert result.value == 1.5
    assert result.metadata["compliant"] is True


def test_lcr_below_threshold():
    algo = LCRAlgorithm()
    result = algo.compute(AlgoInput(data={"hqla": 80.0, "net_outflows": 100.0}))
    assert result.metadata["compliant"] is False


def test_zero_outflows_raises():
    algo = LCRAlgorithm()
    with pytest.raises(ValueError, match="net_outflows"):
        algo.compute(AlgoInput(data={"hqla": 100.0, "net_outflows": 0.0}))


def test_algo_id_and_regulation():
    algo = LCRAlgorithm()
    assert algo.algo_id == "finance.lcr.v1"
    assert algo.regulation["text"] == "CRR2"
```

---

## 7. Vérifier que tout fonctionne

```bash
# Installer en mode éditable
pip install -e ".[dev]"

# Lancer les tests
pytest tests/ -v

# Construire le wheel localement
pip install build
python -m build
ls dist/
# regalgo_finance_lcr-1.0.0-py3-none-any.whl
# regalgo_finance_lcr-1.0.0.tar.gz
```

---

## ✅ Résultat

Vous avez un package :

- **Installable** via `pip install ./dist/regalgo_finance_lcr-1.0.0-py3-none-any.whl`
- **Traçable** : chaque résultat embarque `algo_id`, `regulation`, `inputs_snapshot`
- **Testable** : couverture des cas nominaux et des erreurs métier
- **Conforme** au nommage du registre (`regalgo-<domaine>-<nom>`)

---

## Étape suivante

[→ Publier sur PyPI](02-publish-pypi.md){ .md-button .md-button--primary }
