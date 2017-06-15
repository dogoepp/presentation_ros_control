* équipe

  Qui je suis (nom), je viens de l'équipe de recherche LARSEN, au sein du Loria (UL) et de l'INRIA à Nancy
  subsidiaire :
    l'équipe s'intéresse à ...
    pour ce faire, nous sommes équipés d'un appartement intelligent contenant ...
    et d'un espace d'expérimentation robotique comprenant un iCub, un système de MoCap et les robots de ResiBots

* projet

  ResiBots s'intéresse à ...

* robots
  pour ce faire, nous utilisons des robots à pattes, un bras manipulateur et une base mobile Kuka

* Dynamixel
  Il a été choisi d'utiliser les actionneurs de Robotis (c.à.d. Dynamixel et Dynamixel Pro) pour construire les robots car
  - toute l'électronique est intégrée et les servos sont branchés en BUS, ne nécessitant qu'une interface série/USB et une alimentation
  - le protocole de communication est (presque) complètement documenté, ce qui nous donne une entière liberté sur le logiciel (la libération du SDK par Robotis n'est que relativement récente, l'un de vous a-t-il travaillé avec ?)
  - les actionneurs sont _relativement_ abordables

  Nous avons développé la bibliothèque Libdynamixel. Elle répond aux problématiques suivantes:
  - bibliothèque open source (cf SDK Robotis)
  - écrite en C++
  - fonctionne avec les version 1 et 2 du protocole
  - couvre (presque) tous les actionneurs
  - offre deux approches : bas niveau à base seulement de templates, haut niveau avec de l'héritage et qui masque les différences entre les modèles d'actionneurs

* ros_control

  Maintenant le coeur du sujet. Cette partie est fortement inspirée de la [présentation de Adolfo Rodríguez Tsouroukdissian ](https://vimeo.com/107507546)(Pal Robotics)
  Pourquoi ros_control ?
    - il est conçu pour :
      - Appliquer l'idée même de ROS, réutiliser le travail des autres, aux robots : contrôlers, etc.
      - déjà compatible avec des outils tiers comme MoveIt! et navigation stack
      - moins de code "boilerplate" à écrire
      - prêt pour le temps réel, sans pour autant l'imposer
      - faciliter l'intégration de matériel dans ROS
    - il permet trois modes de commande : position, vitesse et effort
    - pour nous : cela nous permet de communiquer avec nos robots sans nos poser la question du modèle de Dynamixel (sauf le protocole) et de faire évoluer les robots (modèles, nb de servos) avec un minimum de changements logiciels. En somme, moins de code écrit en dur, plus de flexibilité

  ros_control articule le code autour de trois composants :
  - les interfaces matérielles qui s'occupent de l'interaction avec le matériel (récupérer l'état et envoyer la consigne)
  - les contrôleurs qui calculent des consignes en fonction des valeurs de capteurs
  - le controller_manager, une sorte de chef d'orchestre (nous en parlerons plus tard)

  L'interface matérielle :
    - interagit avec le matériel
    - met des ressources à disposition des contrôleurs
    - gère les conflits de ressources
    - une ressource : une seule articulation dans un mode de commande (position, vitesse, effort, etc.)

  Les contrôleurs
    - ne discutent pas directement avec le matériel
    - requièrent des ressources

  ros_control offre aussi des outils
    - des contrôleurs directs simples (voir ros_controllers) dont des

  Le flux des données

  L'interface matérielle donne en fait des pointeurs vers des variables qui seront lues ou modifiées par les contrôleurs

  En résumé ...
* ros_control avec des dynamixels, notre implémentation
  Notre travail est basé sur
  - ros_control_boilerplate qui offre un bon exemple de création d'une interface matérielle (pas pour la création d'un contrôleur)
  - la bibliothèque libdynamixel (_lien_) développée dans l'équipe

  présenter mon schéma en montrant le flux des données.

  En résumé ...
* vidéo de démo (fun)

* au delà
  * ros_control peut faire plus encore ...
  * ros_control devrait dans le future pouvoir faire ...
  * nous aimerions ajouter ...