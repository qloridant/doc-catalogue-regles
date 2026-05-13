# Guide — Automatiser la publication (CI/CD)

Ce guide montre comment configurer un pipeline GitHub Actions qui publie automatiquement votre algorithme sur PyPI et notifie le registre à chaque release.

---

## Prérequis

- Dépôt GitHub avec votre package
- Secrets configurés dans **Settings → Secrets and variables → Actions** :
  - `PYPI_API_TOKEN` : token PyPI de production
  - `TEST_PYPI_API_TOKEN` : token TestPyPI
  - `REGISTRY_TOKEN` : token d'accès au registre public (obtenu via le registre)

---

## Pipeline complet

```yaml title=".github/workflows/release.yml"
name: Release

on:
  push:
    tags:
      - "v*.*.*"   # Déclenché sur git tag v1.2.3

permissions:
  contents: read
  id-token: write   # Requis pour Trusted Publishing (OIDC)

jobs:
  # ── 1. Tests ─────────────────────────────────────────────
  test:
    name: Tests & conformité
    runs-on: ubuntu-latest
    strategy:
      matrix:
        python-version: ["3.10", "3.11", "3.12"]
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python ${{ matrix.python-version }}
        uses: actions/setup-python@v5
        with:
          python-version: ${{ matrix.python-version }}
          cache: pip

      - name: Installer les dépendances
        run: |
          pip install --upgrade pip
          pip install -e ".[dev]"
          pip install regalgo-validator

      - name: Lancer les tests
        run: pytest tests/ -v --tb=short

      - name: Vérifier la conformité au registre
        run: regalgo-validator check .

  # ── 2. Build ──────────────────────────────────────────────
  build:
    name: Build wheel & sdist
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip

      - name: Build
        run: |
          pip install build twine
          python -m build
          twine check dist/*

      - name: Upload artefacts
        uses: actions/upload-artifact@v4
        with:
          name: dist
          path: dist/

  # ── 3. Publish TestPyPI ───────────────────────────────────
  publish-test:
    name: Publier sur TestPyPI
    needs: build
    runs-on: ubuntu-latest
    environment: testpypi
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/

      - name: Publier sur TestPyPI
        uses: pypa/gh-action-pypi-publish@release/v1
        with:
          repository-url: https://test.pypi.org/legacy/
          password: ${{ secrets.TEST_PYPI_API_TOKEN }}

  # ── 4. Publish PyPI ───────────────────────────────────────
  publish-pypi:
    name: Publier sur PyPI
    needs: publish-test
    runs-on: ubuntu-latest
    environment: pypi   # Environnement avec approbation manuelle recommandée
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: dist
          path: dist/

      - name: Publier sur PyPI (Trusted Publishing)
        uses: pypa/gh-action-pypi-publish@release/v1
        # Avec Trusted Publishing OIDC — pas besoin de token explicite
        # Configurer le Trusted Publisher sur pypi.org au préalable

  # ── 5. Notifier le registre ───────────────────────────────
  notify-registry:
    name: Notifier le registre public
    needs: publish-pypi
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Extraire la version du tag
        id: version
        run: echo "VERSION=${GITHUB_REF_NAME#v}" >> $GITHUB_OUTPUT

      - name: Notifier le registre
        run: |
          pip install regalgo-validator
          regalgo-validator notify-registry \
            --package "${{ github.event.repository.name }}" \
            --version "${{ steps.version.outputs.VERSION }}" \
            --token "${{ secrets.REGISTRY_TOKEN }}"
```

---

## Workflow de validation uniquement (PRs)

```yaml title=".github/workflows/ci.yml"
name: CI

on:
  pull_request:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"
          cache: pip
      - run: pip install -e ".[dev]" regalgo-validator
      - run: pytest tests/ -v
      - run: regalgo-validator check .
```

---

## Dépendances de développement dans `pyproject.toml`

```toml
[project.optional-dependencies]
dev = [
    "pytest>=8.0",
    "pytest-cov",
    "regalgo-validator",
    "twine",
    "build",
]
```

---

## Créer une release

```bash
# Tagger une nouvelle version
git tag v1.1.0
git push origin v1.1.0

# Le pipeline se déclenche automatiquement
```

!!! tip "Trusted Publishing (recommandé)"
    Configurez le [Trusted Publishing OIDC](https://docs.pypi.org/trusted-publishers/)
    sur pypi.org pour éviter de stocker des tokens dans vos secrets GitHub.
    Plus sécurisé et sans rotation manuelle de tokens.

---

## Voir aussi

- [Versionner selon CalVer/SemVer réglementaire](versioning.md)
- [Écrire des tests de conformité](compliance-tests.md)
