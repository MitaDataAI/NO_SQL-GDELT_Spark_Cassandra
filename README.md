# Analyse des données NoSQL de GDELT
![image](https://github.com/user-attachments/assets/44604982-9484-47f5-bf76-42d21fbb1650)

****
Le projet GDELT est une base de données mondiale qui collecte, analyse et met à jour en temps réel les événements médiatiques mondiaux issus de millions de sources d’information dans plus de 100 langues.

****
# Les échantillons de GDELT travaillés
On se concentre sur deux bases de données :

## 1. **GDELT Events Database**  
   - Contient les **événements géopolitiques** identifiés dans l’actualité (protestations, conflits, accords, etc.).
   - Champs utilisés :
     - `GlobalEventId`
     - `ActionGeo_CountryCode` (pays de l'événement)
     - `EventDate`, `MonthYear`, `Year`
     - `MentionTranslationInfo` (langue de l’article)
     - `MentionTimeDate`
   - Utilisation :
     - Suivi des événements par pays, date et langue
     - Analyse de relations bilatérales (ex : France – Chine)


## 2. **Global Knowledge Graph (GKG)**  
   - Extrait des **entités, thèmes et sentiments** des articles d’actualité.
   - Champs utilisés :
     - `GkgRecordId`
     - `SourceCommonName` (nom du média)
     - `Themes`, `Persons`, `Locations`
     - `AverageTone` (tonalité moyenne)
     - `Date`, `MonthYear`, `Year`
   - Utilisation :
     - Analyse du contenu médiatique par source
     - Identification des sujets et personnalités abordés
     - Calcul du ton moyen des articles
---
****



<img title="" src="objectif.png" alt="96741f3f-6a94-4881-afc8-8fe3203cbdd1" data-align="center" style="zoom:200%;">

***
# Objectif du point de vue informatique 
- Mettre en œuvre une architecture Big Data distribuée, résiliente et performante.
- Concevoir un système de stockage :
  - **Distribué**
  - **Résilient** (capable de résister à la panne d’un nœud)
  - **Performant** (rapide et robuste)
- Répondre à une problématique concrète :
  - Volume de données initiales (GDELT) : **> 100 Go**
  - Données à stocker dans Cassandra : **> 5 Go**
  - Réalisation de **4 requêtes analytiques spécifiques**

# Pipeline de Données

- **Téléchargement** depuis GDELT (~15 Go)
- **Filtrage & transformation** via Spark
- **Stockage final** dans Cassandra (~1.6 Go)
- **Suppression des données brutes**
- **Requêtes analytiques** via Zeppelin

***
# Architecture du cluster 
- **Spark** : Mode *HA* (High Availability) avec 2 Masters + Zookeeper
- **Cassandra** : Stockage distribué sur les 6 Workers
- **Zeppelin** : Installé sur les 2 Masters comme interface d'analyse
- **Zookeeper** : 3 instances (2 Masters + 1 Worker) pour tolérance aux pannes

# Démarche de configuration brièvement
## 1. Connexion au cluster

- Connexion SSH aux 8 machines fournies par l’école :
```bash
ssh ubuntu@ip_server && ssh tp-hadoop-XX
```

- Création de clés SSH pour une interconnexion sans mot de passe
- Ajout des IPs dans le fichier `/etc/hosts` pour communication fluide
---

## 2. Installation des dépendances
#### Java 8
```bash
sudo apt-get install openjdk-8-jdk
```
Ajout de la variable d’environnement :
```bash
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64/jre
```

#### Python
```bash
sudo apt install python
```

---
## 3. Apache Cassandra (v3.11.3)

- Installé sur les **6 Workers**
- Configuration du fichier `cassandra.yaml` :
  - `cluster_name`, `seed_provider`, `listen_address`, `rpc_address`
  - Distribution automatisée avec `scp` et script shell

- Lancement :
```bash
cd apache-cassandra-3.11.3 && bin/cassandra
nodetool status
```

---

## 4. Apache Spark (v2.3.2)

- Installé sur les **8 machines**
- Mode de déploiement : **Standalone** + configuration **HA** avec Zookeeper
- Fichiers configurés :
  - `spark-env.sh`, `spark-defaults.conf`, `slaves`
  - Connexion automatique à Cassandra via Spark-Cassandra Connector

- Lancement :
```bash
cd spark-2.3.2-bin-hadoop2.7/sbin && ./start-all.sh
```

---

## 5. Apache Zookeeper (v3.5.6)

- Installé sur **3 machines** (2 Masters + 1 Worker)
- Création des dossiers `data/` et `logs/`, configuration du fichier `zoo.cfg`
- Fichier `myid` unique sur chaque nœud
- Intégration dans `spark-env.sh` pour activer la **HA Spark** :
```bash
export SPARK_DAEMON_JAVA_OPTS="... -Dspark.deploy.recoveryMode=ZOOKEEPER ..."
```

---

## 6. Apache Zeppelin (v0.8.2)

- Installé sur les **2 Masters**
- Configuration des fichiers :
  - `zeppelin-env.sh` → Port personnalisé (ex: 9090)
  - `zeppelin-site.xml` → Adresse et port du serveur Zeppelin

- Lancement :
```bash
cd zeppelin-0.8.2-bin-all/bin && ./zeppelin-daemon.sh start
```
- Accessible via navigateur : `http://tp-hadoop-XX:9090`

---

## 🧪 Vérification finale

- Accès Spark UI : `http://<master-ip>:8080` (ou `38080` en mode HA)
- Accès Zeppelin : `http://<zeppelin-ip>:9090`
- Cassandra : vérification des nœuds via `nodetool status`
---

## ✅ Résumé

- ✅ Infrastructure **HA** avec Spark + Zookeeper
- ✅ Stockage distribué et résilient via Cassandra
- ✅ Requêtes interactives via Zeppelin
- ✅ Scripts d’installation et configuration automatisés







