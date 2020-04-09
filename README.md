 ```
     ______________________
    < Ansible Testing Role >          ______
     ----------------------          < Yay! >
            \   ^__^                  ------
             \  (oo)\_______                \   ,__,
                (__)\       )\/\             \  (oo)____
                    ||----w |                   (__)    )\
                    ||     ||                      ||--|| *
```
---

An ansible role to help test ansible playbooks in isolated environment using
docker containers. Creates docker containers and connects them to single network
so they can communicate with each other.

## Testing Workflow

1. create environment - docker containers
    * build image from dockerfile
    * create containers and connect them to network
    * update ansible inventory - set connection to `docker` and become method to
      `su` (no need to configure sudo in docker image)
2. launch playbook
    * in created docker container
    * [import_playbook](https://docs.ansible.com/ansible/latest/modules/import_playbook_module.html)
3. test container state
    * after playbook has finished
    * should be play with asserts (assert module, block with commands + fail
      module,
    * ... anything that will fail when container is not in expected state)
4. destroy environment
    * stop and remove containers, remove network

## Usage

This role contains two sub-roles;

* **setup** which create resources in docker and
* **teardown** that removes them.

### Role parameters

* `ansible_testing_hosts` (list of hosts, default: `[]`) - list of docker
  containers to build and start
    * `name` (string, required) - name and hostname of the container
    * `image` (string, required) - image name
    * `build` (string, optional) - path to directory with Dockerfile
    * `workdir` (string, optional) - container working dir
    * `entrypoint` (string, optional) - container entrypoint
    * `command` (string, optional) - container command
    * `volumes` (list of string, optional) - list of mounts, format:
      `host_path:container_path[:mode]`
* `ansible_testing_network_name` (string, default: `ansible_testing`) - name of
  the network to which containers are connected

*Note: Setup role stores all parameters to host facts so if setup and teardown
role is used from single playbook on same host variables doesn't needs to be
duplicated to play with teardown role.*

### Example

```yaml
# setup environment
- hosts: localhost
  vars:
    ansible_testing_hosts:
      - name: remote.example.com
        image: localhost/test-image:latest
        command: sleep inf
  roles:
    - ansible-testing/setup

# run playbook we would like to test
- include_playbook: playbook.yml

# run some checks
- hosts: remote.example.com
  tasks:
    - command: test -f /etc/new-config.conf

# teardown environment
- hosts: localhost
  roles:
    - ansible-testing/teardown
```

See also examples in `examples` directory.

## Similar projects

* https://github.com/chrismeyersfsu/provision_docker -- looks good but seems not
  maintained, no license - can't fork it and improve...
* https://github.com/ansible-community/molecule -- only for roles
