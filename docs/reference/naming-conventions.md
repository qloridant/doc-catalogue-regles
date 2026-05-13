# Référence — Conventions de nommage

---

## Nom du package PyPI

```
regalgo-<domaine>-<nom-court>
```

| Segment | Règle | Exemples |
|---|---|---|
| `regalgo` | Préfixe fixe, toujours présent | — |
| `<domaine>` | Code court du domaine réglementaire | `civique`, `finance`, `sante`, `enviro`, `fiscal` |
| `<nom-court>` | Acronyme ou nom court de l'algorithme | `droit-vote`, `nsfr`, `drc`, `beges` |

**Exemples valides :**
```
regalgo-civique-droit-vote
regalgo-finance-nsfr
regalgo-sante-drc
regalgo-enviro-beges
regalgo-fiscal-is-taux-reduit
```

**Exemples invalides :**
```
droit-vote-calculator       ← manque le préfixe regalgo-
regalgo_civique_droit_vote  ← underscores au lieu de tirets
regalgo-DroitVote           ← majuscules interdites
```

---

## Nom du module Python

Le module Python utilise des **underscores** (snake_case), correspondant au nom PyPI avec tirets remplacés :

```
regalgo_<domaine>_<nom-court>
```

```
regalgo-civique-droit-vote   →  regalgo_civique_droit_vote
regalgo-sante-drc            →  regalgo_sante_drc
```

---

## `dct:identifier` (algo_id)

```
<domaine>.<nom-court>.<version_majeure>
```

| Segment | Règle | Exemples |
|---|---|---|
| `<domaine>` | Même code que le nom PyPI | `civique` |
| `<nom-court>` | Même code que le nom PyPI | `droit-vote` |
| `<version_majeure>` | `v` suivi du numéro MAJOR | `v1`, `v2` |

```
civique.droit-vote.v1
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

Exemple : `https://registre-algo.gouv.fr/algo/civique/droit-vote/v1`

---

## Nommage des classes Python

| Élément | Convention | Exemple |
|---|---|---|
| Classe principale | `PascalCase` + `Algorithm` | `DroitVoteAlgorithm` |
| Module | `snake_case` | `regalgo_civique_droit_vote/algorithm.py` |
| Tests | `test_` + nom | `test_droit_vote_algorithm.py` |

---

## Codes de domaine reconnus

| Code | Domaine | Autorités typiques |
|---|---|---|
| `civique` | Droits civiques, élections, état civil | Ministère de l'Intérieur, CNIL |
| `social` | Protection sociale | CNAV, CNAM |
| `finance` | Prudentiel, marchés financiers | EBA, AMF, ACPR |
| `sante` | Santé publique, pharmacovigilance | HAS, ANSM |
| `enviro` | Environnement, ESG, GES | ADEME, DREAL |
| `fiscal` | Fiscalité | DGFIP |
| `urbanisme` | Urbanisme, construction | DGALN |
| `transport` | Transports | DGITM |

Pour un domaine non listé, ouvrir une issue sur le dépôt du registre.
