# L’INTELLIGENCE ARTIFICIELLE DANS UNREAL ENGINE 5

## Objectifs du cours
À la fin de cette formation, vous saurez :
- Comprendre les principes fondamentaux de l’IA dans Unreal Engine 5
- Utiliser les composants principaux : AIController, Pawn, Perception, Behaviour Tree, Blackboard
- Créer un PNJ autonome capable de percevoir, décider et agir
- Combiner perception, décision et navigation
- Optimiser et déboguer vos IA dans un projet Unreal Engine 5

---

## PARTIE 1 — Introduction à l’IA de jeu

### 1.1 Qu’est-ce qu’une IA dans un jeu vidéo ?
L’intelligence artificielle dans Unreal n’est pas une “vraie” IA comme dans le machine learning.  
Elle simule des comportements intelligents pour donner l’illusion de vie.

**Objectifs d’une IA de jeu :**
- Réagir aux actions du joueur
- Se déplacer intelligemment (navigation)
- Prendre des décisions cohérentes
- Créer des comportements réalistes ou stratégiques

### 1.2 Architecture générale d’une IA dans Unreal Engine
```

Perception → AIController → Blackboard → Behaviour Tree → Actions (Tasks)

````

1. **Perception** : détecte le joueur ou d’autres stimuli (vue, son, etc.)
2. **AIController** : le cerveau du PNJ, gère les ordres et les états
3. **Blackboard** : mémoire temporaire de l’IA
4. **Behaviour Tree** : prend les décisions selon les conditions
5. **Tasks et Actions** : exécute les comportements (mouvement, tir, fuite…)

### 1.3 Les composants de base

| Élément | Rôle |
|----------|------|
| **Pawn / Character** | Corps physique contrôlé par l’IA |
| **AIController** | Cerveau logique qui décide des actions |
| **Blackboard** | Mémoire (variables de contexte) |
| **Behaviour Tree** | Système de décision hiérarchique |
| **AIPerception** | Capacité à percevoir l’environnement |
| **NavMesh** | Maillage de navigation pour le déplacement |

---

## PARTIE 2 — Mise en place d’une IA simple

### 2.1 Création du projet
1. Créer un projet **Third Person** dans Unreal Engine 5  
2. Vérifier que les modules “AI” et “Navigation” sont activés  
3. Dans le niveau, placer un **Nav Mesh Bounds Volume** couvrant la zone (touche `P` pour visualiser)

### 2.2 Création du PNJ
1. Créer un **Blueprint Class → Character** nommé `BP_Enemy`
2. Dans ses propriétés :
   - `Auto Possess AI` → `Placed in World or Spawned`
   - `AI Controller Class` → `BP_AIController` (à créer)

### 2.3 Création du contrôleur d’IA
1. Créer un **Blueprint Class → AIController** nommé `BP_AIController`
2. Dans le `Event Graph` :

```blueprint
   Event BeginPlay
      → Get Player Character
      → AI MoveTo (Target Actor = Player Character)
````

3. Le PNJ se déplace vers le joueur dès le début du jeu

---

## PARTIE 3 — Perception et réactions

### 3.1 Introduction au système de perception

L’**AIPerception Component** permet à l’IA de détecter des acteurs via plusieurs sens :

* Vue
* Ouïe
* Dommages
* Stimuli personnalisés

### 3.2 Ajouter la perception à l’IA

1. Ouvrir `BP_AIController`
2. Ajouter un **AIPerception Component**
3. Ajouter un **AI Sight Config** :

   * Sight Radius : 2000
   * Lose Sight Radius : 2500
   * Peripheral Vision Angle : 90
   * Detection by Affiliation : détecter ennemis et neutres
4. Connecter l’événement :

```blueprint
   On Perception Updated
      → For Each Actor
         → Cast To BP_ThirdPersonCharacter
         → Set Blackboard Value as Object (Key = TargetActor)
   ```

### 3.3 Ajouter la mémoire de l’IA (Blackboard)

1. Créer un **Blackboard** nommé `BB_Enemy`
2. Ajouter les clés :

   * `TargetActor` (Object → Actor)
   * `LastKnownLocation` (Vector)
3. Le Blackboard conserve les informations perçues par l’IA

---

## PARTIE 4 — Prise de décision : Behaviour Tree

### 4.1 Création du Behaviour Tree

1. Créer un **Behaviour Tree** nommé `BT_Enemy`
2. Dans ses propriétés : `Blackboard Asset = BB_Enemy`
3. Structure de base :

```
   Root
    └── Selector
         ├── Sequence (Chase Player)
         │     ├── Blackboard Condition: TargetActor != None
         │     └── Task: MoveTo(TargetActor)
         └── Sequence (Patrol)
               ├── Task: GetNextPatrolPoint
               └── Task: MoveTo(PatrolLocation)
```

### 4.2 Lier le Behaviour Tree au Controller

Dans `BP_AIController` :

```blueprint
Event BeginPlay
   → Run Behavior Tree (BT_Enemy)
```

### 4.3 Création de Tasks personnalisées

**Task : GetNextPatrolPoint**

* Hérite de `BTTask_BlueprintBase`
* Calcule la position suivante parmi des TargetPoints
* Met à jour `PatrolLocation` dans le Blackboard

**Task : MoveTo**

* Utilise la clé `TargetActor` ou `PatrolLocation` selon le contexte

### 4.4 Ajouter de la logique de combat

Créer un **Decorator** “Check Distance < 600” :

* Si le joueur est proche → attaquer
* Sinon → continuer la poursuite

### 4.5 Service : Mise à jour automatique de la perception

Créer un **Service Blueprint** (hérite de `BTService_BlueprintBase`) :

* Vérifie toutes les secondes si le joueur est visible
* Met à jour `TargetActor` en conséquence

---

## PARTIE 5 — Navigation et intégration

### 5.1 Navigation automatique avec le NavMesh

* Placer un **Nav Mesh Bounds Volume**
* Appuyer sur `P` pour vérifier la zone navigable
* Les nœuds `MoveTo` utilisent automatiquement le NavMesh

### 5.2 Obstacles dynamiques

* Ajouter un **Nav Modifier Component** sur les objets mobiles
* Activer `Dynamic Obstacle = True`
* Le NavMesh s’adapte automatiquement

### 5.3 Liens de navigation

* Placer un **Nav Link Proxy**
* Permet aux PNJ de sauter, grimper ou traverser des zones

---

## PARTIE 6 — Débogage et optimisation

### 6.1 Outils de debug

| Commande                 | Description                             |
| ------------------------ | --------------------------------------- |
| `P`                      | Afficher le NavMesh                     |
| `ShowDebug AI`           | Affiche les infos du AIController       |
| `ShowDebug Navigation`   | Visualise le pathfinding                |
| `ShowDebug BehaviorTree` | Montre le déroulement du Behaviour Tree |

### 6.2 Performance et bonnes pratiques

| Optimisation                         | Effet                           |
| ------------------------------------ | ------------------------------- |
| Désactiver les IA hors caméra        | Réduit la charge CPU            |
| Réduire la fréquence des Services    | Moins de ticks de décision      |
| Simplifier le Behaviour Tree         | Moins de conditions et branches |
| Utiliser les **Navigation Invokers** | NavMesh généré dynamiquement    |

---

## SYNTHÈSE DU COURS

| Élément                     | Fonction                              |
| --------------------------- | ------------------------------------- |
| AIController                | Contrôle la logique du PNJ            |
| AIPerception                | Détecte les stimuli (vue, son, etc.)  |
| Blackboard                  | Mémoire temporaire de l’IA            |
| Behaviour Tree              | Structure de décision hiérarchique    |
| NavMesh                     | Gestion de la navigation              |
| Tasks, Decorators, Services | Éléments modulaires du Behaviour Tree |

---

## POUR ALLER PLUS LOIN

* **EQS (Environment Query System)** : recherche dynamique de positions (abris, cibles, objets)
* **Crowd AI** : gestion de foules et comportements de groupe
* **C++ AI Programming API** :

  * `MoveToActor()`, `MoveToLocation()`
  * `UAIPerceptionComponent`
  * `UBehaviorTreeComponent`

### Lectures recommandées

* *Artificial Intelligence for Games* – Ian Millington
* *Programming Game AI by Example* – Mat Buckland

---

## COMMANDES UTILES

| Commande                 | Effet                            |
| ------------------------ | -------------------------------- |
| `ShowDebug AI`           | Informations du contrôleur IA    |
| `ShowDebug BehaviorTree` | Arbre de décision en temps réel  |
| `ShowDebug Navigation`   | Chemins calculés                 |
| `ai stop`                | Stoppe l’IA                      |
| `ai move`                | Force un mouvement vers un point |

```

