## Introduction

[Demo Video](https://www.youtube.com/watch?v=gdX7VxVnL6U)

[Blog Post](https://dev.to/sashaonion/javaraft-raft-based-distributed-key-value-store-5h0a)

A Java-based implementation of the Raft consensus algorithm, designed to support consistent (CP) distributed systems like distributed key-value stores.

This project includes:

* The core Raft algorithm
* A set of pluggable interfaces for state machines and persistence
* A Spring Boot-based RPC layer for inter-node communication
* A simple in-memory key-value store with HTTP endpoints for PUT, GET, and DELETE operations (as a reference implementation)

Developers can extend this framework by implementing custom state machines and storage backends to build their own distributed services.

Built with Java, Spring Boot, Lombok, and SLF4J.

## Features
Features reference the section number of the [Raft](https://raft.github.io/raft.pdf) paper:
- Leader election (§5.2)
- Log replication (§5.3)
- Election restriction (§5.4.1)
- Committing entries from previous terms (§5.4.2)
- Follower and candidate crashes (§5.5)
- Implementing linearizable semantics (§8)

## Usage

Try out the distributed key-value API with a 3-node Raft cluster.

### Option 1: Run with Docker Compose

Build and start all three Raft nodes:

```bash
docker compose up --build
```

This starts:

```text
node1 -> http://localhost:9090
node2 -> http://localhost:9091
node3 -> http://localhost:9092
```

To stop the cluster:

```bash
docker compose down
```

To simulate a node failure:

```bash
docker stop javaraft-node2
```

To restart the node:

```bash
docker start javaraft-node2
```

### Option 2: Run manually with local Java

Build the JAR, skipping tests:

```bash
mvn clean package -DskipTests
```

Terminal 1:

```bash
java -jar target/distributed_key_value_store-0.0.1-SNAPSHOT.jar --spring.profiles.active=node1
```

Terminal 2:

```bash
java -jar target/distributed_key_value_store-0.0.1-SNAPSHOT.jar --spring.profiles.active=node2
```

Terminal 3:

```bash
java -jar target/distributed_key_value_store-0.0.1-SNAPSHOT.jar --spring.profiles.active=node3
```

## Endpoints

All operations must be sent to the current leader node. Redirection is not implemented, and follower reads/writes are blocked.

In the examples below, `localhost:9090` is used. If node 1 is not the leader, send the request to the leader's port instead:

```text
node1 -> http://localhost:9090
node2 -> http://localhost:9091
node3 -> http://localhost:9092
```

### Get operation

```bash
curl -X GET "http://localhost:9090/raft/client/get?key=myKey"
```

### Put operation

```bash
curl -X POST "http://localhost:9090/raft/client/insert" \
     -H "Content-Type: application/json" \
     -d '{"clientId": "client1", "sequenceNumber": 1, "key": "myKey", "value": "myValue"}'
```

### Delete operation

```bash
curl -X POST "http://localhost:9090/raft/client/delete" \
     -H "Content-Type: application/json" \
     -d '{"clientId": "client1", "sequenceNumber": 2, "key": "myKey"}'
```
