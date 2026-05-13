# Guide — Définir des interfaces interopérables

Ce guide explique comment implémenter `AlgorithmProtocol` pour garantir que votre algorithme est composable avec les autres packages du registre.

---

## Le contrat d'interface

Tous les algorithmes du registre implémentent le `AlgorithmProtocol` défini dans `regalgo-core` :

```python
# pip install regalgo-core
from regalgo_core import AlgorithmProtocol, AlgoInput, AlgoResult
```

Le protocole complet est défini comme un `typing.Protocol` Python :

```python
from typing import Protocol, runtime_checkable

@runtime_checkable
class AlgorithmProtocol(Protocol):
    """Contrat minimal pour tout algorithme du registre."""

    @property
    def algo_id(self) -> str:
        """Identifiant unique : '<domaine>.<nom>.<version_majeure>'"""
        ...

    @property
    def regulation(self) -> dict[str, str]:
        """Référence normative : text, article, authority"""
        ...

    def compute(self, algo_input: AlgoInput) -> AlgoResult:
        """Point d'entrée unique du calcul."""
        ...
```

---

## Implémenter le protocole

```python
from regalgo_core import AlgorithmProtocol, AlgoInput, AlgoResult

class MonAlgorithme:
    """Implémente implicitement AlgorithmProtocol (duck typing)."""

    @property
    def algo_id(self) -> str:
        return "domaine.nom.v1"

    @property
    def regulation(self) -> dict[str, str]:
        return {"text": "REG-XYZ", "article": "Art. 10", "authority": "Autorité"}

    def compute(self, algo_input: AlgoInput) -> AlgoResult:
        value = algo_input.data["param_a"] * 2
        return AlgoResult(
            value=value,
            algo_id=self.algo_id,
            regulation=self.regulation,
            inputs_snapshot=algo_input.data,
        )

# Vérification à l'exécution
assert isinstance(MonAlgorithme(), AlgorithmProtocol)
```

---

## Composer plusieurs algorithmes

### Pattern Pipeline

Enchaîner des algorithmes où la sortie de l'un est l'entrée du suivant :

```python
from regalgo_core import AlgoInput, AlgorithmProtocol

def run_pipeline(
    stages: list[AlgorithmProtocol],
    initial_data: dict,
) -> list:
    """Exécute une séquence d'algorithmes en pipeline."""
    results = []
    current_data = initial_data

    for algo in stages:
        result = algo.compute(AlgoInput(data=current_data))
        results.append(result)
        # La valeur calculée devient une entrée pour le suivant
        current_data = {**current_data, algo.algo_id: result.value}

    return results
```

```python
from regalgo_finance_lcr import LCRAlgorithm
from regalgo_finance_nsfr import NSFRAlgorithm

pipeline = [LCRAlgorithm(), NSFRAlgorithm()]
results = run_pipeline(pipeline, initial_data={
    "hqla": 150.0,
    "net_outflows": 100.0,
    "available_stable_funding": 200.0,
    "required_stable_funding": 180.0,
})
```

### Pattern Agrégateur

Exécuter plusieurs algorithmes en parallèle et agréger :

```python
from concurrent.futures import ThreadPoolExecutor
from regalgo_core import AlgoInput, AlgorithmProtocol

def run_parallel(
    algos: list[AlgorithmProtocol],
    data: dict,
) -> dict[str, float]:
    """Exécute plusieurs algorithmes sur les mêmes données."""
    input_ = AlgoInput(data=data)

    with ThreadPoolExecutor() as executor:
        futures = {
            algo.algo_id: executor.submit(algo.compute, input_)
            for algo in algos
        }

    return {
        algo_id: future.result().value
        for algo_id, future in futures.items()
    }
```

---

## Passer le contexte réglementaire

Pour des besoins d'audit ou de paramétrage contextuel (date d'effet, juridiction…) :

```python
result = algo.compute(AlgoInput(
    data={"hqla": 150.0, "net_outflows": 100.0},
    context={
        "reporting_date": "2024-12-31",
        "jurisdiction": "FR",
        "scenario": "stress",
    }
))
```

Le champ `context` est transmis dans l'`AlgoResult` pour assurer la traçabilité complète.

---

## Vérifier la conformité au protocole

```bash
regalgo-validator check-protocol mon_package.MonAlgorithme
```

```
✅ algo_id présent et format valide
✅ regulation dict avec clés requises
✅ compute() accepte AlgoInput, retourne AlgoResult
✅ isinstance(obj, AlgorithmProtocol) → True
```

---

## Voir aussi

- [Contrat d'interface (référence complète)](../reference/interface-contract.md)
- [Gérer les dépendances entre algorithmes](dependencies.md)
