---
marp: true
theme: default
style: |
  section {background-color: #121114}
  h1,h2,h3 {color: #8393f0}
  p,ul,li,td,th {color: #ccc}
  section table td {background-color: #121114}
  section table th {background-color: #272133; font-weight: bolder}
  pre {background-color: #17151a; color: #ccc}
  blockquote {border: 2px green solid; border-left-width: 15px; padding: 0.5em 15px;font-style: italic }
paginate: true
header: "DORA : Mesurer la performance de livraison logicielle"
footer: "![height:20px](https://raw.githubusercontent.com/DiginamicInternal/PublicAssets/refs/heads/main/Logo-diginamic-color-blk.png)"
---

<center>

![pas Cette dora](https://upload.wikimedia.org/wikipedia/fr/thumb/5/54/Dora_logo_licence.png/250px-Dora_logo_licence.png?utm_source=fr.wikipedia.org&utm_campaign=parser&utm_content=thumbnail)

## Mesurer la performance de livraison logicielle

</center>

---

# Chapitre 1

## Origines et évolutions

Comprendre d'où viennent les métriques, c'est comprendre :

- pourquoi elles sont **cinq**
- pourquoi certaines ont **changé de nom**
- pourquoi les niveaux de performance **bougent chaque année**

---

# 2014 : définir "la performance IT"

Première étude : établir un lien statistique entre **performance IT** et **performance organisationnelle**.

Encore faut-il définir "performance IT" quantitativement.

| Variable candidate | Retenue en 2014 ? |
|---|---|
| Deployment frequency | oui |
| Lead time for changes | oui |
| Mean time to restore | oui |
| Change fail rate | **non** |

---

# Le taux d'échec écarté

Le change fail rate **ne corrèle pas** suffisamment avec les trois autres.

Il ne forme pas un construit latent unique avec elles.

> Il n'était pas jugé inutile.
> Il était jugé **statistiquement différent**.

Dix ans plus tard, ce constat donnera le second facteur.

---

# 2015 : débit et stabilité

- **Débit** : deployment frequency, lead time
- **Stabilité** : MTTR, change fail rate

> Le mythe "la vitesse se paie en stabilité" tombe.

## Les meilleures équipes sont bonnes sur les deux axes simultanément

---

# 2018-2021 : de la disponibilité à la fiabilité

**2018** - introduction de la disponibilité
"IT performance" devient **SDO performance**

**2021** - la disponibilité devient la **fiabilité**
latence · performance · scalabilité

⚠️ Présentée à tort comme "la cinquième métrique"

La fiabilité mesure la performance **opérationnelle**, pas la performance de **livraison**.

---

# 2023 : le MTTR change de nom et de périmètre

| **Avant**                                                        | **Après**                                                                                                          |
|------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| *mean time to restore* - toute panne, quelle qu'en soit la cause | *failed deployment recovery time* - restauration après **qu'un changement mis en production a dégradé le service** |

Une coupure de datacenter **n'entre plus** dans la métrique.

---

# 2024 : cinq métriques, deux facteurs

Le change fail rate est un **proxy du retravail** → on mesure le retravail directement.

| Facteur | Métriques |
|---|---|
| **Débit** | change lead time · deployment frequency · **failed deployment recovery time** |
| **Instabilité** | change fail rate · deployment rework rate |

Le temps de restauration bascule du côté du **débit**.

---

# La frise

- **2014** - 4 variables testées, 3 retenues
- **2015** - débit / stabilité, le mythe tombe
- **2018** - + disponibilité, "SDO performance"
- **2021** - disponibilité → fiabilité
- **2023** - MTTR → failed deployment recovery time
- **2024** - + rework rate : 5 métriques, 2 facteurs
- **2025** - DORA cesse d'être un acronyme, recentrage sur l'IA

---

# Chapitre 2

## Les cinq métriques

---

# Vue d'ensemble

| # | Métrique | Facteur | Question |
|---|---|---|---|
| 1 | Change lead time | Débit | Combien de temps entre "c'est écrit" et "c'est en prod" ? |
| 2 | Deployment frequency | Débit | À quelle cadence livre-t-on de la valeur ? |
| 3 | Failed deployment recovery time | Débit | Quand une livraison casse, en combien de temps se relève-t-on ? |
| 4 | Change fail rate | Instabilité | Quelle proportion de livraisons dégrade la prod ? |
| 5 | Deployment rework rate | Instabilité | Quelle part ne sert qu'à réparer ? |

---

# Avancés et retardés

Les cinq métriques sont à la fois :

- des **indicateurs avancés** de la performance organisationnelle et du bien-être des équipes
- des **indicateurs retardés** de vos pratiques de développement et de livraison

## Elles ne disent pas quoi faire. Elles disent où regarder.

---

# 1. Change lead time

**Du commit en gestion de version au déploiement en production.**

- Départ : *committer date* du commit sur la branche par défaut
- Arrivée : fin du déploiement en production
- Publier **médiane (P50)** et **P90**

Le plus sensible à la **taille des lots**.

---

# 2. Deployment frequency

**Nombre de déploiements sur une période**, ou délai entre deux déploiements.

- Ne compter que `environment = production`
- Un **build** n'est pas un **déploiement**
- Une préproduction n'est pas une production

Le plus sensible à la **taille des lots**.

---

# 3. Failed deployment recovery time

**Temps de restauration après un déploiement ayant dégradé la production.**

- Début : **détection** (alerte) - pas l'ouverture du ticket
- Fin : **service rétabli** - pas le post-mortem rédigé
- Uniquement les incidents **causés par un déploiement**

Le plus sensible à la qualité de l'**observabilité**.

---

# 4. Change fail rate

**Part des déploiements exigeant une intervention immédiate.**

- Cluster *elite* : autour de **5 %**
- Métrique **bruitée** par nature
- Jamais d'objectif d'équipe construit sur elle seule

Le plus sensible à la qualité des **tests** et de l'**observabilité**.

---

# 5. Deployment rework rate

**Part des déploiements non planifiés consécutifs à un incident.** *(introduite en 2024)*

Marqueur à décider **avant** de collecter :

- label `hotfix` sur la PR
- branche `hotfix/*`
- champ dédié dans le pipeline

Le plus sensible à la **discipline de saisie**.

---

# Le chemin commit → prod

```text
commit ──▶ build ──▶ tests ──▶ déploiement ──▶ production
   │                                │              │
   └────── change lead time ────────┘              │
                                                   ▼
                                            incident ?
                                                   │
                          failed deployment recovery time
                                                   │
                                              hotfix ?
```

Chaque métrique se lit comme un **segment** de ce chemin.

---

# Chapitre 2 - sensibilités dominantes

| Métrique | Sensible surtout à |
|---|---|
| Change lead time | taille des lots |
| Deployment frequency | taille des lots |
| Failed deployment recovery time | observabilité et débogage |
| Change fail rate | qualité des tests et observabilité |
| Deployment rework rate | discipline de saisie et dépendances |

Quand une métrique est mauvaise, cette colonne dit **où chercher en premier**.

---

# Chapitre 3

## Ce que DORA ne mesure pas

---

# Les questions hors périmètre

| Question | DORA ? | Où chercher |
|---|---|---|
| Livre-t-on vite et sûrement ? | ✅ | Les cinq métriques |
| Le service tient-il ses promesses ? | ❌ | Fiabilité, SLO/SLI, error budgets |
| Livre-t-on la **bonne** chose ? | ❌ | Métriques produit |
| Cet ingénieur est-il performant ? | ❌ **jamais** | Rien |
| L'équipe est-elle en bonne santé ? | Partiellement | SPACE, DevEx |
| Où sont les goulots ? | Indirectement | Value Stream Mapping |

---

# Les cadres complémentaires

- **SPACE** - satisfaction · performance · activité · communication · efficience
- **DevEx** - boucles de rétroaction · charge cognitive · état de flux
- **VSM** - gestion du flux de valeur bout-en-bout

> DORA recommande de choisir **un** cadre qui parle à votre organisation, plutôt que d'en empiler trois.

---

# Chapitre 4

## Repères de performance

---

# Quatre niveaux, recalculés chaque année

*elite* · *high* · *medium* · *low*

Issus d'une **analyse en clusters** sur l'échantillon de l'enquête.

- Les frontières **bougent** d'une année sur l'autre
- Repères **directionnels**, pas grille de notation
- 2023 → 2024 : *high* passe de **31 % à 22 %**, *low* de **17 % à 25 %**

---

# Distribution 2024

| Cluster | Part |
|---|---|
| Elite | 19 % |
| High | 22 % |
| Medium | 35 % |
| Low | 25 % |

*Elite* : déploiement à la demande · lead time < 1 jour · fail rate ~5 % · restauration < 1 h

---

# Elite vs Low : des ordres de grandeur

## lead time 127× plus court

## 182× plus de déploiements

## fail rate 8× plus bas

## restauration 2 293× plus rapide

---

# L'anomalie de 2024

Pour la première fois, *medium* affiche un fail rate **plus bas** (~10 %) que *high* (~20 %).

DORA a dû trancher entre :

- déployer **souvent** avec plus d'échecs
- déployer **lentement** avec moins d'échecs

Choix retenu : **les premiers**. Les deux récupèrent en moins d'une journée.

> Ne lisez **jamais** une métrique isolément.

---

# Les sept profils de 2025

| Profil | Part | Signature |
|---|---|---|
| Harmonious high-achievers | 20 % | Excellents partout, faible friction |
| Pragmatic performers | 20 % | Solides, engagement moyen |
| Constrained by process | 17 % | Stables mais process inefficaces, burnout |
| Stable and methodical | 15 % | Qualité élevée, rythme lent |
| Legacy bottleneck | 11 % | Réactif permanent, moral dégradé |
| High impact, low cadence | 7 % | Fort impact, forte instabilité |
| Foundational challenges | ~10 % | Mode survie |

---

# Deux enseignements

**1.** Les deux premiers profils font **40 %** de l'échantillon.
Les équipes bonnes en débit **et** en stabilité existent en nombre : le compromis est un mythe, pas un idéal théorique.

**2.** Le diagnostic détermine l'intervention.
*Constrained by process* doit réduire la friction. *High impact, low cadence* doit d'abord automatiser.
**Le même investissement produit des résultats opposés selon le profil.**

---

# Chapitre 5

## Instrumenter la chaîne

---

# Trois sources suffisent

**1. Gestion de version** - commits, PR, branches

**2. CI/CD** - builds, déploiements, environnements

**3. Suivi d'incidents** - détection, résolution, rattachement au déploiement

## Vous avez déjà les trois.

---

# Le contrat de définitions

`dora-definitions.yml`, **versionné avec le code**

```yaml
deployment:
  compte_comme_deploiement: "status success sur env=production"
  exclut: ["preproduction", "rollback automatique"]
incident:
  debut: "horodatage de la détection (alerte)"
  fin: "service rétabli, pas 'post-mortem rédigé'"
  rattachement_deploiement: "champ 'caused_by' à la clôture"
rework:
  marqueur: "PR portant le label 'hotfix' ou branche hotfix/*"
fenetre_de_reference: "90 jours glissants"
agregation: "médiane pour les durées, P90 en complément"
```

---

# Émettre depuis GitHub Actions

```yaml
- name: Créer l'évènement de déploiement
  id: deployment
  uses: actions/github-script@v9
  with:
    script: |
      const d = await github.rest.repos.createDeployment({
        owner: context.repo.owner, repo: context.repo.repo,
        ref: context.sha, environment: 'production',
        required_contexts: [], auto_merge: false });
      return d.data.id;

- name: Déployer
  run: ./scripts/deploy.sh

- name: Marquer le déploiement comme réussi
  if: success()
  uses: actions/github-script@v9
```

*Extrait - workflow complet en 5.3*

---

# L'API Deployments comme source de vérité

L'API Deployments de GitHub est le bon endroit où stocker ces évènements :

- **native** - rien à installer
- **gratuite**
- **requêtable**
- **survit** à un changement d'outillage d'observabilité

⚠️ En production, épinglez les actions sur un **SHA de commit**, pas sur un tag mouvant.

---

# Le même pipeline sous GitLab CI

```yaml
dora-deployment-event:
  stage: .post
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
  script:
    - |
      curl -X POST "$DORA_COLLECTOR/deployments" \
        -H "Content-Type: application/json" \
        -d "{\"sha\":\"$CI_COMMIT_SHA\",
             \"environment\":\"production\",
             \"deployed_at\":\"$(date -u +%FT%TZ)\"}"
```

**La mécanique ne dépend pas de l'outil.** Ce qui compte est le contrat, pas le YAML.

---

# La requête d'agrégation

```sql
SELECT
  (SELECT COUNT(*) FROM deploiements) / 90.0
      AS deployment_frequency_per_day,
  (SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY secondes) / 3600
     FROM lead_times)          AS change_lead_time_hours_p50,
  (SELECT PERCENTILE_CONT(0.9) WITHIN GROUP (ORDER BY secondes) / 3600
     FROM lead_times)          AS change_lead_time_hours_p90,
  (SELECT COUNT(*) FROM echecs)::float
     / NULLIF((SELECT COUNT(*) FROM deploiements), 0)
                               AS change_fail_rate;
```

Le `NULLIF` n'est pas cosmétique : une fenêtre sans déploiement doit produire `NULL`, pas un `0 %` trompeur.

---

# Les règles de calcul

- **Médiane et P90** - jamais la moyenne
- **Fenêtre glissante** de 28 ou 90 jours - jamais 7
- **Une vue par application** - jamais un chiffre unique pour toutes les équipes
- **UTC partout** - un datetime naïf décale silencieusement toutes les durées

---

# Outils prêts à l'emploi

- Solutions commerciales de *software delivery intelligence*
- Projets open source
- ⚠️ **Four Keys** (Google) - **archivé depuis janvier 2024**, ne pas démarrer dessus

## Ne commencez pas par l'outillage

Quick Check → conversation d'équipe → instrumentation

---

# Chapitre 6

## Améliorer : le catalogue de capabilities

---

# Thermomètre et pharmacie

Les **métriques** sont un thermomètre.

Le catalogue de **capabilities** est la pharmacie.

> Une *capability* est une pratique dont la recherche établit qu'elle **prédit** une meilleure performance.

---

# Capabilities techniques

| Capability | Effet principal |
|---|---|
| Intégration continue | lead time ↓, fail rate ↓ |
| Livraison continue | fréquence ↑, recovery ↓ |
| Tests automatisés | fail rate ↓, lead time ↓ |
| **Travail par petits lots** | **le levier le plus rentable, sur les cinq métriques** |
| Développement sur tronc commun | lead time ↓ |
| Architecture faiblement couplée | autonomie, fréquence ↑ |
| Observabilité et supervision | recovery ↓ |

---

# Deux résultats contre-intuitifs

**1. Les CAB* n'améliorent pas la stabilité.**
Corrélation **négative** avec le lead time et la fréquence. **Aucune** corrélation avec le change fail rate.
La revue par les pairs fait mieux - **y compris en environnement régulé**.

**2. La culture organisationnelle prédit la performance de livraison.**
Pas un vœu pieux managérial : un résultat statistique robuste, répété depuis dix ans.

_*CAB : Change Advisory Board — le comité d'approbation des changements (ITIL), instance externe à l'équipe qui valide les mises en production._

---

# La boucle d'amélioration

1. **Établir une ligne de base** - Quick Check
2. **Discuter des points de friction** en équipe
3. **S'engager sur une seule contrainte** - la plus significative
4. **Traduire en plan**, avec des indicateurs avancés propres
5. **Faire le travail**
6. **Vérifier les progrès**
7. **Recommencer**

> Agir sans vérifier est un gaspillage. Vérifier sans agir est une illusion.

---

# Chapitre 7

## Pièges et anti-patterns

---

# Les sept pièges recensés

| Piège | En pratique |
|---|---|
| Métrique devenue objectif | Déploiements vides pour gonfler le compteur |
| La métrique unique | Un seul chiffre pour un système complexe |
| Le secteur comme bouclier | "On est régulés, on ne peut rien changer" |
| Comparer l'incomparable | Une app mobile face à un mainframe |
| Propriété en silo | Le lead time à la dev, le fail rate aux ops |
| La compétition | Le classement tue la remontée des incidents |
| Mesurer au lieu d'améliorer | Six mois de dashboard, zéro changement |

---

# Loi de Goodhart

> Quand une mesure devient un objectif, elle cesse d'être une bonne mesure.

- Formulation d'origine : **Goodhart, 1975**, sur la politique monétaire
- La version courte citée partout est de **Marilyn Strathern**, 1997
- Cousine : la **loi de Campbell**, 1979

**Le mécanisme ne suppose aucune malhonnêteté** : dès qu'une mesure porte un enjeu, l'effort se déplace vers ce qui est mesuré.

---

# L'interdit absolu

## Jamais dans une évaluation individuelle

- Elles mesurent un **système de livraison**, pas une personne
- Elles se **gament** instantanément dès qu'il y a un enjeu de carrière
- Vous détruisez ce que vous cherchiez à mesurer : **la remontée honnête des incidents**

Ce n'est pas une question de sensibilité. C'est une question de **validité**.

---

# Erreurs d'implémentation fréquentes

| Erreur | Correction |
|---|---|
| Moyenne au lieu de médiane | Médiane + P90 |
| Builds comptés comme déploiements | Filtrer sur `environment=production` |
| Recovery toutes causes | Uniquement les incidents causés par un déploiement |
| Toutes les équipes agrégées | Une vue par application |
| Fenêtre de 7 jours | 28 ou 90 jours glissants |
| Datetimes naïfs | UTC partout |

---

# Un fail rate à 0 % n'est pas un trophée

Sur trois mois, c'est presque toujours l'un des trois :

- vous **déployez trop rarement** pour que la métrique ait un sens
- vous **ne détectez pas** vos dégradations
- vos incidents **ne sont pas rattachés** aux déploiements

Le cluster *elite* tourne autour de **5 %**, pas de 0.

> Si c'est parfait, c'est suspect. Si c'est catastrophique, vérifiez d'abord la collecte.

---

# Chapitre 8

## DORA et l'IA - rapport 2025

---

# L'IA est un amplificateur

*State of AI-assisted Software Development*, ~5 000 répondants, 100+ heures d'entretiens.

> L'IA ne crée pas d'organisations performantes.
> Elle **révèle** celles qui le sont déjà.

- **Fondations saines** → l'IA accélère un flux déjà sain
- **Dette technique** → l'IA sature une chaîne déjà engorgée

---

# Le goulot se déplace

L'IA accélère l'**écriture** du code.

Elle n'accélère **pas** :

- la revue
- les tests
- le déploiement
- la restauration après incident

Le goulot d'étranglement se déplace **en aval** et devient plus visible.

---

# Adoption massive, confiance limitée

## ~90 % des développeurs utilisent l'IA au quotidien

La confiance reste très en retrait. Les réserves dépassent l'exactitude du code : perte de compétences, déplacement d'emplois, usages malveillants.

⚠️ L'efficacité de l'IA **réduit les occasions d'apprentissage** des juniors.
Il faut des dispositifs délibérés, pas seulement des licences.

---

# Les sept capabilities qui amplifient l'IA

1. Position claire et communiquée sur l'IA
2. Écosystèmes de données sains
3. Données internes accessibles à l'IA
4. Plateformes internes de qualité
5. **Gestion de version rigoureuse**
6. **Travail par petits lots**
7. **Orientation utilisateur**

Sans le point 7, l'adoption de l'IA a un effet **négatif** : on va plus vite dans la mauvaise direction.

---

# Le paradoxe de la plateforme

Les plateformes internes de qualité corrèlent avec une **légère hausse** de l'instabilité de livraison.

Interprétation DORA : **compensation du risque**.

> Une organisation capable de se relever vite peut se permettre d'expérimenter davantage et d'accepter plus de petits échecs.

Une raison de plus de ne jamais lire le change fail rate isolément.

---

# Si vous déployez de l'IA

**Mesurez les cinq métriques avant et après.**

Si la fréquence de déploiement monte **en même temps** que le change fail rate et le rework rate :

## Vous n'avez pas gagné en performance. Vous avez déplacé le coût en aval.

---

# Respirons ensemble

---

# Ce qu'il faut retenir

- **5 métriques**, **2 facteurs** : débit et instabilité
- Elles se lisent **ensemble**, **par application**, sur une **fenêtre glissante**
- **Médiane et P90**, jamais la moyenne
- **Jamais** d'évaluation individuelle, jamais de classement entre équipes
- Le levier le plus rentable : **réduire la taille des lots**
- Elles ne disent pas quoi faire, elles disent **où regarder**

---

# Par où commencer "lundi matin" ?

1. Faire le **Quick Check** en équipe
2. Écrire le **contrat de définitions**, même imparfait
3. Instrumenter **une seule** application
4. Regarder les chiffres **ensemble**, en rétrospective
5. Choisir **une** contrainte

---

# Ressources

- **dora.dev** - définitions, capabilities, Quick Check
- **conversations.dora.dev** - questions pour animer une rétrospective
- **dora.community** - communauté de pratique
- *Accelerate* - Forsgren, Humble, Kim (2018)

Support complet, glossaire et annexes : `Readme.md`
