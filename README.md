# FOLIO Developer Curriculum Outline
This is an outline of a tutorial that can be given to a group in a workshop or followed by an individual developer in a self-paced fashion.

## Goals
* Set up a running instance of the Stripes Development UI Server
* Set up a running instance of Okapi Gateway
* Demonstrate how to deploy an Okapi Module to a tenant in Okapi Gateway and the Stripes UI Server
* Understand how Okapi Gateway routes requests to modules

## System Requirements
There are two choices: either running the Stripes Development UI Server and the Okapi Gateway directly on a developer’s machine (“on-machine”) or running Stripes and Okapi in a VirtualBox guest.
An Ansible playbook with appropriate roles is used to create the VirtualBox guest, and can also be used to automatically build a developer’s environment (making the playbook target localhost).

* MacOS 10.? or higher (On-machine or VirtualBox)
* Windows 10 or higher (VirtualBox required)
* Linux (On-machine or VirtualBox)

## Prerequisites
Before attending the workshop, participants must meet these requirements.
When in doubt, using the VirtualBox guest machine is recommended.

### On-Machine
* [Java 8 JDK 1.8.0-101](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) or higher
* Maven 3.3.9 or higher
* Node.js 6.x or higher
* [Yarn](https://yarnpkg.com/en/) package manager v0.20.3 or higher

(Note that on MacOS these prerequisites can be installed using [Homebrew](https://brew.sh/).)

### VirtualBox guest
* [VirtualBox 5.1 or higher](https://www.virtualbox.org/wiki/Downloads)
* [Vagrant 1.9.1 or higher](https://www.vagrantup.com/downloads.html)

To download the VirtualBox guest:
1. Make a clean directory and change into it: `mkdir folio-curriculum && cd folio-curriculum`
1. Set up the Vagrantfile: `vagrant init --minimal folio/curriculum`
1. Launch the VirtualBox guest: `vagrant up`
1. Connect to the VirtualBox guest: `vagrant ssh`

## Lessons/Steps
1. [Deploy test Stripes package](01_deploy_test_stripes_module)
1. [Clone, build and explore Okapi](02_clone_build_and_explore_okapi)
1. [Initialize Okapi Gateway from the command line](03_initialize_okapi_from_the_command_line)
1. Real-world application: [set up the FOLIO Users app](04_set_up_the_folio_users_app)
1. Build a skeletal RAML-module-builder module

## Run Jekyll Locally
To view the documentation locally:
* (once) `git clone git@github.com/folio-org/curriculum.git folio-curriculum && cd folio-curriculum && bundle install`
* `bundle exec jekyll serve`
* View the site locally at http://localhost:4000/curriculum/
