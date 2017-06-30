# docker-compose-gitlab
[![Build Status](https://travis-ci.org/inhumantsar/ansible-docker-compose-gitlab.svg?branch=master)](https://travis-ci.org/inhumantsar/ansible-docker-compose-gitlab)
[![Ansible Galaxy](https://img.shields.io/badge/galaxy-inhumantsar.docker--compose--gitlab-blue.svg)](https://galaxy.ansible.com/inhumantsar/docker-compose-gitlab/)

## What?

Creates a local docker-compose based service for GitLab

## Why?

Ansible provides a nice wrapper around docker-compose, offering up useful bits like a system service, tests, and supporting actions.

## How?
### Requirements
* A recent Debian- or RHEL-compatible Linux distribution.
* Python 2.7+ with pip installed.
* At least 2 cores, 4GB RAM, and 50GB of disk space available.

### Python modules
This role will dumbly try to install these if not already present. the
* `docker-compose` >= 1.7.0
* `docker` >= 2.0
  * DO NOT install `docker-py`, even if an error message says to.
  * _*KNOWN ISSUE*_: The `docker` Python library has a bug in v2.4.0 which prevents port mapping in docker-compose.

### Installation
EL7 example tested on RHEL and CentOS.

    yum install -y git gcc python-devel openssl-devel && \
      pip install ansible

    echo -e “- src: geerlingguy.docker\n- src: inhumantsar.docker-compose-gitlab” > requirements.txt
    ansible-galaxy install -r requirements.txt

    echo """---
    > - hosts: localhost
    >   roles:
    >     - geerlingguy.docker
    >     - inhumantsar.docker-compose-gitlab
    > """ > playbook.yml
    ansible-playbook playbook.yml
