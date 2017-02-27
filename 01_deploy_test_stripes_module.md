# Deploy test Stripes module

## Inform the Yarn package manager of the FOLIO UI registry location
(Note: the output of commands is artificially indented from the command line to call out the command lines.)
```
$ yarn config set @folio:registry https://repository.folio.org/repository/npm-folio/
  yarn config v0.20.3
  success Set "@folio:registry" to "https://repository.folio.org/repository/npm-folio/".
  ✨  Done in 0.04s.
```

## Set up a Stripes Development UI Server

Create an empty directory to hold the Stripes UI Server configuration (called `stripes-tutorial-platform`).
(Note: the first line of this block of shell commands saves the 'root' directory from which subsequent directories and files in this tutorial will be created.)
(Note: if using the VirtualBox guest setup, it is recommended to first `cd /vagrant` before creating the empty directory. Doing so makes the Okapi and Stripes files available from the host operating system in the same directory the Vagrantfile file is located.)

```shell
$ export FOLIO_ROOT=`pwd`    # Store the current directory we are in for later use
$ mkdir $FOLIO_ROOT/stripes-tutorial-platform
$ cd $FOLIO_ROOT/stripes-tutorial-platform
```

In that directory, put two files: `package.json` and `stripes.config.js`.

### Contents of `package.json`

`package.json` is a [Node Package Manager (NPM) configuration file](https://docs.npmjs.com/files/package.json).
It is a JSON file that contains two dictionaries: _scripts_ and _dependencies_.
The _scripts_ dictionary specifies name-value pairs of commands that are used by Yarn to build and run the platform.
The _dependencies_ dictionary lists packages (and specific versions) that make up the Stripes client bundles.

At this stage of the Curriculum we are setting up a stand-alone Stripes UI Server instance that does not communicate with an Okapi back-end.
The `package.json` below builds Stripes with a 'trivial' client bundle.

```json
{
  "scripts": {
    "build": "stripes build stripes.config.js",
    "stripes": "stripes",
    "start": "stripes dev stripes.config.js"
  },
  "dependencies": {
    "@folio/stripes-core": "^0.0.9-test",
    "@folio/trivial": "^0.0.2-test"
  }
}
```

### Contents of `stripes.config.js`
`stripes.config.js` contains the configuration details for the Stripes UI Server.
It is referenced in _scripts_ dictionary of `package.json`.
It is a JSON file with two required dictionaries: _config_ and _modules_.
The _config_ dictionary contains one key-value pair setting `disableAuth` to `true`.
(Authentication and authorization are covered later in this tutorial.)
The _modules_ dictionary contains another dictionary of Stripes package and their configuration.
The key in this dictionary is the name of the package to load from the FOLIO UI registry.
The value in this dictionary are parameters that can override the default settings of the Stripes package.

```javascript
module.exports = {
  config: { disableAuth: true },
  modules: {
    '@folio/trivial': {}
  }
};
```
### Build Stripes with the 'trivial' client bundle

Download/update Stripes along with its dependencies and modules, and link them together using the `yarn install` command:

```bash
$ yarn install
  yarn install v0.20.3
  info No lockfile found.
  warning No license field
  [1/4] 🔍  Resolving packages...
  [2/4] 🚚  Fetching packages...
  [3/4] 🔗  Linking dependencies...
  [4/4] 📃  Building fresh packages...
  success Saved lockfile.
  ✨  Done in 40.40s.
```

After the Stripes UI Server is built, run it using the `yarn start` command:

```bash
$ yarn start
  yarn start v0.20.3
  $ stripes dev stripes.config.js
  Listening at http://localhost:3000
  webpack built 554cedd72fbedc2f7499 in 7890ms
```

The Stripes UI Server is now running at http://localhost:3000.
The server will respond after the `webpack built...` message is displayed.

## Interacting with the Stripes UI Server

The Stripes UI Server homepage at http://localhost:3000 looks like the figure below.
![Stripes homepage](pics/01_Stripes_homepage.png)

There is one app the Stripes UI Server -- the "Trivial" app with the green icon.  Click on it to get a form:

![Trivial homepage](pics/01_Trivial_homepage.png)

Type in a greeting and name of your choice and submit the form to see the reply.

![Trivial reply](pics/01_Trivial_reply.png)

This is an example of the Stripes server component communicating with a Stripes browser component.
We have not set up the Okapi part of the FOLIO system, so this interaction is strictly within the Stripes UI Server itself.

The source for the Trivial module is in the stripes-core git repository (https://github.com/folio-org/stripes-core/tree/master/examples/trivial), with the bulk of the work in the [About.js](https://github.com/folio-org/stripes-core/blob/master/examples/trivial/About.js) file.
More details about the state of the object within the module can be seen by viewing the debugging output in the browser's JavaScript console.

![Trivial reply with browser JavaScript console](pics/01_Trivial_reply_with_js_console.png)
