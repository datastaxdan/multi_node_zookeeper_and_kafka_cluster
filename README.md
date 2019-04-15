# Multi Node Zookeeper and Kafka Cluster
This repository is to make it simple to set up a multi node zookeeper ensemble with multi node kafa cluster locally on a mac or Linux machine. This will allow you to get started with kafka quickly for development and educational purposes.

## Download Kafka

The Kafka tarball can be downloaded from the official kafka site by clicking [here](https://kafka.apache.org) 

I am going to be using the scala 2.12 version. Please ensure you use the required version for your environment

1. Download Kafka
```bash
wget https://www-eu.apache.org/dist/kafka/2.2.0/kafka_2.12-2.2.0.tgz
```
2. Extract the Download
```bash
tar xvfz kafka_2.12-2.2.0.tgz
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
### Configure the ZooKeeper Data Directory

This is the directory where we will keep all zookeeper data. This will allow ZooKeeper to persists all configuration and state data to disk so it can survive a reboot/failure.

1. Create the directory and change ownership of directory to zk user
    ```bash
    sudo mkdir -p /data/zookeeper
    sudo chown zk:zk /data/zookeeper
    ```



