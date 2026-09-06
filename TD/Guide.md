# Guide pas à pas

Mode opératoire du TD : les commandes à taper, et ce que vous devez voir après les avoir tapées. Gardez [`Readme.md`](Readme.md) ouvert : c'est lui qui porte les questions auxquelles répondre.

Les valeurs indiquées ici ont été relevées le 6 septembre 2026. Excalidraw est un projet actif : vos chiffres seront proches sans être identiques. Ce qui doit correspondre, ce sont les ordres de grandeur et les rapports entre valeurs.

---

## Étape 0 : le jeton d'accès (5 min, à faire en premier)

L'API GitHub non authentifiée autorise 60 requêtes par heure. Le collecteur en consomme une quinzaine par exécution : sans jeton, la limite est atteinte à la troisième exécution, en phase 2. Avec un jeton, la limite passe à 5 000.

1. Ouvrez <https://github.com/settings/tokens>, puis "Generate new token (classic)".
2. Nom : `TD DORA`. Expiration : 7 jours.
3. Portée : cochez `public_repo` uniquement. Rien d'autre n'est nécessaire.
4. Copiez le jeton affiché. Il ne sera plus affiché ensuite.
5. Exportez-le dans le terminal où vous travaillerez :

```bash
export GITHUB_TOKEN=ghp_xxxxxxxxxxxx          # macOS, Linux, Git Bash
```
```powershell
$env:GITHUB_TOKEN = "ghp_xxxxxxxxxxxx"        # PowerShell
```

Vérifiez que la variable est définie :

```bash
echo $GITHUB_TOKEN        # doit afficher le jeton, pas une ligne vide
```

**Attention :** le jeton ne doit pas entrer dans un fichier du dépôt. S'il part dans un commit, il est compromis même après suppression du commit ; révoquez-le alors depuis la même page.

**Remarque :** la variable ne vit que dans ce terminal. Si vous en ouvrez un autre, refaites l'export.

---

## Étape 1 : forker et cloner (10 min)

### 1.1 Forker

Sur <https://github.com/excalidraw/excalidraw>, bouton "Fork" en haut à droite, puis "Create fork".

**Ce qu'un fork copie :** le code, les branches et les tags. Il ne copie ni les issues, ni les pull requests, ni les déploiements, ni les releases.

Conséquence pour le TD : vous lisez l'historique sur le dépôt amont, `excalidraw/excalidraw`, et vous n'écrivez que sur votre fork, en phase 4. Si vous pointez le collecteur sur votre fork en phase 2, il trouvera zéro déploiement. C'est le comportement attendu, pas une panne.

### 1.2 Cloner

Placez-vous dans le dossier où vous voulez travailler, à côté du dossier `td-dora`, puis :

```bash
git clone --filter=blob:none https://github.com/<votre-compte>/excalidraw.git
cd excalidraw
git rev-list --count HEAD
cd ..
```

`--filter=blob:none` récupère l'historique complet des commits sans le contenu des fichiers. Le TD n'a besoin que des dates de commit.

**Ce que vous devez voir :** un clone d'une dizaine de secondes pour environ 43 Mo, et un peu plus de 4 000 commits.

Si le clone dure plusieurs minutes et pèse des centaines de Mo, l'option `--filter=blob:none` a été omise. Supprimez le dossier et recommencez.

---

## Étape 2 : explorer l'API (30 min)

### 2.1 Les environnements

Ouvrez cette URL dans votre navigateur :

```
https://api.github.com/repos/excalidraw/excalidraw/deployments?per_page=100
```

Cherchez le champ `environment` et relevez les valeurs distinctes. Sous Firefox et Chrome, `Ctrl+F` sur `"environment"` fonctionne ; l'affichage JSON de Firefox permet aussi de filtrer.

**Ce que vous devez voir :** six valeurs distinctes. Trois commencent par `Production`, trois par `Preview`. Une seule correspond au produit lui-même, les autres concernent des exemples d'intégration.

Répondez aux questions 1 à 4 du sujet.

**Remarque :** comptez approximativement combien des 100 déploiements affichés sont des `Preview`. Ce rapport répond à la question 3.

### 2.2 Les statuts d'un déploiement

Relevez l'`id` du déploiement le plus récent dont l'`environment` est `Production – excalidraw`, puis ouvrez :

```
https://api.github.com/repos/excalidraw/excalidraw/deployments/<id>/statuses
```

**Ce que vous devez voir :** un tableau d'états successifs, contenant au moins un objet dont le champ `state` vaut `success`, avec son propre `created_at`, différent de celui du déploiement.

Répondez aux questions 5 et 6 du sujet.

---

## Étape 3 : lancer le collecteur (20 min)

Depuis le dossier `td-dora`, le clone d'excalidraw étant dans le dossier parent :

```bash
python outils/dora_metrics.py \
  --repo excalidraw/excalidraw \
  --git ../excalidraw \
  --environment "Production – excalidraw" \
  --window 90
```

Sous Windows, remplacez `python` par `py` et mettez la commande sur une seule ligne : le `\` de continuation n'existe pas en PowerShell.

**Attention :** le tiret de `Production – excalidraw` est un tiret demi-cadratin, pas un trait d'union. Retapé au clavier, il produit zéro déploiement sans message d'erreur. Copiez-collez la valeur exacte relevée à l'étape 2.1.

**Ce que vous devez voir :** un tableau en deux blocs, `DÉBIT` et `INSTABILITÉ`, avec une dizaine de déploiements sur la fenêtre, un lead time P50 de quelques dizaines d'heures, un P90 de plusieurs jours, et trois lignes à `n/a`.

### Erreurs courantes

| Symptôme | Cause | Résolution |
|---|---|---|
| 0 déploiement, aucune erreur | tiret demi-cadratin retapé au clavier | copier-coller la valeur exacte |
| `Quota de l'API GitHub épuisé` | `GITHUB_TOKEN` non défini dans ce terminal | refaire l'étape 0, vérifier avec `echo $GITHUB_TOKEN` |
| `fatal: not a git repository` | chemin `--git` erroné | vérifier avec `ls ../excalidraw` |
| `lots ignorés (sha absent)` non nul | déploiements sur des commits absents du clone | sans conséquence si le nombre est faible ; sinon `git fetch --all` |
| accents illisibles | console Windows en cp1252 | le script force l'UTF-8 ; utilisez Windows Terminal ou Git Bash |

### Lire le résultat

Remplissez le tableau de la phase 2 du sujet, puis calculez deux rapports qui ne sont pas affichés :

- commits analysés divisés par nombre de lots : la taille moyenne d'un lot ;
- P90 divisé par P50 : l'écart entre le cas courant et le cas défavorable.

**Ce que vous devez observer :** une taille de lot de quelques commits, et un rapport P90/P50 nettement supérieur à 1, de l'ordre de 4 à 5.

Répondez aux questions 7 à 13 du sujet.

**Remarque :** le collecteur accepte `--json`, ce qui permet de coller les valeurs brutes dans votre `reponses.md` sans les recopier.

---

## Étape 4 : le proxy incidents (30 min)

### 4.1 Relancer avec le label

```bash
python outils/dora_metrics.py \
  --repo excalidraw/excalidraw \
  --git ../excalidraw \
  --environment "Production – excalidraw" \
  --window 90 \
  --incident-label bug
```

**Ce que vous devez voir :** un bloc `INCIDENTS` supplémentaire, et un change fail rate toujours à `n/a`.

Notez les deux nombres : issues trouvées, et issues rattachées à un déploiement.

### 4.2 Vérifier dans l'interface

Lancez ces trois recherches dans GitHub et relevez le compteur de résultats affiché en haut de page :

```
repo:excalidraw/excalidraw is:issue label:bug
repo:excalidraw/excalidraw is:issue created:>=2026-06-08
repo:excalidraw/excalidraw is:issue label:bug created:>=2026-06-08
```

Remplacez la date par celle d'il y a 90 jours. Le collecteur l'affiche dans son en-tête, à la ligne "depuis le".

**Ce que vous devez observer :** un nombre élevé pour la première recherche, plus d'une centaine pour la deuxième, et un nombre très faible, voire nul, pour la troisième.

Répondez aux questions 14 à 18 du sujet. La question 16 se joue dans la confrontation de ces trois nombres.

---

## Étape 5 : instrumenter votre fork (50 min)

Cette étape se déroule sur votre fork, dans le clone réalisé à l'étape 1.

### 5.1 Le workflow qui émet les déploiements

Créez `.github/workflows/deploy.yml` en vous appuyant sur le workflow du chapitre 5.3 du support. Il doit :

- se déclencher sur un push vers la branche par défaut ;
- déclarer `permissions: deployments: write` ;
- créer un évènement de déploiement sur `environment: production` ;
- exécuter un déploiement fictif, `echo "déploiement"` suffit : le TD mesure la chaîne, pas l'application ;
- marquer le déploiement en `success`.

**Attention :** l'omission de `permissions: deployments: write` produit une erreur 403 peu explicite dans les logs Actions. C'est l'erreur la plus fréquente de cette étape.

Poussez, puis vérifiez dans l'onglet Actions de votre fork que le workflow passe au vert.

**Ce que vous devez voir :** sur `https://api.github.com/repos/<votre-compte>/excalidraw/deployments`, un déploiement avec `"environment": "production"`.

### 5.2 Produire de l'historique

Réalisez au moins cinq déploiements, en espaçant les commits de quelques minutes pour que les horodatages diffèrent. L'un d'eux doit provenir d'une branche `hotfix/*` :

```bash
git checkout -b hotfix/correctif-urgent
echo "correctif" >> NOTES.md
git commit -am "hotfix: correction urgente"
git push origin hotfix/correctif-urgent
```

Fusionnez ensuite cette branche dans la branche par défaut pour déclencher le déploiement.

### 5.3 Simuler un incident et le rattacher

1. Créez le label `incident` sur votre fork : onglet Issues, puis Labels, puis "New label".
2. Ouvrez une issue avec ce label.
3. Dans le corps de l'issue, ajoutez une ligne indiquant le déploiement en cause :

```
caused_by: 123456789
```

Remplacez le nombre par l'`id` d'un de vos déploiements, relevé dans l'API.

4. Fermez l'issue quelques minutes plus tard : cet écart devient votre temps de restauration.

### 5.4 Mesurer votre fork

```bash
python outils/dora_metrics.py \
  --repo <votre-compte>/excalidraw \
  --git ../excalidraw \
  --environment production \
  --window 90 \
  --incident-label incident \
  --rework-prefix hotfix/
```

**Ce que vous devez voir :** `Change fail rate`, `Failed deployment recovery time` et `Deployment rework rate` affichent cette fois des valeurs chiffrées.

Si le change fail rate reste à `n/a` alors que l'issue existe, vérifiez que la ligne `caused_by:` figure dans le corps de l'issue et non dans un commentaire, et que l'identifiant correspond à un déploiement de la fenêtre.

Répondez aux questions 19 à 21 du sujet.

---

## Étape 6 : lecture critique des outils (25 min)

Pas de manipulation. Ouvrez les trois dépôts et relevez, pour chacun, la date de la dernière version publiée et celle du dernier commit :

- <https://github.com/apache/incubator-devlake>
- <https://github.com/middlewarehq/middleware>
- <https://github.com/dora-team/fourkeys>

**Ce que vous devez observer :** un projet actif, un projet dont le dépôt bouge mais dont les versions publiées datent, et un projet portant le bandeau "This repository has been archived".

Répondez aux questions 22 à 24 du sujet.

---

## Avant de rendre

Votre `reponses.md`, poussé sur votre fork, doit contenir :

- [ ] le `dora-definitions.yml` rempli en phase 0, avec l'heure où vous l'avez figé ;
- [ ] le tableau de résultats de la phase 2 ;
- [ ] les réponses numérotées 1 à 24 ;
- [ ] la ligne de votre contrat qui explique l'écart avec le binôme voisin ;
- [ ] les métriques mesurées sur votre fork en phase 4.
