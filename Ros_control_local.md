name:inverse
layout:true
class: center, middle, inverse
---

# ROS control et son utilisation pour les Dynamixels

.footnote[dorian.goepp@inria.fr]

???
Je suis Dorian Goepp et je viens de l'INRIA Nancy.

Cette présentation va s'articuler en trois grandes parties. Nous commencerons par quelques mots au sujet des Dynamixels, suivis d'une présentation haut niveau de `ros_control` puis verrons un cas concret avec les actionneurs dynamixels.

Mais avant cela, un peu de contexte.

---

## des Dynamixels pour nos robots

---
layout: false
### Le projet ResiBots
<div style="float:right; margin:1em">
    <img width="200px" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/logo_inria.png"/><br/>
    <img width="200px" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/resibots_avatar.png"/>
</div>

Objectif : développer des algorithmes d'apprentissage par essai-erreur

- l'autonomie des robots sur le long terme,
- en gérant des dommages imprévus
- avec des robots abordables

Mots-clefs :
- a-priori issus de simulations
- optimisation bayésienne
- processus gaussiens

???
Le but du projet est de poser les **fondations algorithmiques** permettant à des robots **abordables** de se remettre de **dommages imprévus** en quelques minutes et en **autonomie**.

---
### Robots pour ResiBots

robots à pattes,

.center[<img height="500px" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/Hexaforce-aggressive.jpg"/>]

???
Sur ce robot, chaque patte est dotée de trois degrés de liberté, chacun actionné par un servo-moteur dynamixel.

Nous avons aussi une copie quasi conforme de cet hexapode (sans les capteurs de force) et une version plus grande dont les pattes sont équipées de roues orientables et motorisées TODO : photographie ?

---
### Robots pour ResiBots

robots à pattes, un bras manipulateur et une base mobile Kuka.

.center[<img height="500px" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/omnigrasper.jpg"/>]

???
Ici aussi des dynamixels sont utilisés, pour le bras. C'est cependant une autre gamme que pour les hexapodes

---
### Robots pour ResiBots

robots à pattes, un bras manipulateur et une base mobile Kuka.

.center[<img height="500px" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/optitrack.jpg"/>]




---
class: center, middle, inverse
## Dynamixels

???
Comme vous avez pu le voir, **tous nos robots**, sauf la base mobile Kuka sont **construit** avec des **Dynamixels** de Robotis.

---
### Dynamixels

.center[![un Dynamixel MX-28](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/Dynamixel-full-range.jpg)]

- toute l'électronique est intégrée
- branchement en cascade -> une seule interface série/USB et une seule alimentation
- informations disponibles sur l'état du servo (position, vitesse, température, etc.)
- le protocole de communication est (presque) complètement documenté
- protégés contre les surcharges et surtensions
- les actionneurs sont _relativement_ abordables
- plusieurs gammes

???
Les actionneurs de Robotis sont des servo-moteurs tout intégrés, communiquant avec un protocole série et branchés en cascade.

Protocole documenté : ce qui nous donne une entière liberté sur le logiciel (le dépôt github a été créé en mars 2016)

**déjà travaillé avec le SDK de Robotis ?**

Différentes gammes : pour les différents besoins qu'on peut en avoir

---
### Dynamixels

- permettent un assemblage facile et modulaire

.center[![Quelques exemples de montage pour les Dynamixels](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/rx-28_device_combination.png)]
<!-- .right[![Quelques exemples de montage pour les Dynamixels](file:///home/dgoepp/Images/Dynamixel/Modularité-dynamixels.jpg)] -->

???
Et aussi, ce sont les actionneurs utilisés pour construire nos robots

---
### Robots tiers basés sur les Dynamixels
Poppy ...

.center[<img height="500px" alt="Poppy, le robot humanoïde de l'INRIA Bordeaux" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/poppy.jpg"/>]

???
un projet initié à l'INRIA Bordeaux

---
### Robots tiers basés sur les Dynamixels
Poppy et bien sûr TurtleBot 3

.center[<img width="800px" alt="Poppy, le robot humanoïde de l'INRIA Bordeaux" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/turtlebot3_models.png"/>]

???
Un projet en collaboration avec Robotis, motorisé avec la nouvelle gamme Dynamixel X.

---
## Donc...

- nous travaillons avec ROS
- nos robots sont à base de dynamixels

TODO : deux approches d'intégration à ROS:
- ad-hoc en créant notre propre architecture de commande et choisissant la façon de communiquer avec ROS
- avec ros_control

???
Résumons la présentation. Les robots de ResiBots sont à base de Dynamixels et tournent avec ROS. Il y a plusieurs robots avec différentes versions d'actionneurs. Nous pouvons avoir à ajouter ou retirer des actionneurs.

Pour simplifier tout ça, il serait pratique d'avoir une couche logicielle intégrée à ROS qui gère tous ces actionneurs pour nous.

Et là, quelle surprise ! ros_control existe déjà ! Voyons ce que c'est.

Nous avons donc deux approches possibles TODO


Nous avons choisi d'utiliser ROS_control. Nous verrons plus tard pourquoi (TODO)

---
class:center, middle, inverse
## <tt>ros_control</tt>

.footnote[fortement inspiré de la [présentation](https://vimeo.com/107507546) de Adolfo Rodríguez Tsouroukdissian]

???

Maintenant le cœur du sujet.

---
### <tt>ros_control</tt>

C'est

- un ensemble de paquets ROS
- un petit framework/cadriciel pour l'écriture de contrôleurs de robots


<br/>
Il va nous permettre de :
- distinguer le code chargé de
    - l'_interaction avec le matériel_, de celui chargé de
    - la _boucle d'asservissement_
- réutiliser le code des autres
- accéder aux outils comme MoveIt! et navigation stack
- asservir en temps réel

<br/>
Trois modes de commande : position, vitesse et effort

???
<tt>ros_control</tt> intègre à la fois la définition de la boucle d'asservissement et l'interface matérielle mais les distingue clairement et leur donne une certaine indépendance.

Réutiliser le code des autres : appliquer l'idée à l'origine de ROS;  
  se concrétise par le partage de contrôleurs ou d'interfaces pour des robots

Temps réel : (sans pour autant l'imposer)
---
### <tt>ros_control</tt>

<br/>
<br/>
<br/>
.image[![illustration de ros_control](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/diagrammes/dessin.svg)]

???
<tt>ros_control</tt> articule le code autour de trois composants...

## Le RobotHW :
- interagit avec le matériel
- met des ressources à disposition des contrôleurs
- gère les conflits de ressources
- une ressource : une seule articulation dans un mode de commande (position, vitesse, effort, etc.)

## Les contrôleurs
- ne discutent pas directement avec le matériel
- requièrent des ressources

## controller_manager
- voir plus tard

---
### <tt>ros_control</tt>

<br/>
<br/>
<br/>
.image[![illustration de ros_control](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/diagrammes/dessin-RobotHW-1.svg)]

---
### <tt>ros_control</tt>

<br/>
<br/>
<br/>
.image[![illustration de ros_control](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/diagrammes/dessin-RobotHW-2.svg)]

---
### <tt>ros_control</tt>

<br/>
<br/>
<br/>
.image[![illustration de ros_control](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/diagrammes/dessin-RobotHW-3.svg)]

---
### <tt>ros_control</tt>

<br/>
<br/>
<br/>
.image[![illustration de ros_control](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/diagrammes/dessin-RobotHW-4.svg)]

???
La communication entre l'interface matérielle et le contrôleur se fait par une interface matérielle. Le contrôleur reçoit en fait des pointeurs vers des données écrites ou lues par `RobotHW`.

Les contrôleurs ont des interfaces ROS. Par exemple, un contrôleur de base mobile s'abonnera au sujet `/cmd_vel`.



---
### Circulation des données

<br/>
<br/>
<br/>
.image[![illustration de ros_control](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/diagrammes/dessin-RobotHW-dataflow.svg)]

???
Dans la boucle principale (temps réel), on appelle `read()`, `update()`, et enfin `write()`.

L'interface matérielle donne en fait des pointeurs vers des variables qui seront lues ou modifiées par les contrôleurs

---
### Contrôleurs

<br/>
<br/>
<br/>
.image[![illustration de ros_control](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/diagrammes/dessin-RobotHW-ControllerManager.svg)]

???
Je vais maintenant parler un peu plus des contrôleurs, et du `controller_manager` qui les gère.

---
### Contrôleurs

Plugin chargé dynamiquement par le <tt>controler_manager</tt>.

Un contrôleur peut être dans deux états :
- <tt>stopped</tt>
- <tt>running</tt>

via des services ROS, <tt>controler_manager</tt>

- charge les contrôleurs
- gère leur cycle de vie

Les contrôleurs sont exécutés **périodiquements** et **séquentiellement**

???
<tt>controler_manager</tt> gère les contrôleurs et les charge.

Il permet aussi de **changer** de contrôleur **à la volée**.

En cas d'arrêt d'urgence, l'idée est de faire un **reset** sur le **contrôleur**.

---
class: middle, center, inverse

## dynamixel hardware interface

.footnote[[https://github.com/resibots/dynamixel_control_hw/](https://github.com/resibots/dynamixel_control_hw/)]

???
Jusque là nous avons vu à un haut niveau comment ros_control fonctionne. Il sépare  le matériel des contrôleurs, permet d'en changer à chaud, et offre trois modes de commande.

Les robots sur lesquels je travaille utilisent tous des Dynamixels et nous voulions les intégrer correctement dans ros_control. Voyons donc l'interface matérielle qui en découle.

---
### <tt>ros_control</tt>

Pour nous :

- abstrait la question du modèle de Dynamixel (pour un même protocole)
- suivre l'évolution les robots (nb de servos, nouveaux modèles) avec un minimum de changements logiciels.

En somme, moins de code écrit en dur, plus de flexibilité.

Cependant, les Dynamixels disposent déjà d'un asservissement intégré (type PID).

???
En plus, les outils ROS sont à portée de main (actionlib par exemple).

Les dynamixels sont en effet des actionneurs très complets et comprennent leur propre boucle d'asservissement. Ce qui nous a motivé pour utiliser ros_control (>< ad-hoc) est :
- d'une part l'intégration aux outils ROS
- d'autre part, la perspective de l'écriture d'un algorithme d'asservissement relativement bas niveau, par exemple pour maintenir l'hexapode horizontal, indépendamment de l'état du sol.

---
### <tt>libdynamixel</tt>.blue[\*]

Nous avons développé la bibliothèque <tt>libdynamixel</tt>.blue[\*]. Caractéristiques principales :
- bibliothèque open source
- écrite en C++11
- fonctionne avec les version 1 et 2 du protocole
- couvre (presque) tous les actionneurs
- offre deux approches :
    - template : bas niveau
    - classes : haut niveau, masque les différences entre les modèles d'actionneurs

.footnote[.blue[*][https://github.com/resibots/libdynamixel](https://github.com/resibots/libdynamixel)]

???
Au moment de l'écriture de libdynamixel, le SDK de Robotis n'était pas open source.

---
### <tt>dynamixel_control_hw</tt>

Notre travail est basé sur
- la bibliothèque <tt>libdynamixel</tt> développée dans l'équipe
- <tt>ros_control_boilerplate</tt>.blue[\*]

Fonctionnalités :
- nombre arbitraire d'actionneurs (fichier de configuration)
- commande en radians (pas de conversion)
- décalage et vitesse limite pour chaque actionneur

Charactéristiques :
- fréquence ≥ 50 Hz pour 18 servo-moteurs Dynamixel


.footnote[.blue[\*][https://github.com/davetcoleman/ros_control_boilerplate/](https://github.com/davetcoleman/ros_control_boilerplate/)]
???
Notre travail est basé sur ros_control_boilerplate qui offre un bon exemple de création d'une interface matérielle (pas pour la création d'un contrôleur)

---
### Exemple de fichier de configuration

```yaml
dynamixel_control_hw:
  loop_frequency: 50
  cycle_time_error_threshold: 0.1
  serial_interface: /dev/ttyACM0
  baudrate: 1000000 # in bauds
  dynamixel_timeout: 0.05 # in seconds
  dynamixel_scanning: 0.05 # in seconds
  # correspondance between hardware IDs of the actuators and their names in ROS
  hardware_mapping:
    arm_joint_1: 2
    arm_joint_2: 3
  # offset to be added, in radians, to the position of an actuator
  hardware_corrections:
    "2": 3.14159265359
    "3": 3.14159265359
```

---
### Exemple de fichier de configuration

```yaml
dynamixel_controllers:
  joint_state_controller:
    type: joint_state_controller/JointStateController
    publish_rate: 50

  first_traj_controller:
    type: position_controllers/JointTrajectoryController
    joints:
      - arm_joint_1
      - arm_joint_2
```

---
### Exemple de roslaunch

```xml
<?xml version="1.0"?>

<launch>

  <!-- URDF robot description -->
  <param name="robot_description" command="cat $(find dynamixel_control_hw)/urdf/sample.urdf" />

  <!-- Publish robot's state in the tf tree -->
  <node pkg="robot_state_publisher" type="robot_state_publisher" name="rob_state_pub"/>

  <!-- Parameters for the hardware interface and controllers -->
  <rosparam file="$(find dynamixel_control_hw)/config/traj.yaml"/>

  <!-- launch our hardware interface -->
  <node pkg="dynamixel_control_hw" type="dynamixel_control_hw" name="dynamixel_control_hw"
       output="screen"/>

  <!-- Ask controller_manager to load a controller for our dummy robot -->
  <node name="controller" pkg="controller_manager" type="spawner" respawn="false"
    output="screen" args="/dynamixel_controllers/joint_state_controller
      /dynamixel_controllers/first_traj_controller" />
</launch>
```

---
### Démo avec deux actionneurs

Faire un mini bras à deux degrés de liberté et montrer le résultat.

Montrer :

- Rviz
- rqt controller
- rqt graph

???
L'intégration à Rviz et la disponibilité de <tt>rqt_graph</tt> sont possibles grâce au contrôleur <tt>JointTrajectoryController</tt>
<tt>rqt_controller</tt> est fourni dans le pacquet <tt>ros_controllers</tt> et implémente un client au sens de l'actionlib.

On a donc une bonne intégration avec les outils de l'échosystème ROS (et une démo facile) grâce à <tt>ros_control</tt>.

---
class: middle, center, inverse

## Conclusion

---
### Pour résumer

TODO

(TODO) Un bémol cependant, les résultats obtenus ont requis de se plonger quelques fois dans le code de <tt>ros_control</4tt> ou des outils associés car la documentation du projet est très succinte.

---
### Et au delà
* ros_control peut faire plus encore...
  * intégration avec gazebo
  * transmission interface
  * limites d'articulation
  * interface matérielle combinée
  * politique de gestion des ressources
  * ros_controllers

<br/>
* ros_control devrait dans le futur pouvoir faire...
  * gérer le changement de mode de commande

<br/>
* future work
  * la commande en vitesse (wheel mode)
  * limites angulaires
  * protocole 2


--

???
J'aimerai travailler à une documentation plus complète de <tt>ros_control</tt>


---
class: center, middle, inverse

.footnote[dorian.goepp@inria.fr]