### Due to my community edition I can not use Ultimate Edition features so here is the code for create a topic inside kafka container 

#### First get inside the Kafka container

```bash
docker exec -it <your-kafka-container-id-or-name> bash
```

#### Then run the following command to create a topic named `patient-updates` with 3 partitions and a replication factor of 1

```bash
kafka-topics.sh --create \
  --topic patient-updates \
  --bootstrap-server localhost:9092 \
  --partitions 3 \
  --replication-factor 1
```