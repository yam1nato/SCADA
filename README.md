# Projet d'Automatisation - Système de Tri avec Factory I/O et Control I/O

Ce projet présente la conception, le câblage logique et la sécurisation d'une ligne de tri industrielle automatisée sous **Factory I/O** pilotée par **Control I/O**. Le système gère le défilement de colis, l'identification par capteur de vision numérique, le tri via un pousseur pneumatique, le comptage avec affichage en temps réel, un système d'alertes graduées, ainsi qu'une procédure d'arrêt d'urgence matérielle redondante.

## 📋 Table de Validation de la Configuration

Le système implémente l'intégralité du cahier des charges minimal requis pour l'évaluation et l'enrichit avec plusieurs actionneurs et capteurs complémentaires pour parfaire le scénario industriel.

| Configuration minimale requise | Éléments présents sur le schéma et la scène |
| :--- | :--- |
| **2 Capteurs Booléens** | • `Start Button 0` (Bouton Marche)<br>• `Stop Button 0` (Bouton Arrêt)<br>• `Diffuse Sensor 0` (Capteur optique de passage de colis) |
| **2 Capteurs Register** | • `Potentiometer 0 (V)` (Consigne analogique de vitesse)<br>• `Vision Sensor 0 (Value)` (Caméra de vision numérique d'identification) |
| **2 Actionneurs Booléens** | • `Pusher 0` (Vérin pousseur de tri)<br>• `Warning Light 0` (Gyrophare lumineux d'alerte)<br>• `Alarm Siren 0` (Sirène sonore d'alarme)<br>• `Start Button 0 (Light)` (Voyant bouton marche)<br>• `Stop Button 0 (Light)` (Voyant bouton arrêt) |
| **2 Actionneurs Register** | • `Belt Conveyor (6m) 0 (V)` (Tapis principal à vitesse variable)<br>• `Belt Conveyor (4m) 0 (V)` (Tapis d'évacuation secondaire)<br>• `Digital Display 0` (Afficheur numérique de comptage) |
| **Procédure d’Arrêt d’Urgence** | • `Emergency Stop 0` *(Coup de poing physique câblé en coupure directe et isolée sur l'autorisation des tapis, sur le gyrophare et sur la sirène)* |

---

##  Architecture et Fonctionnement Logique

Le programme Control I/O est structuré en quatre sous-systèmes indépendants et hautement sécurisés :

### 1. Contrôle de la Ligne et Gestion de la Vitesse
* **Marche / Arrêt :** Une bascule `RS` est pilotée par le `Start Button 0` (Set) et le `Stop Button 0` inversé par une porte `NOT` (Reset). Le voyant de marche suit l'état de la bascule.
* **Consigne de Vitesse :** Un potentiomètre analogique délivre une tension (`0` à `10V`). Un bloc `MAX` bride la vitesse minimale à `2.0V`. Cette valeur est multipliée par la sortie filtrée de la bascule `RS`.
* **Sécurisation :** Une porte `AND2` intercepte la sortie de la bascule `RS` avec le bouton `Emergency Stop 0`. Si l'arrêt d'Urgence est enclenché, l'entrée du bloc `ASSIGN` tombe immédiatement à `0`, ce qui fige instantanément les deux moteurs de tapis (`Belt Conveyor 6m` et `4m`).

### 2. Reconnaissance Numérique et Tri Robotisé
* La caméra `Vision Sensor 0 (Value)` lit le code identifiant du colis en transit. 
* Un bloc de comparaison `>=` vérifie si la valeur du colis est supérieure ou égale à `6`. Si la condition est vraie, l'actionneur booléen `Pusher 0` (vérin pneumatique) est activé instantanément pour dévier le colis vers le tapis d'évacuation secondaire de 4 mètres.

### 3. Comptage et Alertes Graduées
* Un capteur optique `Diffuse Sensor 0` détecte le passage des colis triés. Son signal est envoyé vers un détecteur de front descendant `FTRIG` pour incrémenter proprement le bloc compteur `CTU` (Counter Up).
* La valeur courante du compteur (`CV`) est envoyée en direct sur l'actionneur de registre `Digital Display 0` implanté sur le pupitre.
* **Seuil 1 (Alerte Visuelle - Spécification à 5) :** Dès que le compteur atteint ou dépasse `5`, un bloc `>=` active la lumière d'avertissement `Warning Light 0`.
* **Seuil 2 (Alerte Sonore - Spécification à 6) :** Dès que le compteur atteint ou dépasse `6`, un second bloc `>=` déclenche la sirène `Alarm Siren 0`.

### 4. Scénario de Sécurité de l'Arrêt d'Urgence (`Emergency Stop`)
Le bouton coup de poing `Emergency Stop 0` est un contact normalement fermé (NC) envoyant un signal `1` en condition nominale. Lorsqu'il est enfoncé, son état passe à `0`. Afin de respecter l'isolation fonctionnelle exigée, il agit simultanément et sans interférence sur trois branches distinctes :
1. **Arrêt des Tapis :** Il force à `0` le signal d'activation logique en amont du traitement analogique/décimal des tapis (via la porte `AND2` placée juste avant le bloc `ASSIGN`).
2. **Coupure Gyrophare :** Une porte `AND2` placée directement devant l'actionneur `Warning Light 0` prend en entrées le résultat du comparateur de seuil et le signal de l'arrêt d'urgence. Dès l'appui, la lumière se coupe net, peu importe l'état du compteur.
3. **Coupure Sirène :** Une configuration identique avec une porte `AND2` isole l'actionneur `Alarm Siren 0`. La sirène est immédiatement réduite au silence lors du déclenchement de la sécurité.

---

## Visuels du Système

Le dossier du projet comprend trois captures d'écran clés documentant la réalisation :
1. `examen.controlio.png` : Capture complète du schéma logique Control I/O mettant en évidence le câblage propre des blocs logiques, arithmétiques et de comptage ainsi que l'interconnexion de l'arrêt d'urgence.
2. `scene_factory_io_globale.jpg` : Vue d'ensemble de la scène Factory I/O illustrant la disposition en T des tapis roulants, le capteur de vision, le vérin de tri (`Pusher 0`) et le flux des pièces.
3. `pupitre_controle.jpg` : Gros plan sur le coffret électrique d'exploitation équipé du potentiomètre de vitesse, de l'afficheur numérique de pièces, des voyants Marche/Arrêt et du bouton coup de poing rouge `Emergency Stop 0`.