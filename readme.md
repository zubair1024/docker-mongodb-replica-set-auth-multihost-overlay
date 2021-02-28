# MongoDB Replica Set with auth for multi-host setup using ingress network

## How to use?

1. Edit your `/etc/host`. If your running your nodes on 192.168.99.101, 192.168.99.102 and 192.168.99.103 then

   ```shell
   192.168.99.101 mongo1
   192.168.99.102 mongo2
   192.168.99.103 mongo3
   ```

1. Create a `.env` with the necessary data leverage the `example.env` file for reference

1. Initialize a Swarm Manager

   ```shell
   # 192.168.99.101 mongo1
   docker swarm init --advertise-addr 192.168.99.101
   ```

1. Initialize workers

   ```shell
   # 192.168.99.102 mongo2
   # 192.168.99.103 mongo3
   docker swarm join <token>
   ```

1. Create ingress network

   ```shell
   docker network create -d overlay --attachable razr-overlay
   ```

1. start the containers

   ```shell
   # 192.168.99.101 mongo1
   docker-compose -f ./mongo1.yml up -d
   ```

   ```shell
   # 192.168.99.102 mongo2
   docker-compose -f ./mongo2.yml up -d
   ```

   ```shell
   # 192.168.99.103 mongo1
   docker-compose -f ./mongo3.yml up -d
   ```

1. Run the configuration

   Option 1:

   Run the `mongo-init.yml` container.

   Option 2:

   ```shell
   docker exec -it mongo1 sh -c "mongo --port 30001 -u root"
   ```

   Initialize the replica set

   ```javascript
   rs.initiate(
     {
       _id: "rs0",
       protocolVersion: 1,
       version: 1,
       members: [
         {
           _id: 0,
           host: "mongo1:30001",
           priority: 2,
         },
         {
           _id: 1,
           host: "mongo2:30002",
           priority: 0,
         },
         {
           _id: 2,
           host: "mongo3:30003",
           priority: 0,
         },
       ],
     },
     { force: true }
   );
   ```

1. Check the replica set status

   ```javascript
   rs.status();
   ```

If everything is working, you're done 😉
