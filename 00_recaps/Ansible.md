## 1. Concepts fondamentaux

**Ce qu'est Ansible** : outil d'automatisation *agentless* (sans agent). Il se connecte via SSH (Linux) ou WinRM (Windows) depuis un control node vers des managed nodes. Ansible s'installe uniquement sur le control node. Les nœuds gérés ont besoin de Python et d'un accès SSH, c'est tout.

**Les 5 composants clés à maîtriser** :

- **Inventory** — liste des hôtes à gérer (fichier INI, YAML, ou dynamique)
- **Playbook** — fichier YAML décrivant les tâches à exécuter
- **Task** — une action unitaire (installer un paquet, copier un fichier…)
- **Module** — l'unité d'action réutilisable (`apt`, `copy`, `service`, `template`…)
- **Role** — structure de répertoires pour organiser un playbook complexe

---

## 2. L'Inventory

```ini
# inventory.ini — format INI (le plus courant en entretien)
[webservers]
web01 ansible_host=192.168.1.10
web02 ansible_host=192.168.1.11

[dbservers]
db01 ansible_host=192.168.1.20

[prod:children]   # groupe de groupes
webservers
dbservers

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
```

**Variables d'hôte importantes** : `ansible_host`, `ansible_user`, `ansible_port`, `ansible_become`, `ansible_python_interpreter`.

**Inventory dynamique** : un script ou plugin qui génère l'inventory à la volée (ex : `aws_ec2`, `gcp_compute`). Important pour les clouds.

---

## 3. Structure d'un Playbook

```yaml
---
- name: Configure web servers       # nom du play
  hosts: webservers                  # cible de l'inventory
  become: true                       # privilege escalation (sudo)
  vars:
    http_port: 80

  tasks:
    - name: Install nginx
      ansible.builtin.apt:
        name: nginx
        state: present
        update_cache: true

    - name: Start and enable nginx
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    - name: Deploy config
      ansible.builtin.template:
        src: nginx.conf.j2
        dest: /etc/nginx/nginx.conf
      notify: Reload nginx          # déclenche un handler

  handlers:
    - name: Reload nginx
      ansible.builtin.service:
        name: nginx
        state: reloaded
```

Points clés : **`become`** = sudo, **`notify/handlers`** = actions déclenchées seulement si une tâche a changé quelque chose, **`state`** = idempotence.

---

## 4. Les modules les plus importants---

## 5. Variables et précédence

Ansible a 22 niveaux de précédence. Ceux à connaître par cœur, du plus faible au plus fort :

`defaults/main.yml` → `group_vars/all` → `group_vars/<group>` → `host_vars/<host>` → `vars/main.yml` → `vars dans le play` → `extra vars (-e)`

**Règle simple** : les `extra_vars` (`-e`) écrasent toujours tout.

```yaml
# Référencer une variable
- name: Create user
  ansible.builtin.user:
    name: "{{ app_user }}"
    home: "/home/{{ app_user }}"
```

**`register`** — capturer la sortie d'une tâche :
```yaml
- name: Check if file exists
  ansible.builtin.stat:
    path: /etc/myapp.conf
  register: conf_file

- name: Show result
  ansible.builtin.debug:
    msg: "Exists: {{ conf_file.stat.exists }}"
```

---

## 6. Les Roles — structure à connaître

```
roles/
└── nginx/
    ├── defaults/main.yml    # variables par défaut (priorité faible)
    ├── vars/main.yml        # variables fixes (priorité haute)
    ├── tasks/main.yml       # tâches principales
    ├── handlers/main.yml    # handlers
    ├── templates/           # fichiers Jinja2 (.j2)
    ├── files/               # fichiers statiques
    ├── meta/main.yml        # dépendances entre rôles
    └── README.md
```

Utiliser un role dans un playbook :
```yaml
- hosts: webservers
  roles:
    - role: nginx
      vars:
        http_port: 8080
    - common
    - { role: geerlingguy.php, php_version: "8.2" }
```

**Ansible Galaxy** = dépôt communautaire de roles. `ansible-galaxy install geerlingguy.nginx`.

---

## 7. Contrôle de flux et idempotence

```yaml
# when — condition
- name: Install on Debian only
  ansible.builtin.apt:
    name: nginx
  when: ansible_facts['os_family'] == "Debian"

# loop — boucle
- name: Create users
  ansible.builtin.user:
    name: "{{ item }}"
    state: present
  loop:
    - alice
    - bob
    - charlie

# block/rescue/always — try/catch/finally
- block:
    - name: Risky task
      ansible.builtin.command: /usr/bin/risky
  rescue:
    - name: Handle failure
      ansible.builtin.debug:
        msg: "Task failed, recovering..."
  always:
    - name: Always runs
      ansible.builtin.debug:
        msg: "Cleanup"
```

**`changed_when` / `failed_when`** — surcharger la détection de changement/échec, très utile avec `command`/`shell` :
```yaml
- ansible.builtin.command: /usr/bin/check-status
  register: result
  changed_when: false          # ne jamais marquer comme changé
  failed_when: result.rc > 2   # échouer seulement si rc > 2
```

---

## 8. Ansible Vault

Chiffrement des secrets :
```bash
ansible-vault create secrets.yml          # créer fichier chiffré
ansible-vault encrypt vars/passwords.yml  # chiffrer un existant
ansible-vault edit secrets.yml            # éditer
ansible-vault view secrets.yml            # lire

# Utilisation à l'exécution
ansible-playbook site.yml --ask-vault-pass
ansible-playbook site.yml --vault-password-file ~/.vault_pass
```

Vault peut chiffrer une valeur seule (inline) :
```yaml
db_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  ...
```

---

## 9. Commandes essentielles

```bash
# Tester la connectivité
ansible all -i inventory.ini -m ping

# Exécuter une tâche ad-hoc
ansible webservers -i inventory.ini -m apt -a "name=nginx state=present" -b

# Lancer un playbook
ansible-playbook -i inventory.ini site.yml

# Mode dry-run (check mode)
ansible-playbook site.yml --check

# Voir les changements sans les appliquer
ansible-playbook site.yml --check --diff

# Limiter à un sous-ensemble d'hôtes
ansible-playbook site.yml --limit web01

# Lister les tâches sans les exécuter
ansible-playbook site.yml --list-tasks

# Taguer et filtrer des tâches
ansible-playbook site.yml --tags "install,configure"
ansible-playbook site.yml --skip-tags "restart"

# Voir les facts d'un hôte
ansible web01 -i inventory.ini -m setup
```

---

## 10. Questions typiques d'entretien

**"Qu'est-ce que l'idempotence ?"** → Un playbook peut être joué plusieurs fois et produire le même résultat. Si nginx est déjà installé, `state: present` ne fait rien. C'est la philosophie centrale d'Ansible.

**"Différence entre `copy` et `template` ?"** → `copy` pousse un fichier statique. `template` traite un fichier Jinja2 avec les variables Ansible avant de le pousser.

**"Différence entre `command` et `shell` ?"** → `command` n'utilise pas le shell du système (pas de pipes, redirections). `shell` passe par `/bin/sh` donc supporte `|`, `>`, `&&`. Préférer `command` par sécurité.

**"Qu'est-ce que `become` ?"** → Privilege escalation (équivalent de `sudo`). `become: true` + `become_user: root` par défaut. On peut aussi faire `become_user: postgres` pour basculer sur un autre user.

**"Différence entre `vars` et `defaults` dans un role ?"** → `defaults` a la priorité la plus basse — facilement surchargé. `vars` a une priorité haute — difficile à surcharger. Les defaults sont pour les valeurs configurables, vars pour les constantes internes.

**"Comment Ansible gère-t-il les secrets ?"** → Ansible Vault chiffre les fichiers ou valeurs individuelles en AES256. Le mot de passe peut être fourni interactivement ou via fichier/env variable. En CI/CD, on utilise souvent une variable d'environnement `ANSIBLE_VAULT_PASSWORD_FILE`.

**"C'est quoi les facts ?"** → Variables collectées automatiquement sur chaque hôte au début d'un play (`gather_facts: true` par défaut). Contient l'OS, l'IP, le hostname, la RAM, etc. Accessible via `ansible_facts['distribution']` ou `ansible_os_family`.

**"Qu'est-ce qu'un handler ?"** → Une tâche qui ne s'exécute qu'une seule fois à la fin du play, et uniquement si une tâche a émis un `notify`. Sert typiquement à recharger un service après une modification de config, sans le recharger à chaque tâche.

---

## 11. Ansible dans un contexte DevOps/SRE

Pour un entretien DevOps, positionne Ansible dans l'écosystème IaC :

- **Ansible vs Terraform** : Terraform = provisioning d'infra (créer des VMs, réseaux). Ansible = configuration management (ce qu'on installe et configure dedans). Ils se complètent.
- **Ansible vs Chef/Puppet** : Chef/Puppet ont un agent sur les nœuds et un serveur central. Ansible est agentless, plus simple à adopter, moins de moving parts.
- **Ansible + CI/CD** : playbooks déclenchés par GitLab CI/GitHub Actions pour déployer des apps ou provisionner des environnements.
- **Ansible + k8s** : le module `kubernetes.core.k8s` permet de gérer des manifests Kubernetes, bien que `kubectl apply` + ArgoCD soit généralement préféré pour GitOps.

Pour ton projet (k3s + ArgoCD + Ansible sur OVH), la question classique sera : *"pourquoi Ansible ET ArgoCD ?"* La réponse : Ansible provisionne les VMs et installe k3s (couche OS/infra), ArgoCD gère les déploiements applicatifs dans Kubernetes (couche GitOps). Les deux couches sont distinctes.