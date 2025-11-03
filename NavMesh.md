#Le NavMesh dans Unreal Engine 5

## Objectifs du cours
À la fin de cette formation, vous saurez :
- Comprendre le fonctionnement d’un NavMesh (maillage de navigation)
- Générer, visualiser et ajuster le NavMesh dans vos niveaux
- Contrôler les déplacements d’un PNJ via NavMesh et AIController
- Créer des zones spéciales, liens de navigation et obstacles dynamiques
- Optimiser et déboguer la navigation dans un projet Unreal Engine 5

---

## PARTIE 1 — Comprendre le NavMesh

### Qu’est-ce qu’un NavMesh ?
Le **NavMesh (Navigation Mesh)** est une représentation simplifiée de la surface du monde 3D sur laquelle une IA peut marcher.

Il convertit l’environnement en un maillage vert (zones navigables).  
Les IA s’en servent pour calculer un chemin optimal jusqu’à une destination, via un algorithme de pathfinding (A* en interne).

### Pourquoi l’utiliser ?
- Permettre à un PNJ de marcher sans heurter d’obstacles.
- Créer des déplacements réalistes : poursuite, fuite, patrouille.
- Gérer des terrains complexes (escaliers, pentes, plateformes).
- Compatible avec le Behaviour Tree et l’AIController.

### Principe général du système d’IA avec NavMesh
```

AIController
↓
NavMesh Pathfinding
↓
Character Movement Component
↓
Mouvement physique du Pawn

````

Le NavMesh ne déplace pas directement le personnage :  
il fournit un chemin à l’AIController, qui l’envoie ensuite au Character Movement Component.

---

## PARTIE 2 — Mettre en place le NavMesh

### Objectif
Configurer un environnement navigable et faire bouger un PNJ automatiquement.

### 2.1 Créer un projet de base
1. Ouvrir Unreal Engine 5  
2. Choisir le template **Third Person**  
3. Vérifier les plugins :
   - AI Module (activé par défaut)
   - Navigation System

### 2.2 Ajouter un Nav Mesh Bounds Volume
1. Ouvrir le niveau
2. Dans **Place Actors → Volumes**, sélectionner **Nav Mesh Bounds Volume**
3. Le placer dans la scène et redimensionner pour couvrir la zone jouable
4. Appuyer sur **P** pour visualiser la zone navigable (en vert)

Si rien ne s’affiche :
- Vérifier que le sol a la collision activée (`Can Ever Affect Navigation = true`)
- Vérifier que le volume couvre bien la zone

### 2.3 Ajouter un PNJ contrôlé par IA
1. Créer un **Blueprint Character** : `BP_Enemy`
2. Dans les propriétés :
   - `Auto Possess AI` → `Placed in World or Spawned`
   - `AI Controller Class` → `BP_AIController` (à créer)

### 2.4 Créer un AIController simple
1. Créer un **Blueprint Class → AIController** nommé `BP_AIController`
2. Dans son Event Graph :

```blueprint
   Event BeginPlay
      → Get Player Character
      → AI MoveTo (Target Actor = Player Character)
```

3. Lancer le jeu : le PNJ se déplace vers le joueur

Le nœud Blueprint **AI MoveTo** utilise automatiquement le NavMesh pour trouver un chemin.

### 2.5 Créer une patrouille automatique

**Étapes :**

1. Placer deux `TargetPoint` dans la scène (`PointA`, `PointB`)
2. Dans `BP_AIController` :

   ```blueprint
   Variables :
      TargetPoints (Array of TargetPoint)
      CurrentIndex (Integer)

   Event BeginPlay
      → GetAllActorsOfClass(TargetPoint)
      → Set TargetPoints

   Custom Event: MoveToNextPoint
      → AI MoveTo (Target Actor = TargetPoints[CurrentIndex])
      → OnSuccess → Increment Index (modulo)
      → Delay 2s
      → Call MoveToNextPoint
   ```

Le PNJ patrouille de point en point sans jamais se bloquer.

---

## PARTIE 3 — Comprendre et personnaliser le NavMesh

### Objectif

Maîtriser la génération et les paramètres du NavMesh pour obtenir un comportement réaliste.

### 3.1 Les paramètres de génération du NavMesh

Sélectionner **RecastNavMesh** dans l’Outliner (ou afficher via “Show → Navigation”).

| Paramètre       | Description                                                        |
| --------------- | ------------------------------------------------------------------ |
| Agent Radius    | Taille horizontale du PNJ                                          |
| Agent Height    | Hauteur du PNJ                                                     |
| Max Step Height | Hauteur maximale franchissable (marches)                           |
| Max Slope       | Angle maximal de pente grimpable                                   |
| Cell Size       | Résolution de la grille (plus petit = plus précis mais plus lourd) |

### 3.2 Zones de navigation spéciales (Area Class)

Créer des zones avec un coût différent pour le pathfinding :

* Zone lente (boue, sable)
* Zone dangereuse (feu, piège)

**Étapes :**

1. Créer une **Nav Modifier Volume**
2. Dans les propriétés :

   * `Area Class` → `NavArea_Slow`
3. Le PNJ évitera cette zone si possible (coût plus élevé)

On peut créer des `NavArea_Custom` en Blueprint ou C++ pour un contrôle total.

### 3.3 Obstacles dynamiques

Les obstacles peuvent apparaître ou disparaître (portes, véhicules, débris).

**Ajouter un :**

* `Nav Modifier Component` sur un objet mobile
* `Dynamic Obstacle = True`

Le NavMesh se met à jour automatiquement quand l’objet bouge.

### 3.4 Liens de navigation (Nav Link Proxy)

Permet à l’IA de franchir des trous, sauter des plateformes, grimper, etc.

**Étapes :**

1. Placer un `Nav Link Proxy`
2. Déplacer les flèches de départ et d’arrivée
3. L’IA utilisera ce lien lors du calcul du chemin

---

## PARTIE 4 — Déboguer et optimiser la navigation

### 4.1 Visualiser le NavMesh en jeu

* Touche **P** → activer/désactiver la vue du maillage
* `ShowDebug Navigation` → afficher les données de navigation
* `Show Navigation Invoker Locations` → utile pour grands mondes

### 4.2 Navigation Invokers (Open Worlds)

Dans les grands mondes, le NavMesh ne peut pas tout couvrir.

Utiliser le **Navigation Invoker Component** sur le joueur :

* Génère le NavMesh dynamiquement autour de lui
* Réduit la mémoire utilisée

**Étapes :**

1. Dans le Character Blueprint → ajouter `Navigation Invoker`
2. Régler `Tile Generation Radius` (ex : 5000)

### 4.3 Conseils de performance

| Technique                        | Effet                                     |
| -------------------------------- | ----------------------------------------- |
| Réduire la résolution du NavMesh | Moins de calcul, moins précis             |
| Limiter le volume de navigation  | Évite le baking inutile                   |
| Grouper les PNJ                  | Un seul contrôleur pour plusieurs entités |
| Ajuster le `Tile Size UU`        | Calcul plus rapide                        |

### 4.4 Exercice final : NavMesh + IA réactive

Créer une IA qui :

1. Patrouille entre deux points
2. Change de direction quand le joueur la bloque
3. Contourne un obstacle dynamique

Combiner `AI MoveTo` + `Dynamic Obstacle` + `NavModifierVolume`.

---

## SYNTHÈSE DU COURS

| Élément                   | Fonction                                            |
| ------------------------- | --------------------------------------------------- |
| Nav Mesh Bounds Volume    | Définit la zone navigable                           |
| Recast NavMesh            | Contient la configuration du maillage               |
| AI MoveTo / Simple MoveTo | Ordonne un déplacement selon le NavMesh             |
| Nav Modifier Volume       | Crée des zones spéciales (coûts différents)         |
| Nav Link Proxy            | Relie deux zones séparées                           |
| Dynamic Obstacle          | Met à jour le NavMesh en temps réel                 |
| Navigation Invoker        | Génère dynamiquement la navigation autour du joueur |

---

## POUR ALLER PLUS LOIN

* **EQS (Environment Query System)** : trouver dynamiquement des positions dans le monde (abris, cachettes, cibles)
* **Behaviour Tree + NavMesh** : combiner la navigation et la prise de décision
* **C++ NavMesh Interface :**

  * `UNavigationSystemV1::SimpleMoveToActor()`
  * `UNavMovementComponent`

---

## COMMANDES UTILES

| Commande               | Effet                                     |
| ---------------------- | ----------------------------------------- |
| `P`                    | Afficher/masquer le NavMesh               |
| `ShowDebug Navigation` | Informations sur le pathfinding           |
| `RebuildNavigation`    | Reconstruit le NavMesh                    |
| `ai debug`             | Debug complet de l’IA                     |
| `navmesh showpoly`     | Affiche le maillage polygone par polygone |

