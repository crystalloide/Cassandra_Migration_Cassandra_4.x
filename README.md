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

- Installer (option1) ou Démarrer (option2) un cluster Cassandra 3.x à nœud unique
- Créer un schéma et y insérer des données
- Préparer le nœud pour la migration
- Installer Cassandra 4.x
- Démarrer Cassandra 4.x
- Vérifier que le nœud a été migré avec succès

_ Pour aller plus loin : [https://www.datastax.com/learn/whats-new-for-cassandra-4](https://www.datastax.com/learn/whats-new-for-cassandra-4) _

**********************************************************************************************************************************

## Rappel pour retrouver les environnements éventuellement précédemment instanciés dans Gitpod : [ https://gitpod.io/workspaces ](https://gitpod.io/workspaces)

**********************************************************************************************************************************

## Option 1 : Pour lancer notre TP : sur un environnement sans cassandra 3.x préinstallé  :

[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/crystalloide/Cassandra-Migration-Cassandra-3-vers-Cassandra-4/)

**********************************************************************************************************************************


✅ Vérification de la version de Java : 

    java -version
    

✅ Installation de cassandra 3.11.x : 

Download the binary tarball from one of the mirrors on the Apache Cassandra Download site. 

    curl -OL https://downloads.apache.org/cassandra/3.11.16/apache-cassandra-3.11.16-bin.tar.gz

    
The mirrors only host the latest versions of each major supported release. To download an earlier version of Cassandra, visit the Apache Archives.

OPTIONAL: Verify the integrity of the downloaded tarball using one of the methods here. For example, to verify the hash of the downloaded file using GPG:

    gpg --print-md SHA256 apache-cassandra-3.11.16-bin.tar.gz
    
Compare the signature with the SHA256 file from the Downloads site:

    curl -L https://downloads.apache.org/cassandra/3.11.16/apache-cassandra-3.11.16-bin.tar.gz.sha256


Unpack the tarball:

    tar xzvf apache-cassandra-3.11.16-bin.tar.gz
    
The files will be extracted to the apache-cassandra-3.11/ directory. This is the tarball installation location.

Located in the tarball installation location are the directories for the scripts, binaries, utilities, configuration, data and log files:

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
    For information on how to configure your installation, see Configuring Cassandra.


✅ Lancement de Cassandra : 

    cd apache-cassandra-3.11/ && bin/cassandra
    
This will run Cassandra as the authenticated Linux user.

✅ Monitor the progress of the startup with:

    tail -f logs/system.log
    
You can monitor the progress of the startup with:

    tail -f logs/system.log
    
For information on how to configure your installation, see Configuring Cassandra.

✅ Verify that a Cassandra 3.x node is running :

Check the status of Cassandra:

    bin/nodetool status

✅ Verify that the Cassandra version is 3.x:

    nodetool version

✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool version
    ReleaseVersion: 3.11.14

✅ Verify that the Cassandra 3.x node is running:

    nodetool status

✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool status
    
    Datacenter: datacenter1
    =======================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
    UN  127.0.0.1  70.87 KiB  256          100.0%            b10d4523-769e-41d4-aaaf-721ed30f4a22  rack1
    

### Create a keyspace and table, and insert data

In this step, you will create a keyspace and table, and populate them with some data.

✅ Start the CQL shell:

    cqlsh

✅ Create the keyspace:

    CREATE KEYSPACE united_states 
    WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

    USE united_states;

✅ Create the table:

    CREATE TABLE cities_by_state (
    state text,
    name text,
    population int,
    PRIMARY KEY ((state), name)
    );

✅ Insert the top 10 largest U.S. cities by population:

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

✅ Verify that the data has been loaded:

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


✅ Retrieve all the cities in California:

    SELECT * FROM cities_by_state WHERE state = 'California';

✅ Affichage en retour : 

    cqlsh:united_states> SELECT * FROM cities_by_state WHERE state = 'California';
    
     state      | name        | population
    ------------+-------------+------------
     California | Los Angeles |    4085014
     California |   San Diego |    1469490
     California |    San Jose |    1036242
    
    (3 rows)


✅ Exit the CQL shell and clear the screen:

    exit
    clear


✅ You have loaded the data, continue to the next step.

✅ Verify that the node is ready to be upgraded

✅ In this step, we will verify that the Cassandra 3.x cluster is ready to be upgraded. 

## There are nine factors to consider:

### 1. Current state
   
All nodes in the cluster need to be in the ‘Up and Normal’ state. 

Check that there are no nodes in the cluster that are in a state different to Up and Normal.

✅ Verify that all nodes have the UN state:

    nodetool status 
    
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool status 
    Datacenter: datacenter1
    =======================
    Status=Up/Down
    |/ State=Normal/Leaving/Joining/Moving
    --  Address    Load       Tokens       Owns (effective)  Host ID                               Rack
    UN  127.0.0.1  93.26 KiB  256          100.0%            b10d4523-769e-41d4-aaaf-721ed30f4a22  rack1
    

### 2. Disk space

✅ Verify that each node has at least 50% diskspace free:

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



### 3. Errors

✅ Verify that there are no unresolved errors (also, check for warnings) in the log:

    grep -e "WARN" -e "ERROR" cassandra3/logs/system.log
    
✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ grep -e "WARN" -e "ERROR" cassandra3/logs/system.log
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



### 4. Gossip stability

✅ Verify that all entries in the gossip information output have the gossip state STATUS:NORMAL: Check if there are any nodes that have a status other than NORMAL:

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



✅ Check for any records that have a status other than NORMAL:

    nodetool gossipinfo | grep STATUS | grep -v NORMAL
    
✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool gossipinfo | grep STATUS | grep -v NORMAL
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ 


### 5. Dropped messages

✅ Verify that there were no dropped messages in the past 72 hours:

    nodetool tpstats | grep -A 12 Dropped
    
✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool tpstats | grep -A 12 Dropped
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


### 6. Backup disabled

Verify that all automatic backups have been disabled.

This includes disabling Medusa and any scripts that call nodetool snapshot until the upgrade is complete.


### 7. Repair disabled

Verify that repairs have been disabled. 

This includes disabling automated repairs in Reaper.


### 8. Monitoring

Upgrading may result in a temporary reduction in performance, as it simulates a series of temporary node failures. 

Understanding how the upgrade impacts the performance of the system, both during and after, is crucial when working through the process.


### 9. Availability

Confirm that areas of the application that require strong consistency are using the LOCAL_QUORUM consistency level and the replication factor of 3.

When LOCAL_QUORUM is used with a replication factor below 3, all replicas must be available for requests to succeed. 

A rolling restart using this configuration will result in full or partial unavailability while a node is DOWN.

You are now ready to begin the upgrade.


## Prepare the node for migration

In this step, we will prepare the Cassandra 3.x cluster for the upgrade.

✅ Take a snapshot of the node in case you need to roll back the upgrade (nodetool snapshot also flushes the memtables to disk):

    nodetool snapshot
    
✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool snapshot
    Requested creating snapshot(s) for [all keyspaces] with snapshot name [1682847408083] and options {skipFlush=false}
    Snapshot directory: 1682847408083


✅ Stop the node by finding the PID and calling kill:

    pgrep -u gitpod -f cassandra | xargs kill -9

✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ pgrep -u gitpod -f cassandra | xargs kill -9


✅ Use nodetool to verify that the node has been shut down:

    nodetool status
    
✅ Affichage en retour : 

    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ nodetool status
    nodetool: Failed to connect to '127.0.0.1:7199' - ConnectException: 'Connection refused (Connection refused)'.

The node has been shutdown. Continue to the next step.


## Installation de Cassandra 4.x :

In this step, you will dowanload and unpack the Cassandra 4.1.4 distribution.

✅ Download and install Cassandra 4.x:

    wget https://archive.apache.org/dist/cassandra/4.1.4/apache-cassandra-4.1.4-bin.tar.gz

    tar -xzf apache-cassandra-4.1.4-bin.tar.gz

    rm apache-cassandra-4.1.4-bin.tar.gz

    mv apache-cassandra-4.1.4 cassandra4

✅ Affichage en retour : 
    
    gitpod /workspace/cassandra4-migrating-from-cassandra3 (main) $ wget https://archive.apache.org/dist/cassandra/4.1.4/apache-cassandra-4.1.4-bin.tar.gz

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
