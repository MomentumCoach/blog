# Pourquoi on a jeté notre "IA" à la poubelle (pour une formule de 1979)

Quand on lance une startup Tech en 2025, la tentation est grande de mettre de l'"IA" et du Machine Learning partout.
C'est vendeur, c'est "hype", et ça flatte l'ego technique.

C'est exactement le piège dans lequel je suis tombé avec **Momentum Coach**.
Je voulais construire le moteur de recommandation le plus avancé du marché. Résultat ? J'ai fini par tout jeter pour utiliser une formule vieille de 45 ans.

Voici l'histoire technique de ce pivot, et pourquoi la simplicité gagne toujours, même face aux algorithmes les plus sophistiqués.

---

## 1. L'Approche "Ingénieur" : Le Filtre de Kalman

Mon idée initiale était de traiter la condition physique d'un athlète comme un **système dynamique observable**.

L'ambition était majeure : au lieu de demander à l'utilisateur de faire un test initial, je voulais **estimer son profil physiologique complet (VO2max, Seuil Anaérobie)** simplement en analysant son historique Strava (les "Big Data").

En théorie du signal, quand on veut estimer l'état caché d'un système (ici, le niveau physio) à partir de mesures imparfaites (les footings du dimanche), l'outil roi est le **Filtre de Kalman**.

### La Théorie
Le modèle se base sur deux équations matricielles :

1.  **L'Équation d'État (Prediction)** :
    $$x_{k} = A x_{k-1} + B u_{k} + w_{k}$$
    *   $x_k$ : L'état de forme au jour $k$ (Fitness & Fatigue).
    *   $u_k$ : La charge d'entraînement (Training Impulse - TRIMP).
    *   $A, B$ : Matrices de transition (comment la forme évolue naturellement et comment l'entraînement l'impacte).
    *   $w_k$ : Bruit du processus (stress de la vie, maladie...).

2.  **L'Équation de Mesure (Update)** :
    $$z_{k} = H x_{k} + v_{k}$$
    *   $z_k$ : La performance observée (ex: allure cardiaque sur un footing).
    *   $v_k$ : Bruit de mesure (GPS imprécis, dérive cardiaque, chaleur...).

### La Promesse : "Live & Continuous Update"
La grande force théorique de ce modèle est sa capacité à se mettre à jour en continu, sans nouveau test maximal.
L'algorithme ingère chaque sortie :
*   Si vous courez votre footing habituel à 10km/h mais avec 5 puls/min de moins que d'habitude...
*   Le filtre détecte une réduction de l'erreur entre la prédiction et la mesure.
*   Il met à jour sa "Croyance" : votre endurance fondamentale s'est améliorée -> **votre Seuil a augmenté**.

C'était le graal : un plan d'entraînement qui s'adapte dynamiquement après chaque séance, même une récupération active. "Automagique".

### La Réalité : "Garbage In, Garbage Out"
En pratique, c'était un enfer à calibrer.
Le Filtre de Kalman est optimal si le bruit est gaussien et le modèle linéaire. Or, la physiologie humaine n'est ni l'un ni l'autre.

-   **Bruit non-gaussien** : Le filtre est conçu pour lisser du bruit statistique "normal" (Gaussien). Mais un GPS qui décroche ou un bug cardio ne sont pas du bruit, ce sont des aberrations massives (*outliers*). Le filtre, fonctionnant sur des moyennes pondérées, se fait "aspirer" par ces valeurs extrêmes au lieu de les ignorer.

-   **Manque d'observabilité** : La plupart des amateurs font 80% de leurs sorties en endurance fondamentale à basse intensité. Avec ces seules données, il est mathématiquement très difficile (voire impossible) de distinguer une amélioration du **VO2max** d'une variation de la fatigue ou de la température. Le rapport Signal/Bruit est trop faible sur les footings lents.

J'avais construit une usine à gaz qui essayait de deviner la VMA d'un coureur à partir de ses footings de récupération. C'était instable et frustrant.

---

## 2. Le Retour aux Sources : VDOT (Daniels & Gilbert, 1979)

Fatigué de debugger mes matrices de covariance, j'ai ouvert un livre classique : *"Daniel's Running Formula"*.
Le Dr Jack Daniels (physiologiste, médaillé olympique) et le mathématicien Jimmy Gilbert ont publié leurs travaux dès 1979 (*"Oxygen Power"*).

Leur approche est aux antipodes du Big Data : **Peu de données, mais de la donnée de haute qualité.**

### La Mathématique du VDOT
Au lieu d'essayer de deviner le niveau, ils le mesurent.
Le système repose sur deux régressions fondamentales :

1.  **Le coût en oxygène ($VO_2$) d'une vitesse donnée ($v$)** :
    $$VO_2 = 0.182258 \cdot v + 0.000104 \cdot v^2 - 4.60$$
    *Cette formule estime combien d'oxygène vous consommez pour courir à une vitesse $v$ (m/min).*

2.  **Le pourcentage de $VO_2max$ tenable sur une durée ($t$)** :
    *%VO_2max$ = 0.8 + 0.1894393 \cdot e^{-0.012778 \cdot t} + 0.2989558 \cdot e^{-0.1932605 \cdot t}$
    *Cette formule décrit la chute de la capacité à tenir un % de sa puissance max à mesure que le temps passe.*

En combinant ces deux courbes, on obtient le **VDOT**.
Contrairement au VO2max de laboratoire (qui mesure la taille du "moteur"), le VDOT mesure la performance effective. Il encapsule :
*   Le **VO2max** (cardio).
*   L'**Économie de Course** (biomécanique/rendement).
*   Le **Seuil Lactique** (endurance).

C'est un indice de performance composite. Deux coureurs avec le même temps sur 10km ont le même VDOT, même si l'un a un gros moteur et une mauvaise foulée, et l'autre l'inverse. Pour l'entraînement, c'est ce qui compte : **leur vitesse est la même.**

### Le Pouvoir (et les Limites) de l'Extrapolation
La beauté du système Daniels, c'est sa capacité prédictive.
Si vous connaissez votre VDOT (via un test court, comme un 1600m), vous pouvez mathématiquement prédire votre allure sur n'importe quelle distance, du 1500m au Marathon.

$$VDOT_{1500m} \approx VDOT_{Marathon}$$

**Attention cependant :** C'est une équivalence théorique.
Un VDOT calculé sur 1600m a tendance à être optimiste pour un Marathon si l'athlète manque d'endurance.
C'est là que **Momentum Coach** intervient. Le VDOT nous donne votre potentiel *physiologique*. Le but de notre plan d'entraînement est de construire le volume et la résistance nécessaires pour que vous puissiez *tenir* ce potentiel sur la distance cible.

---

## 3. L'Application Pratique : Le Test 1600m (ou vos courses)

Nous avons donc remplacé notre filtre de Kalman instable par un protocole simple.

### Pourquoi 1600m ?
Nous demandons un test de 1600m car c'est :
1.  **Facile à caser** : Ça prend moins de 15 minutes échauffement compris.
2.  **Facile à récupérer** : Contrairement à un 5km ou 10km à fond, ça ne casse pas l'athlète pour la semaine.
3.  **Suffisant** : Mathématiquement, la corrélation est déjà excellente.

Bien sûr, si vous venez de courir un 10km ou un semi-marathon en compétition ("à fond"), nous pouvons utiliser ce chronomètre directement. C'est même encore plus précis pour les longues distances. Mais pour quelqu'un qui démarre un plan, le 1600m est le compromis parfait.

### Étude de Cas : Arthur (Marathon de Barcelone)
Arthur, mon associé, a servi de cobaye.
*   **Test 1600m** : Réalisé en 5:50.
*   **Calcul VDOT** : L'algo sort un VDOT de 51.
*   **Prédiction Seuil** : D'après les tables, son allure seuil (Threshold Pace) doit être **3:50/km**.

Trois semaines plus tard, séance clé : **3 x 8 minutes au Seuil**.
Je lui donne une consigne stricte : *"Ne regarde pas ta montre. Cours à la sensation (RPE 7-8/10) et au cardio."*

Je voulais vérifier si la prédiction mathématique collait à sa physiologie réelle du moment.
Les données Strava sont tombées :
*   Bloc 1 : **3:53/km** (Un poil prudent)
*   Bloc 2 : **3:51/km** ( Ça se règle)
*   Bloc 3 : **3:50/km** (Pile dessus)

La précision est effrayante. Arthur s'est calé naturellement sur l'allure cible, sans même la connaître.
Mon filtre de Kalman, avec ses milliers de points de données bruyants, n'avait jamais réussi à prédire une allure avec une telle fiabilité. Une simple formule de 1979, alimentée par une seule donnée de qualité, a visé juste du premier coup.

---

## Conclusion : Less is More

En tant qu'ingénieurs, nous avons tendance à sur-complexifier. Nous pensons que plus de données + plus d'algos = meilleur résultat.
C'est souvent faux.

Dans le sport comme ailleurs, **la qualité de la donnée surpasse la quantité.**
Un test maximal de 6 minutes vaut mieux que 6 mois de données de jogging bruitées.
