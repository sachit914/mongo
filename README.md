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

