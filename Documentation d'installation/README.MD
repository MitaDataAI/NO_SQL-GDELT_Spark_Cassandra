<hr style="border:3px solid black">

<h1 align="center">#Configuration de cluster#</h1>

<hr style="border:3px solid black">

La configuration de notre projet nécessite de configurer sur le server de l'école. On a **8** machines pour réaliser cette fonction:

- **2 Masters** ;

- **6 Slaves (ou Workers)**  qui sont utiliser pour stoquer des données;
  
  

Dans notre architecture, on décide d'utiliser logiciels: **Spark 2.3.2** et **Cassandra 3.11.3** ce qu'n a trouvé comptable. On a installé **3 Zookeepers** pour contrôler des **deux masters** et **1 worker** pour assurer la résilience de nos travailles 



Notre documentation est organisée dans l'ordre suivant:

1. Lancer le cluster et y accéder

2. Configuration de JAVA et python

3. Installation et configuration de Appache Cassandra sur des **6** Workers du cluster

4. Installation et configuration de Appache Spark sur **8** machines

5. Installation et configuration de Appache Zookeeper sur  **2** Masters et **1** Worker

6. Installation et configuration de Appache Zeppelin sur **2** Masters



****



<h2 align="left">1.Lancer le cluster et y accéder</h2>

### 1.1 Connecter cluster

L'école nous a distribué 8 VMs pour configurer. Pour connecter, taper code sur terminal (Lunix):

```bash
ssh ubuntu@ip_server && ssh tp-hadoop-(Numéro de machine)
```





### 1.2 Création de clé public

Cette étape pour l'interconnexion parmi des machines. 

```bash
ssh-keygen -t rsa -b 4096
```

Notez que la clé doit être créée dans un fichier (**.ssh**). Une fois créée, tapez (**cat id_rsa.pub**) dans le même répertoire de fichiers pour obtenir la clé publique, et copiez cette clé publique dans le fichier de clés publiques (**authorizedkey**) des **7** autres machines.

### 1.3 l'ajout de IPs

Afin de garantir une connexion fluide entre les machines à un stade ultérieur, vous devez copier l'adresse IP de chaque machine sous le fichier (**/home/ubuntu/etc/hosts**).

Pour vérifier que la connexion a réussi, vous pouvez saisir le code :

```bash
ssh ubuntu@(ip de machine)
```

<hr style="border:3px solid black">

<h2 align="left">2.Configuration de JAVA et Python</h2>
