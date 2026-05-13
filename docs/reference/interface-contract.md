# Référence — Contrat d'interface (`AlgorithmProtocol`)

Le contrat d'interface est le mécanisme central de l'interopérabilité entre algorithmes du registre. Il est défini dans le package `regalgo-core` et constitue la traduction Python du modèle CPSV-AP.

---

## `AlgorithmProtocol`

```python
# pip install regalgo-core
from typing import Protocol, runtime_checkable

@runtime_checkable
class AlgorithmProtocol(Protocol):
    """
    Contrat minimal pour tout algorithme du registre.

    Modélise un cpsv:PublicService au sens CPSV-AP :
    - algo_id     → dct:identifier
    - regulation  → cv:hasLegalResource
    - compute()   → produit un cpsv:Output
    """

    @property
    def algo_id(self) -> str:
        """
        Identifiant unique de l'algorithme.
        Format : '<domaine>.<nom>.<version_majeure>'
        Exemple : 'finance.lcr.v1'

        Correspond à dct:identifier dans metadata.json.
        """
        ...

    @property
    def regulation(self) -> dict[str, str]:
        """
        Référence réglementaire minimale.
        Clés requises : 'text', 'article', 'authority'

        Correspond à cv:hasLegalResource dans metadata.json.

        Exemple :
        {
            'text':      'CRR2',
            'article':   'Art. 412',
            'authority': 'EBA',
            'eli':       'http://data.europa.eu/eli/reg/2013/575/oj'
        }
        """
        ...

    def compute(self, algo_input: "AlgoInput") -> "AlgoResult":
        """
        Point d'entrée unique du calcul réglementaire.

        Args:
            algo_input: Entrée normalisée (cv:Evidence).

        Returns:
            AlgoResult: Résultat normalisé (cpsv:Output) avec traçabilité.
        """
        ...
```

---

## `AlgoInput`

Modélise les entrées de l'algorithme, correspondant aux `cv:Evidence` du CPSV-AP.

```python
from dataclasses import dataclass, field
from typing import Any

@dataclass
class AlgoInput:
    """
    Entrée normalisée d'un algorithme réglementaire.
    Correspond à cpsv:hasInput / cv:Evidence dans CPSV-AP.
    """

    data: dict[str, Any]
    """Données de calcul. Clés = dct:identifier des AlgorithmInput."""

    context: dict[str, Any] = field(default_factory=dict)
    """
    Contexte d'exécution (non-calculatoire).
    Exemples : reporting_date, jurisdiction, scenario, fiscal_year.
    Transmis intact dans AlgoResult pour traçabilité.
    """
```

**Exemple :**
```python
AlgoInput(
    data={
        "hqla": 150.0,
        "net_outflows": 100.0
    },
    context={
        "reporting_date": "2024-12-31",
        "jurisdiction":   "FR",
        "scenario":       "baseline"
    }
)
```

---

## `AlgoResult`

Modélise le résultat d'un calcul, correspondant à `cpsv:Output` dans le CPSV-AP.

```python
from dataclasses import dataclass, field
from typing import Any

@dataclass
class AlgoResult:
    """
    Sortie normalisée d'un algorithme réglementaire.
    Correspond à cpsv:produces / cpsv:Output dans CPSV-AP.
    """

    value: Any
    """Valeur calculée principale."""

    algo_id: str
    """dct:identifier de l'algorithme ayant produit ce résultat."""

    regulation: dict[str, str]
    """
    Référence normative complète (copie de AlgorithmProtocol.regulation).
    Garantit la traçabilité même hors contexte.
    """

    inputs_snapshot: dict[str, Any]
    """
    Copie immuable des données d'entrée au moment du calcul.
    Permet la reproductibilité et l'audit.
    """

    context: dict[str, Any] = field(default_factory=dict)
    """Contexte d'exécution transmis depuis AlgoInput."""

    metadata: dict[str, Any] = field(default_factory=dict)
    """
    Métadonnées calculatoires complémentaires.
    Exemples : compliant, threshold, unit, intermediate_values.
    """

    def jsonld_context(self) -> dict:
        """
        Retourne le résultat sérialisé en JSON-LD CPSV-AP.
        Permet l'indexation dans un triplestore.
        """
        return {
            "@context": {
                "cpsv":    "http://purl.org/vocab/cpsv#",
                "cv":      "http://data.europa.eu/m8g/",
                "dct":     "http://purl.org/dc/terms/",
                "regalgo": "https://registre-algo.gouv.fr/ns#"
            },
            "@type":               "cpsv:Output",
            "dct:identifier":      self.algo_id,
            "regalgo:value":       self.value,
            "cv:hasLegalResource": self.regulation,
            "regalgo:inputs":      self.inputs_snapshot,
            "regalgo:context":     self.context,
            "regalgo:metadata":    self.metadata
        }
```

---

## Contrat d'erreurs

Toutes les exceptions levées par `compute()` doivent être des sous-classes de `regalgo.exceptions.AlgoError` :

```python
from regalgo_core.exceptions import (
    AlgoError,           # Base
    AlgoInputError,      # Données d'entrée invalides (ValueError)
    AlgoComplianceError, # Violation d'une règle réglementaire
    AlgoVersionError,    # Incompatibilité de version
)
```

**Exemple :**
```python
from regalgo_core.exceptions import AlgoInputError

def compute(self, algo_input: AlgoInput) -> AlgoResult:
    if algo_input.data.get("net_outflows", 0) <= 0:
        raise AlgoInputError(
            field="net_outflows",
            message="Doit être strictement positif",
            algo_id=self.algo_id
        )
```

---

## Vérification de conformité

```python
from regalgo_core import AlgorithmProtocol

# Vérification à l'exécution (duck typing)
assert isinstance(MonAlgorithme(), AlgorithmProtocol)

# Vérification statique (mypy / pyright)
def run(algo: AlgorithmProtocol, data: dict) -> AlgoResult:
    return algo.compute(AlgoInput(data=data))
```

---

## Voir aussi

- [Guide : Définir des interfaces interopérables](../how-to-guides/interoperability.md)
- [Schéma de métadonnées JSON-LD](metadata-schema.md)
- [Vocabulaire commun CPSV-AP](vocabulary.md)
