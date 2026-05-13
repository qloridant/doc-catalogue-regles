# Guide — Gérer les dépendances entre algorithmes

Certains algorithmes réglementaires dépendent d'autres algorithmes (ex : un calcul de capital réglementaire agrège plusieurs sous-calculs). Ce guide explique comment déclarer et gérer ces dépendances.

---

## Déclarer une dépendance dans `pyproject.toml`

```toml
[project]
dependencies = [
    "regalgo-core>=1.0,<2.0",
    # Dépendance sur un autre algorithme du registre
    "regalgo-finance-exposure>=1.2,<2.0",
    "regalgo-finance-rwa>=1.0,<2.0",
]
```

!!! warning "Toujours borner le MAJOR"
    Un bump MAJOR peut changer silencieusement la sémantique des résultats.
    `regalgo-finance-exposure>=1.0` sans borne haute est un risque de conformité.

---

## Déclarer la dépendance dans `metadata.json`

```json
"regalgo:dependsOn": [
  {
    "@id": "https://registre-algo.gouv.fr/algo/finance/exposure/v1",
    "regalgo:version_constraint": ">=1.2,<2.0"
  }
]
```

---

## Pattern Façade — orchestrer plusieurs algos

```python
from regalgo_core import AlgoInput, AlgoResult
from regalgo_finance_exposure import ExposureAlgorithm
from regalgo_finance_rwa import RWAAlgorithm

class CapitalRequirementAlgorithm:
    """Calcul des exigences de fonds propres — agrège Exposure + RWA."""

    def __init__(self):
        self._exposure = ExposureAlgorithm()
        self._rwa      = RWAAlgorithm()

    @property
    def algo_id(self) -> str:
        return "finance.capital-requirement.v1"

    @property
    def regulation(self) -> dict:
        return {"text": "CRR2", "article": "Art. 92", "authority": "EBA"}

    def compute(self, algo_input: AlgoInput) -> AlgoResult:
        # Étape 1 : calcul de l'exposition
        exposure_result = self._exposure.compute(algo_input)

        # Étape 2 : calcul du RWA à partir de l'exposition
        rwa_input = AlgoInput(
            data={**algo_input.data, "exposure": exposure_result.value},
            context=algo_input.context
        )
        rwa_result = self._rwa.compute(rwa_input)

        # Étape 3 : calcul final
        capital = rwa_result.value * 0.08  # 8% minimum CRR2 Art. 92

        return AlgoResult(
            value=capital,
            algo_id=self.algo_id,
            regulation=self.regulation,
            inputs_snapshot=algo_input.data,
            metadata={
                "exposure":       exposure_result.value,
                "rwa":            rwa_result.value,
                "capital_ratio":  0.08,
                # Traçabilité des algos intermédiaires
                "sub_algos": {
                    self._exposure.algo_id: exposure_result.value,
                    self._rwa.algo_id:      rwa_result.value,
                }
            }
        )
```

---

## Résolution de conflits de version

Si deux algorithmes dépendent de versions incompatibles d'un même sous-algorithme :

```bash
pip install regalgo-finance-capital-requirement
# ERROR: Cannot install regalgo-finance-exposure==1.3 and regalgo-finance-exposure==2.0
```

**Solutions :**

1. **Prioriser la mise à jour** : mettre à jour le package avec la contrainte la plus restrictive
2. **Virtual environments séparés** : isoler les calculs dans des processus distincts
3. **Signaler au mainteneur** : ouvrir une issue sur le registre pour coordonner un alignement de versions

---

## Voir aussi

- [Guide : Versionner selon SemVer réglementaire](versioning.md)
- [Guide : Définir des interfaces interopérables](interoperability.md)
