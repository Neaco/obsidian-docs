## 1. Concepts fondamentaux

**Pipeline** = ensemble ordonné de stages, chaque stage contenant des jobs. Un push sur le repo déclenche automatiquement le pipeline défini dans `.gitlab-ci.yml` à la racine du projet.

**Runner** = agent qui exécute les jobs. Il "poll" le coordinateur GitLab pour récupérer des jobs à exécuter. Les runners sont enregistrés avec un token et un executor.

**Executor** = environnement dans lequel le job tourne. Les principaux :
- `docker` — chaque job dans un conteneur isolé (recommandé)
- `shell` — directement sur la machine du runner
- `kubernetes` — chaque job dans un pod k8s
- `docker+machine` — provision dynamique de VMs (GitLab.com)

---

## 2. Structure du `.gitlab-ci.yml`

```yaml
# Définir les stages dans l'ordre
stages:
  - build
  - test
  - deploy

# Variables globales
variables:
  APP_VERSION: "1.0.0"
  DOCKER_REGISTRY: registry.gitlab.com

# Image Docker par défaut pour tous les jobs
default:
  image: python:3.11
  before_script:
    - pip install -r requirements.txt

# Job minimal
test:unit:
  stage: test
  script:
    - pytest tests/unit/
  artifacts:
    reports:
      junit: report.xml

# Job avec conditions
deploy:prod:
  stage: deploy
  script:
    - ./deploy.sh
  environment:
    name: production
    url: https://myapp.com
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

---

## 3. Anatomie d'un job

```yaml
mon-job:
  stage: test                    # à quel stage appartient ce job
  image: node:20                 # image Docker (override le default)
  
  before_script:                 # exécuté avant script
    - npm ci
  
  script:                        # commandes principales (obligatoire)
    - npm run test
    - npm run build
  
  after_script:                  # exécuté même si script échoue
    - echo "Cleanup"
  
  artifacts:                     # fichiers à conserver après le job
    paths:
      - dist/
      - coverage/
    expire_in: 1 week
    reports:
      coverage_report:
        coverage_format: cobertura
        path: coverage.xml
  
  cache:                         # cache entre jobs/pipelines
    key: $CI_COMMIT_REF_SLUG
    paths:
      - node_modules/
  
  timeout: 30 minutes
  retry: 2                       # retry en cas d'échec
  allow_failure: false
  
  tags:                          # sélectionner un runner spécifique
    - docker
    - linux
```

---

## 4. Le pipeline en détail---

## 5. `rules` vs `only/except` — distinction critique

`only/except` est l'ancienne syntaxe, dépréciée. En entretien, savoir expliquer pourquoi `rules` est préféré.

```yaml
# ANCIEN — only/except (éviter)
deploy:
  only:
    - main
  except:
    - schedules

# NOUVEAU — rules (à utiliser)
deploy:
  rules:
    - if: $CI_COMMIT_BRANCH == "main" && $CI_PIPELINE_SOURCE != "schedule"
      when: on_success
    - when: never           # défaut : ne pas lancer

# Patterns utiles avec rules
job:
  rules:
    # Sur merge request uniquement
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    
    # Seulement si un fichier a changé
    - changes:
        - src/**/*
        - Dockerfile
    
    # Déclenchement manuel
    - when: manual
    
    # Toujours (même si échec précédent)
    - when: always
```

---

## 6. Variables CI/CD

**Variables prédéfinies à connaître par cœur** :

```bash
$CI_COMMIT_SHA          # hash du commit
$CI_COMMIT_SHORT_SHA    # 8 premiers chars
$CI_COMMIT_BRANCH       # nom de la branche
$CI_COMMIT_TAG          # tag si pipeline de tag
$CI_COMMIT_REF_SLUG     # branche slugifiée (safe pour Docker tags)
$CI_PIPELINE_SOURCE     # push / merge_request_event / schedule / web
$CI_PROJECT_NAME        # nom du projet
$CI_PROJECT_PATH        # namespace/projet
$CI_REGISTRY            # adresse du registry GitLab intégré
$CI_REGISTRY_IMAGE      # image de base du projet
$CI_REGISTRY_USER       # user pour s'auth au registry
$CI_REGISTRY_PASSWORD   # password pour s'auth au registry
$CI_ENVIRONMENT_NAME    # nom de l'environnement (si défini)
$CI_JOB_TOKEN           # token éphémère du job (API GitLab)
```

**Définir des variables** : dans le YAML (globalement ou par job), dans Settings > CI/CD > Variables (secrets), ou via `--env` sur un pipeline déclenché via API.

Les variables `protected` ne sont disponibles que sur branches/tags protégés. Les variables `masked` ne s'affichent pas dans les logs.

---

## 7. Artifacts et Cache

Ce sont deux mécanismes distincts, confusion fréquente en entretien.

```yaml
# ARTIFACTS — passer des fichiers entre jobs
build:
  script: npm run build
  artifacts:
    paths:
      - dist/           # disponible pour les jobs suivants
    expire_in: 7 days   # nettoyage automatique
    when: always        # même si le job échoue (utile pour rapports)

deploy:
  script: scp dist/ server:/var/www/
  # dist/ est automatiquement téléchargé depuis build

# CACHE — accélérer les jobs en réutilisant des données
test:
  cache:
    key:
      files:
        - package-lock.json   # invalidé si ce fichier change
    paths:
      - node_modules/
    policy: pull-push   # pull au début, push à la fin (défaut)
    # policy: pull      # télécharger seulement (jobs de lecture)
    # policy: push      # uploader seulement (job qui crée le cache)
```

**La différence clé** : artifacts = données entre jobs du même pipeline. Cache = données entre pipelines différents (performance).

---

## 8. Environments et Deployments

```yaml
deploy:staging:
  stage: deploy
  script:
    - kubectl apply -f k8s/staging/
  environment:
    name: staging
    url: https://staging.myapp.com
    on_stop: stop:staging    # job de teardown

stop:staging:
  stage: deploy
  script:
    - kubectl delete -f k8s/staging/
  environment:
    name: staging
    action: stop
  when: manual
  rules:
    - if: $CI_MERGE_REQUEST_ID   # seulement sur MR

deploy:prod:
  environment:
    name: production
    url: https://myapp.com
  when: manual                   # déclenchement manuel obligatoire
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

Les environments permettent de tracker les déploiements dans GitLab UI, de voir quelle version est en prod, et de rollback facilement.

---

## 9. Patterns avancés courants

**Docker build et push au registry GitLab :**
```yaml
build:image:
  stage: build
  image: docker:24
  services:
    - docker:24-dind      # Docker-in-Docker
  variables:
    DOCKER_TLS_CERTDIR: "/certs"
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_REGISTRY_PASSWORD $CI_REGISTRY
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA
```

**Extend et anchors YAML pour DRY :**
```yaml
# Anchor YAML
.deploy_template: &deploy_template
  image: bitnami/kubectl:latest
  before_script:
    - kubectl config use-context $KUBE_CONTEXT

deploy:staging:
  <<: *deploy_template
  script: kubectl apply -f k8s/staging/

deploy:prod:
  <<: *deploy_template
  script: kubectl apply -f k8s/prod/
```

**`extends` GitLab (préférable aux anchors) :**
```yaml
.base_deploy:
  image: bitnami/kubectl:latest
  before_script:
    - kubectl config use-context $KUBE_CONTEXT

deploy:staging:
  extends: .base_deploy
  script: kubectl apply -f k8s/staging/
```

**Trigger d'un pipeline downstream (multi-projet) :**
```yaml
trigger:deploy:
  stage: deploy
  trigger:
    project: mygroup/myapp-infra
    branch: main
    strategy: depend   # attendre que le pipeline déclenché finisse
```

**Pipeline schedules (cron) :**
Via UI Settings > CI/CD > Schedules. Dans le YAML, détecter avec `$CI_PIPELINE_SOURCE == "schedule"`.

---

## 10. GitLab CI dans un contexte DevOps/SRE

**Intégration avec ton stack** (pertinent pour ton projet) :

```yaml
# Pattern typique GitLab CI + ArgoCD (GitOps)
stages:
  - test
  - build
  - update-manifests   # pas de deploy direct

update:k8s-manifests:
  stage: update-manifests
  script:
    # Mettre à jour le tag d'image dans les manifests Helm/kustomize
    - sed -i "s|image:.*|image: $CI_REGISTRY_IMAGE:$CI_COMMIT_SHORT_SHA|" k8s/deployment.yaml
    - git config user.email "ci@gitlab.com"
    - git add k8s/
    - git commit -m "chore: update image to $CI_COMMIT_SHORT_SHA"
    - git push
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
# ArgoCD détecte le changement et déploie — GitLab CI ne fait jamais kubectl apply
```

---

## 11. Questions typiques d'entretien

**"Différence entre artifact et cache ?"** → Artifact = output d'un job passé aux jobs suivants dans le même pipeline (ex: binaire compilé). Cache = couche de performance entre pipelines successifs (ex: `node_modules`). Le cache peut être invalide ou absent — le job doit fonctionner sans. Les artifacts sont garantis présents pour les jobs qui en dépendent.

**"Différence entre `rules` et `only/except` ?"** → `only/except` est simple mais a des comportements ambigus (notamment sur les MR). `rules` est plus expressif, évalué dans l'ordre, et permet des conditions complexes avec `if`, `changes`, `exists`. En production, toujours utiliser `rules`.

**"Comment sécuriser les secrets dans GitLab CI ?"** → Variables CI/CD dans Settings, marquées `protected` (uniquement branches protégées) et `masked` (ne s'affiche pas dans les logs). Pour les secrets très sensibles : intégration avec HashiCorp Vault ou AWS Secrets Manager via `id_tokens` (OIDC).

**"Comment fonctionne Docker-in-Docker ?"** → Le job tourne dans un conteneur Docker, et pour builder des images il a besoin d'un daemon Docker. `docker:dind` est un service sidecar qui expose ce daemon via TCP. Alternative plus sécurisée : `kaniko` (build sans daemon, sans privilèges root).

**"C'est quoi un runner shared vs spécifique ?"** → Shared : disponible pour tous les projets de l'instance, géré par l'admin. Spécifique (project-specific) : enregistré pour un seul projet, utile pour isoler des environnements sensibles (prod) ou des runners avec accès réseau particulier.

**"Comment accélérer un pipeline lent ?"** → Paralléliser les jobs dans un même stage. Bien configurer le cache (clé fine pour éviter les faux hits). Utiliser des images Docker légères. `interruptible: true` pour annuler des pipelines redondants sur une MR. `needs:` pour déclencher un job dès que ses dépendances directes sont prêtes (sans attendre tout le stage).

**"C'est quoi `needs:` ?"** → Permet à un job de démarrer dès que les jobs listés dans `needs` sont terminés, sans attendre que tout le stage soit fini. Crée un DAG (Directed Acyclic Graph) à la place d'une exécution strictement par stages.

```yaml
deploy:staging:
  stage: deploy
  needs:
    - job: build:image    # démarre dès que build:image est OK
    - job: test:unit      # sans attendre test:e2e
  script: ./deploy.sh staging
```

**"Qu'est-ce que `CI_JOB_TOKEN` ?"** → Token éphémère généré par GitLab pour chaque job, valide uniquement pendant son exécution. Permet de s'authentifier à l'API GitLab, au Container Registry, ou de cloner d'autres repos du même groupe sans stocker de credentials. Révoqué automatiquement à la fin du job.

---

## 12. Positionnement dans l'écosystème

- **GitLab CI vs GitHub Actions** : GitHub Actions est event-driven avec marketplace d'actions. GitLab CI est plus intégré (registry, environments, security scanning natifs). GitLab a `include:` pour partager des templates entre projets, l'équivalent est plus complexe sur GitHub.
- **GitLab CI vs Jenkins** : Jenkins est très flexible mais nécessite beaucoup de maintenance (plugins, updates, Groovy). GitLab CI est "batteries included", config-as-code native, pas de serveur CI séparé à maintenir.
- **GitLab CI + ArgoCD** : GitLab CI pour le CI (build, test, push image). ArgoCD pour le CD (détecter les changements dans le repo de manifests, sync vers k8s). La frontière : GitLab CI ne fait jamais `kubectl apply` en prod dans un workflow GitOps — il met à jour les manifests, ArgoCD s'occupe du reste.