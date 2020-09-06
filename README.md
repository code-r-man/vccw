# VCCW (re-loaded)

[![Build Status](https://travis-ci.org/vccw-team/vccw.svg?branch=master)](https://travis-ci.org/vccw-team/vccw)

This is a Vagrant configuration designed for development of WordPress plugins, themes, or websites based on original `vccw.cc` stack.

List of added/changed features:
* updated `php` version to `7.4`
* added HTTPS

To get started, check out <http://vccw.cc/>

## Configuration

1. Copy `provision/default.yml` to `site.yml`.
1. Edit the `site.yml`.
1. Run `vagrant up`.

### Note

* The `site.yml` has to be in the same directory with Vagrantfile.
* You can add custom options to the `site.yml`.
* Own Certificate Authority (CA) certificate file is located in `provision/playbooks/templates/cacert.pem` (this file is used to import into browser/OS certificate library).
