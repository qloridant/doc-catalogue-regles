# Guide — Déclarer les métadonnées réglementaires

Ce guide explique comment renseigner correctement le fichier `metadata.json` d'un package, champ par champ.

---

## Trouver l'URI EUR-Lex de votre texte réglementaire

Le champ `cv:hasLegalResource` doit référencer le texte normatif via son identifiant **ELI** (European Legislation Identifier).

**Méthode :**

1. Rendez-vous sur [eur-lex.europa.eu](https://eur-lex.europa.eu)
2. Recherchez votre texte (ex : "CRR2 575/2013")
3. L'URI ELI est dans l'onglet "Métadonnées" ou dans l'URL permanente
4. Format : `http://data.europa.eu/eli/reg/<année>/<numéro>/oj`

**Pour les textes français :**

1. Rendez-vous sur [legifrance.gouv.fr](https://www.legifrance.gouv.fr)
2. L'identifiant NOR ou le JORF URI peut être utilisé comme `eli:id_local`
3. Format JORF : `JORFTEXT000<numéro>`

```json
"cv:hasLegalResource": {
  "@type": "cv:LegalResource",
  "@id": "https://registre-algo.gouv.fr/legal/mon-texte",
  "dct:title": {"@value": "Titre du texte", "@language": "fr"},
  "eli:id_local": "CRR2/Art.412",
  "owl:sameAs": {"@id": "http://data.europa.eu/eli/reg/2013/575/oj"}
}
```

---

## Identifier l'autorité compétente

Le champ `cv:hasCompetentAuthority` utilise le **Corporate Body Authority** du Publications Office EU.

**Recherche :**
```bash
curl "https://op.europa.eu/o/opportal-service/euvoc-download-handler?cellarURI=http://publications.europa.eu/resource/authority/corporate-body"
```

Ou via le portail : [op.europa.eu/en/web/eu-vocabularies](https://op.europa.eu/en/web/eu-vocabularies)

| Autorité | URI |
|---|---|
| EBA | `http://publications.europa.eu/resource/authority/corporate-body/EBA` |
| ESMA | `http://publications.europa.eu/resource/authority/corporate-body/ESMA` |
| EIOPA | `http://publications.europa.eu/resource/authority/corporate-body/EIOPA` |

---

## Déclarer la langue de référence

Utiliser les codes de langue du référentiel EU :

```json
"dct:language": {
  "@id": "http://publications.europa.eu/resource/authority/language/FRA"
}
```

Codes courants : `FRA` (français), `ENG` (anglais), `DEU` (allemand).

---

## Déclarer la période de validité

```json
"dct:valid": {
  "dct:start": {"@value": "2024-01-01", "@type": "xsd:date"},
  "dct:end":   {"@value": "2026-12-31", "@type": "xsd:date"}
}
```

Si le texte est encore en vigueur et sans date de fin connue, omettez `dct:end`.

---

## Aligner avec un algorithme existant

Si votre algorithme remplace une version précédente :

```json
"dct:replaces": {
  "@id": "https://registre-algo.gouv.fr/algo/civique/droit-vote/v1"
}
```

Et dans l'ancienne version, déclarez la dépréciation :
```json
"regalgo:status":     "deprecated",
"dct:isReplacedBy": {"@id": "https://registre-algo.gouv.fr/algo/civique/droit-vote/v2"}
```

---

## Valider avant de soumettre

```bash
pip install regalgo-validator
regalgo-validator check-metadata src/mon_module/metadata.json --strict
```

L'option `--strict` active les vérifications des champs recommandés en plus des champs obligatoires.

---

## Voir aussi

- [Schéma complet du metadata.json](../reference/metadata-schema.md)
- [Vocabulaire — mapping Core Vocabularies](../reference/vocabulary.md)
- [Guide alignement référentiels EU — vocabulaire commun DINUM](https://qloridant.github.io/vocabulaire-commun/guides/03-alignement-referentiels/)
