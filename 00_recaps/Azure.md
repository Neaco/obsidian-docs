
## 1. Hiérarchie organisationnelle Azure

**Le Resource Group** est le concept le plus opérationnel : toutes les ressources d'un RG partagent le même lifecycle (on peut tout supprimer d'un coup), la même région par défaut, et les mêmes tags. RBAC et policies peuvent s'appliquer à n'importe quel niveau de la hiérarchie et héritent vers le bas.


## 2. Réseau Azure — les fondations

C'est le sujet le plus piégeux en entretien DevOps Azure. À bien maîtriser.

**VNet (Virtual Network)** = réseau privé isolé dans Azure. Analogie avec un VPC AWS. Chaque VNet a un espace d'adressage CIDR (ex: `10.0.0.0/16`). Les ressources dans le même VNet peuvent se parler. Les VNets sont régionaux.

**Subnet** = subdivision du VNet. On assigne des ressources à un subnet (`10.0.1.0/24` = subnet "web", `10.0.2.0/24` = subnet "db"). Certains services Azure nécessitent leur propre subnet dédié (AKS, App Service Environment, Azure Firewall).

**NSG (Network Security Group)** = firewall stateful à l'entrée/sortie d'un subnet ou d'une NIC. Règles avec priorité (plus petit = plus prioritaire), source/destination IP ou service tag, port, protocole.

```
Règle NSG type :
Priority: 100
Name: Allow-HTTPS-inbound
Direction: Inbound
Source: Internet (service tag)
Destination: 10.0.1.0/24
Port: 443
Protocol: TCP
Action: Allow
```

**Service Tags** = alias pour des plages IP gérées par Azure (`Internet`, `AzureLoadBalancer`, `VirtualNetwork`, `AzureMonitor`…). Évite de gérer des listes d'IP manuellement.

**VNet Peering** = relier deux VNets (même région ou cross-region). Trafic passe par le backbone Microsoft, pas par Internet. Non-transitif : si A peered avec B et B peered avec C, A ne peut pas parler à C sans peering direct.

**Private Endpoint** = injecter un service PaaS (Storage, PostgreSQL, Key Vault…) dans ton VNet via une IP privée. Le service n'est plus accessible depuis Internet, uniquement depuis le VNet.---

## 3. Identité et accès — IAM Azure (RBAC)

**Azure RBAC** = Role-Based Access Control. Trois éléments : un **principal** (user, group, service principal, managed identity), un **rôle** (ensemble de permissions), un **scope** (management group, subscription, resource group, ou ressource individuelle).

**Rôles built-in à connaître :**

- `Owner` — tout faire, y compris gérer les accès
- `Contributor` — tout faire sauf gérer les accès
- `Reader` — lire uniquement
- `User Access Administrator` — gérer les accès uniquement
- Rôles spécifiques : `AcrPull`, `Storage Blob Data Contributor`, `Key Vault Secrets User`…

**Service Principal vs Managed Identity** — distinction critique en entretien :

```
Service Principal = identité pour une application.
Nécessite un client_id + client_secret (ou certificat).
→ À gérer : rotation des secrets, stockage sécurisé.

Managed Identity = identité gérée par Azure pour une ressource Azure.
Pas de secret à gérer — Azure gère le cycle de vie.
System-assigned : liée à une ressource, supprimée avec elle.
User-assigned : ressource indépendante, assignable à plusieurs services.
→ À préférer dès que possible.
```

Usage typique : une VM ou une Container App avec une Managed Identity peut lire des secrets dans Key Vault **sans stocker aucun credential**.


## 4. Services compute — les principaux

**Azure Virtual Machines** : IaaS classique. Tailles (`Standard_D2s_v3` = 2 vCPU, 8 GB RAM). Availability Sets (VMs sur faults domains différents dans un datacenter) vs Availability Zones (VMs dans des datacenters physiques différents dans la région).

**Azure Kubernetes Service (AKS)** : Kubernetes managé. Azure gère le control plane (gratuit). On paie les nœuds (VM pools). Intègre nativement : Managed Identity pour les pods (Workload Identity), Azure CNI pour le réseau, Azure Monitor pour les logs/métriques, Azure Container Registry pour les images.

```bash
# Commandes AKS de base
az aks create --resource-group myRG --name myCluster \
  --node-count 3 --node-vm-size Standard_D2s_v3 \
  --enable-managed-identity --enable-workload-identity

az aks get-credentials --resource-group myRG --name myCluster
kubectl get nodes
```

**Azure Container Apps** : Kubernetes sans gérer Kubernetes. Basé sur KEDA pour l'autoscaling, Dapr pour les microservices. Bon pour des workloads event-driven qui doivent scaler à zéro. Billing à la consommation (CPU/mémoire utilisés, pas les nœuds alloués).

**Azure Functions** : serverless. Billing à l'exécution (1 million d'appels gratuits/mois). Triggers : HTTP, Timer (cron), Queue, Event Hub, Blob…


## 5. Stockage et bases de données

**Azure Storage Account** — contient plusieurs services :
- `Blob Storage` : objets (images, backups, logs). Tiers : Hot, Cool, Archive.
- `File Storage` : partages SMB/NFS montables dans des VMs.
- `Queue Storage` : messages (simple, pour couplage léger).
- `Table Storage` : NoSQL clé-valeur basique.

**Azure PostgreSQL Flexible Server** — la version moderne (vs Single Server, déprécié). Supporte la haute disponibilité avec standby dans une autre zone. Maintenance windows configurables. Private Endpoint pour isoler du réseau public. Extensions PostgreSQL standard supportées.

```terraform
resource "azurerm_postgresql_flexible_server" "main" {
  name                   = "myapp-postgres"
  resource_group_name    = azurerm_resource_group.main.name
  location               = "westeurope"
  version                = "15"
  administrator_login    = "pgadmin"
  administrator_password = var.pg_password
  sku_name               = "GP_Standard_D2s_v3"
  storage_mb             = 32768
  zone                   = "1"

  high_availability {
    mode                      = "ZoneRedundant"
    standby_availability_zone = "2"
  }
}
```

**Azure Key Vault** — stockage sécurisé pour secrets, clés de chiffrement, certificats. Intègre nativement avec Managed Identity (pas de credential pour accéder). Soft delete + purge protection pour éviter les suppressions accidentelles.


## 6. Azure Monitor — observabilité

C'est une plateforme complète, pas un seul outil.

```
Azure Monitor
├── Metrics          — séries temporelles (CPU, mémoire, latence…)
├── Logs (Log Analytics Workspace)
│   ├── Azure Activity Log  — qui a fait quoi sur les ressources
│   ├── Resource Logs       — logs des services Azure (AKS, PG…)
│   └── Custom Logs         — logs applicatifs via agent ou API
├── Alerts           — règles sur métriques ou requêtes KQL
├── Application Insights  — APM pour les apps (traces, dépendances)
└── Dashboards / Workbooks
```

**Log Analytics** utilise **KQL (Kusto Query Language)** — à connaître basiquement :

```kusto
// Erreurs des pods AKS des dernières 24h
ContainerLog
| where TimeGenerated > ago(24h)
| where LogEntry contains "ERROR"
| project TimeGenerated, ContainerName, LogEntry
| order by TimeGenerated desc
| limit 100

// CPU moyen par VM
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)
```

**Alerts** : on définit une condition (métrique dépasse un seuil, requête KQL retourne des résultats) et une action (email, webhook, runbook). Les actions sont groupées dans des **Action Groups**.


## 7. Infrastructure as Code — Terraform sur Azure

Ton terrain de jeu naturel. Les providers et ressources clés :

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.90"
    }
  }
  backend "azurerm" {            # state dans un Storage Account
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatemyapp"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}

provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

# Resource group
resource "azurerm_resource_group" "main" {
  name     = "myapp-prod-rg"
  location = "West Europe"
  tags     = { environment = "prod", team = "platform" }
}

# VNet + Subnet
resource "azurerm_virtual_network" "main" {
  name                = "myapp-vnet"
  address_space       = ["10.0.0.0/16"]
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_subnet" "aks" {
  name                 = "aks-subnet"
  resource_group_name  = azurerm_resource_group.main.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.1.0/24"]
}
```

**Authentification Terraform vers Azure** — ordre de priorité :
1. Variables d'environnement (`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, `ARM_TENANT_ID`, `ARM_SUBSCRIPTION_ID`)
2. Azure CLI (`az login` — pour le développement local)
3. Managed Identity (pour les pipelines CI/CD sur Azure)

En CI/CD GitLab/GitHub Actions → préférer **OIDC** (Workload Identity Federation) pour éviter de stocker des secrets : le pipeline échange un token JWT contre un token Azure AD éphémère.


## 8. Azure Container Apps — ton service de la mission

Puisque c'est dans ton stack de la mission freelance, focus dessus :

```yaml
# azure.containerapp.yaml
properties:
  managedEnvironmentId: /subscriptions/.../managedEnvironments/myenv
  configuration:
    ingress:
      external: true
      targetPort: 8000
      transport: http
    secrets:
      - name: db-password
        keyVaultUrl: https://myvault.vault.azure.net/secrets/db-password
        identity: system
    registries:
      - server: myregistry.azurecr.io
        identity: system    # Managed Identity pour ACR
  template:
    containers:
      - name: myapp
        image: myregistry.azurecr.io/myapp:v1.2.0
        resources:
          cpu: 0.5
          memory: 1Gi
        env:
          - name: DATABASE_URL
            secretRef: db-password
    scale:
      minReplicas: 1
      maxReplicas: 10
      rules:
        - name: http-rule
          http:
            metadata:
              concurrentRequests: "50"
```

**Managed Environment** = le "cluster" sous-jacent, partagé entre plusieurs Container Apps. Intègre un log workspace, un VNet (optionnel), et un runtime Dapr.


## 9. Questions typiques d'entretien Azure

**"Différence entre NSG et Azure Firewall ?"** → NSG = filtrage L4 (IP/port) sur subnet ou NIC, gratuit, basique. Azure Firewall = service managé L4+L7, règles FQDN (filtrer par nom de domaine), inspection TLS, central. NSG pour l'isolation basique entre subnets, Firewall pour le contrôle du trafic sortant vers Internet ou entre VNets.

**"C'est quoi un Private Endpoint vs Service Endpoint ?"** → Service Endpoint = route le trafic vers le service PaaS par le backbone Azure mais le service reste accessible depuis Internet avec des règles firewall. Private Endpoint = injecte le service dans le VNet via une IP privée, le service peut être complètement désactivé depuis Internet. Private Endpoint = plus sécurisé, recommandé.

**"Comment une VM accède à un secret Key Vault sans credential ?"** → System-assigned Managed Identity sur la VM. Assigner le rôle `Key Vault Secrets User` à cette identité sur le Key Vault. Le code de la VM appelle l'endpoint IMDS local (`169.254.169.254`) pour obtenir un token, puis l'API Key Vault. Zéro credential stocké.

**"Différence entre Availability Set et Availability Zone ?"** → Availability Set = distribue les VMs sur des fault domains (racks différents) et update domains (mise à jour séquentielle) dans un même datacenter. Protège contre les pannes hardware locales. Availability Zone = datacenters physiquement séparés dans une région (alimentation, réseau, refroidissement indépendants). Protège contre une panne de datacenter entier. AZ = SLA 99.99%, AS = SLA 99.95%.

**"Comment sécuriser un AKS en production ?"** → Private cluster (API server non exposé sur Internet), Azure CNI (pods avec IP VNet directe), Workload Identity (Managed Identity par pod), RBAC Kubernetes + Azure RBAC intégré, Azure Policy pour les admission controllers (pas de root, images signées), Private Endpoint pour ACR et autres PaaS, Network Policies pour isoler les pods entre eux, Microsoft Defender for Containers pour la détection de menaces.

**"Qu'est-ce que le Terraform backend Azure ?"** → Le state Terraform est stocké dans un Azure Blob Storage (container avec versioning activé). Le locking est géré via Azure Blob leases pour éviter les runs concurrents. En pratique : un Storage Account dédié dans un RG séparé, avec accès restreint par RBAC.


## 10. Commandes `az cli` à connaître

```bash
# Login et contexte
az login
az account list --output table
az account set --subscription "my-subscription"

# Resource groups
az group list --output table
az group create --name myRG --location westeurope

# VMs
az vm list --resource-group myRG --output table
az vm start/stop --name myVM --resource-group myRG

# AKS
az aks list --output table
az aks get-credentials --resource-group myRG --name myCluster
az aks scale --resource-group myRG --name myCluster --node-count 5
az aks upgrade --resource-group myRG --name myCluster --kubernetes-version 1.28

# Container Apps
az containerapp list --output table
az containerapp update --name myapp --resource-group myRG \
  --image myregistry.azurecr.io/myapp:v2.0.0

# Key Vault
az keyvault secret set --vault-name myVault --name mySecret --value "s3cr3t"
az keyvault secret show --vault-name myVault --name mySecret --query value -o tsv

# RBAC
az role assignment list --assignee <object-id> --output table
az role assignment create --assignee <object-id> \
  --role "Contributor" --scope /subscriptions/<sub-id>/resourceGroups/myRG

# Diagnostic / monitoring
az monitor metrics list --resource <resource-id> --metric "Percentage CPU"
az monitor log-analytics query --workspace <workspace-id> \
  --analytics-query "ContainerLog | limit 10"
```
