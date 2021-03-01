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

1. Add labels for each node

   ```shell
   #ssh <-- The manager node server -->
   docker node update --label-add mongo.replica=1 node1
   docker node update --label-add mongo.replica=2 node2
   docker node update --label-add mongo.replica=3 node3

   ```

1. start the swarm cluster as a docker stack

   ```shell
   # 192.168.99.101 mongo1
   docker stack deploy -c docker-compose.yml overlay
   ```

1. Open necessary ports of the swarm for each service

   ```
   # docker service update --publish-rm 30001:30001 overlay_mongo1
   # docker service update --publish-rm 30002:30002 overlay_mongo2
   # docker service update --publish-rm 30003:30002 overlay_mongo3
   # // if publish port was set before, need to remove first and update it.

   docker service update --publish-add 30001:30001 overlay_mongo1
   docker service update --publish-add 30002:30002 overlay_mongo2
   docker service update --publish-add 30003:30003 overlay_mongo3
   ```

1. Check the replica set status

   ```javascript
   rs.status();
   ```

If everything is working, you're done 😉

## How to remove?

1. Remove the Swarm

   ```
   docker stack rm overlay
   ```

1. Remove the volumes
   ```
   docker volume rm $(docker volume ls -qf label=com.docker.stack.namespace=overlay)
   ```
