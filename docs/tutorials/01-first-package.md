# Tutoriel 01 — Mon premier algorithme packagé

**Durée estimée :** ~20 minutes  
**Objectif :** Transformer un algorithme réglementaire en package installable, conforme au standard du registre.

À la fin de ce tutoriel, vous aurez un package qui :

- s'installe avec `pip install`
- expose le contrat d'interface standard (`AlgorithmProtocol`)
- embarque les métadonnées réglementaires minimales requises

L'exemple utilisé tout au long de ce tutoriel est un algorithme fictif déterminant si **une personne a le droit de voter en France** selon le Code électoral (Art. L.2 à L.7). Des implémentations en **Python** et en **Catala** sont proposées.

---

## Prérequis

- Python 3.10 ou supérieur
- `pip` à jour : `pip install --upgrade pip`
- Connaissance basique de la structure d'un module Python
- Installer le package `regalgo`[https://test.pypi.org/project/regalgo/] (defini les modèles de données à utiliser)


---

## 1. Créer la structure du projet

Utilisez `regalgo` pour créer votre dossier au bon format :

```bash
regalgo init <PROJECT_NAME>
cd <PROJECT_NAME>
```

Vous aurez alors la structure suivante :

=== "Python"

    ```
    regalgo-civique-droit-vote/
    ├── pyproject.toml
    ├── README.md
    ├── src/
    │   └── regalgo_civique_droit_vote/
    │       ├── __init__.py
    │       ├── regle.py
    │       └── metadata.json
    └── tests/
        └── test_regle.py
    ```

=== "Catala"

    ```
    regalgo-civique-droit-vote/
    ├── pyproject.toml
    ├── README.md
    ├── catala/
    │   └── droit_vote.catala_fr          # Source Catala
    ├── src/
    │   └── regalgo_civique_droit_vote/
    │       ├── __init__.py
    │       ├── regle.py               # Wrapper Python → Catala
    │       └── metadata.json
    └── tests/
        └── test_regle.py
    ```
---

## 2. Déclarer les métadonnées réglementaires

Ouvrez `src/regalgo_civique_droit_vote/metadata.json` et renseignez :

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
      "cprmv:definition":  "Possède la nationalité française",
      "cprmv:sourceQuote": "Art. L.2",
      "type": "bool"
    },
    {
      "@type":             "cprmv:Rule",
      "dct:identifier":    "age",
      "cprmv:definition":  "Âge de la personne en années",
      "cprmv:sourceQuote": "Art. L.3",
      "type": "int",
      "unit": "années"
    },
    {
      "@type":             "cprmv:Rule",
      "dct:identifier":    "capacite_civique",
      "cprmv:definition":  "Non privé de ses droits civiques",
      "cprmv:sourceQuote": "Art. L.5, L.6",
      "type": "bool"
    },
    {
      "@type":             "cprmv:Rule",
      "dct:identifier":    "inscrit_listes_electorales",
      "cprmv:definition":  "Inscrit sur les listes électorales",
      "cprmv:sourceQuote": "Art. L.7",
      "type": "bool"
    }
  ],
  "cpsv:produces": {
    "@type":            "cprmv:Rule",
    "dct:identifier":   "peut_voter",
    "cprmv:definition": "La personne a le droit de voter",
    "type": "bool"
  },
  "dct:subject": ["election", "civique", "droit-vote", "code-electoral"]
}
```

!!! note "Structure CPRMV"
    Le fichier est un document **JSON-LD** décrivant un `cprmv:DecisionModel` (modèle de décision formalisé).
    Chaque condition légale est une `cprmv:Rule` avec sa `cprmv:sourceQuote` (article de référence).
    Toutes les clés sont documentées dans la [référence du schéma de métadonnées](../reference/metadata-schema.md).

---

## 3. Implémenter l'algorithme

=== "Python"

    Ouvrez `src/regalgo_civique_droit_vote/algorithm.py` :

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

    class DroitVoteAlgorithm:
        """
        Éligibilité au droit de vote en France selon le Code électoral.

        Conditions cumulatives (Art. L.2 à L.7) :
          - Posséder la nationalité française (Art. L.2)
          - Être âgé d'au moins 18 ans (Art. L.3)
          - Ne pas être privé de ses droits civiques (Art. L.5, L.6)
          - Être inscrit sur les listes électorales (Art. L.7)
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
            """
            Détermine si une personne a le droit de voter en France.

            Args:
                algo_input: Entrée contenant les attributs de la personne.

            Returns:
                AlgoResult avec True/False et la traçabilité complète.

            Raises:
                ValueError: Si l'âge est négatif.
            """
            nationalite = bool(algo_input.data["nationalite_francaise"])
            age = int(algo_input.data["age"])
            capacite = bool(algo_input.data["capacite_civique"])
            inscrit = bool(algo_input.data["inscrit_listes_electorales"])

            if age < 0:
                raise ValueError(f"L'âge doit être positif, reçu : {age}")

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

    Ouvrez `catala/droit_vote.catala_fr` :

    ```catala
    > Utilisation Code_Electoral_Français

    ```catala
    # Droit de vote en France
    # Référence : Code électoral, Articles L.2 à L.7
    # https://www.legifrance.gouv.fr/codes/id/LEGITEXT000006070239/

    déclaration champ d'application DroitDeVote:
      # Article L.2 — nationalité française
      entrée nationalite_francaise contenu booléen
      # Article L.3 — âge légal (18 ans)
      entrée age contenu entier
      # Articles L.5-L.6 — absence de privation des droits civiques
      entrée capacite_civique contenu booléen
      # Article L.7 — inscription sur les listes électorales
      entrée inscrit_listes_electorales contenu booléen

      résultat peut_voter contenu booléen

    champ d'application DroitDeVote:
      définition peut_voter égal à
        nationalite_francaise et
        age >= 18 et
        capacite_civique et
        inscrit_listes_electorales
    ```

    !!! tip "Intégration Python ↔ Catala"
        Le compilateur Catala génère un module Python interopérable.
        Le fichier `algorithm.py` agit alors comme une façade respectant `AlgorithmProtocol`
        et délègue le calcul au code compilé depuis le source Catala.

---

## 4. Exposer le package

Ouvrez `src/regalgo_civique_droit_vote/__init__.py` :

```python
from .algorithm import DroitVoteAlgorithm, AlgoInput, AlgoResult

__all__ = ["DroitVoteAlgorithm", "AlgoInput", "AlgoResult"]
```

---

## 5. Configurer `pyproject.toml`

=== "Hatchling (recommandé)"

    ```toml
    [build-system]
    requires = ["hatchling"]
    build-backend = "hatchling.build"

    [project]
    name = "regalgo-civique-droit-vote"
    version = "1.0.0"
    description = "Droit de vote en France — Code électoral Art. L.2"
    readme = "README.md"
    requires-python = ">=3.10"
    license = { text = "MIT" }
    keywords = ["reglementation", "civique", "droit-vote", "election", "regalgo"]
    classifiers = [
        "Programming Language :: Python :: 3",
        "Topic :: Office/Business",
        "Intended Audience :: Developers",
    ]

    [project.urls]
    "Registry" = "https://regles.example.com/civique/droit-vote"
    "Source" = "https://github.com/your-org/regalgo-civique-droit-vote"

    [tool.hatch.build.targets.wheel]
    packages = ["src/regalgo_civique_droit_vote"]

    # Inclusion des métadonnées JSON dans le wheel
    [tool.hatch.build.targets.wheel.force-include]
    "src/regalgo_civique_droit_vote/metadata.json" = "regalgo_civique_droit_vote/metadata.json"
    ```

=== "Setuptools"

    ```toml
    [build-system]
    requires = ["setuptools>=68", "wheel"]
    build-backend = "setuptools.backends.legacy:build"

    [project]
    name = "regalgo-civique-droit-vote"
    version = "1.0.0"
    description = "Droit de vote en France — Code électoral Art. L.2"
    readme = "README.md"
    requires-python = ">=3.10"
    license = { text = "MIT" }
    keywords = ["reglementation", "civique", "droit-vote", "regalgo"]

    [tool.setuptools.packages.find]
    where = ["src"]

    [tool.setuptools.package-data]
    "*" = ["metadata.json"]
    ```

=== "Poetry"

    ```toml
    [tool.poetry]
    name = "regalgo-civique-droit-vote"
    version = "1.0.0"
    description = "Droit de vote en France — Code électoral Art. L.2"
    packages = [{include = "regalgo_civique_droit_vote", from = "src"}]

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
from regalgo_civique_droit_vote import DroitVoteAlgorithm, AlgoInput


ELECTEUR_VALIDE = {
    "nationalite_francaise": True,
    "age": 25,
    "capacite_civique": True,
    "inscrit_listes_electorales": True,
}


def test_peut_voter():
    algo = DroitVoteAlgorithm()
    result = algo.compute(AlgoInput(data=ELECTEUR_VALIDE))
    assert result.value is True


def test_mineur_ne_peut_pas_voter():
    algo = DroitVoteAlgorithm()
    data = {**ELECTEUR_VALIDE, "age": 17}
    result = algo.compute(AlgoInput(data=data))
    assert result.value is False


def test_non_inscrit_ne_peut_pas_voter():
    algo = DroitVoteAlgorithm()
    data = {**ELECTEUR_VALIDE, "inscrit_listes_electorales": False}
    result = algo.compute(AlgoInput(data=data))
    assert result.value is False


def test_etranger_ne_peut_pas_voter():
    algo = DroitVoteAlgorithm()
    data = {**ELECTEUR_VALIDE, "nationalite_francaise": False}
    result = algo.compute(AlgoInput(data=data))
    assert result.value is False


def test_age_negatif_raises():
    algo = DroitVoteAlgorithm()
    with pytest.raises(ValueError, match="âge"):
        algo.compute(AlgoInput(data={**ELECTEUR_VALIDE, "age": -1}))


def test_algo_id_and_regulation():
    algo = DroitVoteAlgorithm()
    assert algo.algo_id == "civique.droit-vote.v1"
    assert algo.regulation["dct:title"] == "Code électoral"
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
# regalgo_civique_droit_vote-1.0.0-py3-none-any.whl
# regalgo_civique_droit_vote-1.0.0.tar.gz
```

---

## ✅ Résultat

Vous avez un package :

- **Installable** via `pip install ./dist/regalgo_civique_droit_vote-1.0.0-py3-none-any.whl`
- **Traçable** : chaque résultat embarque `algo_id`, `regulation`, `inputs_snapshot`
- **Testable** : couverture des cas nominaux et des erreurs métier
- **Conforme** au nommage du registre (`regalgo-<domaine>-<nom>`)

---

## Étape suivante

[→ Publier sur PyPI](02-publish-pypi.md){ .md-button .md-button--primary }
