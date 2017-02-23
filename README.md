# FOLIO Developer Curriculum Outline
This is an outline of a tutorial that can be given to a group in a workshop or followed by an individual developer in a self-paced fashion.

## Goals
* Set up a running instance of Stripes
* Set up a running instance of Okapi gateway
* Demonstrate how to deploy a module to a tenant in Okapi and UI (thing) in Stripes
* Understand how Okapi routes requests to modules

## System Requirements
There are two choices: either running Stripes and Okapi directly on a developer’s machine (“on-machine”) or running Stripes and Okapi in a VirtualBox guest.  An Ansible playbook with appropriate roles is used to create the VirtualBox guest, and can also be used to automatically build a developer’s environment (making the playbook target localhost).

* MacOS 10.? or higher (On-machine or VirtualBox)
* Windows 10 or higher (VirtualBox required)
* Linux (On-machine or VirtualBox)

## Prerequisites
Before attending the workshop, participants must meet these requirements.  When in doubt, using the VirtualBox guest machine is recommended.

### On-Machine
* Java 8 1.8.0-101 or higher
* Maven 3.3.9 or higher
* Node.js 6.x or higher
* [Yarn](https://yarnpkg.com/en/) package manager v0.20.3 or higher
* Stage artifacts in the on-machine Maven and Node repositories (to avoid having to download these prerequisites over the conference facility’s Wi-Fi network)

## VirtualBox guest
* VirtualBox 5.1 or higher
* Download the tutorial guest VM

## Lessons/Steps
1. [Deploy test Stripes module](01_deploy_test_stripes_module.md)
1. [Clone, build and explore Okapi](02_clone_build_and_explore_okapi.md)
1. [Initialize Okapi from the command line](03_initialize_okapi_from_the_command_line)
1. Putting it all together: deploy ui-okapi-console
1. Real-world application: deploy ui-users (Stripes) and mod-users (Okapi)
1. Build a skeletal RAML-module-builder module

## Run Jekyll Locally
* (once) `git clone git@github.com/folio-org/curriculum.git folio-curriculum && cd folio-curriculum && bundle install`
* `bundle exec jekyll serve`
* View the site locally at http://localhost:4000/curriculum/
