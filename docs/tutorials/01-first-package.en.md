# Tutorial 01 — My first packaged algorithm

**Estimated time:** ~20 minutes  
**Goal:** Turn a raw regulatory algorithm into an installable package, compliant with the registry standard.

By the end of this tutorial, you will have a package that:

- installs with `pip install`
- exposes the standard interface contract (`AlgorithmProtocol`)
- embeds the minimum required regulatory metadata

The example used throughout is an algorithm that determines whether **a person has the right to vote in France** under the French Electoral Code (Art. L.2 to L.7). Both **Python** and **Catala** implementations are provided.

---

## Prerequisites

- Python 3.10 or higher
- Up-to-date `pip`: `pip install --upgrade pip`
- Basic knowledge of Python module structure

---

## 1. Create the project structure

Create a folder for your algorithm:

```bash
mkdir regalgo-civique-droit-vote
cd regalgo-civique-droit-vote
```

Create the following structure:

=== "Python"

    ```
    regalgo-civique-droit-vote/
    ├── pyproject.toml
    ├── README.md
    ├── src/
    │   └── regalgo_civique_droit_vote/
    │       ├── __init__.py
    │       ├── algorithm.py
    │       └── metadata.json
    └── tests/
        └── test_algorithm.py
    ```

=== "Catala"

    ```
    regalgo-civique-droit-vote/
    ├── pyproject.toml
    ├── README.md
    ├── catala/
    │   └── droit_vote.catala_fr      # Catala source
    ├── src/
    │   └── regalgo_civique_droit_vote/
    │       ├── __init__.py
    │       ├── algorithm.py           # Python wrapper → Catala
    │       └── metadata.json
    └── tests/
        └── test_algorithm.py
    ```

---

## 2. Declare regulatory metadata

Open `src/regalgo_civique_droit_vote/metadata.json`:

```json
{
  "@context": {
    "cprmv": "https://standaarden.open-regels.nl/standards/cprmv/0.4.0/",
    "cpsv": "http://purl.org/vocab/cpsv#",
    "cv":   "http://data.europa.eu/m8g/",
    "dct":  "http://purl.org/dc/terms/"
  },
  "@id":    "https://regles.gouv.fr/algo/civique/droit-vote/v1",
  "@type":  "cprmv:DecisionModel",
  "dct:title":      "Droit de vote en France",
  "dct:identifier": "civique.droit-vote.v1",
  "dct:version":    "1.0.0",
  "cprmv:isBasedOn": {
    "@id":          "https://www.legifrance.gouv.fr/codes/id/LEGITEXT000006070239/",
    "dct:title":    "Code électoral",
    "dct:coverage": "Art. L.2, L.5, L.6, L.7"
  },
  "cv:hasCompetentAuthority": {
    "dct:title": "Ministère de l'Intérieur"
  },
  "cprmv:method":    ["cprmv:FormalisationMethod", "cprmv:CodificationMethod"],
  "cprmv:validFrom": "2024-01-01",
  "cprmv:hasPart": [
    {
      "@type":             "cprmv:Rule",
      "dct:identifier":    "nationalite_francaise",
      "cprmv:definition":  "Holds French nationality",
      "cprmv:sourceQuote": "Art. L.2",
      "type": "bool"
    },
    {
      "@type":             "cprmv:Rule",
      "dct:identifier":    "age",
      "cprmv:definition":  "Person's age in years",
      "cprmv:sourceQuote": "Art. L.3",
      "type": "int",
      "unit": "years"
    },
    {
      "@type":             "cprmv:Rule",
      "dct:identifier":    "capacite_civique",
      "cprmv:definition":  "Not deprived of civic rights",
      "cprmv:sourceQuote": "Art. L.5, L.6",
      "type": "bool"
    },
    {
      "@type":             "cprmv:Rule",
      "dct:identifier":    "inscrit_listes_electorales",
      "cprmv:definition":  "Registered on electoral rolls",
      "cprmv:sourceQuote": "Art. L.7",
      "type": "bool"
    }
  ],
  "cpsv:produces": {
    "@type":            "cprmv:Rule",
    "dct:identifier":   "peut_voter",
    "cprmv:definition": "The person has the right to vote",
    "type": "bool"
  },
  "dct:subject": ["election", "civique", "droit-vote", "code-electoral"]
}
```

!!! note "CPRMV structure"
    This file is a **JSON-LD** document describing a `cprmv:DecisionModel` (formalised decision model).
    Each legal condition is a `cprmv:Rule` with a `cprmv:sourceQuote` pointing to the relevant article.
    All keys are documented in the [metadata schema reference](../reference/metadata-schema.md).

---

## 3. Implement the algorithm

=== "Python"

    Open `src/regalgo_civique_droit_vote/algorithm.py`:

    ```python
    from __future__ import annotations

    import json
    from pathlib import Path
    from dataclasses import dataclass, field
    from typing import Any


    @dataclass
    class AlgoInput:
        data: dict[str, Any]
        context: dict[str, Any] = field(default_factory=dict)


    @dataclass
    class AlgoResult:
        value: Any
        algo_id: str
        regulation: dict[str, str]
        inputs_snapshot: dict[str, Any]
        metadata: dict[str, Any] = field(default_factory=dict)


    class DroitVoteAlgorithm:
        """
        Voting eligibility in France per the Electoral Code.

        Cumulative conditions (Art. L.2 to L.7):
          - French nationality (Art. L.2)
          - Age >= 18 years (Art. L.3)
          - Not deprived of civic rights (Art. L.5, L.6)
          - Registered on electoral rolls (Art. L.7)
        """

        def __init__(self) -> None:
            _meta_path = Path(__file__).parent / "metadata.json"
            self._metadata = json.loads(_meta_path.read_text())

        @property
        def algo_id(self) -> str:
            return self._metadata["dct:identifier"]

        @property
        def regulation(self) -> dict[str, str]:
            return self._metadata["cprmv:isBasedOn"]

        def compute(self, algo_input: AlgoInput) -> AlgoResult:
            nationalite = bool(algo_input.data["nationalite_francaise"])
            age = int(algo_input.data["age"])
            capacite = bool(algo_input.data["capacite_civique"])
            inscrit = bool(algo_input.data["inscrit_listes_electorales"])

            if age < 0:
                raise ValueError(f"Age must be positive, got: {age}")

            peut_voter = nationalite and age >= 18 and capacite and inscrit

            return AlgoResult(
                value=peut_voter,
                algo_id=self.algo_id,
                regulation=self.regulation,
                inputs_snapshot=algo_input.data,
                metadata={
                    "conditions": {
                        "nationalite_francaise": nationalite,
                        "age_suffisant": age >= 18,
                        "capacite_civique": capacite,
                        "inscrit_listes_electorales": inscrit,
                    }
                },
            )
    ```

=== "Catala"

    Open `catala/droit_vote.catala_fr`:

    ```catala
    > Using Electoral_Code_France

    # Right to vote in France
    # Reference: Electoral Code, Articles L.2 to L.7
    # https://www.legifrance.gouv.fr/codes/id/LEGITEXT000006070239/

    declaration scope RightToVote:
      # Article L.2 — French nationality
      input french_citizen content boolean
      # Article L.3 — legal age (18 years)
      input age content integer
      # Articles L.5-L.6 — civic capacity
      input civic_capacity content boolean
      # Article L.7 — registered on electoral rolls
      input registered_electoral_rolls content boolean

      output can_vote content boolean

    scope RightToVote:
      definition can_vote equals
        french_citizen and
        age >= 18 and
        civic_capacity and
        registered_electoral_rolls
    ```

---

## 4. Write a minimal test

```python
# tests/test_algorithm.py
import pytest
from regalgo_civique_droit_vote import DroitVoteAlgorithm, AlgoInput

VALID_VOTER = {
    "nationalite_francaise": True,
    "age": 25,
    "capacite_civique": True,
    "inscrit_listes_electorales": True,
}

def test_can_vote():
    algo = DroitVoteAlgorithm()
    result = algo.compute(AlgoInput(data=VALID_VOTER))
    assert result.value is True

def test_minor_cannot_vote():
    algo = DroitVoteAlgorithm()
    result = algo.compute(AlgoInput(data={**VALID_VOTER, "age": 17}))
    assert result.value is False

def test_foreigner_cannot_vote():
    algo = DroitVoteAlgorithm()
    result = algo.compute(AlgoInput(data={**VALID_VOTER, "nationalite_francaise": False}))
    assert result.value is False
```

---

## 5. Verify everything works

```bash
pip install -e ".[dev]"
pytest tests/ -v
python -m build
```

---

## Next step

[→ Publish to PyPI](02-publish-pypi.md){ .md-button .md-button--primary }
