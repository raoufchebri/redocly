---
title: Connect to ScyllaDB
---

# Create a ScyllaDB account

First, create an account on [Scylla Cloud](https://cloud.scylladb.com) if you haven't done so already.

<iframe width="100%" height="480" src="https://www.youtube.com/embed/jnWDMiuy9hM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Connect

ScyllaDB is a distributed database, which means that it operates on multiple nodes running independently. When creating a Session you can specify a few known nodes to which the driver will try connecting:

```JavaScript

const cassandra = require('cassandra-driver')

async function connect() {
    const cluster = new cassandra.Client({
        contactPoints: ['SCYALL_URI'],
        localDataCenter: 'DATA_CENTER',
        credentials: {username: 'USERNAME', password: 'PASSWORD'},
        // keyspace: 'your_keyspace'
    })

    const results = await cluster.execute('SELECT * FROM system.clients LIMIT 10')
    results.rows.forEach(row => console.log(JSON.stringify(row)))

    await cluster.shutdown()
}

connect()
```

```Rust
# extern crate scylla;
# extern crate tokio;
use scylla::{Session, SessionBuilder};
use std::error::Error;
use std::time::Duration;
use std::net::{IpAddr, Ipv4Addr, SocketAddr};

#[tokio::main]
async fn main() -> Result<(), Box<dyn Error>> {
    let uri = std::env::var("SCYLLA_URI")
        .unwrap_or_else(|_| "127.0.0.1:9042".to_string());

    let session: Session = SessionBuilder::new()
        .known_node(uri)
        .known_node("127.0.0.72:4321")
        .known_node("localhost:8000")
        .connection_timeout(Duration::from_secs(3))
        .known_node_addr(SocketAddr::new(
            IpAddr::V4(Ipv4Addr::new(127, 0, 0, 1)),
            9000,
        ))
        .build()
        .await?;

    Ok(())
}
```

```Python

#!/usr/bin/python
#
# A simple example of connecting to a cluster
# To install the driver Run pip install scylla-driver
from cassandra.cluster import Cluster, ExecutionProfile, EXEC_PROFILE_DEFAULT
from cassandra.policies import DCAwareRoundRobinPolicy, TokenAwarePolicy
from cassandra.auth import PlainTextAuthProvider


def getCluster():
    profile = ExecutionProfile(load_balancing_policy=TokenAwarePolicy(DCAwareRoundRobinPolicy(local_dc='AWS_EU_WEST_3')))

    return Cluster(
        execution_profiles={EXEC_PROFILE_DEFAULT: profile},
        contact_points=[
            "13.36.216.210", "13.38.174.38", "13.36.114.34"
        ],
        port=9042,
        auth_provider = PlainTextAuthProvider(username='scylla', password='***************'))

print('Connecting to cluster')
cluster = getCluster()
session = cluster.connect()

print('Connected to cluster %s' % cluster.metadata.cluster_name)

print('Getting metadata')
for host in cluster.metadata.all_hosts():
    print('Datacenter: %s; Host: %s; Rack: %s' % (host.datacenter, host.address, host.rack))

cluster.shutdown()

```

```Go

package main

import (
    "fmt"
    "github.com/gocql/gocql"
)

func main() {
    var cluster = gocql.NewCluster("13.36.216.210", "13.38.174.38", "13.36.114.34")
    cluster.Authenticator = gocql.PasswordAuthenticator{Username: "scylla", Password: "***************"}
    cluster.PoolConfig.HostSelectionPolicy = gocql.DCAwareRoundRobinPolicy("AWS_EU_WEST_3")

    var session, err = cluster.CreateSession()
    if err != nil {
        panic("Failed to connect to cluster")
    }

    defer session.Close()

    var query = session.Query("SELECT * FROM system.clients")

    if rows, err := query.Iter().SliceMap(); err == nil {
        for _, row := range rows {
            fmt.Printf("%v\n", row)
        }
    } else {
        panic("Query error: " + err.Error())
    }
}

```

```Java

package com.scylladb.cluster_connection;

import com.datastax.driver.core.*;
import com.datastax.driver.core.policies.DCAwareRoundRobinPolicy;


public class ConnectionExample {
    public void connect_and_query() {
        Cluster cluster = Cluster.builder()
        .addContactPoints("13.36.216.210", "13.38.174.38", "13.36.114.34")
                .withLoadBalancingPolicy(DCAwareRoundRobinPolicy.builder().withLocalDc("AWS_EU_WEST_3").build()) // your local data center
                .withAuthProvider(new PlainTextAuthProvider("scylla", "***************"))
                .build();

        System.out.println("Connected to cluster " + cluster.getMetadata().getClusterName());

        for (Host host: cluster.getMetadata().getAllHosts()) {
          System.out.printf("Datacenter: %s, Host: %s, Rack: %s\n", host.getDatacenter(), host.getEndPoint(), host.getRack());
        }

        Session session = cluster.connect("YOUR_KEYSPACE_NAME");
        System.out.println("Connected to cluster " + metadata.getClusterName())
        ResultSet resultSet = session.execute("SELECT * FROM system.clients LIMIT 10"); // some query

        session.close();
        cluster.close();
    }

}


```
