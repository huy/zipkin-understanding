# Understand Zipkin

Zipkin consists of three processes

1. `zipkin-collector-service`: thrift server listening on port `9410`. It receives and persists span data to span database
2. `zipkin-query-service`: thrift server listening on port `9411`. It accesses span database to answer query from zipkin-web
3. `zipkin-web`: web server that provides UI interface on port `8080`. It connect to zipkin-query to retrieve span data

## Compilation

Run `sbt` from `./bin` folder

    ./bin/sbt

Switch to a project

    >project zipkin-web
    
Compile

    >compile

Create package distribution

    >package-dist

The result is stored in e.g `zipkin-web/dist/zipkin-web.zip`

## Running

### zipkin-collector-service

create package distribution for `zipkin-collector-service`

unzip the `zipkin-collector-service/dist/zipkin-collector-service.zip`

run

    java -jar zipkin-collector-service-1.2.0-SNAPSHOT.jar -f config/collector-dev.scala

the config file `config/collector-*.scala` specifies span database, for dev it is sqlite. 

the collector service utilizes thrift frame transport so we need to use it when writting span feeder.

### zipkin-query-service

create package distribution for `zipkin-query-service`

unzip the `zipkin-query-service/dist/zipkin-query-service.zip`

run

    java -Xmx4G -Xss8M -jar zipkin-query-service-1.2.0-SNAPSHOT.jar -f config/collector-dev.scala

The config file `config/query-*.scala` specifies span database, for dev it is sqlite. `zipkin-query` is memory hungry, the default java memory setting is not enough to make a decent test.

When running make sure that `zipkin-collector` and `zipkin-query` access the same database

### zipkin-web

create package distribution for `zipkin-web`

unzip the `zipkin-web/dist/zipkin-web.zip`

run

    java -Xmx4G -jar zipkin-web-1.2.0-SNAPSHOT.jar -zipkin.web.resourcesRoot=/home/zipkin/deploy/resources

We need to specify `-zipkin.web.resourcesRoot` to point to correct folder containing web static assets

