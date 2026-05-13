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
from regalgo_finance_lcr import LCRAlgorithm

def test_implements_protocol():
    assert isinstance(LCRAlgorithm(), AlgorithmProtocol)

def test_algo_id_format():
    algo = LCRAlgorithm()
    parts = algo.algo_id.split(".")
    assert len(parts) == 3, "Format attendu : domaine.nom.vX"
    assert parts[2].startswith("v"), "La version doit commencer par 'v'"

def test_regulation_required_keys():
    algo = LCRAlgorithm()
    reg = algo.regulation
    assert "text" in reg
    assert "article" in reg
    assert "authority" in reg

def test_result_has_inputs_snapshot():
    algo = LCRAlgorithm()
    data = {"hqla": 100.0, "net_outflows": 80.0}
    result = algo.compute(AlgoInput(data=data))
    assert result.inputs_snapshot == data

def test_result_jsonld_serializable():
    import json
    algo = LCRAlgorithm()
    result = algo.compute(AlgoInput(data={"hqla": 100.0, "net_outflows": 80.0}))
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
    path = files("regalgo_finance_lcr").joinpath("metadata.json")
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

La valeur la plus importante : vérifier avec les **cas d'usage officiels** publiés par l'autorité réglementaire (EBA Q&A, notes ESMA, circulaires HAS…).

```python
# tests/test_compliance.py
import pytest
from regalgo_finance_lcr import LCRAlgorithm, AlgoInput


# Cas d'usage publié par EBA — CRR2 Art. 412 Annex XXV
# Source : EBA/GL/2015/24 — Exemple illustratif §3.2
EBA_REFERENCE_CASES = [
    {
        "description": "Établissement conforme — ratio 1.5",
        "input":       {"hqla": 150_000_000, "net_outflows": 100_000_000},
        "expected_value":     1.5,
        "expected_compliant": True,
    },
    {
        "description": "Établissement non conforme — ratio 0.8",
        "input":       {"hqla": 80_000_000, "net_outflows": 100_000_000},
        "expected_value":     0.8,
        "expected_compliant": False,
    },
    {
        "description": "Seuil exact 100% — limite réglementaire",
        "input":       {"hqla": 100_000_000, "net_outflows": 100_000_000},
        "expected_value":     1.0,
        "expected_compliant": True,
    },
]

@pytest.mark.parametrize("case", EBA_REFERENCE_CASES, ids=lambda c: c["description"])
def test_eba_reference_cases(case):
    algo = LCRAlgorithm()
    result = algo.compute(AlgoInput(data=case["input"]))
    assert result.value == pytest.approx(case["expected_value"], rel=1e-6), \
        f"Valeur attendue : {case['expected_value']}, obtenue : {result.value}"
    assert result.metadata["compliant"] == case["expected_compliant"]
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
