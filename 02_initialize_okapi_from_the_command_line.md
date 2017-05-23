# Initialize Okapi from the command line

## Start the Okapi Gateway
```shell
$ cd $FOLIO_ROOT/okapi   # Set in lesson one of the tutorial
$ java -Dloglevel=DEBUG -jar okapi-core/target/okapi-core-fat.jar dev
  12:08:11 INFO  MainVerticle         git: git@github.com:folio-org/okapi 225c9c1e03c29459da430f93110abb30378e1394
  12:08:11 INFO  MainVerticle         clusterManager not in use
  12:08:11 INFO  MainVerticle         Proxy using inmemory storage
  12:08:12 WARN  Storage              Storage.resetDatabases: NORMAL
  12:08:12 INFO  TenantWebService     All tenants deployed
  12:08:12 INFO  MainVerticle         API Gateway started PID 64161@Walkabout.lan. Listening on port 9130
```

The `dev` parameter starts the Okapi Gateway in 'development mode' (a known, clean state without any modules or tenants defined).
The Okapi Gateway is using an in-memory database (a built-in PostgreSQL database can be specified by adding `-Dstorage=postgres` before the `-jar` parameter).
We are going to run the Okapi Gateway with debugging turned on so you can see the effect of the requests passing through the gateway.
The last line of output tells us that the Okapi Gateway is running on port 9130.
We can use two curl commands to list the Okapi Modules and tenants known to the gateway:

```shell
$ curl -i -w '\n' -X GET http://localhost:9130/_/proxy/modules
  HTTP/1.1 200 OK
  Content-Type: application/json
  Content-Length: 3

  [ ]

$ curl -i -w '\n' -X GET http://localhost:9130/_/proxy/tenants
  HTTP/1.1 200 OK
  Content-Type: application/json
  Content-Length: 3

  [ ]
```

Note that in both cases what was returned from the gateway are empty JSON lists, meaning that the newly initialized Okapi Gateway has no configured modules or tenants.

Paths starting with `/_/` are core Okapi Gateway services.
`/_/proxy` is one core service; it is used to //TODO define
Another core service is `/_/discovery`; it is used to interact with nodes in the Okapi cluster.
More details about these core services can be found in the [Okapi Guide and Reference](https://github.com/folio-org/okapi/blob/master/doc/guide.md#deployment-and-discovery).
In the next section we will use these two core services to register a module and a tenant.

## Defining the _okapi-test-module_ within the Okapi Gateway
Defining a module with the Okapi Gateway occurs in three steps: Registering, Deploying, and Configuring the proxy.

### Registering _okapi-test-module_
To tell Okapi that we want to use the okapi-test-module, we create a JSON structure of a "Module Descriptor" and POST it to Okapi.
The command below creates a Module Descriptor JSON structure.
(Leave the Okapi Gateway running in one window and execute this command in another window.)

```shell
$ cd $FOLIO_ROOT
$ cat > okapi-proxy-test-basic.json <<END
  {
  "id" : "test-module",
    "name" : "Okapi test module",
    "provides" : [ {
        "id" : "test-basic",
        "version" : "2.2.3"
      },
      {
        "id" : "_tenant",
        "version" : "1.0.0"
    } ],
    "routingEntries" : [ {
      "methods" : [ "GET", "POST" ],
      "path" : "/testb",
      "level" : "30",
      "type" : "request-response",
      "permissionsRequired" : [ "test-basic.needed" ],
      "permissionsDesired" : [ "test-basic.extra" ]
    } ],
    "launchDescriptor" : {
      "exec" : "java -Dport=%p -jar okapi-test-module/target/okapi-test-module-fat.jar"
    }
  }
END
```

This is a module descriptor for `test-module` (the first `id` key-value pair).
It `provides` two interfaces: `test-basic` and `_tenant`.
The `test-basic` interface is how clients will communicate with the Okapi Module (as we did directly in the previous lesson).
Interfaces beginning with an underscore (such as `_tenant`) are reserved system interfaces; in this case, an interface that is called when a module is enabled or disabled for a tenant.
The `routingEntries` dictionary tells the gateway:
* which HTTP methods are expected
* the path registered in the gateway for which this interface will receive requests
* the priority of this module relative to other modules registered on the same path as the chain of modules is constructed by the gateway
* the type of response provided by this module interface
* permissions required (enforced by the Okapi Gateway)
* permissions desired (a list of requester's permissions that the business logic of this module needs to know about)
The final entry, `launchDescriptor`, tells the gateway how to start the Okapi Module.

To deploy the module, this JSON is POSTed to the `/_/proxy/modules` core service:

```shell
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
   -d @okapi-proxy-test-basic.json http://localhost:9130/_/proxy/modules
  HTTP/1.1 201 Created
  Content-Type: application/json
  Location: /_/proxy/modules/test-module
  Content-Length: 548

  {
   "id" : "test-module",
   "name" : "Okapi test module",
  ...
  }
```

The Okapi Gateway responds with an [HTTP 201 ("Created")](https://httpstatusdogs.com/201-created) status code and returns the request body as the response body.
There is also a `Location:` header that is the URL of the newly created module.
That address can be used with other HTTP verbs ('GET' to read, 'POST' to update, 'DELETE' to remove) as one would expect for a RESTful resource.
Run these commands to list the modules known to the gateway, delete _test-module_, and list the modules again to see for yourself:

```shell
$ curl -i -w '\n' -X GET http://localhost:9130/_/proxy/modules

$ curl -i -w '\n' -X DELETE http://localhost:9130/_/proxy/modules/test-module

$ curl -i -w '\n' -X GET http://localhost:9130/_/proxy/modules

$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
   -d @okapi-proxy-test-basic.json http://localhost:9130/_/proxy/modules

$ curl -i -w '\n' -X GET http://localhost:9130/_/proxy/modules
```

### Deploying _okapi-test-module_
Although we have registered the module with the Okapi Gateway, we have not yet instantiated the module so there is something available to respond to requests.
The module must also be deployed on one node (or more, in the case of clusters).
First, let's query the discovery service for a list of nodes it knows about in the cluster:

```shell
$ curl -i -w '\n' -X GET http://localhost:9130/_/discovery/nodes
  HTTP/1.1 200 OK
  Content-Type: application/json
  Content-Length: 67

  [ {
    "nodeId" : "localhost",
    "url" : "http://localhost:9130"
  } ]
```

The response body is a JSON document that, for our Okapi Gateway instance running with the 'dev' option, show just one node (`nodeId` of `localhost`).

To deploy the module, we create a "Deployment Descriptor" JSON document:

```shell
$ cat > okapi-deploy-test-basic.json <<END
{
  "srvcId" : "test-module",
  "nodeId" : "localhost"
}
END
```

Then post this document to the `/_/discovery` core service:

```shell
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
   -d @okapi-deploy-test-basic.json http://localhost:9130/_/discovery/modules
  HTTP/1.1 201 Created
  Content-Type: application/json
  Location: /_/discovery/modules/test-module/localhost-9131
  Content-Length: 232

  {
   "instId" : "localhost-9131",
   "srvcId" : "test-module",
   "nodeId" : "localhost",
   "url" : "http://localhost:9131",
   "descriptor" : {
     "exec" : "java -Dport=%p -jar okapi-test-module/target/okapi-test-module-fat.jar"
   }
  }
```

The Okapi Gateway server returns an [HTTP 201 ("Created")](https://http.cat/201) status code responds with a JSON document that includes additional information:
* an instance identifier ('instId') for this particular deployment of the module
* a URL where the instance will receive requests
* a copy of the launch descriptor line from the Module Descriptor

In addition, the gateway returns a `Location:` header that is the URL of this deployed instance.
As with module descriptors, that URL can be used to retrieve information about the deployed instance (HTTP `GET`) and stop the deployed instance (HTTP `DELETE`).
Be sure to also flip back to the terminal session running the Okapi Gateway to see the server-side debug messages.

One important note: in our developer instance, we can access the _okapi-test-module_ directly at the URL http://localhost:9131:

```shell
$ curl -i -w '\n' -X GET http://localhost:9131/testb
  HTTP/1.1 200 OK
  Content-Type: text/plain
  Content-Length: 8

  It works
```

In a production system, there will probably be a firewall that prevents direct access to the Okapi Module interfaces.
Requests for Okapi Module services must go through the Okapi Gateway to ensure that access control and tenant restrictions are honored.

## Creating a tenant
As noted above, all requests should be made through the Okapi Gateway.
Let's try that now:

```shell
$ curl -i -w '\n' -X GET http://localhost:9130/testb
  HTTP/1.1 403 Forbidden
  Content-Type: text/plain
  Content-Length: 14

  Missing Tenant
```

Okapi is inherently a multi-tenant system, so each request to an Okapi Module must be performed on behalf of a tenant.
Let's set up our first tenant:

```shell
$ cat > okapi-tenant.json <<END
{
  "id" : "testlib",
  "name" : "Test Library",
  "description" : "Our Own Test Library"
}
END
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
   -d @okapi-tenant.json http://localhost:9130/_/proxy/tenants
  HTTP/1.1 201 Created
  Content-Type: application/json
  Location: /_/proxy/tenants/testlib
  Content-Length: 91

  {
   "id" : "testlib",
   "name" : "Test Library",
   "description" : "Our Own Test Library"
  }
```
### Enable _test-module_ for the _testlib_ tenant
Now that the tenant is created, we need to enable _test-module_ for that tenant:

```shell
$ cat > okapi-enable-basic.json <<END
{
  "id" : "test-module"
}
END
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
   -d @okapi-enable-basic.json http://localhost:9130/_/proxy/tenants/testlib/modules
  HTTP/1.1 201 Created
  Content-Type: application/json
  Location: /_/proxy/tenants/test-basic
  Content-Length: 25

  {
   "id" : "test-basic"
  }
```

Switch back to the Okapi Gateway terminal to see the debug output for this last request.

```
16:04:04 DEBUG TenantManager        findTenantInterface: prov: [{"id":"test-basic","version":"2.2.3"},{"id":"_tenant","version":"1.0.0"}]
16:04:04 DEBUG TenantManager        findTenantInterface: Looking at test-basic
16:04:04 DEBUG TenantManager        findTenantInterface: Looking at _tenant
16:04:04 DEBUG TenantWebService     enableModule Url: http://localhost:9131 and /_/tenant
16:04:04 DEBUG TenantWebService     Added X-Okapi-Tenant : testlib
16:04:04 DEBUG OkapiClient          OkapiClient: POST request to http://localhost:9131/_/tenant
16:04:04 DEBUG OkapiClient          OkapiClient: adding header Accept: */*
16:04:04 DEBUG OkapiClient          OkapiClient: adding header X-Okapi-Tenant: testlib
16:04:04 DEBUG OkapiClient          OkapiClient: adding header Content-Type: application/json; charset=UTF-8
16:04:04 DEBUG OkapiClient          OkapiClient Buffering response POST request to okapi-test-module tenant service for tenant testlib

16:04:04 INFO  MainVerticle         POST request to okapi-test-module tenant service for tenant testlib
16:04:04 DEBUG OkapiClient          OkapiClient Buffering response {
  "module_to" : "test-module"
}
16:04:04 DEBUG TenantWebService     enableModule: Init request to test-module succeeded
```

In this string of messages you can see the Okapi Gateway finding the tenant and, as a "loopback" OkapiClient, post a request to the reserved `/_/tenant` interface of _okapi-test-module_ on http://localhost:9191 (http://localhost:9191/_/tenant).
This allows the Okapi Module to perform an initialization routine when it is added to a new tenant.
Note the log line that says an `X-Okapi-Tenant: testlib` header is being added to the request being made by OkapiClient.
We, too, must also add an `X-Okapi-Tenant` header when sending a request to an Okapi Module:

```shell
$ curl -i -w '\n' -X GET -H 'X-Okapi-Tenant: testlib' http://localhost:9130/testb
  HTTP/1.1 200 OK
  Content-Type: text/plain
  X-Okapi-Trace: GET - Okapi test module http://localhost:9131/testb : 200 12629us
  Transfer-Encoding: chunked

  It works
```

Also note that the core interfaces behave as we expect them to by returning JSON lists.
Try these commands:

```shell
$ curl -i -w '\n' -X GET http://localhost:9130/_/proxy/tenants

$ curl -i -w '\n' -X GET http://localhost:9130/_/proxy/tenants/testlib/modules
```
