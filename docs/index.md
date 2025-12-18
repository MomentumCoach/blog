# Série : Momentum Infra - De 0 à Production

## Introduction : Le Grand Saut
Je m'appelle Geoffrey et après quelques années d'expérience en tant qu'ingénieur devops, j'ai décidé de me lancer à mon compte avec Arthur, un ami de lycée, pour monter **Momentum Coach**.

Nous sommes une toute petite équipe (deux !), sans levée de fonds, et avec des moyens limités.
Dans cette série, je partage comment je mets à profit mon expérience pour construire une infrastructure solide mais économe. Pas de titres pompeux ici, juste de la tech efficace pour faire tourner notre projet sans brûler du cash inutilement.

L'objectif ? Une infrastructure **résiliente**, **scalable**, et **Low Cost** (50€/mois pour commencer), capable de supporter une application boostée à l'IA.

---

## Le Plan de Bataille

### [Partie 1 : Les Fondations (Kubernetes pour 50€)](./01-cluster-k3s-50-euros.md)
*Article actuel.*
- **Le Hardware** : Pourquoi des VPS OVH "gamme desktop" écrasent le Cloud Public.
- **Le Cluster** : K3s en Haute Disponibilité (HA) sans se ruiner.
- **Networking Basics** : WireGuard pour le réseau privé (Introduction).

### Partie 2 : Le Networking (Cilium & eBPF)
*À venir.*
- **Le Problème** : Sécuriser un cluster sur réseau public sans VPC.
- **La Solution** : **Cilium** (eBPF).
    - Remplacer Kube-Proxy pour la performance.
    - Chiffrement transparent (WireGuard/IPSec).
    - Network Policies : Isoler les namespaces (Zéro Trust).
    - Ingress & Gateway API.

### Partie 3 : La Data (Self-Hosted is King)
*À venir.*
- **Le Constat** : Les bases de données managées (RDS, Cloud SQL) sont le poste de dépense n°1.
- **La Solution** : Utiliser des **Opérateurs Kubernetes** pour gérer le stateful comme des pros.
    - **PostgreSQL** avec CloudNativePG (Backups S3, PITR, HA auto).
    - **Redis** Cluster.
    - **ClickHouse** pour l'analytique.

### Partie 4 : Les Yeux sur le Cluster (Observabilité)
*À venir.*
- **Le Problème** : Datadog est génial, mais coûte plus cher que mes serveurs.
- **La Stack** :
    - **La base** : L'écosystème Prometheus & AlertManager.
    - **VictoriaMetrics** : Une alternative à Prometheus plus légère, plus performante et stockable à long terme (Remote Write).
    - **VictoriaLogs** : Pour arrêter de se battre avec ElasticSearch ou Loki, une solution simple et ultra-rapide pour les logs.
    - **Grafana** : La vitre unique pour visualiser tout ça.

### Partie 5 : GitOps & Déploiement Continu (L'Usine)
*À venir.*
- **Le Flux** : Fini les déploiements manuels. Le code part sur Git, il arrive en prod.
- **Les Outils** :
    - **ArgoCD** : La source de vérité de l'infra.
    - **Skaffold** : Un seul outil pour le Dev (Hot Reload) et la Prod.
    - **GitHub Runners** : Pourquoi je n'utilise pas les runners partagés de GitHub (Spoiler: Performance & Coût).

### Partie 6 : La Stack Applicative (Le Monolithe Distribué)
*À venir.*
- **La Stack** :
    - **Back** : Django (Python) pour la robustesse.
    - **Front** : React & React Native.
    - **Containerization** : Dockerfile optimisés multi-stage.

### Partie 7 : AI Engineering & Agents (Le Cœur du Réacteur)
*À venir.*
- **L'Intelligence** : Momentum Coach n'est pas juste un CRUD, c'est un coach IA.
- **L'Implémentation** :
    - **MCP (Model Context Protocol)** : Création d'un serveur MCP pour donner du contexte à l'IA sur l'app.
    - **Agents** : Comment le Chatbot peut lire la base de données et modifier le plan d'entraînement d'un utilisateur.
    - **Generation** : Pipelines de génération de contenu via llm avec LangGraph.

### Partie 8 : AI-Assisted Workflow (Le Développeur Augmenté)
*À venir.*
- **Le Métier** : Comment je code 3x plus vite seul.
- **Le Workflow** :
    - Utilisation des **LLMs pour coder** (Cursor, Copilot, Antigravity, Claude Code).
    - Patterns de "Code Review" par IA.
    - Comment l'architecture est pensée pour être "AI-readable".

### Partie 9 : Platform Services (Les Outils)
*À venir.*
- **Services Transverses** :
    - **Langfuse Self-Hosted** : Tracing et Debugging des appels LLM (indispensable/coûteux en SaaS).
    - **Vault** : Gestion des secrets centralisée hors du repo Git.
    - **Cert-manager** : Gestion des certificats SSL.

### Partie 10 : La Science du Sport (Bonus Track)
*À venir.*
- **Physiologie** : Analyse de la littérature scientifique sur l'entraînement.
- **Modélisation** : Comment coder la charge d'entraînement (Banister, TRIMP, etc.).
- **Algorithmes** : Transformer des papiers de recherche en features concrètes pour les athlètes.
