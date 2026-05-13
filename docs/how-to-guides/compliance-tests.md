# Guide — Écrire des tests de conformité

Les tests de conformité vérifient que l'algorithme produit les résultats attendus par le texte réglementaire, pas seulement qu'il tourne sans erreur.

---

## Trois niveaux de tests

| Niveau | Ce qu'il vérifie | Obligatoire |
|---|---|---|
| **Contrat** | L'algorithme implémente `AlgorithmProtocol` | ✅ |
| **Métadonnées** | `metadata.json` est valide et complet | ✅ |
| **Réglementaire** | Les résultats sont conformes aux exemples du texte | Fortement recommandé |

---

## Tests de contrat

```python
# tests/test_contract.py
from regalgo_core import AlgorithmProtocol, AlgoInput
from regalgo_civique_droit_vote import DroitVoteAlgorithm

def test_implements_protocol():
    assert isinstance(DroitVoteAlgorithm(), AlgorithmProtocol)

def test_algo_id_format():
    algo = DroitVoteAlgorithm()
    parts = algo.algo_id.split(".")
    assert len(parts) == 3, "Format attendu : domaine.nom.vX"
    assert parts[2].startswith("v"), "La version doit commencer par 'v'"

def test_regulation_required_keys():
    algo = DroitVoteAlgorithm()
    reg = algo.regulation
    assert "text" in reg
    assert "article" in reg
    assert "authority" in reg

def test_result_has_inputs_snapshot():
    algo = DroitVoteAlgorithm()
    data = {
        "nationalite_francaise": True,
        "age": 25,
        "capacite_civique": True,
        "inscrit_listes_electorales": True,
    }
    result = algo.compute(AlgoInput(data=data))
    assert result.inputs_snapshot == data

def test_result_jsonld_serializable():
    import json
    algo = DroitVoteAlgorithm()
    result = algo.compute(AlgoInput(data={
        "nationalite_francaise": True,
        "age": 25,
        "capacite_civique": True,
        "inscrit_listes_electorales": True,
    }))
    # Ne doit pas lever d'exception
    json.dumps(result.jsonld_context())
```

---

## Tests de métadonnées

```python
# tests/test_metadata.py
import json
import pytest
from importlib.resources import files

@pytest.fixture
def metadata():
    path = files("regalgo_civique_droit_vote").joinpath("metadata.json")
    return json.loads(path.read_text())

def test_context_has_required_namespaces(metadata):
    ctx = metadata["@context"]
    for ns in ["cpsv", "cv", "dct", "regalgo", "owl"]:
        assert ns in ctx, f"Namespace manquant : {ns}"

def test_type_includes_cpsv_public_service(metadata):
    types = metadata.get("@type", [])
    if isinstance(types, str):
        types = [types]
    assert "cpsv:PublicService" in types

def test_legal_resource_has_identifier(metadata):
    lr = metadata.get("cv:hasLegalResource", {})
    assert lr.get("eli:id_local") or lr.get("owl:sameAs"), \
        "cv:hasLegalResource doit avoir eli:id_local ou owl:sameAs"

def test_pypi_package_naming(metadata):
    pkg = metadata["regalgo:pypiPackage"]
    assert pkg.startswith("regalgo-"), "Le package doit commencer par regalgo-"
    parts = pkg.split("-")
    assert len(parts) >= 3, "Format attendu : regalgo-<domaine>-<nom>"

def test_inputs_declared(metadata):
    inputs = metadata.get("cpsv:hasInput", [])
    assert len(inputs) >= 1, "Au moins une entrée doit être déclarée"
    for inp in inputs:
        assert "dct:identifier" in inp
        assert "dct:type" in inp
```

---

## Tests réglementaires (exemples normatifs)

La valeur la plus importante : vérifier avec les **cas d'usage officiels** publiés par l'autorité réglementaire (circulaires, notes ministérielles, jurisprudence…).

```python
# tests/test_compliance.py
import pytest
from regalgo_civique_droit_vote import DroitVoteAlgorithm, AlgoInput


# Cas d'usage définis par le Code électoral (Art. L.2 à L.7)
CODE_ELECTORAL_REFERENCE_CASES = [
    {
        "description": "Citoyen français majeur inscrit — peut voter",
        "input": {
            "nationalite_francaise": True,
            "age": 30,
            "capacite_civique": True,
            "inscrit_listes_electorales": True,
        },
        "expected": True,
    },
    {
        "description": "Mineur (17 ans) — ne peut pas voter (Art. L.3)",
        "input": {
            "nationalite_francaise": True,
            "age": 17,
            "capacite_civique": True,
            "inscrit_listes_electorales": True,
        },
        "expected": False,
    },
    {
        "description": "Étranger non naturalisé — ne peut pas voter (Art. L.2)",
        "input": {
            "nationalite_francaise": False,
            "age": 35,
            "capacite_civique": True,
            "inscrit_listes_electorales": True,
        },
        "expected": False,
    },
    {
        "description": "Non inscrit sur les listes — ne peut pas voter (Art. L.7)",
        "input": {
            "nationalite_francaise": True,
            "age": 25,
            "capacite_civique": True,
            "inscrit_listes_electorales": False,
        },
        "expected": False,
    },
    {
        "description": "Privé de droits civiques — ne peut pas voter (Art. L.5)",
        "input": {
            "nationalite_francaise": True,
            "age": 40,
            "capacite_civique": False,
            "inscrit_listes_electorales": True,
        },
        "expected": False,
    },
    {
        "description": "Exactement 18 ans — peut voter (seuil légal)",
        "input": {
            "nationalite_francaise": True,
            "age": 18,
            "capacite_civique": True,
            "inscrit_listes_electorales": True,
        },
        "expected": True,
    },
]

@pytest.mark.parametrize("case", CODE_ELECTORAL_REFERENCE_CASES, ids=lambda c: c["description"])
def test_code_electoral_reference_cases(case):
    algo = DroitVoteAlgorithm()
    result = algo.compute(AlgoInput(data=case["input"]))
    assert result.value == case["expected"], \
        f"Résultat attendu : {case['expected']}, obtenu : {result.value}"
```

---

## Pytest markers recommandés

```python
# pyproject.toml
[tool.pytest.ini_options]
markers = [
    "contract:   Tests du contrat AlgorithmProtocol",
    "metadata:   Tests de validité des métadonnées JSON-LD",
    "compliance: Tests des cas d'usage réglementaires officiels",
]
```

```bash
# Lancer uniquement les tests réglementaires
pytest -m compliance -v

# Lancer tous sauf les tests lents
pytest -m "not slow" -v
```

---

## Voir aussi

- [Contrat d'interface — référence complète](../reference/interface-contract.md)
- [Guide CI/CD — automatiser ces tests](cicd.md)
