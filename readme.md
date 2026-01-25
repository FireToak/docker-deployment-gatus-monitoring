# 📊 Configuration Gatus Monitoring - LoutikCLOUD Status Page

![Gatus Version](https://img.shields.io/badge/Gatus-Latest-blue?style=for-the-badge&logo=gatus)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Status](https://img.shields.io/badge/Status-Live-success?style=for-the-badge)

Ce dépôt contient la configuration complète (Infrastructure as Code) pour le déploiement de la page de statut publique **LoutikCLOUD**. Le système surveille la disponibilité et la performance des services critiques de l'infrastructure via **Gatus**.

🔗 **URL Publique :** [https://status.loutik.fr](https://status.loutik.fr)

---

## 🏗️ Architecture Technique

Le projet repose sur une architecture conteneurisée légère et autonome, placée derrière un reverse proxy sécurisé.

* **Core :** Gatus (Image officielle `ghcr.io/twin/gatus:stable`)
* **Base de données :** SQLite (Fichier `data.db` persisté localement)
* **Exposition :**
    * Port interne conteneur : `8080`
    * Port hôte mappé : `1025`
* **Proxy :** Nginx avec terminaison SSL (Let's Encrypt Wildcard `*.loutik.fr`)

### Flux de données
`Internet (HTTPS)` ➡️ `Nginx (Reverse Proxy)` ➡️ `localhost:1025` ➡️ `Gatus Container` ➡️ `SQLite`

---

## 📂 Structure du Dépôt

```bash
configuration-gatus-monitoring/
├── docker-compose.yaml      # Orchestration du conteneur Gatus (définition service & volumes)
├── status.loutik.fr.conf    # Configuration Nginx (Reverse Proxy & Headers de sécurité)
├── readme.md                # Documentation du projet
└── config/
    ├── config.yaml          # Cœur de la configuration (UI, Endpoints, Alerting)
    └── data.db              # Base de données SQLite (générée automatiquement)

```

---

## 🚀 Services Surveillés

La configuration actuelle (`config/config.yaml`) surveille les groupes de services suivants :

| Groupe | Services | Fréquence | Type de Check |
| --- | --- | --- | --- |
| **🌐 Sites Internet** | Site Vitrine, Portfolio, Documentation | 1 min / 5 min | HTTP Status 200 |
| **💻 Compute** | Hyperviseurs Proxmox (Gideon & Darlene) | 10 sec | HTTP + DNS Custom |
| **🛡️ Sécurité** | SSO Authentik | 1 min | HTTP Status 200 |
| **📝 Collaboratif** | Outline Notes | 2 min | HTTP Status 200 |
| **⚡ Services** | Speedtest | 10 min | HTTP Status 200 |

---

## ⚙️ Installation & Déploiement

### 1. Prérequis

* Docker & Docker Compose installés sur le serveur.
* Port **1025** libre sur l'hôte.
* Nginx installé avec certificats SSL valides.

### 2. Installation Step-by-Step

**Clonage du dépôt**

```bash
git clone https://github.com/FireToak/docker-deployment-gatus-monitoring.git
cd docker-deployment-gatus-monitoring

```

* *`git clone` : Télécharge les fichiers sources du projet sur votre machine locale.*
* *`cd` : Change le répertoire courant pour entrer dans le dossier du projet.*

**Déploiement du conteneur**

```bash
docker compose up -d

```

* *`docker compose up -d` : Construit et démarre les conteneurs définis dans le fichier YAML en arrière-plan (mode "detached").*

**Vérification**

```bash
docker compose ps
docker compose logs -f gatus

```

* *`ps` : Liste les conteneurs actifs pour vérifier que Gatus est "Up".*
* *`logs -f` : Affiche les journaux en temps réel pour s'assurer qu'il n'y a pas d'erreur de chargement de la config.*

### 3. Configuration Nginx (Reverse Proxy)

Copiez le fichier de configuration fourni et activez-le :

```bash
sudo cp status.loutik.fr.conf /etc/nginx/sites-available/
sudo ln -s /etc/nginx/sites-available/status.loutik.fr.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

```

* *`ln -s` : Crée un lien symbolique pour activer le site.*
* *`nginx -t` : Teste la syntaxe de la configuration Nginx avant application (crucial pour éviter de crasher le serveur web).*
* *`systemctl reload` : Recharge la configuration sans couper les connexions actives.*

---

## 🛠️ Personnalisation (config.yaml)

Le fichier `config/config.yaml` contrôle toute la logique. Voici comment le modifier.

### Ajouter un nouvel Endpoint

Insérez ce bloc sous `endpoints:`

```yaml
  - name: "Nom du Service"
    group: "Nom du Groupe"
    url: "[https://service.loutik.fr](https://service.loutik.fr)"
    interval: 1m           # Fréquence (ex: 30s, 1m, 1h)
    conditions:
      - "[STATUS] == 200"  # Condition de succès
    alerts:                # (Optionnel) Alerting
      - type: discord

```

* *`interval` : Définit la fréquence à laquelle Gatus interroge le service.*
* *`conditions` : Critères pour considérer le service comme "UP". Ici, on attend un code HTTP 200 OK.*

### Gérer les Annonces

Décommentez la section `announcement` pour afficher un message en haut de la page de statut (maintenance, incident, info).

```yaml
# ui:
#   announcement:
#     type: warning        # success, info, warning, danger
#     message: "Maintenance prévue ce soir à 22h00."

```

### Modifier l'UI

* **Logo :** Changez l'URL dans `ui.header.logo`.
* **Titre :** Modifiez `ui.header.title`.

---

## 💾 Maintenance & Sauvegardes

### Mise à jour de Gatus

Pour mettre à jour vers la dernière version stable :

```bash
docker compose pull
docker compose up -d

```

* *`pull` : Télécharge la dernière version de l'image Docker spécifiée.*
* *`up -d` : Redémarre le conteneur avec la nouvelle image.*

### Sauvegarde

La base de données est persistante. Pour sauvegarder l'historique de surveillance :

1. Copier le fichier `/config/data.db`.
2. Ou sauvegarder le dossier `config/` complet.

---

## 👤 Auteur

- **Louis MEDO** - *Passionné par l'administration système ❤️* | [Linkedin](https://www.linkedin.com/in/louismedo/) | [Portfolio](https://louis.loutik.fr/) | [GitHub](https://github.com/FireToak) 