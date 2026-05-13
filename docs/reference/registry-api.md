# Référence — API du registre

Le registre expose deux interfaces de découverte : une **API REST** pour les consommateurs simples, et un **endpoint SPARQL** pour les requêtes sémantiques avancées.

---

## API REST

Base URL : `https://registre-algo.gouv.fr/api/v1`

### `GET /algos`

Liste les algorithmes du registre avec filtres.

**Paramètres :**

| Paramètre | Type | Description |
|---|---|---|
| `domain` | `string` | Filtrer par domaine (`finance`, `sante`…) |
| `regulation` | `string` | Filtrer par texte réglementaire (`CRR2`, `DORA`…) |
| `authority` | `string` | Filtrer par autorité (`EBA`, `AMF`…) |
| `status` | `string` | `stable`, `draft`, `deprecated` |
| `q` | `string` | Recherche plein texte |

**Exemple :**
```bash
curl "https://registre-algo.gouv.fr/api/v1/algos?domain=finance&regulation=CRR2"
```

**Réponse :**
```json
{
  "total": 3,
  "items": [
    {
      "algo_id":     "finance.lcr.v1",
      "title":       "Liquidity Coverage Ratio",
      "pypi":        "regalgo-finance-lcr",
      "version":     "1.0.0",
      "domain":      "finance",
      "regulation":  {"text": "CRR2", "article": "Art. 412", "eli": "..."},
      "authority":   "EBA",
      "status":      "stable",
      "registry_uri": "https://registre-algo.gouv.fr/algo/finance/lcr/v1"
    }
  ]
}
```

---

### `GET /algos/{algo_id}`

Retourne le `metadata.json` complet d'un algorithme en JSON-LD.

```bash
curl "https://registre-algo.gouv.fr/api/v1/algos/finance.lcr.v1"
# Content-Type: application/ld+json
```

---

### `GET /algos/{algo_id}/versions`

Historique des versions d'un algorithme.

---

## Endpoint SPARQL

URL : `https://registre-algo.gouv.fr/sparql`

Permet des requêtes sémantiques sur le graphe complet du registre.

### Exemples de requêtes

**Tous les algorithmes finance encadrés par l'EBA :**

```sparql
PREFIX cpsv:    <http://purl.org/vocab/cpsv#>
PREFIX cv:      <http://data.europa.eu/m8g/>
PREFIX dct:     <http://purl.org/dc/terms/>
PREFIX regalgo: <https://registre-algo.gouv.fr/ns#>

SELECT ?algo ?title ?pypi
WHERE {
  ?algo  a cpsv:PublicService ;
         dct:title    ?title ;
         regalgo:pypiPackage ?pypi ;
         cv:hasCompetentAuthority <https://registre-algo.gouv.fr/org/eba> .
}
```

**Tous les algorithmes liés à un texte réglementaire EUR-Lex :**

```sparql
PREFIX cv:   <http://data.europa.eu/m8g/>
PREFIX owl:  <http://www.w3.org/2002/07/owl#>
PREFIX dct:  <http://purl.org/dc/terms/> 

SELECT ?algo ?title
WHERE {
  ?algo  dct:title ?title ;
         cv:hasLegalResource ?lr .
  ?lr    owl:sameAs <http://data.europa.eu/eli/reg/2013/575/oj> .
}
```

**Chaîne de remplacement d'un algorithme :**

```sparql
PREFIX dct: <http://purl.org/dc/terms/>

SELECT ?new_algo ?replaced
WHERE {
  ?new_algo dct:replaces ?replaced .
}
ORDER BY ?new_algo
```

---

## Client Python

```python
from regalgo_core.registry import RegistryClient

client = RegistryClient("https://registre-algo.gouv.fr/api/v1")

# Recherche
results = client.search(domain="finance", regulation="CRR2")
for algo in results:
    print(f"{algo.algo_id} → pip install {algo.pypi_package}")

# Installation automatique
client.install("finance.lcr.v1")
# équivalent à : pip install regalgo-finance-lcr==1.0.0
```

---

## Voir aussi

- [Guide : Intégrer au registre public](../tutorials/03-register-algo.md)
- [Explication : Architecture du registre](../explanation/registry-architecture.md)
