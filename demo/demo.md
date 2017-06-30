## .center[Deploying GitLab with Ansible and Docker Compose]

.center[.img-300[![ansible](img/ansible.png)![gitlab](img/gitlab.png)![docker](img/docker.jpg)]]
---
# Overview
### 1. Ansible Roles
### 2. Tests
### 3. Parameters
### 4. Deployment
### 5. Next Steps


---
# Ansible Roles
### Self-contained atomic units of functionality

> You start to think about modeling *what something is*, rather than how to make *something look like something*.
.right[.small[http://docs.ansible.com/ansible/playbooks_roles.html]]

### Public publishing incentivizes best practices
* Increases focus on quality, reusability, and especially security
* Decreases administrative overhead and support resources required

### _Loads_ of existing work on Ansible Galaxy


???
# Atomic units
* generic behaviour modified by parameters
* applicable to multiple OS types, easy to test on multiple OS's

> It’s no longer “apply this handful of THINGS to these hosts”, you say “these hosts are dbservers” or “these hosts are webservers”. In programming, we might call that “encapsulating” how things work. For instance, you can drive a car without knowing how the engine works.

# Publishing publicly
* forces use of best practices
  * less hard coding (esp passwords and IPs!)
  * automatic publishing requires passing tests
* probably more efficient to patch or rebuild deficient public work than to create from scratch
* free unit testing (travis), free integration and e2e testing from users
* much lower risk of data loss

# Best Practices
* Idempotent as a goal
* Easy to write tests


---
# Ansible Roles: Cookiecutter Template

```
$ pip install cookiecutter

$ cookiecutter https://github.com/inhumantsar/cookiecutter-ansible-role
role_name [dummyrole]: docker-compose-gitlab
      ... other parameters ...

$ ls ansible-docker-compose-gitlab/
defaults  LICENSE  README.md         tasks  test.yml
handlers  meta     requirements.yml  test   vars
```
### Parameters
```json
{
  "role_name": "dummyrole",
  "author": "J. Doe",
  "description": "This role does stuff with things.",
  "company": "Fake Co.",
  "min_ansible_version": "2.1",
  "docker_test_image": "williamyeh/ansible:centos7"
}
```

???
### Cookiecutter provided the project skeleton, including basic CI tests.
### Totally valid do-nothing role out of the box, almost fit for Ansible Galaxy
### Only `checks.sh` needs to change in the tests

---
# Tests
### .travis.yml
```yaml
...
script:
  - ansible-playbook test.yml --syntax-check
  - ansible-playbook test.yml -c local -b

notifications:
  webhooks: https://galaxy.ansible.com/api/v1/notifications/
```

### tasks/main.yml
```yaml
...
- name: Confirm services launched successfully
  assert:
    that:
      - "gitlab.{{ service_name }}{{ service_env }}_gitlab_1.state.running"
      - "gitlab_runner.{{ service_name }}{{ service_env }}_gitlab_runner_1.state.running"
```

???

# Open Source
- free testing
- direct integration with the ansible galaxy
- ansible galaxy incentivizes good tests and well defined roles

# Asserts!
* little wonky looking, but it makes a certain kind of sense
* entirely based on ansible facts
* better tests could go here too
  * eg: hit the api

---
# Parameters
### defaults/main.yml
```yaml
service_env: 'dev'
service_name: 'gitlab'
...
image_tag_gitlab: 'latest'
image_tag_runner: 'alpine'
hostname_gitlab: 'localhost'
hostpath_gitlab_config: '/srv/gitlab/config'
...

gitlab_omnibus_config: |
  gitlab_rails['gravatar_enabled'] = true
#   gitlab_rails['time_zone'] = 'Asia/Tokyo'
```
### Command line overrides
`ansible-playbook playbook.yml -e service_env=prod`

???
# defaults kept in defaults/main.yml, used in tasks/main.yml
* docker-compose varsub doesn't work?
* ansible varsub works well and provides more functionality via jinja filters

# Overrides can be:
* codified into vars/<env>.yml
* written into playbooks
* provided with the `ansible-playbook -e`


---
# Deployment
```bash
yum install -y git gcc python-devel openssl-devel
pip install ansible

echo -e """- src: geerlingguy.docker
- src: inhumantsar.docker-compose-gitlab""" > requirements.txt
ansible-galaxy install -r requirements.txt

echo """---
- hosts: localhost
 roles:
   - geerlingguy.docker
   - inhumantsar.docker-compose-gitlab""" > playbook.yml
ansible-playbook playbook.yml -e service_env=prod
```
???
# Tested on RHEL7. Same general process will work on Amazon Linux, Ubuntu Xenial
# the docker_service task sets up the stack as a daemon
# the asserts confirm that the containers launch successfully.

---
# Next Steps
### Better parameterize GitLab's config
* Ad-hoc, only covering components we care about.

### Add a check which waits until GitLab's API is returning 200s
* Only if GitLab routinely fails on startup

### Use this to stand GitLab up on a production-ready system
.center[.small[![not it](img/123-not-it.jpg)]]

???
