## 1. Le cycle de vie Terraform
\[Rajouter graph]

## 2. Le State — le cœur de Terraform

Le state (`terraform.tfstate`) est le fichier JSON qui mappe ce que Terraform a créé dans le monde réel. C'est la source de vérité de Terraform, pas le code HCL.

**Ce qu'il contient :** chaque ressource managée avec son ID réel dans le provider (ex: l'ID Azure de la VM), ses attributs, ses dépendances, le provider version utilisé.

**Pourquoi c'est critique :**
- Sans state, Terraform ne sait pas ce qui existe déjà → il voudrait tout recréer
- State corrompu ou désynchronisé = situation dangereuse (drift)
- Le state peut contenir des secrets en clair (passwords, keys) → ne jamais le committer en Git

**Remote backend — obligatoire en équipe :**

```hcl
# Azure Blob Storage backend
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatemycompany"
    container_name       = "tfstate"
    key                  = "prod/myapp.tfstate"
  }
}

# S3 backend (AWS)
terraform {
  backend "s3" {
    bucket         = "my-tfstate"
    key            = "prod/myapp.tfstate"
    region         = "eu-west-1"
    encrypt        = true
    dynamodb_table = "tf-state-lock"  # locking DynamoDB
  }
}

# HCP Terraform (Terraform Cloud) — ce que tu utilises
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      name = "myapp-prod"
    }
  }
}
```

**Commandes state à maîtriser :**

```bash
terraform state list                        # lister toutes les ressources trackées
terraform state show azurerm_resource_group.main  # détails d'une ressource
terraform state mv old_name new_name        # renommer une ressource dans le state
                                            # (refactoring sans destroy/recreate)
terraform state rm azurerm_resource_group.main    # retirer du state SANS détruire
                                            # (pour gérer manuellement une ressource)
terraform import azurerm_resource_group.main /subscriptions/.../resourceGroups/myRG
                                            # importer une ressource existante
terraform force-unlock LOCK_ID              # débloquer un state bloqué
terraform refresh                           # synchro state avec la réalité (deprecated → plan -refresh-only)
```

---

## 3. Syntaxe HCL — les blocs essentiels

```hcl
# ── TERRAFORM block ──────────────────────────────────
terraform {
  required_version = ">= 1.5"
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.90"   # ~> = pessimistic constraint : >= 3.90, < 4.0
    }
  }
}

# ── PROVIDER block ────────────────────────────────────
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

# ── RESOURCE block ────────────────────────────────────
resource "azurerm_resource_group" "main" {
  name     = "myapp-${var.environment}-rg"
  location = var.location
  tags     = local.common_tags
}

# ── DATA SOURCE block ─────────────────────────────────
# Lire une ressource existante (non managée par ce Terraform)
data "azurerm_subscription" "current" {}
data "azurerm_client_config" "current" {}

output "tenant_id" {
  value = data.azurerm_client_config.current.tenant_id
}

# ── VARIABLE block ────────────────────────────────────
variable "environment" {
  type        = string
  description = "Deployment environment"
  default     = "dev"
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Must be dev, staging, or prod."
  }
}

# ── LOCAL block ───────────────────────────────────────
locals {
  common_tags = {
    environment = var.environment
    team        = "platform"
    managed_by  = "terraform"
  }
  app_name = "${var.project}-${var.environment}"
}

# ── OUTPUT block ──────────────────────────────────────
output "resource_group_id" {
  value       = azurerm_resource_group.main.id
  description = "The ID of the resource group"
  sensitive   = false  # true = masqué dans les logs
}
```

---

## 4. Variables — types et précédence

**Types HCL :**

```hcl
variable "count"        { type = number }
variable "name"         { type = string }
variable "enabled"      { type = bool }
variable "tags"         { type = map(string) }
variable "ports"        { type = list(number) }
variable "subnets"      { type = set(string) }

# Types complexes
variable "vm_config" {
  type = object({
    size       = string
    disk_gb    = number
    public_ip  = bool
  })
  default = {
    size      = "Standard_D2s_v3"
    disk_gb   = 128
    public_ip = false
  }
}

variable "instances" {
  type = list(object({
    name = string
    size = string
  }))
}
```

**Précédence des variables** (du plus faible au plus fort) :

```
1. default dans le bloc variable
2. terraform.tfvars (chargé automatiquement)
3. terraform.tfvars.json
4. *.auto.tfvars (ordre alphabétique)
5. -var-file="custom.tfvars" (flag CLI)
6. -var="key=value" (flag CLI)
7. TF_VAR_name (variable d'environnement)
```

```bash
# Passage de variables en pratique
terraform apply -var="environment=prod"
terraform apply -var-file="prod.tfvars"
TF_VAR_environment=prod terraform apply
```

---

## 5. Expressions et fonctions — les plus utiles

```hcl
# Références de ressources
azurerm_resource_group.main.id
azurerm_resource_group.main.name
azurerm_resource_group.main.location

# Références de data sources
data.azurerm_subscription.current.id

# Interpolation de strings
name = "myapp-${var.environment}-rg"
name = "${local.app_name}-${random_string.suffix.result}"

# Conditionnelle (ternaire)
sku = var.environment == "prod" ? "Premium" : "Standard"
count = var.create_resource ? 1 : 0

# for expression
tags = { for k, v in var.extra_tags : k => upper(v) }
subnet_ids = [for s in azurerm_subnet.main : s.id]

# fonctions courantes
length(var.subnets)              # longueur d'une liste/map
toset(var.regions)               # convertir liste en set (dédup)
tolist(var.region_set)           # set en liste
merge(local.common_tags, var.extra_tags)   # merger des maps
lookup(var.sku_map, var.env, "default")    # lookup dans une map avec fallback
contains(["prod","staging"], var.env)      # test d'appartenance
coalesce(var.override_name, local.default_name)  # premier non-null
try(var.complex.optional_field, "default")       # accès safe avec fallback
cidrsubnet("10.0.0.0/16", 8, 1)           # calculer un sous-réseau → 10.0.1.0/24
cidrhost("10.0.1.0/24", 5)                # IP d'un host → 10.0.1.5
jsonencode({ key = "value" })             # encoder en JSON
base64encode("my-secret")
```

---

## 6. `count` vs `for_each` — distinction capitale

C'est un point de friction fréquent en entretien.

```hcl
# COUNT — index numérique, fragile
resource "azurerm_resource_group" "envs" {
  count    = length(var.environments)
  name     = "${var.environments[count.index]}-rg"
  location = var.location
}
# Problème : si on retire "staging" de ["dev","staging","prod"]
# Terraform voit l'index 1 disparaître → il détruit staging ET recrée prod !

# FOR_EACH — clé stable, robuste
resource "azurerm_resource_group" "envs" {
  for_each = toset(var.environments)   # ou une map
  name     = "${each.key}-rg"
  location = var.location
}
# each.key   = la clé (string avec toset, clé de map avec map)
# each.value = la valeur (= each.key avec toset, valeur de map)

# FOR_EACH avec map — encore plus expressif
variable "subnets" {
  default = {
    web = "10.0.1.0/24"
    app = "10.0.2.0/24"
    db  = "10.0.3.0/24"
  }
}

resource "azurerm_subnet" "main" {
  for_each             = var.subnets
  name                 = each.key                # "web", "app", "db"
  address_prefixes     = [each.value]            # le CIDR
  virtual_network_name = azurerm_virtual_network.main.name
  resource_group_name  = azurerm_resource_group.main.name
}

# Référencer une ressource for_each
output "subnet_ids" {
  value = { for k, v in azurerm_subnet.main : k => v.id }
}
```

**Règle pratique :** utiliser `count` uniquement pour créer 0 ou 1 instance (toggle). Pour tout ce qui est multiple, préférer `for_each`.

---

## 7. Modules — structurer le code

Un module = un dossier avec des fichiers `.tf`. Le module racine appelle des modules enfants.

**Structure recommandée :**

```
infrastructure/
├── main.tf           # appels aux modules
├── variables.tf
├── outputs.tf
├── providers.tf
├── versions.tf
└── modules/
    ├── networking/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    ├── aks/
    │   ├── main.tf
    │   ├── variables.tf
    │   └── outputs.tf
    └── database/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

**Appeler un module :**

```hcl
# Depuis le module racine
module "networking" {
  source = "./modules/networking"    # chemin local

  # ou depuis le registry public
  source  = "Azure/network/azurerm"
  version = "5.3.0"

  # ou depuis Git
  source = "git::https://github.com/myorg/tf-modules.git//networking?ref=v1.2.0"

  # passer des variables au module
  resource_group_name = azurerm_resource_group.main.name
  location            = var.location
  vnet_cidr           = "10.0.0.0/16"
  subnets             = var.subnets
}

# Accéder aux outputs du module
resource "azurerm_kubernetes_cluster" "main" {
  # ...
  default_node_pool {
    vnet_subnet_id = module.networking.aks_subnet_id  # output du module
  }
}
```

**Dans le module enfant (`modules/networking/outputs.tf`) :**

```hcl
output "aks_subnet_id" {
  value       = azurerm_subnet.aks.id
  description = "ID of the AKS subnet"
}

output "vnet_id" {
  value = azurerm_virtual_network.main.id
}
```

---

## 8. Lifecycle et méta-arguments

```hcl
resource "azurerm_kubernetes_cluster" "main" {
  # ...

  lifecycle {
    # Ne jamais détruire ce cluster (protection prod)
    prevent_destroy = true

    # Ignorer les changements sur ces attributs
    # (ex: autoscaler modifie node_count en dehors de Terraform)
    ignore_changes = [
      default_node_pool[0].node_count,
      tags["LastModified"],
    ]

    # Créer le nouveau avant de détruire l'ancien
    # (zero-downtime pour les ressources qui ne peuvent pas être modifiées en place)
    create_before_destroy = true

    # Déclencher un replace si cette valeur change
    replace_triggered_by = [azurerm_resource_group.main]
  }

  # Dépendance explicite (quand la dépendance n'est pas dans les arguments)
  depends_on = [
    azurerm_role_assignment.aks_network,
    azurerm_subnet.aks,
  ]
}
```

---

## 9. Workspaces — environnements multiples

**Deux approches en entretien :**

**Approche 1 — Workspaces Terraform :**

```bash
terraform workspace new prod
terraform workspace new staging
terraform workspace select prod
terraform workspace list

# Dans le code, accéder au workspace courant
resource "azurerm_resource_group" "main" {
  name = "myapp-${terraform.workspace}-rg"
}
```

**Approche 2 — Répertoires séparés (plus robuste, recommandé en prod) :**

```
environments/
├── dev/
│   ├── main.tf         # appelle les modules
│   ├── dev.tfvars
│   └── backend.tf      # backend avec key "dev/terraform.tfstate"
├── staging/
│   ├── main.tf
│   ├── staging.tfvars
│   └── backend.tf
└── prod/
    ├── main.tf
    ├── prod.tfvars
    └── backend.tf
```

L'approche répertoires est préférable car elle donne un state isolé par environnement (une erreur en prod ne peut pas toucher dev), un historique Git séparé, et une revue de code plus claire.

---

## 10. Terraform en CI/CD — patterns pratiques

**Pattern GitLab CI complet :**

```yaml
variables:
  TF_VERSION: "1.7.0"
  TF_DIR: "./infrastructure"

stages:
  - validate
  - plan
  - apply

.tf_base:
  image: hashicorp/terraform:${TF_VERSION}
  before_script:
    - cd ${TF_DIR}
    - terraform init -input=false
  environment:
    name: ${TF_ENVIRONMENT}

tf:validate:
  extends: .tf_base
  stage: validate
  script:
    - terraform fmt -check -recursive
    - terraform validate
  rules:
    - if: $CI_MERGE_REQUEST_ID

tf:plan:
  extends: .tf_base
  stage: plan
  script:
    - terraform plan -out=tfplan -input=false -var-file="${TF_ENVIRONMENT}.tfvars"
  artifacts:
    paths:
      - ${TF_DIR}/tfplan
    expire_in: 1 day
  rules:
    - if: $CI_COMMIT_BRANCH == "main"

tf:apply:prod:
  extends: .tf_base
  stage: apply
  variables:
    TF_ENVIRONMENT: prod
  script:
    - terraform apply -input=false tfplan
  needs: [tf:plan]
  when: manual
  rules:
    - if: $CI_COMMIT_BRANCH == "main"
```

**Authentification en CI/CD :**

```bash
# Via Service Principal (variables CI/CD)
ARM_CLIENT_ID=...
ARM_CLIENT_SECRET=...
ARM_TENANT_ID=...
ARM_SUBSCRIPTION_ID=...

# Via OIDC (Workload Identity Federation) — recommandé
# GitHub Actions / GitLab CI échangent un JWT contre un token Azure
# → pas de secret à stocker, token éphémère
ARM_USE_OIDC=true
ARM_CLIENT_ID=...       # client_id de l'app registration
ARM_TENANT_ID=...
ARM_SUBSCRIPTION_ID=...
```

---

## 11. Commandes avancées à connaître

```bash
# Formater le code (toujours avant de committer)
terraform fmt -recursive

# Valider la syntaxe (sans appel API provider)
terraform validate

# Voir le graphe de dépendances
terraform graph | dot -Tpng > graph.png

# Plan ciblé (une seule ressource)
terraform plan -target=azurerm_kubernetes_cluster.main

# Apply ciblé
terraform apply -target=module.networking

# Détruire une ressource spécifique
terraform destroy -target=azurerm_resource_group.old_rg

# Plan avec variables inline
terraform plan -var="environment=prod" -var="location=westeurope"

# Plan refresh-only (voir le drift sans rien changer)
terraform plan -refresh-only

# Voir les outputs
terraform output
terraform output resource_group_id
terraform output -json | jq '.subnet_ids.value'

# Tester une expression sans appliquer
terraform console
> cidrsubnet("10.0.0.0/16", 8, 2)
"10.0.2.0/24"
> length(["a","b","c"])
3
```

---

## 12. Questions typiques d'entretien Terraform

**"C'est quoi le drift ?"** → Différence entre le state Terraform et la réalité de l'infra. Arrive quand quelqu'un modifie une ressource manuellement (via portal Azure, az cli…) sans passer par Terraform. `terraform plan -refresh-only` détecte le drift. Solution : soit `terraform apply` pour ramener à l'état déclaré, soit `terraform state rm` + `terraform import` pour réincorporer les changements manuels.

**"Différence entre `count` et `for_each` ?"** → `count` utilise un index numérique — fragile si on supprime un élément au milieu de la liste, car les indices changent et Terraform détruit/recrée des ressources. `for_each` utilise une clé stable (string) — un élément supprimé n'affecte pas les autres. Utiliser `for_each` dès qu'il y a plusieurs instances d'une ressource.

**"C'est quoi `terraform import` et quand l'utiliser ?"** → Permet d'incorporer dans le state une ressource existante (créée manuellement ou par un autre outil). Le code HCL doit déjà exister, `import` ajoute juste l'entrée dans le state. Cas d'usage typique : migration vers Terraform d'une infra existante, ou ressource créée en urgence manuellement qu'on veut maintenant gérer proprement. Depuis Terraform 1.5, on peut aussi déclarer les imports dans le code avec un bloc `import {}`.

**"Comment éviter de stocker des secrets dans le state ?"** → Impossible à 100% — certaines ressources (ex: `azurerm_postgresql_flexible_server` avec `administrator_password`) stockent des valeurs sensibles dans le state. Mitigation : state chiffré au repos (Azure Blob avec CMK, S3 avec SSE), accès restreint au state par RBAC, marquer les outputs avec `sensitive = true`. Idéalement, stocker les secrets dans Key Vault et les passer via data sources.

**"Différence entre `data` source et `resource` ?"** → `resource` = Terraform crée, gère et détruit. `data` = Terraform lit seulement une ressource existante, sans la gérer. Exemple : `data "azurerm_subscription" "current"` lit l'ID de la subscription courante sans rien créer.

**"C'est quoi `depends_on` et quand l'utiliser ?"** → Force une dépendance explicite quand Terraform ne peut pas la déduire automatiquement des références. Exemple : un role assignment doit être créé avant de déployer un AKS qui en a besoin au boot, mais l'AKS ne référence pas directement le role assignment dans ses arguments. Cas rare — en général Terraform déduit les dépendances des références implicites.

**"Workspaces vs répertoires séparés ?"** → Les workspaces partagent le même backend et le même code, avec des states séparés. Simples mais fragiles : une typo dans le workspace name peut appliquer en prod ce qui était prévu pour dev. Les répertoires séparés ont des backends distincts, des states totalement isolés, et une surface d'erreur plus petite. Pour des environnements critiques (prod), la séparation par répertoire est plus sûre.

**"Qu'est-ce que le `terraform.lock.hcl` ?"** → Fichier de lock des versions de providers, généré par `terraform init`. Garantit que toute l'équipe (et la CI) utilise exactement les mêmes versions de providers. À committer en Git. Similaire à un `package-lock.json` en Node.

---

## 13. Bonnes pratiques à placer en entretien

- Toujours utiliser un **remote backend** avec locking (jamais de state local en équipe)
- **`terraform fmt`** et **`terraform validate`** dans la CI avant tout plan
- **`-out=tfplan`** + **`apply tfplan`** en CI pour garantir que ce qui est appliqué est exactement ce qui a été reviewé
- Séparer les **modules réutilisables** des **configurations d'environnement** (DRY sans over-engineering)
- **`prevent_destroy = true`** sur les ressources stateful critiques (base de données, state bucket lui-même)
- **Ne jamais** lancer un `apply` sans avoir lu le plan — surtout les lignes en rouge (destroy/replace)
- Versionner les modules (`?ref=v1.2.0` sur les sources Git, `version = "~> 3.0"` sur le registry)
- Les **outputs** des modules doivent tout exposer ce que les modules parents peuvent avoir besoin — ne pas accéder aux ressources internes d'un module depuis l'extérieur