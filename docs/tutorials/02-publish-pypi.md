# Tutoriel 02 — Publier sur PyPI

**Durée estimée :** ~15 minutes  
**Prérequis :** [Tutoriel 01](01-first-package.md) complété, compte [TestPyPI](https://test.pypi.org/account/register/) créé.

---

## 1. Créer les tokens d'authentification

### Sur TestPyPI (validation)

1. Connectez-vous sur [test.pypi.org](https://test.pypi.org)
2. **Account Settings → API tokens → Add API token**
3. Portée : *Entire account* pour un premier token
4. Copiez le token (format `pypi-...`) — il n'est affiché qu'une fois

### Sur PyPI (production)

Répétez la même démarche sur [pypi.org](https://pypi.org).

!!! warning "Ne committez jamais vos tokens"
    Stockez-les dans votre gestionnaire de secrets (voir [guide CI/CD](../how-to-guides/cicd.md)).

---

## 2. Configurer `~/.pypirc` (optionnel, usage local)

```ini title="~/.pypirc"
[distutils]
index-servers =
    pypi
    testpypi

[pypi]
username = __token__
password = pypi-<votre-token-pypi>

[testpypi]
repository = https://test.pypi.org/legacy/
username = __token__
password = pypi-<votre-token-testpypi>
```

---

## 3. Construire les artefacts de distribution

```bash
# Depuis la racine du projet
pip install --upgrade build
python -m build

# Vérification du contenu
ls -lh dist/
# regalgo_finance_lcr-1.0.0-py3-none-any.whl   (wheel binaire)
# regalgo_finance_lcr-1.0.0.tar.gz              (source distribution)
```

!!! tip "Vérifier le wheel avant publication"
    ```bash
    pip install twine
    twine check dist/*
    # PASSED regalgo_finance_lcr-1.0.0-py3-none-any.whl
    # PASSED regalgo_finance_lcr-1.0.0.tar.gz
    ```

---

## 4. Publier sur TestPyPI

```bash
twine upload --repository testpypi dist/*
```

Vérifiez la publication :
```bash
pip install --index-url https://test.pypi.org/simple/ regalgo-finance-lcr
python -c "from regalgo_finance_lcr import LCRAlgorithm; print(LCRAlgorithm().algo_id)"
# finance.lcr.v1
```

---

## 5. Publier sur PyPI (production)

Une fois la validation sur TestPyPI réussie :

```bash
twine upload dist/*
```

Votre package est maintenant installable par n'importe qui :

```bash
pip install regalgo-finance-lcr
```

---

## 6. Vérifier la page PyPI

Rendez-vous sur `https://pypi.org/project/regalgo-finance-lcr/` et vérifiez que :

- [ ] La description README s'affiche correctement (Markdown rendu)
- [ ] Les classifiers sont présents
- [ ] Le lien *Registry* pointe vers le registre public
- [ ] La version est correcte

---

## ✅ Résultat

Votre algorithme réglementaire est publiquement installable via `pip`. Il est versionné, traçable et prêt à être référencé dans le registre.

---

## Étape suivante

[→ Intégrer au registre public](03-register-algo.md){ .md-button .md-button--primary }
