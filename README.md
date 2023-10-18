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




# mongo sharding

```
docker run --name mongo-config1 -d -p 27019:27019 mongo:latest --configsvr --replSet configrs --bind_ip_all --port 27019

```

### key points

--configsvr: Indicates that this MongoDB instance will act as a config server. Config servers store metadata and configuration settings for a MongoDB sharded cluster.


A MongoDB sharded cluster is a set of servers that distributes the dataset across multiple machines. To manage how the data is distributed and to store metadata about the chunks of data (which portions of the data reside on which shard), MongoDB uses configuration servers.


--replSet configrs: Specifies that the MongoDB instance should be a member of a replica set named "configrs". Replica sets in MongoDB are groups of MongoDB processes that maintain the same dataset. This provides redundancy and high availability.

--bind_ip_all: Tells the MongoDB instance to bind to all available IP addresses. This is useful if you want to allow connections from outside the container, rather than only from other containers or localhost.

--port 27019: Specifies the port on which MongoDB should listen inside the container. Note that because of the -p flag we discussed earlier, this port inside the container will correspond to port 27019 on the host machine.










