[← Retour au sommaire de la série](./00-series-summary.md)

# Comment j'ai construit un Cluster Kubernetes de Production pour 50€/mois (vs 900€+ sur AWS)

**50,36 € par mois.** C'est le prix exact que je paie pour faire tourner un cluster Kubernetes de 3 nœuds en haute disponibilité avec des performances qui coûteraient un salaire sur le Cloud public.

Si vous avez déjà regardé les prix des solutions managées comme EKS (Amazon), GKE (Google) ou AKS (Azure), vous savez que la facture peut grimper très vite. On nous vend souvent la facilité du "Managed Kubernetes", mais on oublie de mentionner le coût d'entrée prohibitif pour avoir de la vraie puissance.

La bonne nouvelle ? Vous n'avez pas besoin d'une carte de crédit d'entreprise pour avoir une infrastructure "Production Grade". Voici comment j'ai fait.

## Partie 1 : Le Matériel (La bonne affaire)

Pour atteindre ce prix, il faut sortir des sentiers battus des grands fournisseurs de Cloud (Hyperscalers). Mon choix s'est porté sur 3 VPS chez OVH avec des spécifications qui feraient rougir une instance EC2 standard.

**La configuration par nœud (x3) :**
*   **Processeur** : 8 vCore
*   **Mémoire** : 24 Go RAM
*   **Stockage** : 200 Go SSD NVMe
*   **Réseau** : 1.5 Gbit/s illimité

J'ai donc un cluster totalisant **24 vCores, 72 Go de RAM et 600 Go de NVMe** pour **50,36 €/mois**.

### Le Logiciel : K3s
Sur ces serveurs, je n'installe pas la distribution Kubernetes standard (qui est lourde), mais **K3s**.

#### Pourquoi K3s et pas "Le Vrai Kubernetes" ?
Une idée reçue tenace est que K3s est une version "au rabais" pour l'IoT. C'est faux. K3s est une distribution **certifiée CNCF**, taillée pour la production.
1.  **Embedded HA (Haute Disponibilité)** : C'est la killer-feature. K3s intègre une gestion automatique de **etcd** (la base de données distribuée du cluster). Avec 3 nœuds, vous avez un quorum HA immédiat et robuste, sans avoir à gérer des certificats complexes ou des topologies externes.
2.  **Ressources** : Une distribution K8s "Upstream" (type `kubeadm`) avec tous les plugins cloud peut consommer 2GB+ de RAM juste pour tourner à vide. K3s, en retirant les drivers inutiles (AWS, Azure, GCP Storage...), tourne avec **<500MB de RAM**. Sur des VPS de 24Go, c'est autant de gagné pour vos applis.
3.  **Simplicité & Flexibilité** : K3s est un binaire unique "Zero Dependencies". Par défaut, il vient avec Flannel pour le réseau, mais ici **nous le désactivons** (`--flannel-backend=none`). Pourquoi ? Pour utiliser **Cilium** et sa puissance eBPF (sécurité, observabilité, performance). K3s nous laisse le choix, et c'est crucial pour la suite. D'ailleurs, le networking Kubernetes et Cilium méritent bien plus qu'un paragraphe : ce sera le sujet d'un article complet à venir.

Voici à quoi ressemble la configuration pour cibler mes serveurs avec Ansible. C'est tout ce dont j'ai besoin pour démarrer l'installation :

```ini
# ansible/inventory/hosts.ini
[k3s_servers]
vps1 ansible_user=ubuntu public_ip=x.x.x.x wg_ip=10.6.0.1
vps2 ansible_user=ubuntu public_ip=x.x.x.x wg_ip=10.6.0.2
vps3 ansible_user=ubuntu public_ip=x.x.x.x wg_ip=10.6.0.3
```

## Partie 2 : La touche "Expert" (Ce n'est pas un jouet)

Ne vous y trompez pas : ce n'est pas un cluster "bricolé" juste pour tester. C'est une infrastructure conçue pour la production.

*   **Haute Disponibilité (HA)** : Avec 3 nœuds, si un serveur tombe en panne, le cluster continue de fonctionner.
*   **GitOps avec ArgoCD** : Je ne touche jamais au cluster manuellement avec `kubectl apply`. Tout est défini en code.
*   **Sécurité Avancée** : J'utilise **Cilium** (basé sur eBPF) pour le réseau et la sécurité, et **Vault** pour la gestion des secrets.

La magie du GitOps réside dans ce simple fichier. En appliquant ce seul manifeste, ArgoCD se réveille et déploie automatiquement toute l'infrastructure :

```yaml
# argocd-apps/root-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: root
  namespace: argocd
spec:
  source:
    repoURL: git@github.com:MomentumCoach/momentum-infra.git
    targetRevision: main
    path: argocd-apps
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
```

## Partie 3 : Le Vrai Coût de la Liberté (Le Comparatif)

C'est ici que ça fait mal pour le Cloud Public. J'ai réalisé une analyse détaillée des coûts pour **les mêmes ressources** (24 vCPU, 72Go+ RAM, 600Go Stockage) chez les principaux fournisseurs.

| Composant | Self-Hosted (OVH VPS) | AWS (EKS) | Google Cloud (GKE) | OVH Public Cloud |
| :--- | :--- | :--- | :--- | :--- |
| **Compute** | **3x VPS-3**<br>(8 vCPU, 24 Go RAM) | **3x t3.2xlarge**<br>(8 vCPU, 32 Go RAM) | **3x e2-standard-8**<br>(8 vCPU, 32 Go RAM) | **3x b2-30**<br>(8 vCPU, 30 Go RAM) |
| **Coût Compute** | ~50,36 € / mois | ~$730 / mois | ~$680 / mois | ~$683 / mois |
| **Control Plane** | **0 €** (K3s) | ~$73 / mois | ~$73 / mois | **0 €** |
| **Stockage** | **Inclus** (600 Go NVMe) | ~$48 / mois (EBS gp3) | ~$60 / mois (SSD) | Inclus |
| **Bande Passante** | **Illimitée** (1.5 Gbps) | ~$0.09 / Go (NAT Gtw) | ~$0.08 / Go | Inclus |
| **TOTAL MENSUEL** | **~50 €** | **~$850+** | **~$810+** | **~$680+** |

### Analyse des Coûts

1.  **AWS (Amazon Web Services)**
    *   L'instance `t3.2xlarge` est l'équivalent le plus proche.
    *   Le Control Plane EKS coûte 0,10$/heure, soit environ 73$/mois, peu importe la taille du cluster.
    *   Le stockage EBS (Elastic Block Store) est facturé à part. Pour égaler les 600 Go de NVMe inclus dans mes VPS, il faut ajouter environ 48$.
    *   **Le piège** : La "NAT Gateway" et le trafic sortant. Sur AWS, vous payez pour chaque Go qui sort de votre VPC. Pour un cluster actif, cela ajoute facilement 50$ à 100$ de plus.

2.  **Google Cloud (GCP)**
    *   L'instance `e2-standard-8` offre des performances similaires.
    *   GKE facture des frais de gestion de cluster (sauf pour le premier cluster zonal, mais en production on veut souvent du régional).
    *   Le stockage Persistent Disk SSD est cher (~0,10$/Go).

3.  **OVH Public Cloud**
    *   Même chez OVH, l'offre "Cloud Public" (instances `b2-30`) est 13 fois plus chère que l'offre VPS pour une puissance brute similaire.
    *   La différence ? Le Public Cloud offre une orchestration automatisée, des IP flottantes à la volée et une scalabilité horizontale infinie. Mais avez-vous besoin de scaler à 100 nœuds dans l'heure ? Probablement pas.

### Le Verdict
Je réalise une économie d'environ **800 € par mois**, soit près de **10 000 € par an**.
Pour le prix d'un seul café par jour, j'ai une puissance de calcul qui coûterait un SMIC mensuel chez AWS.

## Partie 4 : Les Mains dans le Cambouis (Les Inconvénients)

Tout n'est pas rose au pays de l'auto-hébergement. Pour atteindre ce prix plancher, il faut accepter certains compromis que les hyperscalers vous masquent habituellement.

### 1. Le DNS Round-Robin vs Load Balancer Managé
Pour exposer mes services, j'utilise une entrée DNS de type `A` qui pointe vers les IPs de mes 3 VPS.
*   **Le problème** : Si un nœud tombe, le DNS (Round Robin) continue d'envoyer 33% du trafic vers ce nœud mort pendant la durée du cache TTL (souvent 5 à 60 minutes). Vos utilisateurs tomberont sur une page d'erreur 1 fois sur 3.
*   **La solution (Mitigation)** :
    1.  **Load Balancer** : Placer un petit VPS frontal avec HAProxy/Nginx.
    2.  **IP Flottante (Failover IP)** : Certains hébergeurs (comme OVH) permettent d'acheter une IP qui n'est pas attachée physiquement à un serveur mais que l'on peut basculer d'un serveur à l'autre via API. Plutôt que de bricoler un script, l'idéal est d'utiliser un **Operator Kubernetes** (ou un Cloud Controller Manager) qui pilote cela. Si le nœud maître tombe, l'opérateur détecte la panne et appelle l'API de l'hébergeur pour réassigner l'IP flottante à un nœud sain en quelques secondes. C'est du Failover automatique et natif.

### 2. La Complexité du Scaling
Ajouter un nœud ici n'est pas juste un curseur à glisser dans une interface web GUI.
*   **La réalité** : Il faut commander le VPS, attendre la livraison, lancer le playbook Ansible pour le sécuriser, le joindre au cluster WireGuard, puis l'ajouter au cluster K3s.
*   **L'atténuation** : C'est là que l'Infrastructure as Code (**IaC**) est vitale. Avec **Terraform** pour provisionner les serveurs et **Ansible** pour la configuration, j'ai automatisé 90% du processus "Day 0". Mon scaling consiste à changer une variable `count = 4` et lancer une pipeline. Sans ça, ce serait ingérable.

### 3. Le Stockage Local
Mes Pods utilisent le stockage NVMe local des nœuds (HostPath via Local Path Provisioner).
*   **L'inconvénient** : Les données sont "collées" au nœud. Si je perds le nœud `vps1`, les pods qui ont besoin de son volume spécifique ne peuvent pas redémarrer ailleurs.
*   **La stratégie** : Pour les applications stateless (Web, API), ce n'est pas grave. Pour la base de données, c'est critique et cela demande une stratégie spécifique (voir ci-dessous).

### 4. Le Mélange des Genres (Master vs Worker)
Dans une architecture Kubernetes classique (et coûteuse), on sépare les nœuds du Control Plane (Master) des nœuds qui exécutent les applications (Workers).
*   **L'approche économique** : Ici, chaque nœud est à la fois Master et Worker (Hyper-convergé).
*   **Le Gain** : On économise la location de 3 serveurs dédiés uniquement à la gestion du cluster. Cela représente souvent 50% de la facture en moins. De plus, on utilise 100% de la RAM disponible pour nos workloads.
*   **Le Risque** : Si une de vos applications consomme toute la RAM ou le CPU (Memory Leak), elle peut étouffer les processus vitaux de Kubernetes (api-server, etcd) et déstabiliser le nœud entier.
*   **La Mitigation** : Pour dormir tranquille, l'utilisation des **Resource Limits** (Requests/Limits) et des **PriorityClasses** est obligatoire. Il faut garantir au niveau du noyau Linux que les composants système (`kube-system`) ont toujours la priorité absolue sur vos conteneurs applicatifs.

## Partie 5 : Et la Data dans tout ça ? (Teasing)

C'est souvent le point bloquant. "Ne gérez jamais votre propre base de données !" crient les experts. C'est vrai, sauf si vous avez les bons outils modernes comme **CloudNativePG**.

Comment stocker de la donnée critique sur des VPS à 50€ avec la même fiabilité que sur AWS RDS ? Comment gérer les backups S3, le Point-in-Time Recovery et le Failover automatique en moins de 10 secondes ?

Ce sujet est si vaste et passionnant qu'il mérite son propre article. Ce sera justement le sujet de la **Partie 2** de cette série : *"Le Cas de la Base de Données : Adieu Managed SQL, Bonjour Operators"*. Restez connectés, ça va secouer.

## Conclusion

L'auto-hébergement (Self-hosting) n'est pas seulement une question d'économie. C'est avant tout une question de **contrôle** et d'**apprentissage**. En construisant cette infrastructure, j'ai appris dix fois plus sur le fonctionnement interne de Kubernetes qu'en cliquant sur un bouton "Créer un cluster" dans une console AWS.

Dans les prochains articles, je vous montrerai comment j'ai sécurisé ce cluster avec Cilium et comment je gère mes secrets de manière professionnelle avec Vault. Restez connectés !
