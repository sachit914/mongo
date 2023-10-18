# mongo



## replication

Create a Docker network:
```
docker network create mongo-replica-network
```

 Start the first MongoDB container (Primary):
```
docker run --name mongo-primary -d --network mongo-replica-network mongo:latest --replSet myreplica
```

 Start the second and third MongoDB containers (Secondaries):
```
docker run --name mongo-secondary1 -d --network mongo-replica-network mongo:latest --replSet myreplica
docker run --name mongo-secondary2 -d --network mongo-replica-network mongo:latest --replSet myreplica
```


initialize the Replica Set:
```
docker exec -it mongo-primary bash
```

```
mongosh
```

```
rs.initiate(
   {
      _id: "myreplica",
      members: [
         { _id: 0, host: "mongo-primary:27017" },
         { _id: 1, host: "mongo-secondary1:27017" },
         { _id: 2, host: "mongo-secondary2:27017" }
      ]
   }
)
```

Check the Replica Set Status:
```
rs.status()
```

now mongo-primary(master) is used to perform write operation

mono-secondary1 and mongo-secondary2 (slave node) are used to perform read operation


//checking if in all node same data are present

```
docker exec -it mongo-secondary1 bash
```

```
mongosh
```


```
mongosh "mongodb://127.0.0.1:27017/mydatabase?readPreference=secondary"
```

```
db.mycollection.find({ name: "John" })
```



https://chat.openai.com/share/5d6fa2dc-9fe5-4c11-9b20-bd50baa3a348

# mongo sharding


## Run MongoDB Config Servers:
```
docker run --name mongo-config1 -d -p 27019:27019 mongo:latest --configsvr --replSet configrs --bind_ip_all --port 27019

```

#### key points

--configsvr: Indicates that this MongoDB instance will act as a config server. Config servers store metadata and configuration settings for a MongoDB sharded cluster.


A MongoDB sharded cluster is a set of servers that distributes the dataset across multiple machines. To manage how the data is distributed and to store metadata about the chunks of data (which portions of the data reside on which shard), MongoDB uses configuration servers.


--replSet configrs: Specifies that the MongoDB instance should be a member of a replica set named "configrs". Replica sets in MongoDB are groups of MongoDB processes that maintain the same dataset. This provides redundancy and high availability.

--bind_ip_all: Tells the MongoDB instance to bind to all available IP addresses. This is useful if you want to allow connections from outside the container, rather than only from other containers or localhost.

--port 27019: Specifies the port on which MongoDB should listen inside the container. Note that because of the -p flag we discussed earlier, this port inside the container will correspond to port 27019 on the host machine.



## Initialize the Config Replica Set:

```
docker exec -it mongo-config1 mongo --port 27019
```

inside mongodb shell inititalize replica set
```
rs.initiate(
   {
      _id: "configrs",
      configsvr: true,
      members: [
         { _id : 0, host : "localhost:27019" }
      ]
   }
);
```

1) rs.initiate()

This is a MongoDB shell command used to initiate a new replica set. It starts the process of turning standalone MongoDB servers into members of a replica set.
2) _id: "configrs": This specifies the name of the replica set. In this case, the replica set is named "configrs".
3) configsvr: true: This indicates that the replica set is a configuration server replica set for a MongoDB sharded cluster. Configuration servers store the metadata about which chunks of data reside on which shards in a sharded cluster.
4) members: This is an array that defines the initial members of the replica set. Each member in the replica set is represented by a document inside this array.
    - _id : 0: This assigns a unique ID to the replica set member. Every member of a replica set must have a unique _id
    - host : "localhost:27019": This specifies the hostname (or IP address) and port of the MongoDB instance for this member. In this case, it's pointing to MongoDB running on localhost (the same machine where the command is run) and on port 27019.

## Run the Sharding Servers (Shard Nodes):

 ```
docker run --name shard1 -d -p 27020:27020 mongo:latest --shardsvr --bind_ip_all --port 27020
docker run --name shard2 -d -p 27021:27021 mongo:latest --shardsvr --bind_ip_all --port 27021
```

--shardsvr: A MongoDB-specific option that configures the MongoDB instance to run as a shard server in a sharded cluster.

--port 27020 and --port 27021: Specifies the ports on which MongoDB will listen inside their respective containers.

## Run the MongoDB Router (mongos):

```
docker run --name mongos -d -p 27017:27017 mongo:latest mongos --configdb configrs/localhost:27019 --bind_ip_all --port 27017
```

--configdb configrs/localhost:27019: Points the mongos instance to the configuration replica set (configrs)


## Add Shards to the Cluster:

```
docker exec -it mongos mongo
```

```
sh.addShard("localhost:27020");
sh.addShard("localhost:27021");

```

1) Shards: Each shard contains a subset of the sharded data. You can think of each shard as an individual MongoDB instance or, more commonly, a replica set that holds a portion of the dataset.
2) Mongos (Router): The mongos acts as a query router. Applications connect to the mongos, which directs operations to the appropriate shard or shards.
3) Config Servers: These servers maintain the cluster metadata. This metadata, among other things, tracks which chunks of data are on which shards.
4) sh.addShard()
The sh.addShard() command is used to add a new shard to a sharded cluster. You would run these commands from a mongos instance.
5 ) localhost:27020: The first command is instructing the sharded cluster to add a MongoDB instance running on localhost (the same machine where the command is being run) and listening on port 27020 as a new shard.

localhost:27021: Similarly, the second command is adding another MongoDB instance on port 27021 as a separate shard.

## Enable Sharding for a Database:

```
sh.enableSharding("mydb");
```
