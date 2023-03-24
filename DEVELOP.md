
# Overview

To develop an ansible collection, consult [Redhat's Ansible Certification Workflow Guide](https://connect.redhat.com/sites/default/files/2022-08/Ansible%20Certification%20Workflow%20Guide%202022.pdf)

This collection, `styra.opa`, is deployed to:
* [Ansible Galaxy](https://galaxy.ansible.com/) under [styra/opa](https://galaxy.ansible.com/styra/opa).  Galaxy is an open-source repository for ansible collections.


## Mechanics for publishing the collection

To upload an ansible collection to galaxy, you must

```
$ vi ansible_collections/styra/opa/galaxy.yml
$ <bump the version>
```

Then you must build the collection and upload it:

```
$ cd ansible_collections/styra/opa
$ ansible-galaxy collection build .
$ ansible-galaxy collection publish --token TOKEN styra-opa-MAJOR.MINOR.PATCH.tar.gz
```

You may procure a token by logging in to galaxy.ansible.com/USERNAME/preferences and asking for the `API key`

## Additional resources

Example collections
* [Kong](https://github.com/Kong/kong-ansible-collection) Example of an installer for k8s
* [Arista](https://github.com/aristanetworks/ansible-avd) Example of same directory structure and test suite

