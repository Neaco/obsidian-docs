
---

## 1. Le modèle OSI

\[Rajouter graph]
En tant que devops, le modèle OSI sert surtout à localiser un problème. "Le ping passe mais HTTP ne répond pas" → L1-L3 OK, problème L4-L7. 
"Pas de réponse du tout" → chercher L1-L3. Les couches 5 et 6 sont rarement nommées directement en pratique DevOps — c'est L4 et L7 qui dominent.

---

## 2. IP et CIDR — arithmétique réseau

À savoir calculer de tête pour les entretiens.

**Notation CIDR** : `192.168.1.0/24` → le `/24` indique que les 24 premiers bits sont le réseau. Reste 8 bits pour les hôtes = 2⁸ = 256 adresses, dont 2 réservées (réseau + broadcast) → **254 hôtes utilisables**.

```
/32  → 1 IP (host unique)
/31  → 2 IPs (liens point-à-point)
/30  → 4 IPs, 2 utilisables
/29  → 8 IPs, 6 utilisables
/28  → 16 IPs, 14 utilisables
/27  → 32 IPs, 30 utilisables
/26  → 64 IPs, 62 utilisables
/25  → 128 IPs, 126 utilisables
/24  → 256 IPs, 254 utilisables   ← subnet classique
/23  → 512 IPs, 510 utilisables
/22  → 1024 IPs
/20  → 4096 IPs
/16  → 65 536 IPs               ← VNet/VPC typique
/8   → 16 millions IPs

Plages privées RFC 1918 :
10.0.0.0/8
172.16.0.0/12
192.168.0.0/16
```

**Supernetting** : `/23` regroupe deux `/24` consécutifs (ex: `10.0.0.0/24` + `10.0.1.0/24` = `10.0.0.0/23`).

---

## 3. TCP vs UDP — distinction fondamentale**Le TCP three-way handshake** — à connaître par cœur :

```
Client          Serveur
  |--- SYN ------->|    "je veux me connecter, mon seq=100"
  |<-- SYN-ACK ----|    "ok, mon seq=200, j'attends ton ack=101"
  |--- ACK ------->|    "reçu, j'attends seq=201"
  |                |    connexion établie
  
Fermeture (four-way) :
  |--- FIN ------->|
  |<-- ACK --------|
  |<-- FIN --------|
  |--- ACK ------->|
```

`TIME_WAIT` = état après fermeture, dure 2×MSL (~60s) pour absorber les paquets retardés. Peut épuiser les ports locaux sous forte charge.

---

## 4. DNS — le protocole le plus important à maîtriser

**Résolution DNS récursive** : ton navigateur → resolver local (souvent le routeur ou `8.8.8.8`) → root nameserver (13 clusters dans le monde) → TLD nameserver (`.com`, `.fr`…) → authoritative nameserver du domaine.

**Types d'enregistrements à connaître :**

```
A      → nom → IPv4          (api.monsite.com → 1.2.3.4)
AAAA   → nom → IPv6
CNAME  → alias → autre nom   (www → monsite.com)
           ⚠ ne peut pas être à la racine du domaine (apex)
MX     → serveurs de mail
TXT    → texte libre (vérification domaine, SPF, DKIM)
NS     → nameservers autoritaires du domaine
PTR    → IPv4 → nom (reverse DNS)
SRV    → service/port discovery (_http._tcp.example.com)
SOA    → infos de la zone (TTL par défaut, contact admin)
```

**TTL** = durée de cache. Un TTL de 3600 = le resolver garde la réponse 1h avant de redemander. Avant une migration, baisser le TTL à 60s. Après, le remonter.

**`dig` — commandes essentielles :**

```bash
dig api.monsite.com                    # résolution A standard
dig api.monsite.com +short             # juste l'IP
dig api.monsite.com MX                 # enregistrements MX
dig @8.8.8.8 api.monsite.com          # via resolver spécifique
dig api.monsite.com +trace             # trace complète root → autoritative
dig -x 1.2.3.4                         # reverse DNS (PTR)
dig api.monsite.com NS                 # quels sont les nameservers
```

**`nslookup`** = version plus simple, disponible partout (y compris Windows).

**Problèmes DNS courants :**
- Propagation lente → TTL élevé, changement récent
- CNAME à l'apex → utiliser un A record ou un ALIAS (Cloudflare, Route53)
- Split-horizon DNS → résolution différente selon l'origine (interne vs externe) — fréquent avec Private Endpoints Azure
- Cache empoisonné → DNSSEC pour valider l'authenticité

---

## 5. HTTP/HTTPS — protocole applicatif fondamental

**Structure d'une requête HTTP :**

```
GET /api/users?page=1 HTTP/1.1
Host: api.monsite.com
Authorization: Bearer eyJ...
Accept: application/json
Content-Type: application/json

{body si POST/PUT}
```

**Codes de statut à connaître :**

```
2xx — Succès
  200 OK
  201 Created
  204 No Content

3xx — Redirection
  301 Moved Permanently  (cache côté client)
  302 Found              (temporaire, pas de cache)
  304 Not Modified       (cache valide)

4xx — Erreur client
  400 Bad Request
  401 Unauthorized       (non authentifié)
  403 Forbidden          (authentifié mais pas autorisé)
  404 Not Found
  429 Too Many Requests  (rate limiting)

5xx — Erreur serveur
  500 Internal Server Error
  502 Bad Gateway        (proxy/LB ne peut pas joindre le backend)
  503 Service Unavailable (backend surchargé ou down)
  504 Gateway Timeout    (backend trop lent)
```

**HTTP/1.1 vs HTTP/2 vs HTTP/3 :**

- HTTP/1.1 : une requête à la fois par connexion TCP (head-of-line blocking). Keep-alive pour réutiliser la connexion.
- HTTP/2 : multiplexage — plusieurs requêtes en parallèle sur une seule connexion TCP. Headers compressés (HPACK). Push serveur.
- HTTP/3 : remplace TCP par QUIC (basé sur UDP). Élimine le head-of-line blocking TCP. Connexion plus rapide (0-RTT handshake). Utilisé par Cloudflare, Google, etc.

---

## 6. TLS — comment ça fonctionne vraiment

**TLS handshake (TLS 1.3 simplifié) :**

```
Client                          Serveur
  |-- ClientHello ─────────────>|   versions supportées, cipher suites, random
  |<── ServerHello ─────────────|   version choisie, certificat, clé publique
  |<── Certificate ─────────────|   
  |     (vérifie la chaîne de confiance via CA)
  |-- Finished ────────────────>|   clé de session dérivée des deux côtés
  |<── Finished ────────────────|
  |  données chiffrées          |
```

**Ce que vérifie le client :**
1. Le certificat est signé par une CA de confiance (dans le trust store OS/navigateur)
2. Le nom dans le certificat correspond au domaine (CN ou SAN)
3. La date de validité (not before / not after)
4. Le certificat n'est pas révoqué (OCSP)

**Certificates — types à connaître :**
- DV (Domain Validated) — juste la possession du domaine prouvée. Gratuit avec Let's Encrypt.
- OV (Organization Validated) — identité de l'organisation vérifiée.
- EV (Extended Validation) — vérification poussée, ancien cadenas vert.
- Wildcard (`*.monsite.com`) — couvre tous les sous-domaines d'un niveau.
- SAN (Subject Alternative Name) — un certificat pour plusieurs domaines.

**Commandes utiles :**

```bash
# Voir le certificat d'un site
openssl s_client -connect api.monsite.com:443 -servername api.monsite.com

# Voir les détails d'un certificat local
openssl x509 -in cert.pem -text -noout

# Vérifier la chaîne complète
openssl verify -CAfile chain.pem cert.pem

# Date d'expiration
openssl x509 -enddate -noout -in cert.pem
echo | openssl s_client -connect api.monsite.com:443 2>/dev/null | openssl x509 -noout -dates
```

---

## 7. Load balancing — niveaux et algorithmes

**L4 vs L7 :**
- L4 (TCP/UDP) : le LB voit juste l'IP/port. Décision de routage rapide, transparente. Exemples : Azure Load Balancer, AWS NLB, HAProxy en mode TCP.
- L7 (HTTP/HTTPS) : le LB lit le contenu de la requête. Peut router selon le path (`/api` → backend API, `/static` → CDN), les headers, le cookie de session. Peut faire du TLS termination (déchiffrer ici, HTTP vers les backends). Exemples : Nginx, Azure Application Gateway, AWS ALB.

**Algorithmes de distribution :**
- Round Robin : tour à tour (défaut)
- Least Connections : vers le serveur le moins chargé
- IP Hash : même IP client → même serveur (sticky sessions sans cookie)
- Weighted : certains serveurs reçoivent plus de trafic (blue/green, canary)
- Random : aléatoire (surprenant mais souvent aussi bon que round robin à grande échelle)

**Health checks** : le LB sonde régulièrement les backends (TCP connect, HTTP GET + code attendu). Un backend qui rate N fois est retiré du pool. Important : le health check doit tester la vraie santé applicative, pas juste TCP.

---

## 8. Réseau dans les conteneurs et Kubernetes

C'est là que ça devient crucial pour un profil DevOps/SRE.

**Docker networking :**

```bash
# Modes réseau Docker
bridge     # réseau virtuel privé, NAT vers l'extérieur (défaut)
host       # partage la stack réseau du host directement
none       # pas de réseau
overlay    # réseau entre plusieurs hosts Docker (Swarm)

# Un conteneur expose le port 8080, mappé sur le host :
docker run -p 80:8080 myapp
# → traffic :80 sur le host → NAT → :8080 dans le conteneur
```

**Kubernetes networking — 4 problèmes distincts :**

1. **Pod-to-Pod** (même node) : via un bridge virtuel (`cbr0` ou équivalent CNI). Les pods ont des IPs routables directement.
2. **Pod-to-Pod** (nodes différents) : le CNI s'en occupe. Flannel = encapsulation VXLAN (overlay). Calico = BGP entre nodes (underlay, plus performant). Cilium = eBPF.
3. **Pod-to-Service** : via `kube-proxy` qui maintient des règles iptables/IPVS. Le DNS cluster (`CoreDNS`) résout `mon-service.namespace.svc.cluster.local` vers la ClusterIP du service.
4. **External-to-Pod** : via `NodePort`, `LoadBalancer` (crée un LB cloud), ou `Ingress` (L7, un seul LB pour tout).

**Services Kubernetes :**

```yaml
ClusterIP    # IP virtuelle interne au cluster uniquement (défaut)
NodePort     # expose un port sur chaque node (30000-32767)
LoadBalancer # provisionne un LB cloud (crée une IP publique)
ExternalName # alias DNS vers un service externe
```

**Ingress** = ressource K8s + un Ingress Controller (Nginx, Traefik, HAProxy). L'Ingress Controller est le seul LoadBalancer, il route selon les règles :

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myapp-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: api.monsite.com
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: api-v1-svc
            port:
              number: 80
  tls:
  - hosts:
    - api.monsite.com
    secretName: api-tls-cert
```

**Network Policies** = firewall L3/L4 entre pods (nécessite un CNI compatible comme Calico ou Cilium) :

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}       # tous les pods du namespace
  policyTypes:
  - Ingress
  # aucune règle ingress = tout bloquer
```

---

## 9. NAT, routage, et outils de diagnostic

**NAT (Network Address Translation) :**
- SNAT (Source NAT) : modifier l'IP source. Permet à des machines avec IP privée de sortir sur Internet via une IP publique. Le routeur/firewall garde une table de translation.
- DNAT (Destination NAT) : modifier l'IP destination. Utilisé pour les load balancers, port forwarding.
- Masquerade : SNAT dynamique où l'IP publique n'est pas fixe.

**Routage :** chaque hôte a une table de routage. Pour chaque paquet, le kernel cherche la route la plus spécifique (`longest prefix match`). La route par défaut (`0.0.0.0/0`) est le dernier recours.

```bash
ip route show          # table de routage Linux
ip route add 10.0.0.0/8 via 10.1.0.1    # ajouter une route
ip route del 10.0.0.0/8
```

**Outils de diagnostic réseau — les indispensables :**

```bash
# Connectivité de base
ping 8.8.8.8               # ICMP echo — est-ce que l'hôte répond ?
ping6 2001:4860:4860::8888 # IPv6

# Tracé du chemin
traceroute 8.8.8.8         # Linux (UDP par défaut)
traceroute -T 8.8.8.8      # via TCP (passe mieux les firewalls)
tracepath 8.8.8.8          # alternative sans root
mtr 8.8.8.8                # traceroute continu + stats perte/latence

# DNS
dig api.monsite.com
dig @8.8.8.8 api.monsite.com +trace

# Ports et connexions
ss -tulnp                  # sockets en écoute (remplace netstat)
ss -tnp                    # connexions TCP établies
netstat -tulnp             # équivalent plus ancien

# Test de connectivité TCP
nc -zv api.monsite.com 443          # est-ce que le port répond ?
nc -zv api.monsite.com 5432         # PostgreSQL accessible ?
telnet api.monsite.com 80           # vieux mais universel

# Analyse de trafic
tcpdump -i eth0 port 80             # capturer le trafic HTTP
tcpdump -i eth0 host 10.0.0.5      # tout le trafic vers/depuis une IP
tcpdump -i eth0 -w capture.pcap    # sauvegarder pour Wireshark
tcpdump -i any -n 'port 53'        # tout le trafic DNS

# Interfaces et IPs
ip addr show                        # toutes les IPs (remplace ifconfig)
ip link show                        # interfaces
ip neigh show                       # table ARP

# curl pour tester HTTP/HTTPS
curl -v https://api.monsite.com/health          # verbose
curl -I https://api.monsite.com               # headers seulement
curl -k https://api.monsite.com              # ignorer erreur TLS
curl --resolve api.monsite.com:443:1.2.3.4  # forcer une IP
curl -w "\n%{http_code} %{time_total}s\n" https://api.monsite.com
```

---

## 10. Questions typiques d'entretien réseau

**"Que se passe-t-il quand tu tapes `https://google.com` dans un navigateur ?"**
C'est la question reine. La réponse complète couvre : résolution DNS (resolver local → root → TLD → authoritative) → connexion TCP (SYN/SYN-ACK/ACK) → TLS handshake (ClientHello, certificat, clé de session) → requête HTTP GET → réponse 200 + HTML → rendu + ressources supplémentaires (JS, CSS via HTTP/2 multiplexé).

**"Différence entre un routeur et un switch ?"** → Switch = L2, route par adresse MAC à l'intérieur d'un réseau local. Routeur = L3, route par adresse IP entre réseaux différents. Un routeur connaît la table de routage, un switch apprend la table MAC dynamiquement (flooding puis mémorisation).

**"C'est quoi ARP ?"** → Address Resolution Protocol. Permet de trouver l'adresse MAC correspondant à une IP dans le même subnet. Broadcast "qui a l'IP 10.0.0.5 ?" → la machine répond avec sa MAC. Le résultat est mis en cache dans la table ARP (`ip neigh show`). Problème : ARP spoofing (attaque MITM en LAN).

**"Pourquoi un `502 Bad Gateway` ?"** → Le reverse proxy (Nginx, LB) a bien reçu la requête mais n'a pas pu obtenir de réponse valide du backend. Causes : backend down, crash de l'app, timeout, backend qui répond une réponse HTTP invalide. À distinguer du `504 Gateway Timeout` (même cause mais le backend a répondu trop lentement plutôt que pas du tout).

**"C'est quoi un MTU et pourquoi ça pose problème ?"** → Maximum Transmission Unit = taille maximale d'un paquet en octets. Ethernet standard = 1500 bytes. Avec VXLAN (overlay réseau pour conteneurs/cloud), l'encapsulation ajoute ~50 bytes d'overhead → les paquets dépassent 1500 bytes et sont fragmentés ou droppés si PMTUD est cassé. Solution : réduire le MTU des interfaces virtuelles à 1450 bytes dans les environnements overlay. Symptôme classique : les petits paquets passent, les gros (comme les requêtes TLS avec de gros certificats) sont droppés.

**"Qu'est-ce que VXLAN ?"** → Virtual Extensible LAN. Protocole d'encapsulation L2-over-L3 qui permet de créer des réseaux overlay entre hosts. Le trafic Ethernet est encapsulé dans des paquets UDP (port 4789). Utilisé par Flannel (Kubernetes), Docker Swarm, Azure VNet. Permet de traiter des VMs/pods sur des hosts différents comme s'ils étaient sur le même LAN, même à travers des routeurs.

**"Comment déboguer un problème réseau dans un pod Kubernetes ?"**

```bash
# Lancer un pod de debug dans le namespace
kubectl run debug --image=nicolaka/netshoot -it --rm -- bash

# Depuis le pod de debug :
ping nom-du-service            # test DNS + connectivité
dig nom-du-service             # résolution DNS
curl http://nom-du-service:80/health   # test HTTP
nc -zv nom-du-service 5432    # test TCP

# Depuis l'extérieur :
kubectl exec -it mon-pod -- sh
kubectl port-forward svc/mon-service 8080:80  # tester localement
```

---

## 11. Sécurité réseau — concepts à placer en entretien

**Ingress vs Egress** : trafic entrant (vers tes services) vs sortant (de tes services vers l'extérieur). Penser aux deux. Beaucoup de teams sécurisent l'ingress mais laissent l'egress totalement ouvert — ce qui permet à un attaquant de contacter son C2 ou d'exfiltrer des données.

**Zero Trust Network** : ne plus supposer que le réseau interne est de confiance. Authentifier et autoriser chaque connexion (même interne). Implémenté via mTLS entre services (chaque service a son propre certificat et vérifie celui de l'autre), Network Policies Kubernetes, service mesh (Istio, Linkerd).

**mTLS** = mutual TLS. Les deux parties (client ET serveur) présentent un certificat. Utilisé dans les service meshes et les communications inter-services sensibles.

**CORS (Cross-Origin Resource Sharing)** = mécanisme browser qui autorise (ou non) des requêtes JavaScript vers un domaine différent de la page courante. Configuré côté serveur avec des headers (`Access-Control-Allow-Origin`). N'est pas un mécanisme de sécurité serveur — le serveur reçoit quand même la requête, c'est le browser qui bloque la réponse.

---

## 12. Connexion avec ton stack

Pour tes entretiens DevOps, le réseau atterrit toujours dans des contextes concrets :

- **k3s sur OVH** : Flannel VXLAN par défaut, MTU à surveiller. CoreDNS pour la résolution interne. `kubectl get svc` pour comprendre quelle IP est exposée.
- **ArgoCD + GitOps** : ArgoCD s'appuie sur DNS pour résoudre l'API k8s et les repos Git. Les Network Policies peuvent bloquer ArgoCD si mal configurées.
- **Azure VNet** : tout ce qu'on a vu hier — Private Endpoints, NSG, peering. La plupart des problèmes de connectivité Azure viennent d'un NSG qui bloque ou d'un Private DNS Zone manquant pour résoudre l'endpoint privé.
- **FastAPI + PostgreSQL** : PostgreSQL écoute sur le port 5432/TCP. En conteneur, le service Kubernetes crée une ClusterIP stable — l'app se connecte à `postgres-svc:5432`, CoreDNS résout, kube-proxy route vers le pod. Si ça ne marche pas : `nc -zv postgres-svc 5432` depuis un pod de debug.