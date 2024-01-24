<hr style="border:3px solid black">

<h1 align="center">#Configuration de cluster#</h1>

<hr style="border:3px solid black">

La configuration de notre projet nécessite de configurer sur le server de l'école. On a **8** machines pour réaliser cette fonction:

- **2 Masters** ;

- **6 Slaves (ou Workers)**  qui sont utiliser pour stoquer des données;
  
  

Dans notre architecture, on décide d'utiliser logiciels: **Spark 2.3.2** et **Cassandra 3.11.3** ce qu'n a trouvé compatible. On a installé **3 Zookeepers** pour contrôler des **deux masters** et **1 worker** pour assurer la résilience de nos travailles 



Notre documentation est organisée dans l'ordre suivant:

1. Lancer le cluster et y accéder

2. Configuration de JAVA et python

3. Installation et configuration de Appache Cassandra sur des **6** Workers du cluster

4. Installation et configuration de Appache Spark sur **8** machines

5. Installation et configuration de Appache Zookeeper sur  **2** Masters et **1** Worker

6. Installation et configuration de Appache Zeppelin sur **2** Masters
   
   

****



<h2 align="left">1.Lancer le cluster et y accéder</h2>

#### 1.1 Connecter cluster

L'école nous a distribué 8 VMs pour configurer. Pour connecter, taper code sur terminal (Lunix):

```bash
ssh ubuntu@ip_server && ssh tp-hadoop-(Numéro de machine)
```



#### 1.2 Création de clé public

Cette étape pour l'interconnexion parmi des machines. 

```bash
ssh-keygen -t rsa -b 4096
```

Notez que la clé doit être créée dans un fichier (**.ssh**). Une fois créée, tapez (**cat id_rsa.pub**) dans le même répertoire de fichiers pour obtenir la clé publique, et copiez cette clé publique dans le fichier de clés publiques (**authorizedkey**) des **7** autres machines.



#### 1.3 l'ajout de IPs

Afin de garantir une connexion fluide entre les machines à un stade ultérieur, vous devez copier l'adresse IP de chaque machine sous le fichier (**/home/ubuntu/etc/hosts**).

Pour vérifier que la connexion a réussi, vous pouvez saisir le code :

```bash
ssh ubuntu@(ip de machine)
```



****

<h2 align="left">2.Configuration de JAVA et Python</h2>

#### 2.1 Installer Java

L'installation de Java est cruciale. Il s'agit de l'environnement de travail du logiciel pour ce projet. Avant l'installation, il faut vérifier comptabilité des logiciels. Pous nous, on a choisi **java 8**:

```bash
sudo apt-get update && sudo apt-get upgrade && sudo apt-get install openjdk-8-jdk
```

Les deux premières commandes sont des commandes de mise à jour du système. Si le système est nouveaux, vous n'avez pas besoin d'exécuter.

Ensuite, vous devez ajouter le chemin d'accès à java à la variable d'environnement(**.barshrc**).

```bash
java -XshowSettings:properties -version # properties java
```

Le code ci-dessus permet de trouver le chemin d'accès à l'installation de Java.

```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
```

 Après l'avoir ajouté à la variable d'environnement, vous pouvez le vérifier en exécutant (**echo $$JAVA_HOME**).

#### 2.2 Installer Python

Python peut être installé après l'installation de cassandra, car il nécessite des logiciels compatibles. S'il n'y a pas d'exigence de compatibilité, vous pouvez l'indiquer :

```bash
sudo apt install python
```



****

<h2 align="left">3.Installation et configuration de Cassandra</h2>

#### 3.1 Installer Cassandra

Encore une fois, un rappel:vous devez vous assurer de la compatibilité des logiciels avant de les installer, faute de quoi vous passerez beaucoup de temps à chercher une solution au problème. Pour nous,  on a choisi **Cassandra 3.11.3** , dans le site de [site Apache Cassandra](http://www.apache.org/dyn/closer.lua/cassandra/3.11.3/) :

```bash
cd /home/ubuntu/ && wget http://archive.apache.org/dist/cassandra/3.11.3/apache-cassandra-3.11.3-bin.tar.gz && tar -xzvf apache-cassandra-3.11.3-bin.tar.gz
```

Si besoin, vous pouvez supprimer le fichier de gz :

```bash
rm -r apache-cassandra-3.11.3-bin.tar.gz
```

Ensuite, vous devez ajouter le chemin d'cassandra à la variable d'environnement(**.barshrc**)

```bash
export PATH="$PATH:/home/ubuntu/apache-cassandra-3.11.3/bin"
```

#### 3.2 Configurer cassandra fichier <u>Cassandra.ymal</u>

  Pour activer la mise en cluster entre les nœuds, il faut configurer fichier(**<u>Cassandra.ymal</u>**). Dans le fichier, nous devons modifier des lignes suivant:

- **cluster_name** : nom de cluster(par defauct "Test Cluster")

- **seed_provider** : IPs de machines qui doivent faire le cluster

- **listen_address** : IP de machine actuelle qui laisse cluster trouve le noeud.

- **rpc_address** : 0.0.0.0 (ici, on peut mettre l'ip de machine actuelle, mais pour les connexions de cassandra, il faut ajouter ip après **cqlsh**)

- **broadcast_rpc_address** : 1.2.3.4(quand rpc_address est 0.0.0.0, il faut ajouter ce lignes, sinon, commenter cette ligne)

```
cluster_name: 'Test Cluster'

seed_provider:
    # Addresses of hosts that are deemed contact points. 
    # Cassandra nodes use this list of hosts to find each other and learn
    # the topology of the ring.  You must change this if you are running
    # multiple nodes!
    - class_name: org.apache.cassandra.locator.SimpleSeedProvider
      parameters:
          # seeds is actually a comma-delimited list of addresses.
          # Ex: "<ip1>,<ip2>,<ip3>"
          - seeds: "tp-hadoop-17,tp-hadoop-18,tp-hadoop-19,tp-hadoop-20,tp-hadoop-23,tp-hadoop-24"

listen_address: tp-hadoop-17
rpc_address: 0.0.0.0
broadcast_rpc_addres:1.2.3.4
```

Lorsque nous avons fini de configurer une machine, nous pouvons utiliser la commande **scp** pour la copier sur une autre machine.

```bash
    scp /home/ubuntu/apache-cassandra-3.11.3/conf/cassandra.yaml ubuntu@$node:/home/ubuntu/apache-cassandra-3.11.3/conf/
```

Grâce à la connexion entre les machines, nous pouvons écrire des fichiers **sh** pour un déploiement en un clic.

```bash
!/bin/bash
# Liste des noms d'hôtes ou adresses IP des nœuds
nodes=("ip_1" "ip_2" "ip_3" "ip_4","ip_5")

for node in "${nodes[@]}"; do
    # Supprimer l'ancien fichier cassandra.yaml
    ssh ubuntu@$node "rm /home/ubuntu/apache-cassandra-3.11.3/conf/cassandra.yaml"

    # Copier le fichier de base cassandra.yaml vers le nœud
    scp /home/ubuntu/apache-cassandra-3.11.3/conf/cassandra.yaml ubuntu@$node:/home/ubuntu/apache-cassandra-3.11.3/conf/

    # Modifier le fichier cassandra.yaml sur le nœud
    ssh ubuntu@$node "sed -i 's/listen_address:.*/listen_address: $node/' /home/ubuntu//home/ubuntu/apache-cassandra-3.11.3/conf/cassandra.yaml

done

```



****

<h2 align="left">4.Installation et configuration de Spark</h2>

 Un rappel, les étapes suivant pour réaliser la fonction cluster :

1. Installer Apache-SparK sur les Masters et Workers

2. Configurer le fichier **spark-env.sh**

3. Configurer le fichier **spark-default.conf**

4. Ajouter les dépendances pour connecter Spark et Cassandra

5. Lancer les Masters et les Workers

#### 4.1 Installer Spark

  Pour installation, on a choisi **Spark 2.3.2** , Vous pouvez chercher des vesion qui est compatible sur le [site](https://archive.apache.org/dist/spark/spark-2.3.2/) .

```bash
cd /home/ubuntu && wget https://archive.apache.org/dist/spark/spark-2.3.2/spark-2.3.2-bin-hadoop2.7.tgz && tar -xzvf spark-2.3.2-bin-hadoop2.7.tgz && rm spark-2.3.2-bin-hadoop2.7.tgz
```

Ensuite, vous devez ajouter le chemin d'Spark à la variable d'environnement(**.barshrc**)

```bash
export SPARK_HOME=/home/ubuntu/spark-2.3.2-bin-hadoop2.7
export PATH=$PATH:$SPARK_HOME/bin
```

#### 4.2 Configurer le fichier *<u>spark-env.sh</u>*

Accéder au dossier(**/home/ubuntu/spark-2.3.2-bin-hadoop2.7/conf**), Copier `spark-env.sh.template` et `spark-defaults.conf.template` en `spark-env.sh` et `spark-defaults.conf` . 

```bash
cd /home/ubunut/spark-2.3.2-bin-hadoop2.7/conf && cp spark-env.sh.template spark-env.sh && cp spark-defaults.conf.template spark-defaults.conf
```

Sur masters:

```bash
export SPARK_LOCAL_IP=IP De master
export SPARK_MASTER_HOST=IP De master
export SPARK_MASTER_OPTS="-Dspark.deploy.recoveryMode=ZOOKEEPER -Dspark.deploy.zookeeper.url=tp-ha>
```

Sur workers:

```bash
export SPARK_LOCAL_IP=ip de worker
export SPARK_MASTER_HOST=IPs de masters
```



#### 4.3 Configurer le fichier *<u>spark-defaults.conf</u>*

Sur des machines:

```bash
spark.master                        spark://tp-hadoop-21:7077,tp-hadoop-22:7077
spark.jars                          /home/ubuntu/spark-2.3.2-bin-hadoop2.7/jars/spark-cassandra-connector_2.11-2.4.0.jar
spark.cassandra.connection.host     tp-hadoop-17,tp-hadoop-18,tp-hadoop-19,tp-hadoop-20,tp-hadoop-23,tp-hadoop-24
```

On peut résumer:

- `spark.master` : spark://PRIVATE_DNS_MASTER1:7077,PRIVATE_DNS_MASTER2:7077

- `spark.jars` : spark-cassandra-connector_2.11-2.4.0

- `spark.cassandra.connection.host` : IPs de workers
  
  

#### 4.4 Installer Spark-cassandra-connector

```bash
cd /home/ubuntu/spark-2.3.2-bin-hadoop2.7/jars && wget https://repo1.maven.org/maven2/com/twitter/jsr166e/1.1.0/jsr166e-1.1.0.jar && wget https://repo1.maven.org/maven2/com/datastax/spark/spark-cassandra-connector_2.11/2.4.0/spark-cassandra-connector_2.11-2.4.0.jar
```


