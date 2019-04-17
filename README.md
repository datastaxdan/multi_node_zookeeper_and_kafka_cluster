# Multi Node Zookeeper and Kafka Cluster
This repository is to make it simple to set up a multi node zookeeper ensemble with multi node kafa cluster locally on a mac or Linux machine. This will allow you to get started with kafka quickly for development and educational purposes.

## Download Kafka

The Kafka tarball can be downloaded from the official kafka site by clicking [here](https://kafka.apache.org) 

I am going to be using the scala 2.12 version. Please ensure you use the required version for your environment

1. Download Kafka
```bash
cd /opt
wget https://www-eu.apache.org/dist/kafka/2.2.0/kafka_2.12-2.2.0.tgz
```
2. Extract the Download
```bash
tar xvfz kafka_2.12-2.2.0.tgz
rm -Rf ar xvfz kafka_2.12-2.2.0.tgz
```
## Configure ZooKeeper

### Set up the zookeeper user
1. Create a User to run the zookeeper service
    ```bash 
    sudo user add zk -m
    ```
    The -m Flag creates a home directory for the zk user. This is where we will base all of our zookeeper configuration from.

2. Ensure Bash is the default shell for the zookeeper user
    ```bash
    sudo usermod --shell /bin/bash zk
    ```

3. Configure a pw for the zookeeper user
    ```bash
    sudo passwd zk
    ```

4. Add the zk user to the sudo group to allo running of provelaged commands
    ```bash
    usermod -aG sudo zk
    ```
5. Log in with the zk user
    ```bash
    su -l zk
    ```
### Configure the ZooKeeper Ensemble

This is the directory where we will keep all zookeeper data. This will allow ZooKeeper to persists all configuration and state data to disk so it can survive a reboot/failure. As we are running 3 nodes locally i will create 3 seperate data dariectory folders, one for each node

1. Create the directory and change ownership of directory to zk user
    ```bash
    sudo mkdir -p /data/zookeeper1
    sudo mkdir -p /data/zookeeper2
    sudo mkdir -p /data/zookeeper3
    sudo chown zk:zk /data/zookeeper1
    sudo chown zk:zk /data/zookeeper2
    sudo chown zk:zk /data/zookeeper3
    ```

2. Next we need to configure the first zookeeper config file. The zookeeper.properties file is the file that holds the configuration for the zookeeper instances. Again we need to create 3 of these config files. We will configure the first file and use this as a template to configure the others.
    * Copy the template file for the first instance
    ```bash
    sudo cp /opt/kafka_2.12-2.2.0/config/zookeeper.properties /opt/kafka_2.12-2.2.0/config/zookeeper1.properties
    ```
    * Edit the zookeeper1.config file with your editor of choice and add the following to the bottom of the file. a copy of the file can be founfd in this repo.

        * tickTime=2000 
        * initLimit=5
        * syncLimit=2
        * server.1=localhost:2666:3666
        * server.2=localhost:2667:3667
        * server.3=localhost:2668:3668

    * Now we need to copy this file for the other 2 instances
    ```bash
    sudo cp /opt/kafka_2.12-2.2.0/config/zookeeper1.properties /opt/kafka_2.12-2.2.0/config/zookeeper2.properties

    sudo cp /opt/kafka_2.12-2.2.0/config/zookeeper1.properties /opt/kafka_2.12-2.2.0/config/zookeeper3.properties
    ```
    * Now we just need to change the client port and data directory for in these 2 files. I am going to use the below confguration
        
        * zookeeper1.properties - clientport=2181
        * zookeeper1.properties - dataDir=/data/zookeeper1
        * zookeeper2.properties - clientport=2182
        * zookeeper2.properties - dataDir=/data/zookeeper2
        * zookeeper1.properties - clientport=2183
        * zookeeper1.properties - dataDir=/data/zookeeper3

    * Finally we need to give each node an ID. This is done by creating an ID file in each zookeeper node directory with a single digit

        * zookeeper1 = 1
        * zookeeper2 = 2
        * zookeeper3 = 3

    ```bash
    sudo echo "1" > /data/zookeeper1/myid
    sudo echo "2" > /data/zookeeper2/myid
    sudo echo "3" > /data/zookeeper3/myid
    ```
    * Finally, we start zookeeper using the scripts we have configured. I like to do this in 3 seperate termianl windows
    ```bash
    sudo /opt/kafka_2.12-2.2.0/bin/zookeeper-server-start.sh /opt/kafka_2.12-2.2.0/config/zookeeper1.properties

    sudo /opt/kafka_2.12-2.2.0/bin/zookeeper-server-start.sh /opt/kafka_2.12-2.2.0/config/zookeeper2.properties

    sudo /opt/kafka_2.12-2.2.0/bin/zookeeper-server-start.sh /opt/kafka_2.12-2.2.0/config/zookeeper3.properties
    ```
     
### Configure Kafka
The kafka configuration is kept in a file called server.properties
We need to create a copy of this for each kafka broker we want to run. In this case im creating a 3 node kafka cluster

1. Copy the template file
   ```bash
    sudo cp /opt/kafka_2.12-2.2.0/config/server.properties /opt/kafka_2.12-2.2.0/config/server1.properties
   ```
2. We now need to edit this file and add our zookeeper nodes to the zookeeper.connect
    * zookeeper.connect=localhost:2181,localhost:2182,localhost:2183
    * log.dir = /opt/kafka1/logs

3. Next we need to create copies of this file for the other 2 nodes
    ```bash
        sudo cp /opt/kafka_2.12-2.2.0/config/server1.properties /opt/kafka_2.12-2.2.0/config/server2.properties

        sudo cp /opt/kafka_2.12-2.2.0/config/server1.properties /opt/kafka_2.12-2.2.0/config/server3.properties
    ```

4. Next we need to edit these files with the below changes

    * kafkabroker1
        
        * brokerid = 0
        * port = 9092
        * log.dirs = /opt/kafka1/logs
  
    * kafkabroker2

        * brokerid = 1
        * port = 9093
        * log.dirs = /opt/kafka2/logs 

    * kafkabroker3

        * brokerid = 2
        * port = 9094
        * log.dirs = /opt/kafka3/logs

5. Now that we have configured kafka we can now start each of the kafka brokers. Again im going to keep 3 seperate terminal windows so i can see live logs. I like to do this as part of my development workflow.
    ```bash
    sudo /opt/kafka_2.12-2.2.0/bin/kafka-server-start.sh /opt/kafka_2.12-2.2.0/config/server1.properties

    sudo /opt/kafka_2.12-2.2.0/bin/kafka-server-start.sh /opt/kafka_2.12-2.2.0/config/server2.properties

    sudo /opt/kafka_2.12-2.2.0/bin/kafka-server-start.sh /opt/kafka_2.12-2.2.0/config/server3.properties
    ```


