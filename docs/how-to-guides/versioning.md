# Guide — Versionner selon CalVer/SemVer réglementaire

Le versioning d'un algorithme réglementaire est distinct du versioning logiciel classique : une modification normative (nouveau texte, nouvelle autorité) peut être rétrocompatible *fonctionnellement* mais **incompatible réglementairement**.

---

## Schéma de version recommandé

Le registre adopte un **SemVer enrichi** :

```
MAJOR.MINOR.PATCH[.NORM_DATE]
```

| Composant | Signification | Exemple |
|---|---|---|
| `MAJOR` | Rupture de l'interface **ou** changement de texte réglementaire de référence | `2.0.0` |
| `MINOR` | Ajout de paramètres optionnels, nouvelles sorties non-breaking | `1.1.0` |
| `PATCH` | Correction de bug, clarification documentaire, aucun impact sur les résultats | `1.0.1` |
| `.NORM_DATE` | Optionnel — date d'entrée en vigueur de la norme (`YYYYMMDD`) | `1.0.0.20240101` |

---

## Quand incrémenter MAJOR

| Situation | Action |
|---|---|
| Le texte réglementaire de référence change (ex : CRR2 → CRR3) | **MAJOR** |
| La signature de `compute()` change (nouveaux paramètres obligatoires) | **MAJOR** |
| La formule de calcul change de façon incompatible avec les résultats antérieurs | **MAJOR** |
| L'`algo_id` change | **MAJOR** (en pratique, créer un nouveau package) |

## Quand incrémenter MINOR

| Situation | Action |
|---|---|
| Ajout d'un paramètre optionnel avec valeur par défaut | **MINOR** |
| Ajout de champs dans `AlgoResult.metadata` | **MINOR** |
| Ajout d'une entrée optionnelle dans `AlgoInput.context` | **MINOR** |

## Quand incrémenter PATCH

| Situation | Action |
|---|---|
| Correction d'un bug de calcul (résultat incorrect → correct) | **PATCH** + changelog détaillé |
| Amélioration des messages d'erreur | **PATCH** |
| Mise à jour de la documentation embarquée | **PATCH** |

---

## Coexistence de versions réglementaires

Quand une norme est mise à jour (ex : révision d'un texte prudentiel), **ne remplacez pas** la version précédente — maintenez les deux :

```
regalgo-civique-droit-vote       # Droit de vote selon Code électoral (actuel)
regalgo-civique-droit-vote-2026  # Droit de vote selon révision 2026 (nouveau texte)
```

Ou via des extras dans le même package :

```toml
[project.optional-dependencies]
v2024 = []   # comportement par défaut
v2026 = ["regalgo-civique-droit-vote-2026-adapter>=1.0"]
```

---

## Déclarer la version dans `metadata.json`

```json
{
  "version": "1.2.0",
  "effective_date": "2024-01-01",
  "supersedes": "1.1.0",
  "changelog": [
    {
      "version": "1.2.0",
      "date": "2024-06-01",
      "type": "MINOR",
      "description": "Ajout du champ stress_scenario dans AlgoInput.context"
    }
  ]
}
```

---

## Compatibilité des dépendances

Dans `pyproject.toml`, utilisez des contraintes de compatibilité strictes pour les algorithmes dont vous dépendez :

```toml
[project]
dependencies = [
    "regalgo-core>=1.0,<2.0",         # Même MAJOR = compatible
    "regalgo-finance-exposure>=2.1",   # MINOR minimum requis
]
```

!!! warning "Ne jamais utiliser `>=x.y` sans borne haute pour les algorithmes réglementaires"
    Un MAJOR bump peut changer silencieusement la sémantique des résultats.
    Toujours borner : `>=1.0,<2.0`.
