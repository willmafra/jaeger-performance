## Simple standalone example of writing spans to Jaeger

# Start Cassandra and create the keyspace
+ `docker run --name=cassandra --rm -it -p 7000:7000 -p 9042:9042 cassandra:3.9 `
+ `MODE=test ./plugin/storage/cassandra/schema/create.sh | cqlsh `

# Start the Collector and Agent
+ `export CASSANDRA_KEYSPACE_NAME=jaeger_v1_test`
+ `export CASSANDRA_CLUSTER_IP=<ctual ip of cassandra is running on, not localhost`
+ `docker run -it -e COLLECTER_QUEUE_SIZE=300000 -e CASSANDRA_SERVERS=${CASSANDRA_CLUSTER_IP} -e CASSANDRA_KEYSPACE=${CASSANDRA_KEYSPACE_NAME} --rm -p14267:14267 -p14268:14268 jaegertracing/jaeger-collector:latest` 
+ `docker run -it -e PROCESSOR_JAEGER_BINARY_SERVER_QUEUE_SIZE=100000 -e PROCESSOR_JAEGER_COMPACT_SERVER_QUEUE_SIZE=100000 -e COLLECTOR_HOST_PORT=${CASSANDRA_CLUSTER_IP}:14267 -p5775:5775/udp -p6831:6831/udp -p6832:6832/udp -p5778:5778/tcp jaegertracing/jaeger-agent:latest
`
# Optional: Start the UI
+ `docker run -it -e CASSANDRA_SERVERS=${CASSANDRA_CLUSTER_IP} -e CASSANDRA_KEYSPACE=${CASSANDRA_KEYSPACE_NAME} -p16686:16686  jaegertracing/jaeger-query:latest`
`

# Optional: Run from source instead of docker

+ Start cassandra
+ cd to the Jaeger source directory and build.
+ `MODE=test ./plugin/storage/cassandra/schema/create.sh | cqlsh `
+ Collector: `go run ./cmd/collector/main.go  --cassandra.keyspace=${CASSANDRA_KEYSPACE_NAME} --collector.queue-size=300000`
+ Agent: `go run ./cmd/agent/main.go   --collector.host-port=localhost:14267 --processor.jaeger-binary.server-queue-size=300000 --processor.jaeger-compact.server-queue-size=300000 --processor.zipkin-compact.server-queue-size=300000 `
 (NOTE:  I'm not sure which of these we need...maybe compact for 6831, zipkin-compact for 5775?)

#Checkout and run the tests
Clone this repo and cd into it `git@github.com:Hawkular-QE/SimpleJaegerExample.git`

### Run the test

#### I configured this to create 300 000 spans in 30 seconds, 
+ export THREAD_COUNT=100
+ export ITERATIONS=3000
+ export DELAY=10
+ export USE_AGENT_OR_COLLECTOR=collector /// if you want,defaults to agent
+ export JAEGER_MAX_QUEUE_SIZE=100000  // This is the default

Optional for the test: CASSANDRA_CLUSTER_IP defaults to localhost CASSANDRA_KEYSPACE_NAME defaults to jaeger_v1_test
+ `mvn clean -Dtest=SimpleTest#createTracesTest test`

### Between test runs do this to clear out the traces table
+ `cqlsh --keyspace=jaeger_v1_test --execute="truncate traces;"
`
### Just get a count of traces in Cassandra
+ `mvn clean -Dtest=SimpleTest#countTraces test`



