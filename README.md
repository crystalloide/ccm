CCM (Cassandra Cluster Manager)
====================================================

WARNING - CCM configuration changes using updateconf does not happen according to CASSANDRA-17379
-------------------------------------------------------------------------------------------------

Après CASSANDRA-15234, pour prendre en charge les tests de mise à niveau de Python, 

CCM updateconf remplace le nouveau nom de clé et la nouvelle valeur au cas où l'ancien nom de clé et la nouvelle valeur sont fournis.

Par exemple, si vous ajoutez dans la config  `permissions_validity_in_ms`, cela remplacera

`permissions_validity` dans la valeur par défaut dans cassandra.yaml 

Cela était nécessaire pour garantir une surcharge correcte car CCM cassandra.yaml a des clés triées lexicographiquement. 

CASSANDRA-17379 a été ouvert pour améliorer l'expérience utilisateur et déprécier la surcharge des paramètres dans cassandra.yaml. 

Dans CASSANDRA 4.1+, par défaut, Cassandra refuse de démarrer avec une configuration contenant à la fois les anciennes et les nouvelles clés de configuration pour le même paramètre. 

Il faut démarrer Cassandra avec `-Dcassandra.allow_new_old_config_keys=true` pour povour overrider.

Pour des raisons historiques, les clés de configuration en double dans cassandra.yaml sont autorisées par défaut, 

démarrez Cassandra avec  `-Dcassandra.allow_duplicate_config_keys=false` pour empêcher cela. 

Veuillez noter que key_cache_save_period, row_cache_save_period, counter_cache_save_period 

ne seront affectés que par `-Dcassandra.allow_duplicate_config_keys`. 

Le ticket CASSANDRA-17949 a été ouvert pour décider de l'avenir de updateconf dans CCM après CASSANDRA-17379 : 

Jusque-là, gardez à l'esprit que les anciens paramètres remplacent les nouveaux paramètres dans cassandra.yaml lors de l'utilisation de updateconf

même si  `-Dcassandra.allow_new_old_config_keys=false` est mis par défaut. 

Remarque : 

Ne surchargez pas les paramètres dans CCM si possible. 

De plus, les modifications mentionnées ne sont effectuées que dans la branche principale. 

La meilleure façon de gérer Cassandra 4.1 dans CCM à ce stade est probablement de définir les paramètres 

`-Dcassandra.allow_new_old_config_keys=false` et `-Dcassandra.allow_duplicate_config_keys=false` 

afin d'interdire tout type de surcharge lors de l'utilisation des versions principales/publiées de CCM


CCM (Cassandra Cluster Manager)
====================================================

C'est un script/bibliothèque pour créer, lancer et supprimer un cluster Apache Cassandra en localhost.

L'objectif de ccm et ccmlib est de faciliter la création, la gestion et la destruction d'un petit cluster Cassandra sur une machine locale. 

Il est donc destiné à tester un cluster Cassandra.


Débutant dans le développement Python ?
--------------------------

Python a évolué depuis le début du développement de CCM. 

pip a remplacé easy_install, Python 3 a remplacé Python 2.7, 

et pyenv et virtualenv sont fortement recommandés pour gérer plusieurs versions et dépendances Python pour des applications Python spécifiques.


Vous êtes maintenant prêt à installer Python à l'aide de pyenv. 

Pour éviter d'obtenir une version qui échouera avec certains aspects de CCM, vous pouvez installer la version 3.9.16 de Python :
`pyenv install 3.9.16`

Pour créer l'environnement virtuel virtualenv, à partir de votre dépôt git comme répertoire de travail actuel, 

pour créer un environnement virtuel pour CCM : lancer la commande : 
`python3 -m venv --prompt ccm venv` 

Ensuite, faire `source venv/bin/activate` pour activer venv pour le terminal actuel

et `deactivate` pour le quitter


Vous êtes maintenant prêt à configurer le venv avec CCM et ses dépendances de test. 

`pip install -e <path_to_ccm_repo>` 

pour installer CCM et ses dépendances d'exécution à partir de requirements.txt, 

afin que la version de CCM que vous exécutez pointe vers le code sur lequel vous travaillez activement. 

Il n'y a pas d'étape de build ou de package car vous modifiez les fichiers Python en cours d'exécution à chaque fois que vous appelez CCM.

Vous y êtes presque. Il ne vous reste plus qu'à ajouter les dépendances de test qui ne sont pas dans `requirements.txt`. 

`pip install mock pytest requests` pour finir de configurer votre environnement de développement !


Prerequis
------------

- Une installation Python fonctionnelle (testée pour fonctionner avec Python 2.7).

- Voir `requirements.txt` pour les prérequis de runtime

- `mock` et `pytest` pour les tests

- ant (http://ant.apache.org/)

- Java : actuellement, Cassandra est compilé avec Java 8 ou 11 et est limité aux fonctionnalités et dépendances du langage JDK 8.

  Il existe plusieurs sources pour le JDK : Azul Zulu est une bonne option.

- Si vous souhaitez créer plusieurs clusters de nœuds, le moyen le plus simple est d'utiliser plusieurs alias de bouclage.

  Sur les distributions Linux modernes, il n'y a rien à faire.

  Notez que la section d'utilisation suppose qu'au moins 127.0.0.1, 127.0.0.2 et 127.0.0.3 sont disponibles.

### Pré-requis optionnels : 

- Paramiko (http://www.paramiko.org/): Paramiko ajoute la possibilité d'exécuter CCM à distance :
  `pip install paramiko`

Remarque : la machine distante doit être configurée avec un serveur SSH et un CCM fonctionnel. 

Lorsque vous travaillez avec plusieurs nœuds, chaque adresse IP exposée doit être dans un ordre séquentiel. 

Par exemple, le dernier numéro du 4e octet d'une adresse IPv4 doit commencer par 1 (par exemple 192.168.33.11). 

Consultez le fichier [Vagrantfile](misc/Vagrantfile) pour obtenir de l'aide sur la configuration de la machine CCM distante.

Installation
------------

ccm utilise la librairie python distutils, donc à partir du répertoire source, lancer :

    sudo ./setup.py install

ccm est disponible sur [Python Package Index][pip]:

    pip install ccm

[pip]: https://pypi.org/project/ccm/


Test sur environnement Gitpod :
-----

#### 1°) Avant de commencer, pour voir vos environnements Gitpod éventuellement déjà provisionnés : [cliquer ici](https://gitpod.io/workspaces)

#### 2°) Pour instancier l'environnement Gitpod de démonstration accessible avec un simple navigateur web :
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/crystalloide/ccm)

#### Cet environnement permet de déployer un cluster Cassandra nœuds avec Gitpod 
#### Attention : c'est ici uniquement à des fins de développement et de formation 
 


Usage
-----

Disons que l'on souhaite démarrer un cluster Cassandra à 3 nœuds :

### Version courte :

    ccm create test -v 2.0.5 -n 3 -s

Il faudra bien sûr remplacer la version `2.0.5` par la version de Cassandra souhaitée.


### Version longue : 

ccm fonctionne à partir d'un arbre source Cassandra (pas à partir de jars). 

Il existe deux manières d'indiquer à ccm comment trouver les sources :

  1. Si on a téléchargé *et* compilé les sources de Cassandra,

     on peut à ccm de les utiliser en lançant un nouveau cluster avec :

        ccm create test --install-dir=<path/to/cassandra-sources>

     ou, à partir de ce répertoire d'arborescence source, simplement

          ccm create test

  3. On peut demander à ccm d'utiliser une version publiée de Cassandra.

     Par exemple, pour utiliser Cassandra 2.0.5, exécutez

          ccm create test -v 2.0.5

     ccm va télécharger le binaire (depuis http://archive.apache.org/dist/cassandra),

     et configurer le nouveau cluster pour l'utiliser.

     Cela signifie que cette commande peut prendre quelques minutes la première fois que vous créez un cluster pour une version donnée.

     ccm enregistre la source compilée dans ~/.ccm/repository/,

     donc la création d'un cluster pour cette version sera beaucoup plus rapide la deuxième fois que vous l'exécuterez

     (notez cependant que si vous créez beaucoup de clusters avec des versions différentes, cela prendra de l'espace disque).

Une fois le cluster créé, vous pouvez le peupler de 3 nœuds avec :

    ccm populate -n 3


Après cela, exécutez :

    ccm start

Cela démarrera 3 nœuds avec respectivement les IP 127.0.0.[1, 2, 3] sur le port 9160 pour Thrift, 

le port 7000 pour la communication interne du cluster et les ports 7100, 7200 et 7300 pour JMX. 

Vous pouvez vérifier que le cluster est correctement configuré avec :

    ccm node1 ring

Vous pouvez ensuite démarrer un 4ème nœud avec :

    ccm add node4 -i 127.0.0.4 -j 7400 -b

(populate n'est qu'un raccourci pour ajouter plusieurs nœuds la 1ère fois)

ccm fournit un certain nombre de commodités, comme le vidage de tous les nœuds du cluster :

    ccm flush

ou un seul nœud :

    ccm node2 flush

Vous pouvez également consulter facilement le fichier de log d'un nœud donné avec :

    ccm node1 showlog

Enfin, vous pouvez vous débarrasser de l’ensemble du cluster (ce qui arrêtera le nœud et supprimera toutes les données) avec :

    ccm remove

La liste des autres commandes fournies est disponible via :

    ccm

Chaque commande est ensuite documentée via le paramètre `-h` (ou `--help`). 

Par exemple `ccm add -h` décrit les options pour `ccm add`


### Utilisation à distance (SSH/Paramiko)

Tous les exemples d'utilisation ci-dessus fonctionneront exactement de la même manière pour une machine configurée à distance ; 

cependant, des options à distance sont nécessaires pour établir une connexion à la machine distante avant d'exécuter les commandes CCM :

| Argument | Valeur | Description |
| :--- | :--- | :--- |
| --ssh-host | string | Nom d'hôte ou adresse IP à utiliser pour la connexion SSH |
| --ssh-port | int | Port à utiliser pour la connexion SSH<br/>Valeur par défaut : 22 |
| --ssh-username | string | Nom d'utilisateur à utiliser pour l'authentification par nom d'utilisateur/mot de passe ou par clé publique |
| --ssh-password | string | Mot de passe à utiliser pour le nom d'utilisateur/mot de passe ou phrase de passe de clé privée à l'aide de l'authentification par clé publique |
| --ssh-private-key | filename | Clé privée à utiliser pour la connexion SSH |

#### Précision :

Certaines commandes nécessitent que les fichiers soient situés sur le serveur distant. 

Ces commandes sont prétraitées, les transferts de fichiers sont initiés et des mises à jour sont apportées à la valeur de l'argument pour l'exécution à distance de la commande CCM :

| Parameter | Description |
| :--- | :--- |
| `--dse-credentials` | Copie le fichier d'informations d'identification DSE local sur un serveur distant |
| `--node-ssl` | Copie de manière récursive le répertoire SSL du nœud sur le serveur distant |
| `--ssl` | Copie récursivement le répertoire SSL sur un serveur distant |

#### Version courte

    ccm --ssh-host=192.168.33.11 --ssh-username=vagrant --ssh-password=vagrant create test -v 2.0.5 -n 3 -i 192.168.33.1 -s

__Remarque__: `-i` est utilisé pour ajouter un préfixe IP pendant le processus de création afin de garantir que les nœuds communiquent en utilisant l'adresse IP appropriée pour leur nœud

### Distribution des sources

Si vous souhaitez utiliser une distribution source au lieu du binaire par défaut à chaque fois (par exemple, pour l'intégration continue), 

vous pouvez préfixer la version cassandra avec `source:`, par example:

```
ccm create test -v source:2.0.5 -n 3 -s
```

### Gestion automatique de la version en cas d'anomalie :

Si 'binary:' or 'source:' ne sont pas explicitement spécifiés dans votre chaîne de version, 

alors ccm reviendra à la construction de la version demandée à partir de git s'il ne peut pas accéder aux miroirs Apache.


### Git et GitHub

Pour utiliser la dernière version du [canonical Apache Git repository](https://gitbox.apache.org/repos/asf?p=cassandra.git), utilisez le nom de la version `git:branch-name`, 

par exemple :

```
ccm create trunk -v git:trunk -n 5
```

et pour télécharger une branche à partir d'un fork GitHub de Cassandra, vous pouvez préfixer le référentiel et la branche avec :`github:`, 

par exemple :

```
ccm create patched -v github:jbellis/trunk -n 1
```

### Complétion de la ligne de commande Bash

ccm possède de nombreuses sous-commandes pour les commandes de cluster ainsi que pour les commandes de nœud, 

et parfois vous ne vous souvenez pas exactement du nom de la sous-commande que vous souhaitez appeler. 

De plus, les lignes de commande peuvent être longues en raison de noms de cluster ou de nœud longs.

Tirez parti de la fonctionnalité de complétion programmable de bash pour rendre l'utilisation de ccm plus agréable. 

Copiez `misc/ccm-completion.bash` quelque part dans votre répertoire personnel (ou /etc si vous souhaitez le rendre accessible à tous les utilisateurs de votre système) 

et sourcez-le dans votre  `.bash_profile`:

```
. ~/scripts/ccm-completion.bash
```

Une fois configuré,  `ccm sw<tab>` s'étend à `ccm switch `, par exemple. 

La sous-commande `switch` dispose d'une logique de complétion supplémentaire pour aider à compléter le nom du cluster. 

Ainsi,`ccm switch cl<tab>` s'étendrait à `ccm switch cluster-58` si le cluster-58 est le seul cluster dont le nom commence par « cl ». 

En cas d'ambiguïté, appuyez  `<tab>` une deuxième fois pour afficher les choix correspondants :

```
$ ccm switch cl<tab>
    ... becomes ...
$ ccm switch cluster-
    ... then hit tab twice ...
cluster-56  cluster-85  cluster-96
$ ccm switch cluster-8<tab>
    ... becomes ...
$ ccm switch cluster-85
```

Il détermine dynamiquement les sous-commandes disponibles en fonction du CCM invoqué. 

Ainsi, les utilisateurs exécutant plusieurs CCM (ou un CCM qu'ils mettent continuellement à jour avec de nouvelles commandes) fonctionneront automatiquement.

Le script de complétion s'appuie sur le fait que ccm possède deux sous-commandes cachées :

* show-cluster-cmds - émet les noms des sous-commandes du cluster.
* 
* show-node-cmds - émet les noms des sous-commandes du nœud.

Ainsi, cela ne fonctionnera pas avec des versions suffisamment anciennes de ccm.

Essai
-----------------------

Creation de l'environnement virtel :

    python3 -m venv ccm

`pip install` installe toutes les dépendances ainsi que`mock` et `pytest`. 

Lancer `pytest` à partir de la racine du référentiel pour exécuter les tests.

Débogage à distance
-----------------------

Si vous souhaitez vous connecter à vos nœuds Cassandra avec un débogueur distant, vous devez passer le paramètre `-d` (ou `--debug`) à la commande populate :

    ccm populate -d -n 3

Cela remplira 3 nœuds sur l'IP 127.0.0.[1, 2, 3] en configurant le débogage à distance sur les ports 2100, 2200 et 2300. 

Le thread principal ne sera pas suspendu, vous n'aurez donc pas besoin de vous connecter à un débogueur distant pour démarrer un nœud.

Vous pouvez également spécifier un port distant avec le paramètre `-r` (ou `--remote-debug-port`) lors de l'ajout d'un nœud :

    ccm add node4 -r 5005 -i 127.0.0.4 -j 7400 -b

Où les choses sont stockées :
-----------------------

Par défaut, ccm stocke toutes les données de nœud et les fichiers de configuration sous `~/.ccm/cluster_name/`.

Cela peut être remplacé en utilisant le paramètre `--config-dir` avec chaque commande.

DataStax Enterprise
-------------------

CCM 2.0 prend en charge la création et l'interaction avec les clusters DSE. 

L'option --dse doit être utilisée avec la commande `ccm create`. 

Voir `ccm create -h` pour obtenir de l'aide


CCM Lib
-------

Les fonctions ccm sont disponibles par programmation via ccmlib. 

Cela pourrait être utilisé pour implémenter des tests automatisés contre Cassandra. 

Voici un exemple simple d'utilisation de ccmlib :

    import ccmlib.cluster

    CLUSTER_PATH="."
    cluster = ccmlib.cluster.Cluster(CLUSTER_PATH, 'test', cassandra_version='2.1.14')
    cluster.populate(3).start()
    [node1, node2, node3] = cluster.nodelist()

    # do some tests on the cluster/nodes. To connect to a node through thrift,
    # the host and port to a node is available through
    #   node.network_interfaces['thrift']

    cluster.flush()
    node2.compact()

    # do some other tests

    # after the test, you can leave the cluster running, you can stop all nodes
    # using cluster.stop() but keep the data around (in CLUSTER_PATH/test), or
    # you can remove everything with cluster.remove()

--
Merci à Sylvain Lebresne <sylvain@datastax.com>
