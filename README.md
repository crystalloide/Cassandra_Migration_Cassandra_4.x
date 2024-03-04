**********************************************************************************************************************************
# Migration de Cassandra 3.11.x vers 4.x
**********************************************************************************************************************************
## Principe et réalisation d'une migration d'un cluster Apache Cassandra de la version 3.11.x vers la version 4.x :
**********************************************************************************************************************************

Pour simplifier ici, on effectuera une migration d'un cluster *à nœud unique*. 

Les clusters de production Cassandra ont toujours plusieurs nœuds. 

Par conséquent, les étapes comportent des notes décrivant le travail *supplémentaire* nécessaire à la mise à niveau des clusters *multi-nœuds*.

**********************************************************************************************************************************
## Dans ce TP, nous allons :

- Installer puis démarrer un cluster Cassandra 3.x à nœud unique
- Créer un schéma et y insérer des données
- Préparer le nœud pour la migration
- Installer Cassandra 4.x
- Démarrer Cassandra 4.x
- Vérifier que le nœud a été migré avec succès

_ Pour aller plus loin : [https://www.datastax.com/learn/whats-new-for-cassandra-4](https://www.datastax.com/learn/whats-new-for-cassandra-4) _

**********************************************************************************************************************************

## Rappel pour retrouver les environnements éventuellement précédemment instanciés dans Gitpod : [ https://gitpod.io/workspaces ](https://gitpod.io/workspaces)

**********************************************************************************************************************************

## Pour lancer notre TP  (environnement sans cassandra 3.x préinstallé)  :

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/crystalloide/Cassandra_Migration_Cassandra_4.x)

**********************************************************************************************************************************

✅ Installation de la version 8 de Java pour cassandra 3.11.x : 

    sdk install java 8.0.345.fx-zulu

Remarque : faire "Y" pour que cette version devienne celle par défaut.


✅ Vérification de la version de Java : 

    java -version
    

✅ Installation de la version 8 de Java pour cassandra 3.11.x : 

    which python2 
    
    sudo apt-get install python2

    python2 -V
    

✅ Mise à jour du Path : 
   
    echo $PATH
    
    export PATH="/home/gitpod/.pyenv/bin:$PATH"
    export VIRTUALENVWRAPPER_PYTHON=/usr/bin/python2.7
    export PATH="$GITPOD_REPO_ROOT/capache-cassandra-3.11.16/bin:$PATH"
    export PATH="$GITPOD_REPO_ROOT/apache-cassandra-3.11.16/tools/bin:$PATH"

    echo $PATH 


✅ Installation de cassandra 3.11.x : 

Téléchargement du tarball avec les binaires sur l'un des sites miroirs de la fondation Apache : 

    curl -OL https://downloads.apache.org/cassandra/3.11.16/apache-cassandra-3.11.16-bin.tar.gz

    
Vérification de l'intégrité du tarball ainsi téléchargé avec le hash en utilisant GPG : 

    gpg --print-md SHA256 apache-cassandra-3.11.16-bin.tar.gz
    
Comparaison de la signature du fichier Tarball avec le contenu du fichier SHA256 récupéré en ligne :

    curl -L https://downloads.apache.org/cassandra/3.11.16/apache-cassandra-3.11.16-bin.tar.gz.sha256


Décompression du tarball:

    tar xzvf apache-cassandra-3.11.16-bin.tar.gz
    
Les fichiers sont extraits dans le répertoire apache-cassandra-3.11.16/

Cela correspond donc au "tarball installation location".

Dans ce répertoire d'installation, on retrouve les sous-répertoires contenant : les scripts, binaires, utilitaires, fichiers de configuration, data et fichiers de log :

    <tarball_installation>/
    bin/		
    conf/		
    data/		
    doc/
    interface/
    javadoc/
    lib/
    logs/		
    pylib/
    tools/		
    
    location of the commands to run cassandra, cqlsh, nodetool, and SSTable tools
    location of cassandra.yaml and other configuration files
    location of the commit logs, hints, and SSTables
    location of system and debug logs <5>location of cassandra-stress tool
   


✅ Lancement de Cassandra : 

    cd /workspace/Cassandra_Migration_Cassandra_4.x/apache-cassandra-3.11.16
    
    bin/cassandra
    
Cela lancera Cassandra comme user Linux authentifié.

✅ Suivi du lancement de Cassandra :

    tail -f tail -f /workspace/Cassandra_Migration_Cassandra_4.x/apache-cassandra-3.11.16/logs/system.log
    

✅ Vérification que le noeud Cassandra 3.x est bien lancé :

    bin/nodetool status


✅ Vérification de la version de Cassandra (3.11.x attendu):

    bin/nodetool version


✅ Affichage en retour : 

    ReleaseVersion: 3.11.16


✅ Vérification que le noeud Cassandra 3.11.x est bien opérationnel : 

    bin/nodetool status

✅ Affichage en retour : 

    Datacenter: datacenter1
    =======================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
    UN  127.0.0.1  70.87 KiB  256          100.0%            b10d4523-769e-41d4-aaaf-721ed30f4a22  rack1
    

### Création d'un keyspace et d'une table, puis insertion de données :

✅ On installa CQLSH :

    pip install -U cqlsh

    
✅ On lance une session de shell CQL :

    bin/cqlsh


✅ On créée ensuite le keyspace:

    CREATE KEYSPACE united_states 
    WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

    USE united_states;


✅ On créée enfin la table :

    CREATE TABLE cities_by_state (
    state text,
    name text,
    population int,
    PRIMARY KEY ((state), name)
    );

✅ On insère les données sur les 10 plus importantes villes par état des US en terme de population :

    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('New York','New York City',8622357);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('California','Los Angeles',4085014);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('Illinois','Chicago',2670406);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('Texas','Houston',2378146);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('Arizona','Phoenix',1743469);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('Pennsylvania','Philadelphia',1590402);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('Texas','San Antonio',1579504);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('California','San Diego',1469490);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('Texas','Dallas',1400337);
    INSERT INTO cities_by_state (state, name, population) 
      VALUES ('California','San Jose',1036242);

✅ On vérifie que les données ont bien été chargées :

    SELECT * FROM cities_by_state;


✅ Affichage en retour : 

    cqlsh:united_states> SELECT * FROM cities_by_state;
    
     state        | name          | population
    --------------+---------------+------------
     Pennsylvania |  Philadelphia |    1590402
     New York | New York City |    8622357
     Arizona |       Phoenix |    1743469
     Illinois |       Chicago |    2670406
     California |   Los Angeles |    4085014
     California |     San Diego |    1469490
     California |      San Jose |    1036242
     Texas |        Dallas |    1400337
     Texas |       Houston |    2378146
     Texas |   San Antonio |    1579504

    (10 rows)


✅ On extrait les données des villes situées en California:

    SELECT * FROM cities_by_state WHERE state = 'California';


✅ Affichage en retour : 

    cqlsh:united_states> SELECT * FROM cities_by_state WHERE state = 'California';
    
     state      | name        | population
    ------------+-------------+------------
     California | Los Angeles |    4085014
     California |   San Diego |    1469490
     California |    San Jose |    1036242
    
    (3 rows)


✅ On sort du shell CQL et on efface l'écran :

    exit
    clear


✅ Vérification que le noeud est prêt pour la montée de version :

## Il y a 9 points à regarder pour s'assurer que tout est prêt : 

### 1. Statut actuel des noeuds : 
   
✅ Vérification que tous les noeuds sont à l'état "UN" (Up et Normal) :

    nodetool status 

    
✅ Affichage en retour : 

    Datacenter: datacenter1
    =======================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
    UN  127.0.0.1  93.26 KiB  256          100.0%            b10d4523-769e-41d4-aaaf-721ed30f4a22  rack1
    

### 2. Espace disque : 

✅ Vérification que chaque noeud a au moins 50% d'espace libre :

    df -h

✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ df -h
    Filesystem                                Size  Used Avail Use% Mounted on
    /.workspace/mark                          1.4T  150G  1.3T  11% /
    tmpfs                                      64M     0   64M   0% /dev
    /dev/sda5                                 442G  9.0G  433G   3% /dev/termination-log
    shm                                        64M     0   64M   0% /dev/shm
    tmpfs                                      32G     0   32G   0% /sys/firmware
    /dev/mapper/lvm--disk-gitpod--workspaces   30G  263M   30G   1% /workspace
    /dev/mapper/lvm--disk-containerd--mount   1.4T  150G  1.3T  11% /etc/hostname
    tmpfs                                      32G  4.3M   32G   1% /tmp
    tmpfs                                      32G     0   32G   0% /proc/acpi
    tmpfs                                      64M     0   64M   0% /proc/keys
    tmpfs                                      32G     0   32G   0% /proc/scsi
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ 



### 3. Absence d'erreur dans les logs : 

✅ Vérification qu'il n'y a pas d'erreurs non traitées (et aussi de warnings) :

    grep -e "WARN" -e "ERROR" cassandra3/logs/system.log
    
    
✅ Affichage en retour : 

    WARN  [main] 2023-04-30 09:23:34,134 DatabaseDescriptor.java:503 - Small commitlog volume detected at cassandra3/bin/../data/commitlog; setting commitlog_total_space_in_mb to 7680.  You can override this in cassandra.yaml
    WARN  [main] 2023-04-30 09:23:34,135 DatabaseDescriptor.java:530 - Small cdc volume detected at cassandra3/bin/../data/cdc_raw; setting cdc_total_space_in_mb to 3840.  You can override this in cassandra.yaml
    WARN  [main] 2023-04-30 09:23:34,657 DatabaseDescriptor.java:579 - Only 29.747GiB free across all data volumes. Consider adding more capacity to your cluster or removing obsolete snapshots
    WARN  [main] 2023-04-30 09:23:35,496 NativeLibrary.java:189 - Unable to lock JVM memory (ENOMEM). This can result in part of the JVM being swapped out, especially with mmapped I/O enabled. Increase RLIMIT_MEMLOCK or run Cassandra as root.
    WARN  [main] 2023-04-30 09:23:35,497 StartupChecks.java:136 - jemalloc shared library could not be preloaded to speed up memory allocations
    WARN  [main] 2023-04-30 09:23:35,497 StartupChecks.java:169 - JMX is not enabled to receive remote connections. Please see cassandra-env.sh for more info.
    WARN  [main] 2023-04-30 09:23:35,581 SigarLibrary.java:174 - Cassandra server running in degraded mode. Is swap disabled? : false,  Address space adequate? : true,  nofile limit adequate? : true, nproc limit adequate? : true 
    WARN  [main] 2023-04-30 09:23:35,583 StartupChecks.java:311 - Maximum number of memory map areas per process (vm.max_map_count) 65530 is too low, recommended value: 1048575, you can change it with sysctl.
    WARN  [main] 2023-04-30 09:23:35,610 StartupChecks.java:332 - Directory cassandra3/bin/../data/data doesn't exist
    WARN  [main] 2023-04-30 09:23:35,621 StartupChecks.java:332 - Directory cassandra3/bin/../data/commitlog doesn't exist
    WARN  [main] 2023-04-30 09:23:35,621 StartupChecks.java:332 - Directory cassandra3/bin/../data/saved_caches doesn't exist
    WARN  [main] 2023-04-30 09:23:35,622 StartupChecks.java:332 - Directory /workspace/cassandra4-migrating-from-cassandra3/cassandra3/bin/../data/hints doesn't exist
    WARN  [main] 2023-04-30 09:23:45,426 SystemKeyspace.java:1142 - No host ID found, created b10d4523-769e-41d4-aaaf-721ed30f4a22 (Note: This should happen exactly once per node).



### 4. Etat du Gossiping :

✅ Vérification que toutes les informations listées dans le report Gossipingont un statut normal (STATUS:NORMAL)

    nodetool gossipinfo


✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool gossipinfo
    localhost/127.0.0.1
      generation:1682846625
      heartbeat:591
      STATUS:18:NORMAL,-1130954909809462762
      LOAD:584:95503.0
      SCHEMA:369:b004a9c9-d03f-3acf-9260-41c79c3b3593
      DC:7:datacenter1
      RACK:9:rack1
      RELEASE_VERSION:4:3.11.14
      RPC_ADDRESS:3:127.0.0.1
      NET_VERSION:1:11
      HOST_ID:2:b10d4523-769e-41d4-aaaf-721ed30f4a22
      RPC_READY:20:true
      SSTABLE_VERSIONS:5:big-me
      TOKENS:17:<hidden>



✅ On regarde donc s'il n'y a pas un autre statut présent que "NORMAL" :

    nodetool gossipinfo | grep STATUS | grep -v NORMAL

    
✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool gossipinfo | grep STATUS | grep -v NORMAL
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ 


### 5. Vérification de l'absence de messages "Dropped" récemment : 

✅ On regarde pour les 72 heures passées :

    nodetool tpstats | grep -A 12 Dropped

    
✅ Affichage en retour : 

    Message type           Dropped
    READ                         0
    RANGE_SLICE                  0
    _TRACE                       0
    HINT                         0
    MUTATION                     0
    COUNTER_MUTATION             0
    BATCH_STORE                  0
    BATCH_REMOVE                 0
    REQUEST_RESPONSE             0
    PAGED_RANGE                  0
    READ_REPAIR                  0


### 6. Vérification que les Backups ont bie nété désactivés : 

On vérifie que tous les backups automatiques sont désactivés.

Ceci englobe Medusa et n'importe quel script appelant "nodetool snapshot" tant que la montée de version n'est pas terminée.


### 7. Vérification que les "Repair" sont désactivés :

On vérifie que tous les repairs sont désactivés.

Ceci englobe les repairs automatiques définis dans Reaper.


### 8. Niveau de performance / Monitoring :

La montée de version se traduit par une baisse temporaire de la performance, du fait que cela se traduit par une succession d'arrêt temporaire de chacun des noeuds. 

Il est donc ncessaire d'anticiper l'impact de la montée de version (de type "rolling upgrade") sur les performances du systeme, avant et après la migration.


### 9. Disponibilité : 

Vérification que les applications qui nécessitent une forte cohérence utilisent bien le niveau de cohérence (CL) à LOCAL_QUORUM et le facteur de réplication de 3.

Lorsque LOCAL_QUORUM est utilisé avec un facteur de réplication inférieur à 3, tous les réplicas doivent être disponibles pour que les requêtes aboutissent.

Un redémarrage progressif utilisant cette configuration entraînerait une indisponibilité totale ou partielle lorsqu'un nœud est DOWN.


## Préparation du nœud pour la migration :


✅ Prise d'un snapshot du noeud qui sera utilisé en cas de rollback suite à la migration :

   Rappel : "nodetool snapshot" va flusher les memtables sur dique

    nodetool snapshot

    
✅ Affichage en retour : 

    Requested creating snapshot(s) for [all keyspaces] with snapshot name [1682847408083] and options {skipFlush=false}
    Snapshot directory: 1682847408083


✅ Arrêt du noeud en recherchant son PID puis en passant la commande "kill" :

    pgrep -u gitpod -f cassandra | xargs kill -9


✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ pgrep -u gitpod -f cassandra | xargs kill -9


✅ Utilisation de la commande "nodetool" pour vérifier que le noeud a bien été arrêté : 

    nodetool status
    
✅ Affichage en retour : 

    nodetool: Failed to connect to '127.0.0.1:7199' - ConnectException: 'Connection refused (Connection refused)'.


## Installation de Cassandra 4.1.x :


✅ Download and install Cassandra 4.1.x:

    wget https://dlcdn.apache.org/cassandra/4.1.4/apache-cassandra-4.1.4-bin.tar.gz

    tar -xzf apache-cassandra-4.1.4-bin.tar.gz

    rm apache-cassandra-4.1.4-bin.tar.gz

    mv apache-cassandra-4.1.4 cassandra4

✅ Affichage en retour : 


    Connecting to archive.apache.org (archive.apache.org)|138.201.131.134|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 49653805 (47M) [application/x-gzip]
    Saving to: ‘apache-cassandra-4.1.4-bin.tar.gz’

    apache-cassandra-4.0.5-bi 100%[=====================================>]  47.35M  51.8MB/s    in 0.9s    

    2023-04-30 09:39:25 (51.8 MB/s) - ‘apache-cassandra-4.1.4-bin.tar.gz’ saved [49653805/49653805]

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ 
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ tar -xzf apache-cassandra-4.1.4-bin.tar.gz
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ 
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ rm apache-cassandra-4.1.4-bin.tar.gz
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ 
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ mv apache-cassandra-4.1.4 cassandra4
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ 




✅ Update the PATH variable:

    export PATH="$GITPOD_REPO_ROOT/cassandra4/bin:$PATH"
    export PATH="$GITPOD_REPO_ROOT/cassandra3/tools/bin:$PATH"
    
✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ export PATH="$GITPOD_REPO_ROOT/cassandra4/bin:$PATH"
    /bin:$PATH"
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ export PATH="$GITPOD_REPO_ROOT/cassandra3/tools/bin:$PATH"

## Lancement de Cassandra 4.x :

In this step, you will configure cassandra.yaml and start Cassandra 4.x.

✅ Change the number of virtual nodes to 256 since the 3.x node had 256 and the 4.x node is set to 16 by default. 

Set num_tokens to 256 in Cassandra 4.x:

    sed -i 's/num_tokens: 16/num_tokens: 256/' cassandra4/conf/cassandra.yaml


✅ Point the Cassandra 4.x node to the Cassandra 3.x node data files:

    sed -i 's|# data_file_directories:|data_file_directories:|' cassandra4/conf/cassandra.yaml
    
    sed -i "s|#     - /var/lib/cassandra/data|    - $GITPOD_REPO_ROOT/cassandra3/data/data|" cassandra4/conf/cassandra.yaml



✅ Start the Cassandra 4.x node:

    cassandra

Look for the state "jump to NORMAL" message to indicate that the node is running.



✅ Clear the screen :

    clear

✅ Affichage du statut du noeud en cassandra 4.x :

    nodetool status

✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool status
    Datacenter: datacenter1
    =======================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address    Load        Tokens  Owns (effective)  Host ID                               Rack 
    UN  127.0.0.1  161.73 KiB  256     100.0%            b10d4523-769e-41d4-aaaf-721ed30f4a22  rack1


## Verify that the node has been successfully migrated

In this step, you will verify that the Cassandra node has been upgraded and that the data is still available.

✅ Verify that the Cassandra version is 4.x:

    nodetool version
    
✅ Affichage en retour : 

    nodetool version
    ReleaseVersion: 4.0.5


✅ Verify that the node is in the UP and NORMAL (UN) state:

    nodetool status

✅ Affichage en retour : 

    Datacenter: datacenter1
    =======================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address    Load        Tokens  Owns (effective)  Host ID                               Rack 
    UN  127.0.0.1  161.73 KiB  256     100.0%            b10d4523-769e-41d4-aaaf-721ed30f4a22  rack1

    
✅ Verify that there are no errors:

    grep -e "WARN" -e "ERROR" cassandra4/logs/system.log


✅ Start the CQL shell:

    cqlsh

✅ Use the keyspace:

    USE united_states;

✅ Verify that the data is accessible:

    SELECT * FROM cities_by_state;

## If you can see the data, you have successfullly upgraded from Cassandra 3.x to 4.x!

✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ cqlsh
    Connected to Test Cluster at 127.0.0.1:9042
    [cqlsh 6.0.0 | Cassandra 4.0.5 | CQL spec 3.4.5 | Native protocol v5]
    Use HELP for help.
    cqlsh> USE united_states;
    cqlsh:united_states> SELECT * FROM cities_by_state;
    
     state        | name          | population
    --------------+---------------+------------
     Pennsylvania |  Philadelphia |    1590402
     New York     | New York City |    8622357
     Arizona      |       Phoenix |    1743469
     Illinois     |       Chicago |    2670406
     California   |   Los Angeles |    4085014
     California   |     San Diego |    1469490
     California   |      San Jose |    1036242
     Texas        |        Dallas |    1400337
     Texas        |       Houston |    2378146
     Texas        |   San Antonio |    1579504

    (10 rows)
    cqlsh:united_states> 


**********************************************************************************************************************************
## Fin du TP - Migration du cluster 3.11.x en cluster 4.x
**********************************************************************************************************************************
