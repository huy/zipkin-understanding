# Understand Zipkin

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

Zipkin consists of three processes

1. zipkin-collector: thrift server listening on port 9410. It receives and persists span data to span database
2. zipkin-query: thrift server listening on port 9411. It accesses span database to answer query from zipkin-web
3. zipkin-web: web server that provides UI interface on port 8080. It connect to zipkin-query to retrieve span data


