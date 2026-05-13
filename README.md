# Registre des Algorithmes Réglementaires — Documentation

Documentation officielle pour packager, publier et rendre interopérables des algorithmes réglementaires sur PyPI.

**Format :** [Diátaxis](https://diataxis.fr/) | **Thème :** [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) | **Déployé sur :** GitHub Pages

## Accès rapide

→ **[Lire la documentation](https://your-org.github.io/algo-registry-docs/)**

## Structure

```
docs/
├── index.md                          # Accueil
├── tutorials/                        # 🎓 Apprendre en faisant
│   ├── 01-first-package.md
│   ├── 02-publish-pypi.md
│   └── 03-register-algo.md
├── how-to-guides/                    # 🔧 Recettes pratiques
│   ├── choose-build-tool.md
│   ├── versioning.md
│   ├── interoperability.md
│   ├── compliance-tests.md
│   ├── cicd.md
│   ├── dependencies.md
│   └── metadata.md
├── reference/                        # 📐 Spécifications
│   ├── vocabulary.md
│   ├── metadata-schema.md
│   ├── interface-contract.md
│   ├── package-structure.md
│   ├── naming-conventions.md
│   └── registry-api.md
└── explanation/                      # 💡 Concepts
    ├── registry-architecture.md
    ├── why-interoperability.md
    ├── algo-lifecycle.md
    └── governance.md
```

## Développement local

```bash
pip install -r requirements-docs.txt
mkdocs serve
# → http://127.0.0.1:8000
```

## Contribuer

Les contributions sont bienvenues. Ouvrir une PR sur la branche `main`.  
La documentation est déployée automatiquement à chaque merge.

## Liens

- [Vocabulaire commun DINUM](https://qloridant.github.io/vocabulaire-commun/)
- [SEMIC — Core Vocabularies EU](https://joinup.ec.europa.eu/collection/semantic-interoperability-community-semic)
- [CPSV-AP Specification](https://semiceu.github.io/CPSV-AP/)
- [ELI — European Legislation Identifier](https://eur-lex.europa.eu/eli-register/)
# doc-catalogue-regles
