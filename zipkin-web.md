# zipkin-web

`zipkin-web` consists of 2 parts

1. scala backend: talk to `zipkin-query-service` to get span data, render page using `mustachejava`
2. javascript frontend: control UI

## scala backend

Scala backend utilizes twitter scala infrastructure libraries mainly

* finagle - https://github.com/twitter/finagle
* twitter server - https://github.com/twitter/twitter-server

Detail of dependencies libraries is in `project/Project.scala`

## javascript frontend

Javascript frontend is designed as single page application and utilizes various javascript libararies maily

* jquery
* requirejs - http://requirejs.org/
* flightjs - https://github.com/flightjs/flight: is used to bind event handler to DOM object
* bootstrap - https://github.com/twbs/bootstrap: provides CSS based template/components for UI responsive design

Any pages use layout `templates/v2/layout.mustache`

```html
    <script src='/app/libs/jquery/jquery.min.js'></script>
    <script src='/app/libs/jquery-timeago/jquery.timeago.js'></script>
    <script src='/app/libs/chosen/chosen.jquery.min.js'></script>
    <script src='/app/libs/jquery-cookie/jquery.cookie.js'></script>
    <script src='/app/libs/bootstrap-datepicker/js/bootstrap-datepicker.js'></script>
    <script src="/app/libs/momentjs/min/moment.min.js"></script>

    <script src='/app/libs/bootstrap/js/bootstrap.min.js'></script>
    <script src='/app/libs/requirejs/require.js' data-main='/app/js/main.js'></script>
```

The main entry point for the `flightjs` application is managed by `requirejs` in `app/js/main.js`

```js
    require(
      [
        'flight/lib/compose',
        'flight/lib/registry',
        'flight/lib/advice',
        'flight/lib/logger',
        'flight/lib/debug'
      ],
    
      function(compose, registry, advice, withLogging, debug) {
```

## Example of request

The http request `/trace/3` goes to the server, then it is routed according to `web/Main.scala`

```scala
    Seq(
      ("/app/", handlePublic(resourceDirs, typesMap, publicRoot)),
      ("/public/", handlePublic(resourceDirs, typesMap, publicRoot)),
      ("/", addLayout andThen handleIndex(queryClient)),
      ("/traces/:id", addLayout andThen handleTraces(queryClient)),
```     
      
The `addLayout` is filter object in `web/Handlers.scala`

```scala
    val addLayout: Filter[Request, Renderer, Request, Renderer] =
      Filter.mk[Request, Renderer, Request, Renderer] { (req, svc) =>
        svc(req) map { renderer =>
          response: Response => {
            renderer(response)
            val r = MustacheRenderer("v2/layout.mustache", Map("body" -> response.contentString))
            r(response)
          }
        }
      }
```

`templates/v2/layout.mustache` is the template for all pages. It loads bundle of css and js files required for display and control UI elements of each page.

Method`handleTraces` in `web/Handlers.scala` returns `Service[Request, Render]`

```scala
    def handleTraces(client: ZipkinQuery[Future]): Service[Request, Renderer] =
      Service.mk[Request, Renderer] { req =>
        pathTraceId(req.path.split("/").lastOption) map { id =>
          client.getTraceCombosByIds(Seq(id), getAdjusters(req)) flatMap {
            case Seq(combo) => Future.value(renderTrace(combo))
            case _ => NotFound
          }
        } getOrElse NotFound
      }
```

`ZipkinQuery[Future]` is thrift client side that sends request to the `zipkin-query-service` using thrift protocol.
Method `renderTrace` in `web/Handlers.scala` performs final rendering of the result

```scala
    private[this] def renderTrace(combo: TraceCombo): Renderer = {
      val trace = combo.trace.toTrace
      val traceStartTimestamp = trace.getStartAndEndTimestamp.map(_.start).getOrElse(0L)
      val childMap = trace.getIdToChildrenMap
      val spanMap = trace.getIdToSpanMap
```
