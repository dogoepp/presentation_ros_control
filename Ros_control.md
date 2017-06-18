name:inverse
layout:true
class: center, middle, inverse
---

# ROS control et son utilisation pour les Dynamixels

???
Je suis Dorian Goepp et je viens de l'INRIA Nancy.

Cette présentation va s'articuler en trois grandes parties. Nous commencerons par quelques mots au sujet des Dynamixels, suivis d'une présentation haut niveau de `ros_control` puis verrons un cas concret avec les actionneurs dynamixels.

Mais avant cela, un peu de contexte.

---

## des Dynamixels pour nos robots

---
layout: false
exclude: true
### Équipe Larsen

LARSEN : Lifelong Autonomy and interaction skills for Robots in a Sensing ENvironment

<img width="100px" style="float:left" alt="Loria (UL)" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/logo_loria.jpg"/>
<img width="200px" style="float:right" alt="l'INRIA" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/logo_inria.jpg"/>

--
exclude:true

Détails :

-  l'équipe s'intéresse à ...
-  pour ce faire, nous sommes équipés d'un appartement intelligent contenant ...
-  et d'un espace d'expérimentation robotique comprenant un iCub, un système de MoCap et les robots de ResiBots

---
layout: false
### Le projet ResiBots <img width="200px" style="float:right; margin:1em" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/resibots_avatar.png"/>

Le but du projet est de poser les fondations algorithmiques permettant à des robots abordables de se remettre de dommages imprévus en quelques minutes et en autonomie.

Approche :

- générer des a-prioris par des simulations
- utiliser ces a-prioris pour choisir un comportement/une démarche qui maximise l'objectif

---
### Robots pour ResiBots

robots à pattes,

.center[<img height="500px" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/Hexaforce-aggressive.jpg"/>]

---
### Robots pour ResiBots

robots à pattes, un bras manipulateur et une base mobile Kuka.

.center[<img height="500px" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/omnigrasper.jpg"/>]

---
### Robots pour ResiBots

robots à pattes, un bras manipulateur et une base mobile Kuka.

.center[<img height="500px" src="file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/optitrack.jpg"/>]




---
class: center, middle, inverse
## Dynamixels

???
Tout, sauf la base mobile Kuka est construit avec des Dynamixels de Robotis.

---
### Dynamixels

.center[![un Dynamixel MX-28](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/Dynamixel-full-range.jpg)]

???
Les actionneurs de Robotis sont des servo-moteurs tout intégrés, communiquant avec un protocole série et branchés en cascade.

--
- toute l'électronique est intégrée
- protégés contre les surcharges et surtensions
- branchement en cascade -> une seule interface série/USB et une seule alimentation
- le protocole de communication est (presque) complètement documenté
    - ce qui nous donne une entière liberté sur le logiciel (la libération du SDK par Robotis n'est que relativement récente, l'un de vous a-t-il travaillé avec ?)
- les actionneurs sont _relativement_ abordables
- ils y a plusieurs gammes, allant du très petit au assez gros (les Pro).

---
### Dynamixels

- permettent un assemblage facile et modulaire

.center[![Quelques exemples de montage pour les Dynamixels](file:///home/dgoepp/Documents/RosControl/Presentation-TechDays-2017/images/rx-28_device_combination.png)]
<!-- .right[![Quelques exemples de montage pour les Dynamixels](file:///home/dgoepp/Images/Dynamixel/Modularité-dynamixels.jpg)] -->

???
Et aussi, ce sont les actionneurs utilisés pour construire nos robots

---
## Donc...

- nous travaillons avec ROS
--

- nos robots sont à base de dynamixels

???
Résumons la présentation. Les robots de ResiBots sont à base de Dynamixels et tournent avec ROS. Il y a plusieurs robots avec différentes versions d'actionneurs. Nous pouvons avoir à ajouter ou retirer des actionneurs.

Pour simplifier tout ça, il serait pratique d'avoir une couche logicielle intégrée à ROS qui gère tous ces actionneurs pour nous.

---
class:center, middle, inverse
## <tt>ros_control</tt>

.footnote[fortement inspiré de la [présentation](https://vimeo.com/107507546) de Adolfo Rodríguez Tsouroukdissian]

???
Et là, quelle surprise ! ros_control existe déjà ! Voyons ce que c'est.

Maintenant le cœur du sujet.

---
### <tt>ros_control</tt>

C'est

- un ensemble de paquets ROS
- un petit framework/cadriciel pour l'écriture de contrôleurs de robots

--

<br/>
Il va nous permettre de :
- séparer le code spécifique au matériel et la boucle de contrôle

--
- réutiliser le code des autres

???
appliquer l'idée à l'origine de ROS;  
se concrétise par le partage de contrôleurs ou d'interfaces pour des robots

--
- accéder aux outils comme MoveIt! et navigation stack

--
- asservir en **temps réel**

???
(sans pour autant l'imposer)

--

<br/>
Trois modes de commande : position, vitesse et effort


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
Revenons à l'illustration précédente et parlons un peu du `controller_manager`. Il gère le cycle de vie des contrôleurs via des services.

---
### Contrôleurs

Plugin chargé dynamiquement par le <tt>controler_manager</tt>.

Un contrôleur peut être dans deux états :
- stopped
- running

???
<tt>controler_manager</tt> gère les contrôleurs et les charge

--

Les contrôleurs sont exécutés **périodiquements** et **séquentiellement**..

--

Via des services ROS, <tt>controler_manager</tt>

- charge les contrôleurs
- gère leur cycle de vie

---
### Goodies de ROS control

Les possibilités avancées offertes par ros_control

- transmission interface
- joint limit
- combined hardware interface
- resource claim policies
- changement de contrôleur à la volée

???

En supplément : des contrôleurs simples sans asservissement (dans ros_controllers)

Autres possibilités : arrêt d'urgence, politique personalisée de gestion des ressources.

---
### <tt>ros_control</tt>

Jusque là nous avons vu à un haut niveau comment ros_control fonctionne. Il sépare  le matériel des contrôleurs, permet d'en changer à chaud, de gérer les limites d'articulation et trois types de commande.

Comme dit, les robots sur lesquels je travaille utilisent tous des Dynamixels et nous voulions les intégrer correctement dans ros_control. Voyons donc l'interface matérielle qui en découle.

---
class: middle, center, inverse

## dynamixel hardware interface

.footnote[[https://github.com/resibots/dynamixel_control_hw/](https://github.com/resibots/dynamixel_control_hw/)]

---
### <tt>ros_control</tt>

Pour nous :

- abstrait la question du modèle de Dynamixel (pour un même protocole)
- suivre l'évolution les robots (nb de servos, nouveaux modèles) avec un minimum de changements logiciels.

--

En somme, moins de code écrit en dur, plus de flexibilité.

???
En plus, les outils ROS sont à portée de main (actionlib par exemple).

---
### <tt>libdynamixel</tt>.blue[\*]

Nous avons développé la bibliothèque Libdynamixel. Elle répond aux problématiques suivantes:
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

.footnote[.blue[\*][https://github.com/davetcoleman/ros_control_boilerplate/](https://github.com/davetcoleman/ros_control_boilerplate/)]

???
Notre travail est basé sur ros_control_boilerplate qui offre un bon exemple de création d'une interface matérielle (pas pour la création d'un contrôleur)

---
### Exemple d'utilisation

```yaml
dynamixel_control_hw:
  loop_frequency: 10
  cycle_time_error_threshold: 0.2
  serial_interface: /dev/ttyUSB0
  baudrate: 1000000 # in bauds
  dynamixel_timeout: 0.05 # in seconds
  dynamixel_scanning: 0.05 # in seconds
  # correspondance between hardware IDs of the actuators and their names in ROS
  hardware_mapping:
    first: 1
    second: 26
  # offset to be added, in ticks, to the position of an actuator
  hardware_corrections:
    "26": 45
```

---
### Exemple de fichier de config

```yaml
dynamixel_controllers:
  joint_state_controller:
    type: joint_state_controller/JointStateController
    publish_rate: 10

  first_position_controller:
    type: position_controllers/JointPositionController
    joint: first

  second_position_controller:
    type: position_controllers/JointPositionController
    joint: second
```

---
### Exemple de roslaunch

```xml
<?xml version="1.0"?>

<launch>

  <!-- URDF robot description -->
  <param name="robot_description" command="cat $(find dynamixel_control_hw)/urdf/sample.urdf" />
  <!-- Publish robot's state in the tf tree -->
  <node pkg="robot_state_publisher" type="robot_state_publisher" name="rob_state_pub" />

  <!-- Parameters for the hardware interface and controllers -->
  <rosparam file="$(find dynamixel_control_hw)/config/sample.yaml"/>

  <!-- launch our hardware interface -->
  <node pkg="dynamixel_control_hw" type="dynamixel_control_hw" name="dynamixel_control_hw"
        output="screen"/>

  <!-- Start a controller for our dummy robot -->
  <node name="controller" pkg="controller_manager" type="spawner" respawn="false"
    output="screen" args="/dynamixel_controllers/joint_state_controller
                          /dynamixel_controllers/first_position_controller
                          /dynamixel_controllers/second_position_controller" />
</launch>

```

---
### Démo avec deux actionneurs

Faire un mini bras à deux degrés de liberté et montrer le résultat.

Montrer :

- Rviz
- rqt controller
- rqt graph

---
class: middle, center, inverse

## Conclusion

---
### Pour résumer

---
### Et au delà
  * ros_control peut faire plus encore ...
      * intégration avec gazebo
  * ros_control devrait dans le future pouvoir faire ...
      * gérer le changement de mode de commande
  * nous aimerions ajouter ...
      * la commande en vitesse (wheel mode)
      * limites angulaires
      * arrêt d'urgence
  * j'aimerai travailler à une documentation plus complète de <tt>ros_control</tt> pour en faciliter l'accès

---
class: center, middle, inverse
