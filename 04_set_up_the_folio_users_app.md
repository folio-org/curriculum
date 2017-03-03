# Set up the Okapi Users app

In lesson one, we deployed Stripes and demonstrated communication between the browser and the Stripes components.
In lessons two and three, we deployed the Okapi Gateway as well as a test Okapi Module and examined the communication between them.
In lesson four we are connecting Stripes to the Okapi Gateway and adding the Users app.
There are two components to the Users app: the Stripes UI component and the Okapi Module component.
We will start first with the Stripes UI component.

## Add the Users app UI component to the Stripes UI Server
Remember in $FOLIO_ROOT/stripes-tutorial-platform we have two configuration files: `package.json` and `stripes.config.js`.
Each will need changes to add the Users app.

### Modify _package.json_
The `package.json` file needs a new entry in the dependency section.
The new file should look like:
```JavaScript
{
  "scripts": {
    "build": "stripes build stripes.config.js",
    "stripes": "stripes",
    "start": "stripes dev stripes.config.js"
  },
  "dependencies": {
    "@folio/stripes-core": "^0.0.11",
    "@folio/users": "^0.0.1-test",
    "@folio/trivial": "^0.0.2-test"
  }
}
```

### Modify _stripes.config.js_
The `stripes.config.js` file needs not only a line for adding the Users UI component; it also needs connection information for the Okapi Gateway.
```javascript
module.exports = {
  okapi: { 'url':'http://localhost:9130', 'tenant':'testlib' },
  config: { reduxLog: true, disableAuth: true },
  modules: {
    '@folio/trivial': {},
    '@folio/users': {}
  }
};
```

The `okapi` line in `stripes.config.js` gives the Stripes UI Server the URL of the Okapi gateway and the name of the tenant being used by this UI Server instance.

### Rebuild Stripes Server
With the Users UI components added to the UI Server configuration, use the `yarn install` command to download and configure the necessary modules.
```shell
$ cd $FOLIO_ROOT/stripes-tutorial-platform
$ yarn install
  yarn install v0.20.3
  warning No license field
  [1/4] 🔍  Resolving packages...
  [2/4] 🚚  Fetching packages...
  [3/4] 🔗  Linking dependencies...
  warning "eslint-import-resolver-webpack@0.8.1" has unmet peer dependency "eslint-plugin-import@>=1.4.0".
  [4/4] 📃  Building fresh packages...
  success Saved lockfile.
  ✨  Done in 12.12s.
$ yarn start
  yarn start v0.20.3
  $ stripes dev stripes.config.js
  Listening at http://localhost:3000
  webpack built 97b8243748d40fd3a9c1 in 14004ms
```

The Stripes portion of the Users app is now running at  [http://localhost:3000/users](http://localhost:3000/users).
At this point you likely receive one of three possible errors:
* `ERROR: in module @folio/users operation FETCH on resource users failed, saying: Network request failed`: the Okapi Gateway is not running; see the Java command line at the [start of lesson three](http://dev.folio.org/curriculum/03_initialize_okapi_from_the_command_line#start-the-okapi-gateway)
* `ERROR: in module @folio/users operation FETCH on resource users failed, saying: No such Tenant testlib`: your Okapi Gateway is newly restarted and does not have the _testlib_ tenant; return to [lesson three under the Creating a Tenant heading](03_initialize_okapi_from_the_command_line#creating-a-tenant) to post `okapi-tenant.json` to `/_/proxy/tenants`.
* `ERROR: in module @folio/users operation FETCH on resource users failed, saying: ` (empty error message): you are in the right place.  The Users Okapi Module is not yet [registered](03_initialize_okapi_from_the_command_line#registering-okapi-test-module), [deployed](03_initialize_okapi_from_the_command_line#deploying-okapi-test-module) and/or [enabled](03_initialize_okapi_from_the_command_line#enable-test-module-for-the-testlib-tenant) for the 'testlib' tenant.  The lack of an error message is a [known issue](https://issues.folio.org/browse/STRIPES-224).

If you did not receive an error message, then your Okapi Gateway is properly configured.
The next tutorial section may be review.

## Add the Users app Okapi Module to the Okapi Gateway

### Fetch and build the Users app Okapi Module
```shell
$ cd $FOLIO_ROOT
$ git clone --recursive https://github.com/folio-org/mod-users.git
  Cloning into 'mod-users'...
  remote: Counting objects: 645, done.
  remote: Compressing objects: 100% (41/41), done.
  remote: Total 645 (delta 9), reused 0 (delta 0), pack-reused 595
  Receiving objects: 100% (645/645), 88.87 KiB | 0 bytes/s, done.
  Resolving deltas: 100% (201/201), done.
  Submodule 'raml-util' (https://github.com/folio-org/raml.git) registered for path 'raml-util'
  Cloning into '/Users/peter/code/folio-trial/mod-users/raml-util'...
  remote: Counting objects: 409, done.
  remote: Compressing objects: 100% (7/7), done.
  remote: Total 409 (delta 2), reused 0 (delta 0), pack-reused 397
  Receiving objects: 100% (409/409), 85.73 KiB | 0 bytes/s, done.
  Resolving deltas: 100% (238/238), done.
  Submodule path 'raml-util': checked out 'b8dcdf7e1f2d9b5c4f4cc8591edf79d07deb9df6'
$ cd mod-users
$ mvn install
  [...]
  [INFO] ------------------------------------------------------------------------
  [INFO] BUILD SUCCESS
  [INFO] ------------------------------------------------------------------------
  [INFO] Total time: 01:21 min
  [INFO] Finished at: 2017-02-24T16:52:58-05:00
  [INFO] Final Memory: 74M/532M
  [INFO] ------------------------------------------------------------------------
```

### Register and Deploy the Users app Okapi Module
The Git repository for the Users app Okapi Module has a [Module Descriptor](https://github.com/folio-org/mod-users/blob/master/ModuleDescriptor.json) that can be used to register the Users app Okapi Module.

```shell
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
   -d @ModuleDescriptor.json http://localhost:9130/_/proxy/modules
  HTTP/1.1 201 Created
  Content-Type: application/json
  Location: /_/proxy/modules/users-module
  Content-Length: 852

  {
    "id" : "users-module",
    "name" : "users",
   [...]
  }
```

You will also need to deploy the module with a Deployment Descriptor:

```shell
$ cd $FOLIO_ROOT
$ cat > okapi-deploy-mod-users.json <<END
{
  "srvcId" : "users-module",
  "nodeId" : "localhost",
  "descriptor" : {
    "exec" : "java -jar ../mod-users/target/mod-users-fat.jar -Dhttp.port=%p embed_postgres=true"
  }
}
END
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
  -d @okapi-deploy-mod-users.json http://localhost:9130/_/discovery/modules
  HTTP/1.1 201 Created
  Content-Type: application/json
  Location: /_/discovery/modules/users-module/localhost-9131
  Content-Length: 245

  {
    "instId" : "localhost-9131",
    "srvcId" : "users-module",
    "nodeId" : "localhost",
    "url" : "http://localhost:9131",
    "descriptor" : {
      "exec" : "java -jar ../mod-users/target/mod-users-fat.jar -Dhttp.port=%p embed_postgres=true"
    }
  }
```

Finally, you'll need to enable the Okapi Users app module for the test tenant:

```
$ cat > okapi-enable-users.json <<END
{
  "id" : "users-module"
}
END
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
   -d @okapi-enable-users.json http://localhost:9130/_/proxy/tenants/testlib/modules
HTTP/1.1 201 Created
Content-Type: application/json
Location: /_/proxy/tenants/testlib/modules/users-module
Content-Length: 27

{
  "id" : "users-module"
}
```

The FOLIO Users app is now available at [http://localhost:3000/users](http://localhost:3000/users).
You'll see the message `No results found for "". Please check your spelling and filters` because no users have been added.

## Add users to FOLIO
For testing purposes, the FOLIO development team has JSON documents representing facticious users that can be used to populate the dev FOLIO environment.
The JSON documents are located in [https://github.com/folio-org/folio-ansible/tree/master/roles/mod-users-data/files](https://github.com/folio-org/folio-ansible/tree/master/roles/mod-users-data/files).

```shell
$ cd $FOLIO_ROOT
$ curl -s -O https://raw.githubusercontent.com/folio-org/folio-ansible/master/roles/mod-users-data/files/User000.json
$ curl -s -O https://raw.githubusercontent.com/folio-org/folio-ansible/master/roles/mod-users-data/files/User001.json
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
    -H 'X-Okapi-Tenant: testlib' -d @User000.json http://localhost:9130/users
  HTTP/1.1 201 Created
  Content-Type: application/json
  Location: 82738e56-517b-4878-aece-f275d0445f59
  Host: localhost:9130
  User-Agent: curl/7.51.0
  Accept: */*
  X-Okapi-Tenant: testlib
  X-Okapi-Url: http://localhost:9130
  X-Okapi-Permissions-Required: users.create
  X-Okapi-Module-Permissions: {}
  X-Okapi-Trace: POST - users http://localhost:9131/users : 201 349307us
  Transfer-Encoding: chunked

  {
    "username" : "dante",
    "id" : "EF84048E-559F-48EF-BE07-CF6AB8E91670",
    "active" : true,
    "type" : "patron",
    "patron_group" : "on_campus",
    "meta" : {
      "creation_date" : "2009-10-09",
      "last_login_date" : "1986-04-16"
    },
    "personal" : {
      "last_name" : "Shanahan",
      "first_name" : "Shakira",
      "email" : "oral@harvey-dach-and-medhurst.ht"
    }
  }
$ curl -i -w '\n' -X POST -H 'Content-type: application/json' \
    -H 'X-Okapi-Tenant: testlib' -d @User001.json http://localhost:9130/users
  HTTP/1.1 201 Created
  Content-Type: application/json
  Location: 6cb3ef6f-cab5-4516-8bfb-da53bf6e5fdc
  Host: localhost:9130
  User-Agent: curl/7.51.0
  Accept: */*
  X-Okapi-Tenant: testlib
  X-Okapi-Url: http://localhost:9130
  X-Okapi-Permissions-Required: users.create
  X-Okapi-Module-Permissions: {}
  X-Okapi-Trace: POST - users http://localhost:9131/users : 201 40984us
  Transfer-Encoding: chunked

  {
    "username" : "darrell",
    "id" : "73F41A0B-B5BC-4413-875B-3334E0436DD6",
    "active" : true,
    "type" : "patron",
    "patron_group" : "on_campus",
    "meta" : {
      "creation_date" : "1998-03-15",
      "last_login_date" : "2000-02-27"
    },
    "personal" : {
      "last_name" : "Oberbrunner",
      "first_name" : "Eladio",
      "email" : "dayna@zulauf-jast.la"
    }
  }
```

You can also use the _New User_ button in the browser interface to add a user.
