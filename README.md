# lngraph

lngraph imports Lightning Network data into [Neo4j](https://neo4j.com/product/) for local exploration of your node and its view of the network. This is **not intended to be a public explorer**.

The purpose is to explore local information (channels, chain transactions, peers) and private data (channel balances, fees, bytes sent and received) but also public (all other nodes and channels).

![lngraph screenshot](https://raw.githubusercontent.com/xsb/lngraph/master/img/screenshot.png)

![lngraph screenshot 2](https://raw.githubusercontent.com/xsb/lngraph/master/img/screenshot2.png)

As of today this tool is just a Neo4j importer. All data is persisted in Neo4j and explored using [Neo4j Browser](https://neo4j.com/developer/guide-neo4j-browser/). Source data files have to be manually extracted using `lncli` so the node and network state is completely static.

I started working on this as a way to learn more about both the Lightning Network and Neo4j. If you see something that can be improved (or that it's just broken) feel free to open an issue or a pull request.

## How to use

Clone the repository and install the `lngraph` binary. You will need [Go installed](https://golang.org/dl/) in your system.:

```sh
git clone git@github.com:xsb/lngraph.git
cd lngraph
go install
```

Run Neo4j and the Neo4j Browser using [Docker](https://docs.docker.com/install).

```sh
docker run -d \
    --name=lngraph_neo4j \
    --publish=7474:7474 \
    --publish=7687:7687 \
    --env=NEO4J_AUTH=none \
    --volume=$HOME/.lngraph/neo4j:/data \
    neo4j
```

Use `lngraph` to get data from LND and index it in the Neo4j database that we previously launched. By default it connects to localhost (this is why we launched Neo4j with Docker previously) but you can specify a database URL with the `-url` flag.

```sh
lngraph \
    -lnd-grpc 127.0.0.1:10009 \
    -macaroon /path/to/readonly.macaroon \
    -tls-cert /path/to/tls.cert
```

Open [http://localhost:7474/](http://localhost:7474/) in your browser to open the Neo4j Browser UI.

## Neo4j queries

The Neo4j query language is called Cypher, check the [documentation](https://neo4j.com/developer/cypher/) to learn more about it.

A good start is to visualise your node, your channels, the nodes you have opened channels with, the blockchain transactions that opened these channels and your peers in the network.

```cypher
MATCH (mynode:Node)-[:OPENED]->(mychannels:Channel)<-[:FUNDED]-(mytransactions:Transaction)
MATCH (othernodes:Node)-[:OPENED]->(mychannels:Channel)
MATCH (mynode:Node)-[:PEER]->(peers:Node)
RETURN mynode, mychannels, mytransactions, othernodes, peers
```

You can edit the color and size of the nodes and relationships, the property that is used as a name for each type of node, etc. But notice that Neo4j Browser will store configuration in Local Storage, not in the shared Docker volume.
