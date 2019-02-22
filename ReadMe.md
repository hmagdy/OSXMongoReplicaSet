Simple MongoDB Replica Set on Mac
---

In MongoDB, a replica set is simply a set (or cluster if you like fancy names) that consists of at least two mongod instances that replicate data amongst one another. Such setup increases redundancy and ensures high availability of the database. It may also improve its read capacity.

##Concepts
In a replica set there is always only one primary mongod instance that is used to manage write operations. Secondary instances are only readable and asynchronously replicate data from the primary mongod instance. Replica sets also provide automated failover which means that if the primary member fails, secondary ones will automatically try to elect a new primary among them.

The operations log (abbreviated as oplog) is a special feature (implemented as a capped collection) of replica set that records each operation modifying any data stored in the databases. In order to modify the database configured as a replica set, MongoDB starts by applying write operations on the primary member. Next, the operation is recorded on the primary’s oplog. The secondary members are now ready to replicate this oplog. Once replicated, they can apply these operations to themselves in an asynchronous process. Operations in the oplog are idempotent. Readable members may not have the latest changes at all times, but eventually, they will contain the same version of the oplog. When the system allows gradual propagation of changes we say that read operations to a primary have strict consistency, while read operations to secondaries have eventual consistency.

- Development Setup

Setting up a replica set on OSX is pretty straightforward. I assume that everything is performed on the localhost with no security nor optimisation taken into consideration. In practice, while in development mode, the number of mongod instances can be even reduced to a single one. In that case there is obviously no replication, but we have access to the oplog, which, in some scenarios, may be useful.

Let's run a mongod instance as a member of test replica set with the following command:

    
    brew services stop mongodb
    sudo rm -rf rs0 rs0.conf rs1 rs1.conf var/ rs2 rs2.conf
    sudo sh ./setup.sh
    
    
Do not forget to set in slave shell

    rs.slaveOk()
    
and use 
    
    mongodb://127.0.0.1:27018/XYZ?replicaSet=rs0

to avoid `MongoError: not master and slaveOk=false` error
    
Next, let's connect to this instance and configure its replica set as shown below:

    mongo --port 27017
    mongo --port 10718

You can setup more replica sets by:
    
    sudo sh ./rs.sh 3 ----> mongo --port 27020
    sudo sh ./rs.sh 4 ----> mongo --port 27021
    sudo sh ./rs.sh 5 ----> mongo --port 27022
    
Note: you can connect to each on with PORT = (27017+ID)    

Thanks:

- https://zaiste.net/mongodb_replica_set_and_osx_setup/
- https://medium.com/@katopz/minimal-mongodb-replica-set-osx-76dc9dc36018


Commands:

    brew services restart mongodb
    brew services start mongodb
    brew services stop mongodb
    sudo rm -rf rs0 rs0.conf rs1 rs1.conf var/ rs2 rs2.conf
    db.adminCommand({setFeatureCompatibilityVersion: "3.6"})
    db.adminCommand({replSetStepDown: 86400, force: 1})
       
    rs.initiate({_id:"rs0", members: [{_id:0, host:"127.0.0.1:27017"}, {_id:1, host:"127.0.0.1:27018"}]})
    
    > rs.initiate({
       "_id" : "rs0",
       "members" : [
        {
         "_id" : 0,
         "host" : "localhost:27017"
        },
        {
         "_id" : 1,
         "host" : "localhost:27018"
        },
        {
         "_id" : 2,
         "host" : "localhost:27019"
        }
       ]
    })
     