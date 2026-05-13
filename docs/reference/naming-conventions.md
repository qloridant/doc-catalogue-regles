# Référence — Conventions de nommage

---

## Nom du package PyPI

```
regalgo-<domaine>-<nom-court>
```

| Segment | Règle | Exemples |
|---|---|---|
| `regalgo` | Préfixe fixe, toujours présent | — |
| `<domaine>` | Code court du domaine réglementaire | `finance`, `sante`, `enviro`, `fiscal` |
| `<nom-court>` | Acronyme ou nom court de l'algorithme | `lcr`, `nsfr`, `drc`, `beges` |

**Exemples valides :**
```
regalgo-finance-lcr
regalgo-finance-nsfr
regalgo-sante-drc
regalgo-enviro-beges
regalgo-fiscal-is-taux-reduit
```

**Exemples invalides :**
```
lcr-calculator          ← manque le préfixe regalgo-
regalgo_finance_lcr     ← underscores au lieu de tirets
regalgo-LCR             ← majuscules interdites
```

---

## Nom du module Python

Le module Python utilise des **underscores** (snake_case), correspondant au nom PyPI avec tirets remplacés :

```
regalgo_<domaine>_<nom-court>
```

```
regalgo-finance-lcr    →  regalgo_finance_lcr
regalgo-sante-drc      →  regalgo_sante_drc
```

---

## `dct:identifier` (algo_id)

```
<domaine>.<nom-court>.<version_majeure>
```

| Segment | Règle | Exemples |
|---|---|---|
| `<domaine>` | Même code que le nom PyPI | `finance` |
| `<nom-court>` | Même code que le nom PyPI | `lcr` |
| `<version_majeure>` | `v` suivi du numéro MAJOR | `v1`, `v2` |

```
finance.lcr.v1
finance.nsfr.v1
sante.drc.v2
```

!!! warning "Immuabilité de l'algo_id"
    Un `dct:identifier` ne change **jamais** pour une version MAJOR donnée.
    Tout changement de MAJOR crée un nouvel identifiant (`v1` → `v2`),
    voire un nouveau package si la rupture est fondamentale.

---

## URI du registre

```
https://registre-algo.gouv.fr/algo/<domaine>/<nom-court>/<version_majeure>
```

Exemple : `https://registre-algo.gouv.fr/algo/finance/lcr/v1`

---

## Nommage des classes Python

| Élément | Convention | Exemple |
|---|---|---|
| Classe principale | `PascalCase` + `Algorithm` | `LCRAlgorithm` |
| Module | `snake_case` | `regalgo_finance_lcr/algorithm.py` |
| Tests | `test_` + nom | `test_lcr_algorithm.py` |

---

## Codes de domaine reconnus

| Code | Domaine | Autorités typiques |
|---|---|---|
| `finance` | Prudentiel, marchés financiers | EBA, AMF, ACPR |
| `sante` | Santé publique, pharmacovigilance | HAS, ANSM |
| `enviro` | Environnement, ESG, GES | ADEME, DREAL |
| `fiscal` | Fiscalité | DGFIP |
| `social` | Protection sociale | CNAV, CNAM |
| `urbanisme` | Urbanisme, construction | DGALN |
| `transport` | Transports | DGITM |

Pour un domaine non listé, ouvrir une issue sur le dépôt du registre.
