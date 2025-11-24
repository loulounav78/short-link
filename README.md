# URL Shortener - Service de Raccourcissement d'URLs

Un service web performant de raccourcissement et de gestion d'URLs construit en Go, offrant une redirection instantanée, des analytics asynchrones et un monitoring d'URL automatisé.

[![Go Version](https://img.shields.io/badge/Go-1.24+-00ADD8?style=flat&logo=go)](https://go.dev/)

## 📋 Table des Matières

- [Fonctionnalités](#-fonctionnalités)
- [Architecture](#-architecture)
- [Prérequis](#-prérequis)
- [Installation](#-installation)
- [Configuration](#-configuration)
- [Utilisation](#-utilisation)
- [API Endpoints](#-api-endpoints)
- [Technologies](#-technologies)
- [Équipe](#-équipe)

## ✨ Fonctionnalités

### Fonctionnalités Principales

- **🔗 Raccourcissement d'URLs**
  - Génération de codes courts uniques (6 caractères alphanumériques)
  - Utilisation de `crypto/rand` pour une sécurité maximale
  - Gestion intelligente des collisions avec système de retry

- **⚡ Redirection Instantanée**
  - Redirection HTTP 302 sans latence
  - Analytics asynchrones via goroutines et channels bufferisés
  - Aucun impact sur les performances de redirection

- **📊 Suivi des Clics**
  - Enregistrement asynchrone des clics (User-Agent, IP, timestamp)
  - Pool de workers pour traitement parallèle
  - Buffer de 1000 événements pour absorber les pics de trafic

- **🔍 Surveillance d'URLs**
  - Vérification périodique de la disponibilité des URLs (configurable)
  - Notifications de changement d'état dans les logs
  - Utilisation de requêtes HTTP HEAD pour optimiser les performances

- **🖥️ Interface Dual**
  - API RESTful complète avec Gin
  - CLI riche avec Cobra pour l'administration

## 🏗️ Architecture

### Structure du Projet

Le projet suit le pattern **Clean Architecture** avec séparation claire des responsabilités :
```
url-shortener/
├── cmd/                        # Points d'entrée de l'application
│   ├── root.go                # Commande racine Cobra
│   ├── server/
│   │   └── server.go          # Lance serveur, workers et monitoring
│   └── cli/
│       ├── create.go          # Création de liens via CLI
│       ├── stats.go           # Affichage des statistiques
│       └── migrate.go         # Migrations de base de données
├── internal/                   # Code privé de l'application
│   ├── api/
│   │   └── handlers.go        # Handlers HTTP (Gin)
│   ├── models/
│   │   ├── link.go            # Modèle Link (GORM)
│   │   └── click.go           # Modèle Click (GORM)
│   ├── services/
│   │   ├── link_service.go    # Logique métier des liens
│   │   └── click_service.go   # Logique métier des clics
│   ├── repository/
│   │   ├── link_repository.go # Accès données liens
│   │   └── click_repository.go# Accès données clics
│   ├── workers/
│   │   └── click_workers.go   # Workers asynchrones
│   ├── monitor/
│   │   └── url_monitor.go     # Surveillance périodique
│   └── config/
│       └── config.go          # Gestion configuration (Viper)
├── configs/
│   └── config.yaml            # Fichier de configuration
├── go.mod                     # Dépendances Go
├── go.sum                     # Checksums des dépendances
├── main.go                    # Point d'entrée principal
└── README.md                  # Documentation
```

## 💻 Prérequis

### Système d'Exploitation

- **✅ Linux** (Testé et recommandé)
- **⚠️ Windows** : Nécessite l'installation de [GCC](https://www.mingw-w64.org/) pour compiler le driver SQLite
  - Installer [MinGW-w64](https://www.mingw-w64.org/downloads/)
  - Ou utiliser [TDM-GCC](https://jmeubank.github.io/tdm-gcc/)
- **✅ macOS** : Support natif

### Logiciels Requis

- **Go 1.24+** - [Télécharger Go](https://go.dev/dl/)
- **GCC/C Compiler** (pour SQLite driver)
  - Linux: `sudo apt-get install build-essential` (Debian/Ubuntu)
  - macOS: Installé avec Xcode Command Line Tools
  - Windows: Voir section ci-dessus
- **Git** - [Télécharger Git](https://git-scm.com/downloads)

### Vérification de l'Installation

```bash
# Vérifier Go
go version

# Vérifier GCC (nécessaire pour SQLite)
gcc --version
```

## 📥 Installation

### 1. Cloner le Projet

```bash
git clone https://github.com/loulounav78/short-link.git
cd short-link
```

### 2. Installer les Dépendances

```bash
go mod tidy
```

### 3. Compiler le Projet

```bash
go build -o url-shortener
```

**Note Windows**: Si vous rencontrez l'erreur `gcc: not found`, installez MinGW-w64 et ajoutez-le à votre PATH.

## ⚙️ Configuration

Le fichier `configs/config.yaml` contient toutes les configurations :

```yaml
server:
  port: 8080                          # Port du serveur HTTP
  base_url: "http://localhost:8080"   # URL de base pour les liens courts

database:
  name: "url_shortener.db"            # Fichier SQLite

analytics:
  buffer_size: 1000                   # Taille du buffer du channel
  worker_count: 5                     # Nombre de workers asynchrones

monitor:
  interval_minutes: 5                 # Intervalle de vérification des URLs
```

### Variables par Défaut

Si le fichier `config.yaml` est absent, l'application utilisera les valeurs par défaut ci-dessus.

## 🚀 Utilisation

### 1. Initialiser la Base de Données

Avant la première utilisation, créez les tables :

```bash
./url-shortener migrate
```

**Sortie attendue :**
```
2025/11/24 10:00:00 Configuration loaded: Server Port=8080, DB Name=url_shortener.db...
Migrations de la base de données exécutées avec succès.
```

### 2. Démarrer le Serveur

Lancez le serveur avec tous les composants (API, workers, monitoring) :

```bash
./url-shortener run-server
```

**Le serveur démarre sur** `http://localhost:8080`

**Logs affichés :**
```
2025/11/24 10:00:00 Repositories initialisés.
2025/11/24 10:00:00 Services métiers initialisés.
2025/11/24 10:00:00 Starting 5 click worker(s)...
2025/11/24 10:00:00 Moniteur d'URLs démarré avec un intervalle de 5m0s.
2025/11/24 10:00:00 Serveur démarré sur :8080
```

### 3. Commandes CLI

#### Créer un Lien Court

```bash
./url-shortener create --url="https://www.example.com/very-long-url"
```

**Sortie :**
```
URL courte créée avec succès:
Code: mB8pRz
URL complète: http://localhost:8080/mB8pRz
```

#### Afficher les Statistiques

```bash
./url-shortener stats --code="mB8pRz"
```

**Sortie :**
```
Statistiques pour le code court: mB8pRz
URL longue: https://www.example.com/very-long-url
Total de clics: 42
```

#### Aide des Commandes

```bash
./url-shortener --help
./url-shortener create --help
./url-shortener stats --help
```

## 📡 API Endpoints

### Base URL
```
http://localhost:8080
```

### Endpoints Disponibles

#### 1. Health Check

```http
GET /health
```

**Réponse :**
```json
{
  "status": "ok"
}
```

---

#### 2. Créer un Lien Court

```http
POST /api/v1/links
Content-Type: application/json

{
  "long_url": "https://www.example.com/very-long-url"
}
```

**Réponse (201 Created) :**
```json
{
  "short_code": "mB8pRz",
  "long_url": "https://www.example.com/very-long-url",
  "full_short_url": "http://localhost:8080/mB8pRz"
}
```

**Exemple avec curl :**
```bash
curl -X POST http://localhost:8080/api/v1/links \
  -H "Content-Type: application/json" \
  -d '{"long_url":"https://www.example.com/test"}'
```

---

#### 3. Redirection

```http
GET /{shortCode}
```

**Comportement :**
- Redirige vers l'URL longue (HTTP 302)
- Enregistre le clic de manière asynchrone
- Pas de latence ajoutée

**Exemple :**
```bash
curl -L http://localhost:8080/mB8pRz
# Redirige automatiquement vers l'URL longue
```

---

#### 4. Statistiques d'un Lien

```http
GET /api/v1/links/{shortCode}/stats
```

**Réponse (200 OK) :**
```json
{
  "short_code": "mB8pRz",
  "long_url": "https://www.example.com/very-long-url",
  "total_clicks": 42
}
```

**Exemple avec curl :**
```bash
curl http://localhost:8080/api/v1/links/mB8pRz/stats
```

---

### Codes d'Erreur

| Code | Description |
|------|-------------|
| 200 | Succès |
| 201 | Ressource créée |
| 302 | Redirection |
| 400 | Requête invalide |
| 404 | Lien non trouvé |
| 500 | Erreur serveur |

## 🛠️ Technologies

### Frameworks & Bibliothèques

- **[Gin](https://gin-gonic.com/)** - Framework web HTTP performant
- **[Cobra](https://cobra.dev/)** - Création d'interfaces CLI
- **[Viper](https://github.com/spf13/viper)** - Gestion de configuration
- **[GORM](https://gorm.io/)** - ORM pour Go
- **[SQLite](https://www.sqlite.org/)** - Base de données embarquée

### Design Patterns

- **Repository Pattern** - Abstraction de l'accès aux données
- **Service Layer** - Logique métier centralisée
- **Dependency Injection** - Via interfaces
- **Worker Pool** - Traitement asynchrone avec goroutines

### Concepts Go Utilisés

- Goroutines & Channels
- Interfaces
- Error Handling
- Struct Tags (JSON, GORM)
- Context & Graceful Shutdown

## 👥 Équipe

Ce projet a été développé dans le cadre du TP Go Final.

### Développeurs

- **Samuel CHARTON** - [@Darukity](https://github.com/darukity)
- **Loris NAVARRO** - [@Loulounav78](https://github.com/Loulounav78)
- **Gaëtan MAIRE** - [@TheD0Om](https://github.com/TheD0Om)

## 🐛 Dépannage

### Erreur: `gcc: not found` (Windows)

**Solution :**
1. Installez [MinGW-w64](https://www.mingw-w64.org/downloads/)
2. Ajoutez `C:\mingw64\bin` à votre PATH
3. Redémarrez votre terminal
4. Vérifiez: `gcc --version`

### Le serveur ne démarre pas

**Vérifications :**
- Port 8080 déjà utilisé ? Changez le port dans `config.yaml`
- Base de données migrée ? Exécutez `./url-shortener migrate`
- Permissions fichier ? Vérifiez les droits d'écriture

### Erreur de compilation SQLite

**Solution Linux :**
```bash
sudo apt-get install build-essential
```

**Solution macOS :**
```bash
xcode-select --install
```

## 🔗 Liens Utiles

- [Documentation Go](https://go.dev/doc/)
- [Gin Framework](https://gin-gonic.com/docs/)
- [GORM Guide](https://gorm.io/docs/)
- [Cobra CLI](https://cobra.dev/)

---

**Made with ❤️ in attempt to get a good grade**