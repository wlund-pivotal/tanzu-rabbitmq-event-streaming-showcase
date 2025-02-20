# Spring SCDF


# Build account-generator-source Docker Images

## step 1 - Change to project dir

```shell
    cd ~/projects/rabbitmq/tanzu-rabbitmq-event-streaming-showcase/
```

## step 2 - build docker (optional if complete lab related to Quorum Queues)

```shell
mvn -pl applications/account-generator-source -am spring-boot:build-image
```

## step 3 - load docker to kubernetes kind (optional if complete lab related to Quorum Queues)

```shell
kind load docker-image account-generator-source:0.0.1-SNAPSHOT
```


--------------

## Open SCDF Dashboard

## step 1 - add account-generator-source

```shell
k port-forward deployment/scdf-server 9393:8080 &
```

open [http://localhost:9393/dashboard](http://localhost:9393/dashboard)

Click Add Application -> Register application

name: account-generator-source
type: source
URI: docker:account-generator-source:0.0.1-SNAPSHOT

Click -< Import Application

## step 2 - add account-generator-source

open [http://localhost:9393/dashboard](http://localhost:9393/dashboard])

Click Add Application -> Register application

name: account-geode-sink
type: sink
URI: docker:account-geode-sink:0.0.1-SNAPSHOT

Click -< Import Application

## step create account pipeline with 

open [http://localhost:9393/dashboard](http://localhost:9393/dashboard)

Click Streams -> Streams -> Create Streams


```definition
account-generator-source --server.port=8080 | account-geode-sink --server.port=8080 --spring.profiles.active=stream --spring.application.name=scf-account-sink-stream --spring.rabbitmq.stream.host=rabbitmq --spring.data.gemfire.pool.locators=gemfire1-locator-0.gemfire1-locator[10334]
```

Click Create Stream 

Name: scf-acct

In Terminal

```shell
cd ~/dataServices/scdf
curl -OL https://cloud-native-data.s3.amazonaws.com/spring-cloud-dataflow-shell-2.8.1.jar
java -jar spring-cloud-dataflow-shell-2.8.1.jar

```

```shell
stream deploy --name scf-acct --properties "deployer.account-generator-source.kubernetes.requests.memory=1Gi, deployer.account-geode-sink.kubernetes.requests.memory=1Gi, deployer.account-generator-source.kubernetes.limits.memory=1Gi, deployer.account-geode-sink.kubernetes.limits.memory=1Gi"
```


```shell
stream list
```



## step 5 - review Account data GemFire region/table

```shell
    kubectl exec gemfire1-locator-0 -- gfsh -e "connect" -e "query --query='select id, bank_id, label, number, product_code from /Account'"
```

## destroy region

```shell
    kubectl exec gemfire1-locator-0 -- gfsh -e "connect" -e "destroy region --name=Account"
```

## destroy region

```shell
    kubectl exec gemfire1-locator-0 -- gfsh -e "connect" -e "create region --name=Account --type=PARTITION"
```

```shell
    kubectl exec gemfire1-locator-0 -- gfsh -e "connect" -e "query --query='select id, bank_id, label, number, product_code from /Account'"
```

## Create pipeline with replay

## In SCDF shell


```shell
cd ~/dataServices/scdf
java -jar spring-cloud-dataflow-shell-2.8.1.jar
```

```shell
stream destroy --name scf-acct
```

See argument --rabbitmq.streaming.replay=true

```shell
stream create --name scdf-account-stream-replay --definition "account-generator-source --server.port=8080 | account-geode-sink --rabbitmq.streaming.replay=true --server.port=8080 --spring.profiles.active=stream --spring.application.name=scf-account-sink-stream --spring.rabbitmq.stream.host=rabbitmq --spring.data.gemfire.pool.locators=gemfire1-locator-0.gemfire1-locator[10334]"
```

```shell
stream deploy --name scdf-account-stream-replay --properties "deployer.account-generator-source.kubernetes.requests.memory=1Gi, deployer.account-geode-sink.kubernetes.requests.memory=1Gi, deployer.account-generator-source.kubernetes.limits.memory=1Gi, deployer.account-geode-sink.kubernetes.limits.memory=1Gi"
```

```shell
watch kubectl get pods
```




View data in GemFire 

```shell
    kubectl exec gemfire1-locator-0 -- gfsh -e "connect" -e "query --query='select id, bank_id, label, number, product_code from /Account'"
```
