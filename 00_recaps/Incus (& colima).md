
# 📦 Incus — Doc de référence rapide

## C'est quoi ?

**Incus** est un gestionnaire de **conteneurs système** et de **machines virtuelles**, fork communautaire de LXD. Il permet de lancer des instances légères (Linux, mais aussi Windows en VM) en quelques secondes, avec une interface simple et un réseau géré automatiquement.

> Pense à Docker pour les process → Incus pour les **OS complets**.

---

## 🍎 macOS — Installation via Colima

Incus tourne nativement sur Linux. Sur Mac, on passe par **Colima** (VM légère basée sur Lima) qui expose un environnement Linux local.

### Installation

```bash
brew install colima incus
```

### Démarrer Colima avec le runtime Incus

```bash
colima start --runtime incus
```

> Options utiles : `--cpu 4 --memory 8 --disk 60` pour allouer des ressources.

### Arrêter / relancer

```bash
colima stop
colima start
colima status
```

---

## 🚀 Premiers pas

### Initialiser Incus (première fois)

```bash
incus admin init
# Répondre aux questions ou tout laisser par défaut
```

### Lancer une instance rapidement

```bash
incus launch images:ubuntu/24.04 mon-ubuntu
incus launch images:debian/12    mon-debian
incus launch images:alpine/3.19  mon-alpine
```

### Ouvrir un shell dans l'instance

```bash
incus exec mon-ubuntu -- bash
incus exec mon-ubuntu -- sh          # pour Alpine
```

### Lancer une commande one-shot

```bash
incus exec mon-ubuntu -- apt update
incus exec mon-ubuntu -- ls /etc
```

---

## 🗂️ Gestion des instances

|Action|Commande|
|---|---|
|Lister les instances|`incus list`|
|Démarrer|`incus start <nom>`|
|Arrêter (gracieux)|`incus stop <nom>`|
|Arrêter (forcé)|`incus stop <nom> --force`|
|Redémarrer|`incus restart <nom>`|
|Supprimer|`incus delete <nom>`|
|Supprimer (forcé)|`incus delete <nom> --force`|
|Infos détaillées|`incus info <nom>`|
|État en temps réel|`incus monitor`|

---

## 💾 Images

```bash
# Chercher une image
incus image list images: | grep ubuntu
incus image list images: ubuntu

# Images locales déjà téléchargées
incus image list

# Supprimer une image locale
incus image delete <fingerprint>
```

**Sources d'images courantes :**

|Source|URL|
|---|---|
|Images officielles|`images:`|
|Ubuntu Cloud|`ubuntu:`|
|Ubuntu daily|`ubuntu-daily:`|

---

## 📁 Fichiers & transferts

```bash
# Copier un fichier vers l'instance
incus file push ./mon-fichier.txt mon-ubuntu/root/

# Copier depuis l'instance
incus file pull mon-ubuntu/etc/hosts ./hosts-local

# Éditer directement un fichier dans l'instance
incus file edit mon-ubuntu/etc/hosts
```

---

## 🌐 Réseau

```bash
# Voir l'IP de l'instance
incus list
incus info mon-ubuntu | grep inet

# Lister les réseaux
incus network list

# Détails d'un réseau
incus network info incusbr0
```

---

## 💡 Snapshots

```bash
# Créer un snapshot
incus snapshot create mon-ubuntu snap1

# Lister les snapshots
incus snapshot list mon-ubuntu

# Restaurer
incus snapshot restore mon-ubuntu snap1

# Supprimer un snapshot
incus snapshot delete mon-ubuntu snap1
```

---

## ⚙️ Configuration & ressources

```bash
# Limiter CPU et RAM
incus config set mon-ubuntu limits.cpu 2
incus config set mon-ubuntu limits.memory 2GiB

# Voir la config d'une instance
incus config show mon-ubuntu

# Voir / éditer la config complète
incus config edit mon-ubuntu
```

---

## 🖥️ Machines virtuelles

```bash
# Lancer une VM au lieu d'un conteneur
incus launch images:ubuntu/24.04 ma-vm --vm

# Lancer avec plus de ressources
incus launch images:ubuntu/24.04 ma-vm --vm \
  --config limits.cpu=4 \
  --config limits.memory=4GiB
```

---

## 📄 Déclarer des machines en YAML

Incus permet de définir des instances de façon reproductible via deux mécanismes complémentaires : les **profils** (config réutilisable) et le **cloud-init** (provisioning au premier démarrage).

### Profils — config réutilisable

Un profil est un bloc YAML qui définit des ressources, devices et config appliqués à une ou plusieurs instances.

```bash
# Créer un profil vide
incus profile create webserver

# L'éditer en YAML
incus profile edit webserver
```

Exemple de profil `webserver.yaml` :

```yaml
config:
  limits.cpu: "2"
  limits.memory: 2GiB
  user.user-data: |
    #cloud-config
    packages:
      - nginx
      - curl
    runcmd:
      - systemctl enable --now nginx
devices:
  eth0:
    name: eth0
    network: incusbr0
    type: nic
  root:
    path: /
    pool: default
    size: 20GiB
    type: disk
description: "Serveur web nginx"
```

```bash
# Importer le profil depuis un fichier
incus profile edit webserver < webserver.yaml

# Lancer une instance avec ce profil
incus launch images:ubuntu/24.04 web1 --profile default --profile webserver

# Voir les profils existants
incus profile list

# Voir le profil appliqué à une instance
incus config show web1 --expanded
```

> Le profil `default` est toujours là en base (réseau + disque). On l'empile avec les profils custom.

---

### Cloud-init — provisioning au boot

La clé `user.user-data` accepte un fichier **cloud-config** standard. Elle est lue au premier démarrage de l'instance.

Exemple `cloud-init.yaml` autonome :

```yaml
#cloud-config
hostname: mon-serveur

users:
  - name: deploy
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    ssh_authorized_keys:
      - ssh-ed25519 AAAA... ta_cle_publique

packages:
  - python3
  - git
  - vim

write_files:
  - path: /etc/motd
    content: |
      Bienvenue sur mon instance Incus 🚀

runcmd:
  - apt-get upgrade -y
  - timedatectl set-timezone Europe/Paris
```

```bash
# Injecter cloud-init au lancement
incus launch images:ubuntu/24.04 mon-serveur \
  --config user.user-data="$(cat cloud-init.yaml)"

# Vérifier que cloud-init a bien tourné dans l'instance
incus exec mon-serveur -- cloud-init status --wait
incus exec mon-serveur -- cat /var/log/cloud-init-output.log
```

---

### Workflow complet : instance reproductible en YAML

```bash
# 1. Créer et importer le profil
incus profile create myapp
incus profile edit myapp < myapp-profile.yaml

# 2. Lancer N instances identiques
for i in 1 2 3; do
  incus launch images:ubuntu/24.04 app-$i \
    --profile default \
    --profile myapp
done

# 3. Vérifier
incus list
```

---

## 🤖 Ansible — Inventaire dynamique

Ansible peut **découvrir automatiquement** toutes les instances Incus en cours d'exécution grâce au plugin d'inventaire `community.general.incus`.

### Prérequis

```bash
ansible-galaxy collection install community.general
pip install ansible
```

### Fichier d'inventaire `incus.yml`

Créer un fichier `inventory/incus.yml` :

```yaml
---
plugin: community.general.incus
remotes: # si incus est lancé à travers colima
  - "colima"
groups:
  servers: "'server' in inventory_hostname"
host_fqdn: false #default true
keyed_groups: # 1 group per key found
  - key: ansible_incus_config['image.os'] | lower
```

### Connection via le plugin

Dans `group_vars/all.yml` :
```yaml
---
ansible_connection: community.general.incus
ansible_incus_remote: colima # le nom du remote à utiliser
```


### (à valider) Tagger ses instances pour les grouper

```bash
# Ajouter un label à une instance
incus config set web1 user.role=webserver
incus config set web2 user.role=webserver
incus config set db1  user.role=database
incus config set db1  user.env=prod
```

Ces labels deviennent automatiquement des groupes Ansible : `role_webserver`, `role_database`, `env_prod`, etc.

### Tester l'inventaire

```bash
# Voir tous les hôtes et groupes détectés
ansible-inventory -i inventory/incus.yml --list

# Affichage en arbre lisible
ansible-inventory -i inventory/incus.yml --graph

# Tester un ping sur toutes les instances
ansible -i inventory/incus.yml all -m ping
```

### Plugin de connexion Incus (optionnel)

Par défaut, Ansible se connecte en SSH. Pour utiliser **directement `incus exec`** (sans SSH configuré dans les instances) :

```yaml
# inventory/incus.yml — ajouter :
compose:
  ansible_connection: "'community.general.incus'"
  ansible_incus_host: inventory_hostname
```

Ou dans le playbook / `group_vars/all.yml` :

```yaml
ansible_connection: community.general.incus
```

> Avec ce plugin de connexion, pas besoin de SSH ni de clé dans les instances — idéal pour le dev local.

### Exemple de playbook ciblant des groupes dynamiques

```yaml
# playbook.yml
---
- name: Configurer les webservers
  hosts: role_webserver
  become: true
  tasks:
    - name: Installer nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Démarrer nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

- name: Configurer les bases de données
  hosts: role_database
  become: true
  tasks:
    - name: Installer postgresql
      ansible.builtin.apt:
        name: postgresql
        state: present
```

```bash
# Lancer le playbook
ansible-playbook -i inventory/incus.yml playbook.yml

# Limiter à un groupe
ansible-playbook -i inventory/incus.yml playbook.yml --limit role_webserver

# Mode dry-run
ansible-playbook -i inventory/incus.yml playbook.yml --check
```

### Structure de projet recommandée

```
mon-projet/
├── inventory/
│   └── incus.yml          # inventaire dynamique
├── group_vars/
│   ├── all.yml            # vars communes (ansible_connection, etc.)
│   ├── role_webserver.yml
│   └── role_database.yml
├── profiles/
│   ├── webserver.yaml     # profil incus
│   └── database.yaml
├── cloud-init/
│   ├── webserver.yaml
│   └── database.yaml
└── playbook.yml
```

---

## 🔁 Cas d'usage courants

### Environnement de dev isolé

```bash
incus launch images:ubuntu/24.04 dev
incus exec dev -- bash
# → apt install, pip install... sans polluer le host
```

### Tester un script sur plusieurs distros

```bash
for distro in ubuntu/24.04 debian/12 alpine/3.19; do
  incus launch images:$distro test-$distro
  incus exec test-$distro -- sh -c "echo Hello from $distro"
  incus delete test-$distro --force
done
```

### Snapshot avant opération risquée

```bash
incus snapshot create mon-ubuntu avant-migration
# ... faire l'opération ...
# Si ça plante :
incus snapshot restore mon-ubuntu avant-migration
```

### Copier une instance

```bash
incus copy mon-ubuntu mon-ubuntu-copie
incus start mon-ubuntu-copie
```

---

## 📖 Ressources

- Doc officielle : [linuxcontainers.org/incus/docs](https://linuxcontainers.org/incus/docs/main/)
- Images dispo : [images.linuxcontainers.org](https://images.linuxcontainers.org)
- Colima : [github.com/abiosoft/colima](https://github.com/abiosoft/colima)
- Plugin inventaire Ansible : [community.general.incus inventory](https://docs.ansible.com/ansible/latest/collections/community/general/incus_inventory.html)
- Plugin connexion Ansible : [community.general.incus connection](https://docs.ansible.com/ansible/latest/collections/community/general/incus_connection.html)
- Cloud-init : [cloudinit.readthedocs.io](https://cloudinit.readthedocs.io)