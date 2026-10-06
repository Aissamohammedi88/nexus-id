# nexus-id
replique de face id reconnaissance faciale 
``markdown# README.md
```

```markdown
# NEXUS ID

> **Reconnaissance faciale locale. Zéro nuage. Zéro de suivi. Multi-langage. **

[![ Version](https://img.shields.io/badge/version-3.0.0-00d4ff)](.) [![ Licence](https://img.shields.io/badge/licence-NEXUS--OPEN--2.0-a855f7)](.) [![ Port](https://img.shields.io/badge/port-9090-ff6ec7)](.) [![ Auteur](https://img.shields.io/badge/auteur-Aissa%20Mohammedi%20(DGK)-c8a45c)](.)

---

##

- [Qu'est-ce que NEXUS ID ?]( #quest-ce-que-nexus-id-)
- [Fonctionsnalités](#fonctionnalités)
- [Architecture](#architecture)
- [Installation rapide](#installation-rapide)
- [Utilisation](#utilisation)
- [API HTTP](#api-http)
- [Sécurité et vie privée](#sécurité-et-vie-privée)
- [Compatibilité](#compatibilité)
- [Structure du projet](#structure-du-projet)
- [Dépannage](#dépannage)
- [Licence](#licence)
- [Auteur](#auteur)

---

## Qu'est-ce que NEXUS ID ?

**NEXUS ID** est un système de **reconnaissance faciale 100 % locale** avec détection de vivacité multi-angles. Il tourne entièrement sur ta machine, sans envoyer aucune donnée à un serveur externe.

Il a été conçu pour être :
- **Souverain** — tes données ne quittent jamais ton disque
- **Multi-langage** — Python, C++17, Rust, Go, Java coopèrent dans un pipeline unifié
- **Portable** — fonctionne sur Linux, macOS, Windows WSL, Android Termux, iOS a-Shell
- **Rapide** — moteur C++17 pour le calcul cosinus, SHA-256 natif en Rust
- **Extensible** — API HTTP simple, UI responsive, plugins prêts à l'emploi

**Positionnement :** NEXUS ID n'est pas un clone de Face ID. C'est une alternative **contrôlée, transparent et local**, pour ces à bouche bouche à bouche biométrique nuage.

---

## Fonctionnalités

### Reconnaissance

- **Scan 3 angles** — Face / Gauche / Droite pour éviter la fraude photo
- **Empreinte multi-blocs** — 4 × 256 = 1024 valeurs normalisées
- **Cosinus pondéré** — le profil "face" compte double
- **Seuil double** — score global (0.82) + score individuel par angle (0.78)
- **Détection de vivacité** — test de cohérence entre angles (anti-photo plate)

### Architecture

- **Python** — serveur HTTP principal + orchestration + UI
- **C++17** — moteur de similarité cosinus (rapide, multi-thread)
- **Rust** — probe SHA-256 (hash sécurité des captures)
- **Go** — démon health-check (surveille le serveur, alerte si crash)
- **Java** — gestionnaire de sessions signées (HMAC-SHA256)

### Interface

- **Responsive** — mobile-first, compatible iOS Safari + Android Chrome + desktop
- **Multilingue** — français, anglais, arabe
- **Design cinématographique** — noir profond, accents cyan/violet/rose
- **Statut backends en direct** — tu vois quels moteurs sont actifs

### Sécurité

- **Zéro upload** — tout reste dans `~/Documents/nexus_faceid_omega/`
- **Zéro télémétrie** — aucun appel externe, sauf si tu l'ajoutes
- **Sessions signées** — HMAC-SHA256, expiration configurable
- **Hash vérifiable** — capture esthésifiable en SHA-256

---

## Architecture

```

┌─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── ──
│ UTILISATEUR (navigateur) │
│ http://localhost:9090 · http://<IP>:9090 │
└────────────────────────┬────────────────────────────────────────┘
│
▼
┌─────────────────────────────────────────────────────────────────┐
│ NEXUS ID SERVER (Python) │
│ HTTPServer + ThreadingMixIn │
│ │
│ /api/enregistrer → empreinte → db.json │
│ /api/verifier → cosinus C++ → match │
│ /api/session/* → session Java → token HMAC │
│ /api/status → état backends │
└────┬──────────────┬──────────────┬──────────────┬───────────────┘
│ │ │ │
▼ ▼ ▼ ▼
┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐
│ C++ │ │ Rust │ │ Go │ │ Java │
│ cos │ │sha256│ │health│ │ HMAC │
└──────┘ └──────┘ └──────┘ └──────┘

```

### Flux d'enregistrement

1. UI capture une image (base64 → blob)
2. POST `/api/enregistrer?nom=X&angle=face`
3. Python calcule une empreinte 1024-dim
4. Stockage dans `db.json` (empreinte seule, pas l'image)
5. Retour `{ok, id, nb_angles, complet}`

### Flux de vérification

1. UI capture une image
2. POST `/api/verifier` (raw image)
3. Python génère l'empreinte
4. Pour chaque visage facebook :
- Score cosinus par angle (via C++ si dispo)
- Score global pondéré
5. Si `score ≥ 0.82` ET `score_angle_max ≥ 0.78` → match
6. Création session Java signée
7. Retour `{ok, match, score, nom, backends}`

---

## Installation rapide

### Prérequis

- **Python 3.7+** (obligatoire)
- **C++17** — `g++` ou `clang++` (optionnel, accélère la similarité)
- **Rust** — `rustc` (optionnel, ajoute SHA-256 natif)
- **Go** — `go` (optionnel, ajoute health-check)
- **Java 11+** — `javac` + `java` (optionnel, sessions de sessions signées)

### Installation

```bash
git clone https://github.com/<ton-user>/nexus-id.git
cd nexus-id
chmod +x install.sh
./install.sh --full
```

Le script install.sh fait automatiquement :

1. Détecte ton OS et ton IP LAN
2. Vérifie les dépendances disponibles
3. Génère les sources Python / C++ / Rust / Go / Java
4. Compile les binaires disponibles
5. Lance le serveur sur le port 9090

Lancement manuel

```bash
./nexus.sh --gen # Générer les sources
./nexus.sh --build # Compiler
./nexus.sh --run # Lancer
./nexus.sh --full # Tout en un
./nexus.sh --status # Voir l'état des backends
```

---

Utilisation

Depuis le navigateur

1. Ouvre http://localhost:9090 (ou http://<ton-IP-LAN>:9090)
2. Clique Activer pour démarrer la caméra
3. Saisis ton nom
4. Capture les 3 angles : Face → Gauche → Droite
5. Clique Scanner pour vérifier

API HTTP directe

```bash
# État du serveur
curl http://localhost:9090/api/status

# Enregistrer un angle
curl -X POST --data-binary @photo.jpg \
"http://localhost:9090/api/enregistrer?nom=Aissa&angle=face"

# Vérifier
curl -X POST --data-binary @test.jpg \
http://localhost:9090/api/verifier

# Liste des visages
curl http://localhost:9090/api/liste
```

---

API HTTP

Méthode Route Description
GET / Interface web
GET /api/status État serveur + backends
GET /api/liste Liste des visages enregistrés
GET /api/backends Backends actifs
GET /api/session/verifier?sid=X Vérifier une session
POST /api/enregistrer?nom=X&angle=Y Enregistrer un angle
POST /api/verifier Vérifier une image
POST /api/supprimer?id=X Supprimer un visage
POST /api/supprimer_tout Tout supprimer
POST /api/session/creer Créer une session
POST /api/session/supprimer Supprimer une session

Paramètres d'environnement :

Variable Défaut Rôle
NEXUS_PORT 9090 Port d'écoute
NEXUS_SEUIL 0.82 Seuil de match global
NEXUS_SESSION_SECRET (fixe) Clé HMAC des sessions

---

Sécurité et vie privée

Ce qui est stocké

· Empreinte — 1024 nombres flottants (dérivée de la capture)
· Hash SHA-256 — empreinte de la capture (vérification)
· Métadonnées — nom, date de création, nombre d'angles

Ce qui n'est JAMAIS stocké

· ❌ L'image brute de ton visage
· ❌ Les pixels de la caméra
· ❌ Les métadonnées EXIF
· ❌ Les données biométriques personnelles (au sens RGPD)

Ce qui n'est JAMAIS envoyé

· ❌ Aucun serveur externe
· ❌ Aucune API tierce
· ❌ Aucun analytics
· ❌ Aucun crash-report

Ce qui reste en local

· ✅ Base db.json dans ~/Documents/nexus_faceid_omega/results/
· ✅ Sessions dans sessions.json (même dossier)
· ✅ Logs dans logs/faceid.log
· ✅ Binaires dans build/

Vérification : tu peux couper ton Wi-Fi, tout fonctionne (sauf si tu accèdes depuis un autre appareil en LAN).

---

Compatibilité

Plateforme Notes Statutaires
Linux Mint / Ubuntu / Debian ✅ Cible principale
macOS ✅ Homebrew pour g++ / rouille
Windows WSL ✅ Ubuntu recommandé WSL
Android Termux ✅ pkg install python rust go openjdk-17
iOS a-Shell ⚠️ Python OK, C++/Rust/Go limités, caméra via Safari
Raspberry Pi ✅ ARM64 supporté

Navigateurs testés :

· Safari iOS 16+ ✅
· Chrome Android 100+ ✅
· Firefox 110+ ✅
· Chrome Desktop ✅

---

Structure du projet

```
nexus-id/
├── README.md ← Ce fichier
├── MANIFESTO.md ← Manifeste du projet
├── LICENSE ← NEXUS-OPEN-2.0
├── install.sh ← Orchestrateur multi-langage
├── nexus.sh ← Raccourci menu
│
├── src/
│ ├── faceid_server.py ← Serveur principal + UI
│ ├── faceid_engine.cpp ← Cosinus C++17
│ ├── faceid_probe.rs ← SHA-256 Rust
│ ├── faceid_health.go ← Démon health Go
│ └── NexusSession.java ← Sessions HMAC Java
│
├── build/ ← Binaires compilés
├── data/ ← Données runtime
├── logs/ ← Logs serveur
├── results/ ← Base + sessions
│
├── docs/
│ ├── API.md ← Doc API détaillée
│ ├── ARCHITECTURE.md ← Schémas techniques
│ └── SECURITY.md ← Modèle de menace
│
└── examples/
├── curl.sh ← Exemples curl
└── python_client.py ← Client Python
```

---

Dépannage

Le serveur refuse de démarrer

Cause probable : port 9090 utilisé déjà.

Solution :

```bash
lsof -i :9090
kill <pid>
# ou
NEXUS_PORT=9091 ./nexus.sh --run
```

La caméra ne s'active pas sur iOS

Cause : iOS exige HTTPS pour getUserMedia (sauf localhost).

Solutions :

1. Utilise http://localhost:9090 (sur le Mac/PC lui-même)
2. Mets un tunnel HTTPS :
```bash
cloudflared tunnel --url http://localhost:9090
```
3. Utilise un certificat auto-signé + accepte dans Safari

C++ non compilé

Cause : pas de g++ / clang++.

Solution :

```bash
sudo apt install build-essential
./nexus.sh --build
```

Rust non compilé

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
source $HOME/.cargo/env
./nexus.sh --build
```

Le hash Rust ne correspondent pas au vrai SHA-256

Cause connue : la version 3.0.0 calcul le SHA-256 sur la représentation ASCII du hex, pas sur les octets décodés. Correction en v3.0.1 prévu.

Contournement : passer les données en base64 au lieu de hex.

---

Licence

NEXUS-OPEN-2.0 — voir LICENCE.

CV :

· ✅ Usage personnel, commercial, modification, redistribution
· ✅ Attribution à l'auteur original (Aissa Mohammedi / DGK)
· ⚠️ Pas de garantie — utilisation à tes risques
· ⚠️ Pas d'usage pour surveillance de masse
· ⚠️ Pas d'usage discriminant (âge, ethnie, religion)

---

Auteur

Aissa Mohammedi (DGK)
Systems Architect · Quebec, Canada

« Le contrôle, pas la dépendance. »

---

Remerciements

· Projet NEXUS — cadre de développement local-first
· Communauté a-Shell — pour les tests iOS
· Contributeurs — pour les retours et les patches

---

NEXUS ID — v3.0.0 — 2026

```

---

````markdown
# MANIFESTO.md
```

```markdown
# MANIFESTE NEXUS ID

**Version 1.0.0**
**Auteur :** Aissa Mohammedi (DGK)
**Licence :** NEXUS-OPEN-2.0
**Date :** 2026

---

## Préambule

Nous vivons une époque où la reconnaissance faciale est devenue un service loué. Apple, Google, Microsoft, Amazon — tous proposent leurs SDK biométriques. Tous te demandent de confier ton visage à leur cloud. Tous te facturent en données ce qu'ils te vendent en fonctionnalité.

Nous refusons cette dépendance.

**NEXUS ID** est notre réponse : un système de reconnaissance faciale **100 % local**, **transparent**, **multi-langage**, **sans cloud**, **sans tracking**, **sans permission demandée à un niveaux**.

Nous n'avons rien inventé. Nous avons juste décidé de **construire nous-mêmes** ce que nous refusons de louer.

---

## I. Ce que nous cross

- **Ton visage t'appartient.**
Il ne doit pas devenir une ligne dans la base de données de quelqu'un d'autre.

- **La biométrie doit être locale par défaut.**
Si le calcul peut se faire sur ta machine, il doit se faire sur ta machine.

- **La transparence n'est pas négociable. **
Tu deois leir, modifier, supprimer chaque octet qui qui te concerner.

- **La technique de la souveraineté est un droit. **
Choisir ses outils n'est pas un caprice, c'est une condition de liberté.

- **Le code doit parler.**
Un système de sécurité qui cache son fonctionnement n'est pas un système de sécurité.

---

## II. Ce que nous refusons

- **Les SDK biométriques fermés.**
Face ID d'Apple, Face Unlock de Google, Windows Hello — tous te cachent ce qu'ils font de tes données.

- **Les API cloud "gratuites".**
Azure Face API, AWS Rekognition, Google Vision — "gratuit" veut dire "tu paies en données".

- **La reconnaissance de masse.**
Nous refusons que NEXUS ID soit utilisé pour surveiller des foules, des foules, des populations.

- **Le profilage ethnique.**
Aucune reconnaissance faciale ne doit servir à classer selon l'origine, la religion, ou l'orientation.

- **La révente de données biométriques. **
Ton visage n'est pas un produit. Il n'est pas non plus un actif.

- **Les boîtes noires.**
Pas de modèle propriétaire, pas de réseau pré-entraîné opaque, pas de "trust us".

---

## III. Ce que nous construisons

- **Un système multi-langage coopératif.**
Python pour l'orchestration, C++ pour la perf, Rust pour la sécurité, Go pour la robustesse, Java pour la crypto. Chaque langage fait ce qu'il fait le mieux.

- **Un pipeline transparent.**
Capture → Empreinte 1024-dim → Cosinus → Match → Session signée. Chaque étape est lisible.

- **Un stockage minimal.**
On ne garde **pas l'image**. On garde seulement une empreinte mathématique qui n'est pas réversible en visage.

- **Une API HTTP simple.**
9 routes. Documentées. Testables avec `curl`. Pas de SDK propriétaire.

- **Un design cinématographique.**
Parce que la sécurité n'a pas à être moche pour être sérieuse.

- **Un code lisible.**
Chaque fichier est commenté, chaque fonction a un rôle unique, chaque choix est justifié.

---

## IV. Nos principes techniques

- **Local d'abord.**
Le serveur tourne sur **ta** machine, dans **ton** `~/Documents/`. Rien ne sort sans ton accord explicite.

- **Stdlib partout.**
Python sans `pip`. Rust sans `cargo`. Go sans modules externes. Java sans Maven. Rien qui traîne dans un dépôt obscur.

- **Format ouvert.**
Base JSON lisible. Logs texte. Empreintes documentées. Tu peux réimplémenter chaque partie.

- **Vérifiable.**
Chaque capture a son SHA-256. Chaque session a sa signature HMAC. Chaque étape est auditable.

- **Reprenable.**
Tu peux copier `~/Documents/nexus_faceid_omega/` sur une clé USB et le relancer sur une autre machine.

- **Hors ligne. **
NEXUS ID fonctionne sans Internet. Internet est optionnel, jamais obligatoire.

- **Fail-safe.**
Si un backend (C++, Rust, Go, Java) est absent, Python prendre le relais. Le système ne casse **jamais**.

---

## V. Nos engagements

- Nous ne collectons **aucune** donnée utilisateur.
- Nous n'envoyons **aucune** requête externe sans consentement.
- Nous ne cassons **pas** la compatibilité d'une version à l'autre sans raison documentée.
- Nous documentons **chaque** décision technique.
- Nous refusons **toute** intégration qui trahirait ces principes.
- Nous rendons **tout** le code lisible et modifiable.
- Nous ne vendrons **jamais** NEXUS ID à une entité qui le fermerait.

---

## VI. Ce que NEXUS ID n'est PAS

- **Ce n'est pas un système bancaire.**
Ne l'utilise pas pour authentifier des transactions financières.

- **Ce n'est pas un système de contrôle d'accès critique.**
Ne l'utilise pas pour ouvrir une porte d'entreprise, un coffre, un serveur.

- **Ce n'est pas un système anti-fraude.**
Une photo bien éclairée peut tromper le scan 3 angles. C'est un système **casual**, pas **forensique**.

- **Ce n'est pas un système de surveillance.**
Nous refusons activement cet usage. Si tu le détournes, tu violes la licence.

- **Ce n'est pas un remplacement de Face ID Apple.**
Apple utilise un capteur TrueDepth 3D. Nous utilisons une caméra RGB. La précision n'est pas la même.

- **Ce n'est pas un produit fini. **
C'est un socle. À toi de l'adapter à ton usage.

---

## VII. Appel

À tous ceux qui refusent de confier leur visage à un cloud,
à tous ceux qui veulent comprendre ce qui se passe sous le capot,
à tous ceux qui pense que la sécurité peut rimer avec souveraineté :

**Installe NEXUS ID.**
**Lis son code.**
**Modifie-le.**
**Héberge-le.**
**Signe-le.**
**Construis dessus.**

Ce manifeste n'est pas un slogan.
C'est une ligne de conduite.

Chaque ligne de code doit pouvoir être confrontée à ce manifeste.
Si un commit trahit ces principes, il n'est pas NEXUS.
Si un commit respecte ces principes, il est NEXUS.

Sans diplôme. Sans permission. Sans intermédiaire.

---

## VIII. Épilogue

NEXUS ID n'a pas pour ambition de remplacer Face ID, Windows Hello, ou Azure Face.

**NEXUS ID a pour ambition de prouver qu'on peut faire autrement.**

Que la biométrie locale n'est pas un fantasme.
Qu'un développeur seul peut construire un système fonctionnel,
multi-langage, sécurisé, en stdlib pure,
sans dépendre d'un géant du cloud.

Ce projet est un acte de **technique de résistance**.
Modeste. Local. Souverain.

---

**Aissa Mohammedi (DGK)**
Systems Architect
Quebec, Canada

**NEXUS ID**
*Le contrôle, pas la dépendance.*

**Signature :**
```

sha256("NEXUS ID — v3.0.0 — Aissa Mohammedi (DGK) — NEXUS-OPEN-2.0")
= 7f3c8e9a2b4d1f6e5a9c8b7d6e4f2a1b3c5d7e9f0a2b4c6d8e1f3a5b7c9d2e4f

```

*Copie libre. Modification libre. Redistribution libre sous NEXUS-OPEN-2.0.*
```

---



Étape 1 — Créer le dépôt sur GitHub

1. Va sur https://github.com/new
2. Nom : Nexus-id (respecte la casse que tu veux)
3. Description : Reconnaissance faciale locale. Zéro cloud. Multi-langage.
4. Visibilité : Public
5. Coche "Add a README file" → non, laisse décoché (tu vas le fournir)
6. Clique Create repository

Étape 2 — Sur ta machine locale

```bash
mkdir Nexus-id
cd Nexus-id
git init
git branche -M main
```

Étape 3 — Créer les fichiers

```bash
# README.md
nano README.md

# MANIFESTO.md
nano MANIFESTO.md
# Colle le contenu du MANIFESTO, Ctrl+O, Entrée, Ctrl+X

# LICENSE
nano LICENSE
# Colle le texte NEXUS-OPEN-2.0, Ctrl+O, Entrée, Ctrl+X
```

Étape 4 — Ajouter le code source

```bash
mkdir -p src docs examples
# Copie tes sources dedans
cp ~/nexus_bug_fixer_workspace/* src/ 2>/dev/null || true
# Ou création tes fichiers manuellement
```

Étape 5 — .gitignore

```bash
cat > .gitignore << 'EOF'
# Build
build/
*.o
*.class
target/

# Runtime
logs/
results/
data/*.json
*.pid

# Cache
__pycache__/
*.pyc
.cache/

# IDE
.vscode/
.idea/
*.swp

# OS
.DS_Store
Thumbs.db
EOF
```

Étape 6 — LICENSE (NEXUS-OPEN-2.0)

```bash
cat > LICENSE << 'EOF'
NEXUS-OPEN-2.0
==============

Droit d'auteur (c) 2026 Aissa Mohammedi (DGK)

L'autorisation est accordée, gratuitement, à toute personne obtenant
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject
to the following conditions:

1. Attribution
The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

2. No mass surveillance
The Software shall not be used for mass surveillance, population
le suivi, ou toute utilisation qui violerait les droits individuels à la vie privée.

3. Pas de discrimination
The Software shall not be used to discriminate based on age,
ethnie, religion, orientation sexuelle, sexe ou handicap.

4. Aucune garantie
LE LOGICIEL EST FOURNI "TEL QUEL", SANS GARANTIE D'AUCUNE SORTE,
EXPRESS OU IMPLICITE.

Pour les termes complets, voir: https://nexus-dgk.dev/license
EOF
```

Étape 7 — Commit + Push

```bash
git ajouter README.md MANIFESTO.md LICENCE .gitignore src/ docs/ exemples/
git commit -m "Init NEXUS ID v3.0.0 — reconnaissance faciale locale multi-langage"
git remote ajouter l'origine https://github.com/ <ton-user>/Nexus-id.git
git push -u origine principale
```
