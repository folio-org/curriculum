# Clone, Build, and Explore Okapi

## Clone Okapi

```shell
$ cd $FOLIO_ROOT   # Set in lesson one of the tutorial
$ git clone --recursive https://github.com/folio-org/okapi.git
  Cloning into 'okapi'...
  remote: Counting objects: 11346, done.
  remote: Compressing objects: 100% (55/55), done.
  remote: Total 11346 (delta 21), reused 0 (delta 0), pack-reused 11283
  Receiving objects: 100% (11346/11346), 1.95 MiB | 415.00 KiB/s, done.
  Resolving deltas: 100% (5820/5820), done.
$ cd okapi
$ mvn install
  [...]
  [INFO] ------------------------------------------------------------------------
  [INFO] Reactor Summary:
  [INFO]
  [INFO] okapi .............................................. SUCCESS [  0.464 s]
  [INFO] okapi-common ....................................... SUCCESS [  5.545 s]
  [INFO] okapi-test-module .................................. SUCCESS [  1.191 s]
  [INFO] okapi-test-auth-module ............................. SUCCESS [  0.731 s]
  [INFO] okapi-test-header-module ........................... SUCCESS [  0.670 s]
  [INFO] okapi-core ......................................... SUCCESS [ 27.901 s]
  [INFO] ------------------------------------------------------------------------
  [INFO] BUILD SUCCESS
  [INFO] ------------------------------------------------------------------------
  [INFO] Total time: 37.231 s
  [INFO] Finished at: 2017-02-27T16:56:58-05:00
  [INFO] Final Memory: 62M/490M
  [INFO] ------------------------------------------------------------------------
```

## Interact with the test modules as if you are the Okapi Gateway
The `mvn install` command builds _okapi-core_ (the Okapi Gateway server) and _okapi-common_ (utilities used by both the gateway server and by modules) along with three simple test Okapi Modules.
Before starting the Okapi Gateway, we are going to look at one of the three test Okapi Modules and interact with them as if we are the Okapi Gateway.
Okapi is an implementation of the "API Gateway" microservices pattern.
As such, the Okapi Gateway accepts RESTful requests from clients and routes them through a series of RESTful interfaces ("Okapi Modules") to build a response that is ultimately returned to the client.
(For more details about the Okapi architecture, see the [Okapi Guide and Reference](https://github.com/folio-org/okapi/blob/master/doc/guide.md#architecture).)

To see what this interaction between _okapi-core_ and Okapi Modules looks like, let's start an Okapi Module and send it some requests via `curl`.

### Interact with Okapi-test-module
The _Okapi-test-module_ is a very simple Okapi Module.
If you make a GET request to it, it will reply "It works".
If you POST something to it, it will reply with "Hello" followed by whatever you posted.
First we start the test module:

```shell
$ cd $FOLIO_ROOT/okapi
$ java -jar okapi-test-module/target/okapi-test-module-fat.jar
  13:53:00 INFO  MainVerticle         Starting okapi-test-module 42510@Walkabout.lan on port 8080
  13:53:00 INFO  ertxIsolatedDeployer Succeeded in deploying verticle
```

With the _Okapi-test-module_ now listening on port 8080, in another terminal window send a simple `curl` command.
(Note that the `-i` command line option tells `curl` to output the response headers in addition to the response body, and the `-w '\n'` option adds a newline to the end of the response body to ensure the shell prompt starts on a new line.)

```shell
$ curl -i -w '\n' http://localhost:8080/testb
  HTTP/1.1 200 OK
  Content-Type: text/plain
  Content-Length: 8

  It works
```

Next make a HTTP POST request (using `-x POST`) and send the string `Testing Okapi` (using the -d command line option):

```shell
$ curl -i -w '\n' -X POST -d "Testing Okapi" http://localhost:8080/testb
  HTTP/1.1 200 OK
  Content-Type: text/plain
  Transfer-Encoding: chunked

  Hello Testing Okapi
```

Okapi modules would typically send and receive JSON content bodies, but in these examples simple strings are sent and returned.

### Interact with Okapi-test-module with headers
As a RESTful interface, the Okapi Gateway communicates key data to Okapi Modules and to the client using HTTP headers.
For example: as Okapi Modules are chained together by the Okapi Gateway, a module may want to signal to the gateway that it encountered an exception and must interrupt the chain.

If you haven't done so already, start _Okapi-test-module_ in a terminal window:

```shell
$ cd $FOLIO_ROOT/okapi
$ java -jar okapi-test-module/target/okapi-test-module-fat.jar
  13:53:00 INFO  MainVerticle         Starting okapi-test-module 42510@Walkabout.lan on port 8080
  13:53:00 INFO  ertxIsolatedDeployer Succeeded in deploying verticle
```

Next, in a separate terminal, send an HTTP GET request with an `X-my-header: blah` header (using the `-H` command line argument):

```shell
$ curl -i -w '\n' -X GET -H 'X-my-header: blah' http://localhost:8080/testb
  HTTP/1.1 200 OK
  Content-Type: text/plain
  Content-Length: 12

  It worksblah
```

This example appends the contents of the `X-my-header` to the response body.
If we add an 'X-stop-here' header, the module returns the `X-Okapi-Stop` header (which would trigger the exception handling in the Okapi gateway):

```shell
$ curl -i -w '\n' -X GET -H 'X-my-header: blah' \
    -H 'X-stop-here: because I said so.' \
    http://localhost:8080/testb
  HTTP/1.1 200 OK
  Content-Type: text/plain
  X-Okapi-Stop: because I said so.
  Content-Length: 12

  It worksblah
```

(Note that there is not an `X-stop-here` request header defined in Okapi.  This is a header specific to the _Okapi-test-module_ that forces the return of the Okapi-defined `X-Okapi-Stop` response header.)

The corresponding Okapi Module code that is handling this interaction can be found in the [okapi-test-module/.../MainVerticle.java]( https://github.com/folio-org/okapi/blob/master/okapi-test-module/src/main/java/org/folio/okapi/sample/MainVerticle.java) file.

Return to the terminal window with _Okapi-test-module_ running and press Control-C to exit it.

### Interact with Okapi-test-auth-module
_Okapi-test-auth-module_ is a simple module that illustrates authentication in Okapi.
The Okapi Gateway itself does not handle authentication; rather, it delegates authentication to an Okapi Module that operates at a very high level in the chain of module requests orchestrated by the Okapi Gateway.

First we start the test auth module:

```shell
$ cd $FOLIO_ROOT/okapi
$ java -jar okapi-test-auth-module/target/okapi-test-auth-module-fat.jar
  02:15:42 INFO  MainVerticle         Starting auth 26062@contrib-jessie on port 9020
  02:15:42 INFO  ertxIsolatedDeployer Succeeded in deploying verticle
```

The `/login` path of _Okapi-test-auth-module_ takes a simple JSON document with a 'username' and a 'password' value.
If the password is the same as the username with the string '-password' appended, then the authentication is successful and a token is returned.

```shell
$ curl -i -w '\n' -X POST -H 'X-Okapi-Tenant: blah' \
    -d '{"username": "seb", "password": "seb-password"}' \
    http://localhost:9020/login
  HTTP/1.1 200 OK
  Content-Type: application/json
  X-Okapi-Token: dummyJwt.eyJzdWIiOiJzZWIiLCJ0ZW5hbnQiOm51bGx9.sig
  Content-Length: 47

  {"username": "seb", "password": "seb-password"}
```

Now try to send a JSON document in which the password does not match the expected value.
Note that the response returns an [HTTP 401 "Unauthorized"](https://http.cat/401) status.

Another path supplied by _Okapi-test-auth-module_ is `/check`; it checks an _X-Okapi-Token_ to see if it is valid.
```shell
$ curl -i -w '\n' -X GET -H 'X-Okapi-Tenant: blah' \
    -H 'X-Okapi-Token: dummyJwt.eyJzdWIiOiJzZWIiLCJ0ZW5hbnQiOm51bGx9.sig' \
    http://localhost:9020/check
  HTTP/1.1 202 Accepted
  X-Okapi-Token: dummyJwt.eyJzdWIiOiJzZWIiLCJ0ZW5hbnQiOm51bGx9.sig
  X-Okapi-Module-Tokens: {}
  Content-Type: text/plain
  Transfer-Encoding: chunked
```

Note that the response returns an [HTTP 202 "Accepted"](https://httpstatusdogs.com/202-accepted) status.
Try sending a request without the required _X-Okapi-Tenant_ header.
All requests are expected to send the _X-Okapi-Tenant_ header.
